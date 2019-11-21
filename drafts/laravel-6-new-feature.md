# Laravel 6.0 新特性

## 语义化的版本控制

Laravel 和官方扩展包将正式遵循语义化版本，即版本号 X.Y.Z 严格遵循以下规范

* X - 主版本号，当你做了不兼容的 API 修改，递增主版本号；
* Y - 次版本号，当你做了向下兼容的功能性新增，递增次版本号；
* Z - 修订号，当你做了向下兼容的问题修正，递增修订号；

更多关于语义化版本的信息，可参考 [语义化版本 2.0.0 | Semantic Versioning](https://semver.org/lang/zh-CN/)

## 分离前端脚手架

安装

```bash
$ composer require laravel/ui
```

生成脚手架

```bash
$ php artisan ui vue
$ php artisan ui react
```

生成注册登录

```bash
$ php artisan ui vue --auth
$ php artisan ui react --auth
```

## ignition

[Introduction - Flare Docs](https://flareapp.io/docs/ignition-for-laravel/introduction)

配置编辑器(需要先安装对应的 [handle](https://github.com/inopinatus/sublime_url))，在页面中，可以定位

```
IGNITION_EDITOR=sublime
```

配置主题

```
IGNITION_THEME=dark
```

在线 tinker

```bash
composer require facade/ignition-tinker-tab --dev
```

在线编辑器

```bash
$ composer require facade/ignition-code-editor --dev
```

自我诊断工具

```bash
$ composer require facade/ignition-self-diagnosis --dev
```

## 懒集合

懒集合是 PHP 生成器的应用。

生成器：

* 生成器允许你在 foreach 代码块中写代码来迭代一组数据而不需要在内存中创建一个数组；
* 生成器函数与普通函数类似，但是可根据需要 yield 多次，以便生成需要迭代的值



我们真正迭代完每个用户之后，filter 回调才会执行，从而大大减少了内存的使用：

```php
$users = App\User::cursor()->filter(function ($user) {
    return $user->id > 500;
});

foreach ($users as $user) {
    echo $user->id;
}
```

## 子查询增强功能

## 队列中间件
