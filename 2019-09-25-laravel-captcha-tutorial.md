# Laravel 快速添加验证码

> 注：本文使用的 Laravel 版本为 `5.8`。

创建应用

```php
$ laravel new Captcha
```

配置数据库

```toml
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=captcha
DB_USERNAME=root
DB_PASSWORD=123456
```

执行迁移

```sh
$ php artisan migrate
```

使用  Laravel 自带的 `auth` 组件

```sh
$ php artisan make:auth
```

安装扩展包

```sh
$ composer require mews/captcha
```

发布配置文件

```sh
$ php artisan vendor:publish --provider='Mews\Captcha\CaptchaServiceProvider'
```

根据自己需要进行配置，本文使用默认配置，因此省略该步骤。配置后记得清空配置信息缓存。

```sh
$ php artisan config:cache
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

![-w1240](qiniu.larahacks.cn/15543463922140.jpg)

本地路由测试

```
127.0.0.1:8000/captcha  // 默认使用 default 配置
127.0.0.1:8000/captcha/flat  // 使用 flat 配置
127.0.0.1:8000/captcha/api // 测试 API 返回
```

测试成功后，我们来为注册视图添加验证码，可添加到密码后面

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

该步骤中，我们在 URL 后面附带了随机数，防止浏览器缓存。

![-w740](qiniu.larahacks.cn/15553341862464.jpg)


最后，只需要在注册控制器中添加验证即可

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
