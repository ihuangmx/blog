# 重载

PHP 的重载跟 Java 的重载不同，不可混为一谈。Java 允许类中存在多个同名函数，每个函数的参数不相同，而 PHP 中只允许存在一个同名函数。例如，Java 的构造函数可以有多个，PHP 的构造函数则只能有一个。

PHP 的重载是指 **通过魔术方法对属性和类的动态创建**

* 属性的重载 -  `__get` 与 `__set`
* 方法的重载 -  `__call` 与 `__callStatic`

例如，Laravel 的请求类实现了属性重载，使代码变得更加的简洁

```php
$name = $request->name;
```
该属性在类中并不存在，而是通过魔术方法来访问的，具体实现如下

```php
public function __get($key)
{
    return Arr::get($this->all(), $key, function () use ($key) {
        return $this->route($key);
    });
}
```
这种实现方式的应用非常广泛，简单的归纳实现的原理

```php
class Foo
{	
	private $params = [];
	
	function __construct(array $params = [])
	{
		$this->params = $params;
	}

	public function __set($name, $value)
    {
        $this->params[$name] = $value;
    }

	public function __get($name)
    {
    	return $this->params[$name];
    }

    public function __isset($name)
    {
        return isset($this->params[$name]);
    }

    public function __unset($name)
    {
        unset($this->params[$name]);
    }
}
```