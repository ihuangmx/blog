# Laravel 快速添加验证码

> 注：本文使用的 Laravel 版本为 `6.x`。

## 生成注册登录

Laravel 6.x 版本需要进行前端的初始化

```bash
$ composer require laravel/ui
```

生成 `bootstrap` 脚手架

```bash
php artisan ui bootstrap
php artisan ui:auth bootstrap
```

编译前端资源

```bash
$ cnpm install && npm run dev
```

## 添加验证码

安装扩展包

```bash
$ composer require mews/captcha
```

发布配置文件

```sh
$ php artisan vendor:publish --provider='Mews\Captcha\CaptchaServiceProvider'
```

`captcha` 提供了一系列辅助方法

```php
captcha(); // 图片
captcha_src(); // URL
captcha_img(); // HTML
```

辅助方法可传入配置项

```php
captcha_img('inverse');
captcha_src('flat');
```

同时，该扩展包默认注册了供用户获取验证码的路由

```bash
$ php artisan route:list | grep captcha
# captcha/{config?}
# captcha/api/{config?}
```

本地测试

```
127.0.0.1:8000/captcha  // 默认使用 default 配置
127.0.0.1:8000/captcha/flat  // 使用 flat 配置
127.0.0.1:8000/captcha/api // 测试 API 返回
```

在密码字段后面添加验证码 - URL 后面附带了随机数，防止浏览器缓存

```php
// /resources/views/auth/register.blade.php
<div class="form-group row">
    <label for="captcha" class="col-md-4 col-form-label text-md-right">验证码</label>

    <div class="col-md-6">
       
        <input id="captcha" class="form-control{{ $errors->has('captcha') ? ' is-invalid' : '' }}" name="captcha" required>
       
        <img class="thumbnail mt-3 mb-2" src="{{ captcha_src() }}" onclick="this.src='/captcha?'+Math.random()" title="点击图片重新获取验证码">

    @if ($errors->has('captcha'))
      <span class="invalid-feedback" role="alert">
        <strong>{{ $errors->first('captcha') }}</strong>
      </span>
    @endif
  </div>
</div>
```

最后，只需要在注册控制器中添加验证

```php
// /app/Http/Controllers/Auth/RegisterController.php

protected function validator(array $data)
{
    return Validator::make($data, [
        'name' => ['required', 'string', 'max:255'],
        'email' => ['required', 'string', 'email', 'max:255', 'unique:users'],
        'password' => ['required', 'string', 'min:8', 'confirmed'],
        'captcha' => ['required', 'captcha'],
    ]);
}
```


