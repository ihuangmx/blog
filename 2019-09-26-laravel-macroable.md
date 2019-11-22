> Laravel 提供的 `Macroable` 可以在不改变类结构的情况为其扩展功能，本文将教你从零开始构建一个 `Macroable`。

`Macroable` 的核心是基于匿名函数的绑定功能，先来回顾下匿名函数的绑定功能。

## 预备知识

PHP 可通过匿名函数的绑定功能来扩展类或者实例的功能。

定义类

```php
class Foo
{
}
```

定义匿名函数

```php
$join = function(...$string){
    return implode('-', $string);
}
```

使用 `bindTo` 为**类的实例**添加 `join` 功能

```php
$foo = new Foo();
$bindFoo = $join->bindTo($foo, Foo::class);
$bindFoo('a', 'b', 'c');  //  "a-b-c"
```

PHP 7 之后引入了  `call` 方法更高效的实现了该功能

```php
$foo = new Foo();
$join->call($foo, 'a', 'b', 'c'); // "a-b-c"
```

对于本例而言，使用 `bind` 方法进行静态绑定更贴合实际场景

```php
$bindClass = \Closure::bind($join, null, Foo::class);
$bindClass('a', 'b', 'c');  // "a-b-c"
```

如果还没看懂的话，可以参考我之前写的 [PHP 核心特性 - 匿名函数](https://learnku.com/articles/35863)。

## 通过匿名函数扩展类的功能

了解了匿名函数的绑定功能后，就可以对其进行简单的封装了。首先，定义一个数组用来保存要添加的功能列表

```php
<?php

trait Macroable {
    // 保存要扩展的功能
    protected static $macros = [];

    // 添加要扩展功能
    public static function macro($name, $macro)
    {
        static::$macros[$name] = $macro;
    }
}
```

`macros` 属性保存了要添加的功能名及实现，在类中使用该 `Trait`

```php
class Foo 
{
    use Macroable;
}
```

添加 `join` 功能

```php
Foo::macro('join', function(...$string){
    return implode('-', $string);
});
```

`join` 功能及对应的实现已经保存到了 `macros` 数组中。接下来是调用 `join` 方法

```php
Foo::join('a', 'b', 'c')
```

由于 `Foo` 中的 `join` 静态方法不存在，会自动将方法名和参数转发到 `__callStatic` 魔术方法中。因此，在魔术方法中手动调用绑定的匿名函数即可

```php
public static function __callStatic($name, $parameters)
{   
    // 获取匿名函数
    $macro = static::$macros[$name];

    // 绑定到类
    $bindClass = \Closure::bind($macro, null, static::class);

    // 调用并返回调用结果
    return $bindClass(...$parameters);
}
```

测试

```php
echo Foo::join('a', 'b', 'c'); // a-b-c
```

动态扩展与静态扩展的实现原理完全一样

```php
public function __call($name, $parameters) 
{   
    // 获取匿名函数
    $macro = static::$macros[$name];

    // 调用并返回调用结果
    return $macro->call($this, ...$parameters);
}
```

测试

```php
$foo = new Foo();
echo $foo->join('a', 'b', 'c'); // 'a-b-c'
```

## 通过对象实例来扩展类的功能

之前，我们通过匿名函数的方式扩展类的功能

```php
Foo::macro('join', function(...$string){
    return implode('-', $string);
});
```

现在，我们考虑如何通过对象的方式来实现同样的功能。首先，将匿名函数改造成类

```php
final class Join
{
    public function __invoke(...$string)
    {
        return implode('-', $string);
    }
}
```

当以函数的方式调用该类时，就会激活 `__invoke` 方法

```php
$join = new Join();
$join('a', 'b', 'c'); // a-b-c
```

现在，将 `Join` 的实例添加到类中，实现同样的效果

```php
Foo::macro('join', new Join());
```

只需要对原有的 `__callStatic` 方法增加一层判断即可。如果是匿名函数则绑定该匿名函数并调用，如果是对象则以函数的方式调用对象，激活对象的 `__invoke` 方法。

```php
public function __call($name, $parameters) 
{
    $macro = static::$macros[$name];

    if($macro instanceof Closure){
        return $macro->call($this, ...$parameters);
    }

    return $macro(...$parameters);
}

public static function __callStatic($name, $parameters)
{
    $macro = static::$macros[$name];

    // 闭包
    if($macro instanceof Closure){
        $bindClass = \Closure::bind($macro, null, static::class);
        return $bindClass(...$parameters);
    }

    // 对象实例，则激活该对象
    return $macro(...$parameters);
}
```

测试

```php
Foo::join('a', 'b', 'c');  // a-b-c
```

## 同时扩展多个方法

最后，Laravel 的 `Macroable` 还实现了同时扩展多个方法。

原理其实很简单，将功能类似的方法定义在一个类中

```php
final class Str
{   
    public function join()
    {   
        // 返回匿名函数
        return function(...$string){
            return implode('-', $string);
        };
    }

    public function split() 
    {   
        // 返回匿名函数
        return function(string $string){
            return explode('-', $string);
        };
    }
}
```

每个方法都返回了匿名函数，我们只需要将每个匿名函数添加到 `$macros` 列表中即可，只需要用到 PHP 的反射功能即可实现。


```php
public static function mixin($mixin) 
{
    // 通过反射获取对象的 ReflectionMethod  列表
    $methods = (new \ReflectionClass($mixin))->getMethods(
        \ReflectionMethod::IS_PUBLIC | \ReflectionMethod::IS_PROTECTED
    );

    // 遍历 ReflectionMethod 列表，依次保存到 $macros 中
    foreach ($methods as $method) {
        $method->setAccessible(true);
        // 依次激活该对象的每个方法，每个方法返回的匿名函数刚好保存在 $macros 中
        static::macro($method->name, $method->invoke($mixin));
    }
}
```

测试

```php
Foo::mixin(new Str());
Foo::join('a', 'b', 'c');
Foo::split('a-b-c');
```

当然，这个功能没多大作用，还不如直接用 `Trait` 来的直观方便。
