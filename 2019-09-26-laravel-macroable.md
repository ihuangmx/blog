# 深入浅出 Laravel Macroable

## 预备知识

在构建 Laravel Macroable 之前，需要掌握的知识点有：

* PHP 魔术方法
* PHP 延迟绑定
* PHP 反射

## 使用 Macroable

```php
class Foo 
{
	use Illuminate\Support\Traits\Macroable;
}
```

为类扩展 `join` 方法

```php
Foo::macro('join', function(...$string) {
	return implode('-', $string);
});

// test
Foo::join('a','b','c'); // a-b-c
```

在扩展的方法中使用类的静态属性

```php
class Foo 
{
	use Illuminate\Support\Traits\Macroable;
	static $separator = '-';
}

Foo::macro('join', function(...$string) {
	return implode(static::$separator, $string);
});
```

## 动手实现 Macroable