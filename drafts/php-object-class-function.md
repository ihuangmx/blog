# PHP 函数库之类与对象

## 废弃

一些函数已经被**废弃**或者**移除**，请不要使用它们

* `__autoload` - 7.2 版本废弃
* `call_user_method_array` - 7.0 版本移除
* `call_user_method` - 7.0 版本移除

## 判断

### 类的存在性检查

相关函数

* `class_exists` - 判断类是否存在
* `interface_exists` - 判断接口是否存在
* `trait_exists` - 判断 Trait 是否存在

第二个参数用来决定如果尚未加载，是否使用自动加载。

```php
class_exists ( string $class_name [, bool $autoload = true ] ) : bool
interface_exists ( string $interface_name [, bool $autoload = true ] ) : bool
trait_exists ( string $traitname [, bool $autoload = true ] ) : bool
```

示例 - 广泛的类存在性检查函数

```php
function common_class_exists(string $class): bool
{
    return class_exists($class, false) || interface_exists($class, false) || trait_exists($class, false);
}
```

### 类成员的存在性检查

相关函数：

* `property_exists` - 检查属性是否存在
* `method_exists` — 检查方法是否存在

```php
method_exists ( mixed $object , string $method_name ) : bool
property_exists ( mixed $class , string $property ) : bool
```

示例 - 实现一个回调函数，用户可通过方法或者属性来定义回调的 URL

```php
trait RedirectsUsers
{
    public function redirectPath()
    {
        if (method_exists($this, 'redirectTo')) {
            return $this->redirectTo();
        }

        return property_exists($this, 'redirectTo') ? $this->redirectTo : '/home';
    }
}
```

### 类关系判断

相关函数：

`is_a` — 对象属于该类或该类的父类，返回`TRUE`
`is_subclass_of` — 对象是该类的子类，返回 `TRUE`

```php
is_a ( object $object , string $class_name [, bool $allow_string = FALSE ] ) : bool
is_subclass_of ( object $object , string $class_name ) : bool
```

示例

```php
interface A {}
interface B {}
class BaseFoo implements B {}
class Foo extends BaseFoo implements A{}

$foo = new Foo();

// 对象
is_a($foo, 'BaseFoo'); // true
is_a($foo, 'Foo'); // true
is_a($foo, 'A'); // true

// 类
is_a('Foo', 'BaseFoo'); // false
is_a('Foo', 'BaseFoo', true);  // true, 传入第三个参数，代表允许使用类名而不是示例

is_subclass_of($foo, 'Foo'); // false
is_subclass_of($foo, 'BaseFoo'); // true
is_subclass_of($foo, 'B'); // true
```

实际情况中，更多的是使用操作符 `instanceof`

```php
$foo instanceof Foo; // true
$foo instanceof A; // true
$foo instanceof B; // true
```
## 操作

相关函数：

* `class_alias()` - 为一个类创建别名

```php
class_alias ( string $original , string $alias [, bool $autoload = TRUE ] ) : bool
```

示例 - 类别名加载器，用于管理类的别名

```php
class AliasLoader
{
    private $aliases;

    public function __construct(array $aliases)
    {
        $this->aliases = $aliases;
    }

    public function load($alias)
    {
        if (isset($this->aliases[$alias]))
        {
            return class_alias($this->aliases[$alias], $alias);
        }
    }
    
}

class LongLongLongLongFoo {}

$aliases = [
    'Foo' => 'LongLongLongLongFoo',
    'Bar' => 'LongLongLongLongBar',
];

$loader =  new AliasLoader($aliases);
$loader->load('Foo');

$foo = new Foo();
var_dump($foo);  // object(LongLongLongLongFoo)#3395
```

## 获取

### 获取全部

相关函数：

* `get_declared_traits` — 返回所有已定义的 traits 的数组
* `get_declared_interfaces` — 返回一个数组包含所有已声明的接口
* `get_declared_classes` — 返回由已定义类的名字所组成的数组

这些函数很少需要用到

```php
foreach (get_declared_classes() as $class) {
    $r = new \ReflectionClass($class);
}
```

### 获取类

相关函数

`get_called_class` — 后期静态绑定类的名称，在类外部使用返回 `false`
`get_class` — 返回对象的类名
`get_parent_class` — 返回对象或类的父类名

```php
get_declared_classes ( void ) : array
get_class ([ object $object = NULL ] ) : string
get_parent_class ([ mixed $obj ] ) : string
```

示例 - 抛出异常时获取异常的类

```php
throw (new ModelNotFoundException)->setModel(get_class($this->model));
```

### 获取类成员

相关函数：

* get_class_methods — 返回由类的方法名组成的数组
* get_class_vars — 返回由类的默认属性组成的数组
* get_object_vars — 返回由对象属性组成的关联数组

示例 - 获取类中的所有访问器属性

```php

class Foo {

    public function getFullNameAttribute()
    {
        
    }

    public function getTextAttribute()
    {
        
    }

    public static function getMutatorMethods()
    {
        preg_match_all('/(?<=^|;)get([^;]+?)Attribute(;|$)/', implode(';', get_class_methods(static::class)), $matches);

        return $matches[1];
    }
}

Foo::getMutatorMethods()
// [
//     "FullName",
//     "Text",
// ]
```

