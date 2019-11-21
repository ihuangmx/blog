# PHP OPcache

## 原理

首先，来看看 PHP 代码的执行过程：

1. Lexing - 将 PHP 代码转换为语言片段 (Tokens)
2. Parse - 将 Tokens 转换成简单而有意义的表达式
3. Compile - 将表达式编译成字节码(OpCode)
4. Excute - 顺次执行字节码，每次一条，从而实现 PHP 脚本的功能。

从执行步骤中可以看出，如果可以缓存字节码的话，就可以跳过前面的步骤，从而大大提高 PHP 应用的性能。

<img src="imgs/opcache-life-cycle.png" alt="opcache-life-cycle.png" style="zoom:24%;" />

## OPcache

在 PHP 5.5 之前，不少扩展都实现了字节码的缓存，比如 Zend 公司开发的 Zend Optimizer Plus。该扩展也随着 PHP 5.5 一起发布，并改名为 OPcache。也就是说，从 5.5 开始，OPcache 成了 PHP 的默认绑定扩展。该扩展通过将 PHP 脚本预编译的字节码存储到共享内存中，省去了每次加载和解析 PHP 脚本的开销。

### 参数说明

OPcache 扩展可通过 `php.ini` 来进行配置，以下是一些常用的配置参数说明，后面对应的默认值。

```
opcache.memory_consumption = 128
```

分配的内存大小，如果应用的脚本数较少的话，可适当调小。

```
opcache.interned_strings_buffer = 8
```

存储预留字符串的内存大小。PHP 解释器在背后会找到相同字符串的多个实例,把这个字符串保存在内存中,如果再次使用相同的字符串,PHP 解释器会使用指针。这么做能节省内存。默认情况下,PHP 驻留的字符串会隔离在各个 PHP 进程中。这个设置能让 PHP-FPM 进程池中的所有进程把驻留字符串存储到共享的缓冲区中,以便在 PHP-FPM 进程池中的多个进程之间引用驻留字符串。

```
opcache.max_accelerated_files = 10000
```

OPcache 哈希表中可存储的脚本文件数量上限。该值的设置要大于应用的文件数量。

```
opcache.validate_timestamps = 1
```

该选项的值为 1 或者 0， 1 代表启用，一旦启用该项， OPcache 会每隔 `revalidate_freq` 设定的秒数来检查脚本是否更新，开发环境下，一般将其设置为 1。而生产环境下，一般将其设置为 0，然后通过重启 Web 服务器或者使用 PHP 提供的 `opcache_reset()` 或者 `opcache_invalidate()` 函数来手动重置 OPcache。


```
revalidate_freq = 2
```

脚本更新的检查时间周期，一旦将该值设置为 0，那么每个请求 OPcache 都会检查脚本更新。由于生产环境下的 `validate_timestamps` 通常设置为 0，所以该值往往只在开发环境下有意义。


```
enable_cli = 0 
```

仅针对 CLI 版本的 PHP 启用操作码缓存，默认不开启。

### 推荐配置

以下是 PHP 官方的推荐设置

```
opcache.memory_consumption=128
opcache.interned_strings_buffer=8
opcache.max_accelerated_files=4000
opcache.revalidate_freq=60
opcache.enable_cli=1
```

以下是 《Modern PHP》一书的推荐配置

```
opcache.memory_consumption=64
opcache.interned_strings_buffer=16
opcache.max_accelerated_files=4000
opcache.revalidate_freq=0
opcache.validate_timestamps  开发环境为 1，生产环境为 0
```

## 相关函数

### 说明

PHP 官方提供了与 OPcache 相关的函数

```php
opcache_compile_file ( string $file ) : boolean
```

该函数可以用于在不用运行某个 PHP 脚本的情况下，编译该 PHP 脚本并将其添加到字节码缓存中去。 该函数可用于在 Web 服务器重启之后初始化缓存，以供后续请求调用。

```php
opcache_get_configuration ( void ) : array
```

该函数将返回缓存实例的配置信息。

```php
opcache_get_status ([ boolean $get_scripts = TRUE ] ) : array
```

该函数将返回缓存实例的状态信息。

```php
opcache_invalidate ( string $script [, boolean $force = FALSE ] ) : boolean
```

该函数的作用是使得指定脚本的字节码缓存失效。 如果 `force` 没有设置或者传入的是 `FALSE`，那么只有当脚本的修改时间 比对应字节码的时间更新，脚本的缓存才会失效。

```php
opcache_is_script_cached ( string $file ) : bool
```

该函数用于检测 PHP 脚本是否已经被缓存。

```php
opcache_reset ( void ) : boolean
```

该函数将重置整个字节码缓存。 在调用 `opcache_reset()` 之后，所有的脚本将会重新载入并且在下次被点击的时候重新解析。有网友指出，该函数在 Cli 模式下不会生效。

### 应用

利用这些函数，可以方便的清空 OPcache 缓存，比如将其封装成脚本，在代码更新后自动执行脚本

```bash
#!/bin/bash
WEBDIR=/var/www/html/
RANDOM_NAME=$(head /dev/urandom | tr -dc A-Za-z0-9 | head -c 13)
echo "<?php opcache_reset(); ?>" > ${WEBDIR}${RANDOM_NAME}.php
curl http://localhost/${RANDOM_NAME}.php
rm ${WEBDIR}${RANDOM_NAME}.php
```

或者说利用这些函数来创建可视化操作面板。例如，`opcache-status` 项目是一个展示 Opcache Status 的单个页面，可以很方便的部署在自己的服务器上。以下本地查看的流程

```bash
$ git clone https://github.com/rlerdorf/opcache-status.git --depth=1
$ cd opcache-status
$ php -S localhost:8000 opcache.php
```

参考资料

* [深入理解PHP原理之Opcodes | 风雪之隅](http://www.laruence.com/2008/06/18/221.html)
* [rlerdorf/opcache-status: A one-page opcache status page](https://github.com/rlerdorf/opcache-status)
* [PHP: OPcache - Manual](https://www.php.net/manual/zh/book.opcache.php)
* [Modern PHP（中文版） (豆瓣)](https://book.douban.com/subject/26635862/)
* [PHP Performance I: Everything You Need to Know About OpCode Caches – Engine Yard Developer Center](https://support.cloud.engineyard.com/hc/en-us/articles/205411888-PHP-Performance-I-Everything-You-Need-to-Know-About-OpCode-Caches)