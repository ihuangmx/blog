# Mac 下 PHP 的安装

## PHP

Mac 环境下推荐使用 `brew` 来安装和切换 PHP 版本。

安装 

```bash
$ brew install php@7.3
```

brew 官方只能安装 PHP [官方支持版本](https://www.php.net/supported-versions.php)。如果要支持旧版本，则需要添加其他 `tap`

```bash
$ brew tap exolnet/homebrew-deprecated
```

查看安装信息（扩展、相关目录、相关命令等等）

```bash
$ brew info php
```

相关命令

```bash
$ brew services start php
$ brew services stop php
$ brew services restart php
```

PHP 版本的切换

```bash
$ brew unlink php@7.3 && brew link --force --overwrite php@5.6
```

## Composer 

安装 `composer`

```bash
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php -r "if (hash_file('sha384', 'composer-setup.php') === 'a5c698ffe4b8e849a443b120cd5ba38043260d5c4023dbf93e1558871f1f07f58274fc6f4c93bcfd858c6bd0775cd8d1') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
php composer-setup.php
php -r "unlink('composer-setup.php');"
```

快捷使用

```bash
mv composer.phar /usr/local/bin/composer
sudo chmod +x /usr/local/bin/composer
```

使用阿里云镜像

```bash
$ composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/
```

## 代码校验

安装代码校验工具

```bash
$ composer global require friendsofphp/php-cs-fixer
```

集成到编辑器中，这里以 Sublime Text 为例，使用最新的 `PRS@12` 代码规范

```json
{
	"shell_cmd": "php-cs-fixer fix $file --rules=@PSR12"
}
```

## 安装扩展

查看已加载的扩展

```bash
$ php -r "print_r(get_loaded_extensions());"
```

查看已安装的扩展

```bash
$ php -m
```

安装 ImageMagick

```bash
$ brew install imagemagick
$ pecl install imagick
```

安装 phpunit

```bash
$ composer global require --dev phpunit/phpunit
```

安装 laravel

```bash
$ composer global require laravel/installer
```

安装 swoole

```bash
$ pecl install swoole
```
