# 自定义命令快速查看 Laravel 日志

Unix 的 `tail` 命令可以用来查看文件的，例如查看 `2019-09-26` 的最后 100 行日志

```sh
$ cd storage/logs  
$ tail -100 laravel-2019-09-26.log 
```

将 `tail` 封装成 `artisan` 指令，可实现命令行快速查看日志，[spatie/laravel-tail](https://github.com/spatie/laravel-tail) 扩展包实现了该功能，类似功能实现如下。

创建命令

```bash
$ php artisan make:command LaravelTailCommand 
```

需要做两部分的功能：

1. 获取最新的日志文件;
2. 转化成终端的命令执行;

## 获取最新的日志文件

依赖注入 `Filesystem`

```php
use Illuminate\Filesystem\Filesystem;
.
.
.
class LaravelTailCommand extends Command
{
    /**
     * The filesystem instance.
     *
     * @var \Illuminate\Filesystem\Filesystem
     */
    protected $files;

    /**
     * Create a new command instance.
     *
     * @return void
     */
    public function __construct(Filesystem $files)
    {
        parent::__construct();
        $this->files = $files;
    }
}
```

获取全部日志文件

```php
/**
 * 获取全部日志文件
 * 
 * @return array
 */
public function allLogFiles() : array
{
    return $this->files->allFiles(storage_path('logs'));
}
```

判断某个日志文件是否为 `laravel` 日志文件

```php
use SplFileInfo;
.
.
.
/**
 * 是否为 Laravel 日志文件
 * 
 * @param  SplFileInfo $file 
 * @return boolean           
 */
public function isLaravelLogFile(SplFileInfo $file) : bool
{
    return strpos($file->getBaseName(),'laravel') !== false;
}
```

获取最新的日志文件

```php
/**
 * 获取最新的日志路径
 * 
 * @return string | false
 */
public function findLatestLogFile() 
{   
    $logFile = collect($this->allLogFiles())
        ->filter(function(SplFileInfo $file){
            return $this->isLaravelLogFile($file);
        })
        ->sortByDesc(function (SplFileInfo $file) {
            return $file->getMTime();
        })
        ->first();

    return $logFile
        ? $logFile->getPathname()
        : false;
}
```

## 转化成终端的命令执行

默认显示 100 行，且默认只显示日志摘要，不显示详情

```php
/**
 * The name and signature of the console command.
 *
 * @var string
 */
protected $signature = 'tail
                        {--lines=100 : 输出行数}
                        {--stack : 是否显示详细信息，默认为不显示}';
```

对于是否显示详情的处理

```php
/**
 * 根据参数来判断是否显示详情
 * 
 * @return null | string
 */
public function getFilters()    
{   
    return $this->option('stack') 
        ? null
        : '| grep -i -E "^\[[0-9]{4}\-[0-9]{2}\-[0-9]{2} [0-9]{2}:[0-9]{2}:[0-9]{2}\]" --color';

}
```

最终处理

```php
use Symfony\Component\Process\Process;
.
.
.
/**
 * Execute the console command.
 *
 * @return mixed
 */
public function handle()
{
    $latestLogPath = $this->findLatestLogFile();

    $lines = $this->option('lines');
    $filters = $this->getFilters();
    $tail = "tail -n {$lines} -f ".escapeshellarg($latestLogPath)." {$filters}";
    $process = new Process($tail);
    $process->setTty(true)->run();
}
```

完整代码

```php
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;
use Illuminate\Filesystem\Filesystem;
use SplFileInfo;
use Symfony\Component\Process\Process;

class LaravelTailCommand extends Command
{

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'tail
                            {--lines=100 : 输出行数}
                            {--stack : 是否显示详细信息，默认为不显示}';

    /**
     * The console command description.
     *
     * @var string
     */
    protected $description = '快速查看最新日志';

    /**
     * The filesystem instance.
     *
     * @var \Illuminate\Filesystem\Filesystem
     */
    protected $files;

    /**
     * Create a new command instance.
     *
     * @return void
     */
    public function __construct(Filesystem $files)
    {
        parent::__construct();
        $this->files = $files;
    }

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $latestLogPath = $this->findLatestLogFile();

        $lines = $this->option('lines');
        $filters = $this->getFilters();
        $tail = "tail -n {$lines} -f ".escapeshellarg($latestLogPath)." {$filters}";
        $process = new Process($tail);
        $process->setTty(true)->run();
    }

    /**
     * 根据参数来判断是否显示详情
     * 
     * @return null | string
     */
    public function getFilters()    
    {   
        return $this->option('stack') 
            ? null
            : '| grep -i -E "^\[[0-9]{4}\-[0-9]{2}\-[0-9]{2} [0-9]{2}:[0-9]{2}:[0-9]{2}\]" --color';

    }

    /**
     * 获取全部日志文件
     * 
     * @return array
     */
    public function allLogFiles() : array
    {
        return $this->files->allFiles(storage_path('logs'));
    }

    /**
     * 是否为 Laravel 日志文件
     * 
     * @param  SplFileInfo $file 
     * @return boolean           
     */
    public function isLaravelLogFile(SplFileInfo $file) : bool
    {
        return strpos($file->getBaseName(),'laravel') !== false;
    }

    /**
     * 获取最新的日志路径
     * 
     * @return string | false
     */
    public function findLatestLogFile() 
    {   
        $logFile = collect($this->allLogFiles())
            ->filter(function(SplFileInfo $file){
                return $this->isLaravelLogFile($file);
            })
            ->sortByDesc(function (SplFileInfo $file) {
                return $file->getMTime();
            })
            ->first();

        return $logFile
            ? $logFile->getPathname()
            : false;
    }

    
}
```

使用

```bash
$ php artisan tail
$ php artisan tail --stack
$ php artisan tail --lines=500
```
