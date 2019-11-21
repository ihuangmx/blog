# 在 Laravel 自动转换 PHP 变量

## 介绍

laracasts 出品的  [PHP-Vars-To-Js-Transformer](https://github.com/laracasts/PHP-Vars-To-Js-Transformer) 包可自动将 PHP 变量转化为 JavaScript 变量。

使用说明

```php
JavaScript::put('foo', 'bar');  
// window.foo = "bar";

JavaScript::put([
    'foo' => 'bar',
    'age' => 29
]);  
// window.foo = "bar";window.age = 29;"
```

接下来将动手实现该扩展包。

## 实现

PHP-Vars-To-Js-Transformer 扩展包主要完成两项工作

1. 将 PHP 变量转化为 JavaScript 变量；
2. 利用视图事件自动输出 JavaScript 变量；

定义类，该类传入需要绑定的视图以及 `JavaScript` 变量的作用域

```php
namespace App\Services\JavaScript;

class Transformer
{   
    /**
     * 命名空间
     * 
     * @var string
     */
    private $namespace;

    /**
     * 要输出变量的视图
     * 
     * @var array
     */
    private $views;

    function __construct($views, string $namespace = 'window')
    {
        $this->namespace = $namespace;
        $this->views = str_replace('/', '.', (array)$views);
    }

    // 主要业务逻辑，todo
    public function put()
    {
    	
    }
```

### 转化变量

`put` 函数支持两种类型的传参

* `JavaScript::put('foo', 'bar')`
* `JavaScript::put($arr)`

这两种传参都需要标准化成数组

```php
use InvalidArgumentException;

public function put()
{
	$input = $this->normalizeInput(func_get_args());
}

/**
 * 格式化输入
 * 
 * @param  string | array $variables 
 * @throws InvalidArgumentException
 * @return array
 */
public function normalizeInput($arguments) : array
{   
    if( is_array($arguments[0]) ){
        return $arguments[0];
    }

    if(count($arguments) == 2){
        return [
            $arguments[0] => $arguments[1]
        ];
    }

    throw new InvalidArgumentException('参数输入错误');
}
```

将输入标准化成数组后，还需要进一步将其转化为 JavaScript 变量，关键点在于 PHP 变量的值可能有多种类型（基本类型、数组、对象等等），需要进行不同的处理

```php
use Illuminate\Contracts\Support\Arrayable;
use Illuminate\Contracts\Support\Jsonable;
use JsonSerializable;

public function put()
{
	$input = $this->normalizeInput(func_get_args());
	$js = $this->constructJavaScript($input);
}

/**
 * 转化 PHP 变量
 * 
 * @param  array $variables PHP 变量
 * @return string
 */
public function constructJavaScript($variables)
{
    return collect($variables)->map(function($data, $key){
        return "{$this->namespace}.{$key} = ".$this->convertToJavaScript($data).";";
    })->implode('');
}

/**
 * 格式化值
 * 
 * @param mix $data 
 * 
 * @return mix
 */
public function convertToJavaScript($data)
{
    if($data instanceof Jsonable){
        return $data->toJson();
    } 

    if ($data instanceof JsonSerializable) {
        return json_encode($data->jsonSerialize());
    }

    if ($data instanceof Arrayable) {
        return json_encode($data->toArray());
    }

    return json_encode($data);
}
```

当命名空间不为 `window` 时，还需要添加额外的声明语句。例如，传入的作用域为 `laravel`，则需要添加声明语句 `window.laravel = window.laravel || {};`，具体实现如下

```php

public function put()
{
	$input = $this->normalizeInput(func_get_args());
    $js = $this->constructJavaScript($input);
    $js = $this->constructNamespace().$js;
}

/**
 * 构造命名空间
 * 
 * @return string
 */
public function constructNamespace() : string
{
    if($this->namespace == 'window'){
        return '';
    }

    return "window.{$this->namespace} = window.{$this->namespace} || {};";
}
```

### 输出变量

为绑定的视图添加对应的事件，以便自动输出变量

```php
public function put()
{   
    $input = $this->normalizeInput(func_get_args());
    $js = $this->constructJavaScript($input);
    $js = $this->constructNamespace().$js;
    
    foreach ($this->views as $view) {
        app('events')->listen("composing: {$view}", function () use ($js) {
            echo "<script>{$js}</script>";
        });
    }
}
```

完整代码

```php
<?php

namespace App\Services\JavaScript;

use Illuminate\Contracts\Support\Arrayable;
use Illuminate\Contracts\Support\Jsonable;
use InvalidArgumentException;
use JsonSerializable;

class Transformer
{   
    /**
     * 命名空间
     * 
     * @var string
     */
    private $namespace;

    /**
     * 要输出变量的视图
     * 
     * @var array
     */
    private $views;

    function __construct($views, string $namespace = 'window')
    {
        $this->namespace = $namespace;
        $this->views = str_replace('/', '.', (array)$views);
    }

    public function put()
    {   
        $input = $this->normalizeInput(func_get_args());
        $js = $this->constructJavaScript($input);
        $js = $this->constructNamespace().$js;
        
        foreach ($this->views as $view) {
            app('events')->listen("composing: {$view}", function () use ($js) {
                echo "<script>{$js}</script>";
            });
        }
    }

    /**
     * 转化 PHP 变量
     * 
     * @param  array $variables PHP 变量
     * @return string
     */
    public function constructJavaScript(array $variables) : string
    {
        return collect($variables)->map(function($data, $key){
            return "{$this->namespace}.{$key} = ".$this->convertToJavaScript($data).";";
        })->implode('');
    }

    /**
     * 格式化值
     * 
     * @param mix $data 
     * 
     * @return mix
     */
    public function convertToJavaScript($data)
    {
        if($data instanceof Jsonable){
            return $data->toJson();
        } 

        if ($data instanceof JsonSerializable) {
            return json_encode($data->jsonSerialize());
        }

        if ($data instanceof Arrayable) {
            return json_encode($data->toArray());
        }

        return json_encode($data);
    }

    /**
     * 格式化输入
     * 
     * @param  string | array $variables 
     * @throws InvalidArgumentException
     * @return array
     */
    public function normalizeInput($arguments) : array
    {   
        if( is_array($arguments[0]) ){
            return $arguments[0];
        }

        if(count($arguments) == 2){
            return [
                $arguments[0] => $arguments[1]
            ];
        }

        throw new InvalidArgumentException('参数输入错误');
    }

    /**
     * 构造命名空间
     * 
     * @return string
     */
    public function constructNamespace() : string
    {
        if($this->namespace == 'window'){
            return '';
        }

        return "window.{$this->namespace} = window.{$this->namespace} || {};";
    }
}
```

### 运行

```php
$input = [
	'foo' => 'bar',
	'age' => 29,
	'collection' => collect(['aaa', 'bbb', 'ccc'])
];

$views = ['footer'];

$transform = new Transformer($views, 'laravel');
$transform->put($input);
```

## 优化

`Transformer` 类主要完成了两项工作：转化变量以及绑定变量到视图。该类的实现违反了 **开放-封闭** 原则

> 实体（类、方法等）应当对扩展开放，对修改封闭。

对于绑定变量到视图这一行为而言，不同的框架有不同的实现方法，是可变的行为，应当将其分离出来，隐藏于接口背后。

```php
<?php

namespace App\Services\JavaScript;

interface ViewBinderInterface
{
    /**
     * 绑定变量到视图
     *
     * @param string $js
     */
    public function bind($js);
}
```

Laravel 对绑定变量到视图这一行为的实现

```php
<?php

namespace App\Services\JavaScript;

use Illuminate\Contracts\Events\Dispatcher;

class ViewBinder implements ViewBinderInterface
{
	/**
	 * 事件分发器
	 * 
	 * @var Dispatcher
	 */
    protected $event;

    /**
     * 变量绑定的视图
     *
     * @var string
     */
    protected $views;

    /**
     * Create a new Laravel view binder instance.
     *
     * @param Dispatcher   $event
     * @param string|array $views
     */
    public function __construct(Dispatcher $event, $views)
    {
        $this->event = $event;
        $this->views = str_replace('/', '.', (array)$views);
    }

    /**
     * Bind the given JavaScript to the view.
     *
     * @param string $js
     */
    public function bind($js)
    {
        foreach ($this->views as $view) {
            $this->event->listen("composing: {$view}", function () use ($js) {
                echo "<script>{$js}</script>";
            });
        }
    }
}
```

`Transformer`  类就可以简化成

```php
<?php

namespace App\Services\JavaScript;

use App\Services\JavaScript\ViewBinderInterface;
use Illuminate\Contracts\Support\Arrayable;
use Illuminate\Contracts\Support\Jsonable;
use InvalidArgumentException;
use JsonSerializable;

class Transformer
{   
    /**
     * 命名空间
     * 
     * @var string
     */
    private $namespace;

    /**
     * @var App\Services\JavaScript\ViewBinderInterface
     */
    private $viewBinder;


    function __construct(ViewBinderInterface $viewBinder, string $namespace = 'window')
    {
        $this->namespace = $namespace;
        $this->viewBinder = $viewBinder;
    }

    public function put()
    {   
        $js = $this->constructNamespace().$this->constructJavaScript($this->normalizeInput(func_get_args()));
        $this->viewBinder->bind($js);

        return $js;
    }

}

```

运行

```php
$input = [
	'foo' => 'bar',
	'age' => 29,
	'hello' => collect(['aaa', 'bbb', 'ccc'])
];

$viewBinder = new ViewBinder(
	app('events'), 
	['footer']
);
$transform = new Transformer($viewBinder, 'laravel');
$transform->put($input);
```

## 集成到 Laravel 项目中

注册容器

```php
public function register()
{
    $this->app->singleton('JavaScript', function ($app) {
        return new Transformer(
            new ViewBinder($app['events'], ['footer']),
            'window'
        );
    });
}
```

创建门面

```php
<?php

namespace App\Services\JavaScript;

use Illuminate\Support\Facades\Facade;

class JavaScriptFacade extends Facade
{
    protected static function getFacadeAccessor()
    {
        return 'JavaScript';
    }
}
```

注册门面

```php
'JavaScript' => App\Services\JavaScript\JavaScriptFacade::class,
```




