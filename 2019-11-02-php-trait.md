# Trait

> PHP 的 Trait 语法很简单，更重要的是理解 Trait 的使用场景。

## 提出

为什么 PHP 会引入 `Trait`? 我们先来看看软件开发中的两种常用代码复用模式，**继承**和**组合**。

* 继承：强调 **父类与子类** 的关系，即子类是父类的一个特殊类型；
* 组合：强调 **整体与局部** 的关系，侧重的一种需要的关系；

软件开发中有一条原则，叫做**组合优于继承**。这是因为**从耦合度来看，继承要高于组合**。继承关系中，子类与父类保持着高度的依赖关系，加上 PHP 不支持多继承，为了避免重写编写代码，很多功能都被统一封装到父类中。这样做有两个坏处：一是随着继承的层数和子类的增加，代码复杂度不断增加，大量的方法都将面临着重写；二是这些功能对于一些子类来说可能是不必要的，破坏了代码的封装性。

`Trait` 的提出弥补了 PHP 对组合支持的不足，一个 `Trait` 就相当于一个模块，不同的 `Trait` 以组合的方式注入到类中。我们以 Laravel 的控制器为例，来介绍下继承和组合是如何在具体的场景中使用的。

首先，**底层的代码应当多使用组合**。Laravel 的底层控制器只继承了一个简单的控制器 `Illuminate\Routing\Controller`，结构相对稳定。同时，控制器使用了不同的  `Trait` 来组织代码，避免了对象的臃肿，极大程度的保持了架构的灵活性。

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Foundation\Auth\Access\AuthorizesRequests;
use Illuminate\Foundation\Bus\DispatchesJobs;
use Illuminate\Foundation\Validation\ValidatesRequests;
use Illuminate\Routing\Controller as BaseController;

class Controller extends BaseController
{
    use AuthorizesRequests, DispatchesJobs, ValidatesRequests;
}
```

而**具体的业务逻辑或顶层代码应当多使用继承**，这样能够大大提高的开发效率

```php
<?php
use App\Http\Controllers\Controller;

class UserController extends Controller
{
}
```

以上就是继承和组合的简单介绍。接下来看看 `Trait` 的具体使用。

## 使用

### 规范

Symfony 编码规范建议在每个 `Trait` 之后添加 `Trait` 关键字。

```php
namespace Symfony\Contracts\Translation;

trait TranslatorTrait {

}
```

PSR-12 规范建议在每个 `Trait` 使用一个 `use` 语句来声明，同时 `Trait` 与类的其他成员需要保持一行空行。

```php
class ClassName
{
    use FirstTrait;
    use SecondTrait;
    use ThirdTrait;

    public $a;
}
```

### 成员

`Trait` 中可包含属性、方法 与 抽象方法，这三者的结合既可以复用代码，也可以对代码的使用作出一些约定，例如 Laravel 中的自动维护 `slug` 字段

```php
<?php

namespace App\Traits;

use Illuminate\Support\Str;

trait HasSlug 
{   
    public static function bootSluggable()
    {
        static::saving(function ($model) {
            $model->slug = Str::slug(
                $model->getAttribute( $model->sluggable() )
            );
        });
    }

    /**
     * Slug 字段
     * 
     * @return string
     */
    abstract public function sluggable(): string;

}
```

`Trait` 中也可以包括静态属性和静态方法，以下是一个单例模式的简单封装。

```php
trait Singleton
{
    private static $instance;

    public static function getInstance() {
        if (!(self::$instance instanceof self)) {
            self::$instance = new self;
        }
        return self::$instance;
    }
}
```

每个 `Trait` 中可以包含其他 `Trait`，进一步提高了代码的灵活性

```php
<?php
trait Hello
{
    function sayHello() {
        echo "Hello";
    }
}

trait World
{
    function sayWorld() {
        echo "World";
    }
}

class MyWorld
{
    use Hello;
    use World;
}
```

### Trait 与类的冲突处理

当存在同名方法时，当前类的方法会覆盖 `Trait` 中的方法，而 `Trait` 中的方法会覆盖父类的方法。

```php
<?php
// 父类，优先级最低
class Base {
    public function sayHello() {
        echo 'Hello ';
    }
}

// Trait 优先级大于父类
trait SayWorld {
    public function sayHello() {
    	parent::sayHello();
        echo 'World!';
    }
}

// 当前类，优先级最高
class MyHelloWorld extends Base {
    use SayWorld;
}

$o = new MyHelloWorld();
$o->sayHello();  // Hello, World
```

当存在同名属性时，类的属性**必须**与 `Trait` 的属性兼容（相同的访问性、相同的初始值），否则会报**致命错误**。

```php
<?php
trait Foo {
    public $same = true; 
    public $different = false;
}

class Bar {
    use Foo;
    public $same = true; // 合法
    public $different = true; // fatal error
}
```

### Trait 与 Trait 的冲突处理

当一个类包含多个 `Trait` 时，不同 `Trait` 之间可能会存在属性和方法的冲突。

当存在相同属性时，属性**必须**兼容，跟 `Trait` 与类的冲突处理类似

```php
Trait A {
	public $a = 'foo';
}
Trait B {
	public $a = 'foo';
}
class Foo 
{
	use A;
	use B;
}
```

当存在方法冲突时，需要使用 `insteadof` 来手动处理冲突，否则会报致命错误

```php
<?php
Trait A {
	public function foo()
	{
		return "A foo";
	}
}

Trait B {
	public function foo()
	{
		return "B foo";
	}
}

class Bar {
    use A, B {
    	B::foo insteadof A;  // 用 B 的 foo 方法来代替 A
    }
}

$bar = new Bar();
echo $bar->foo();  // B foo
```

这时候如果想要保留 A 的 `foo` 方法，可以用 `as` 定义别名来进行调用。注意，起别名仅仅代表可以用别名来调用该方法，仍然需要用 `insteadof` 处理冲突

```php
class Bar {
    use A, B {
    	B::foo insteadof A;  // 用 B 的 foo 方法来代替 A
    	A::foo as aFoo; // A 的  foo 方法用 aFoo 来调用
    }
}

$bar = new Bar();
echo $bar->aFoo(); // A foo
```

`as` 关键字还可以用来更改方法法的访问控制

```php
<?php

Trait A {
	public function foo()
	{
		return "A foo";
	}
}

class Bar {
    use A {
    	A::foo as private;
    }
}

$bar = new Bar();
echo $bar->foo(); // Fatal error: Uncaught Error: Call to private method Bar::foo()
```

这两者可以结合起来用，这时候原有方法的访问控制就不会受到影响

```php
<?php

Trait A {
	public function foo()
	{
		return "A foo";
	}
}

class Bar {
    use A {
    	A::foo as private aFoo;
    }
}

$bar = new Bar();
echo $bar->foo(); // A foo，照常调用
echo $bar->aFoo(); // 被设置成私有方法，因此报错。Fatal error: Uncaught Error: Call to private method Bar::foo()
```
