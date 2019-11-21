# PHP 内置服务器

PHP 内置了一个 Web 服务器，可用于本地开发。

进入目录后启动服务器，默认会去匹配 `index.html` 或 `index.php` 文件

```sh
$ cd public
$ php -S localhost:8000
$ php -S localhost:8000 -c php.ini  # 指定配置文件
```

也可以使用 `-t` 参数字指定目录：

```sh
$ php -S localhost:8000 -t public/
```

或者指定某一脚本，指定后则 **始终先访问** 此脚本

```sh
$ php -S localhost:8000 router.php
```

若脚本的返回值若为 `false`，则会以静态资源的方式返回同名文件，利用这点，可进行简单的返回资源判断

```php
<?php
// router.php
if (preg_match('/\.(?:png|jpg|jpeg|gif)$/', $_SERVER["REQUEST_URI"]))
    return false;    // 直接返回请求的文件
else { 
    echo "<p>Welcome to PHP</p>";
}
```