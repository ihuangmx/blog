# 命名空间

## 提出

在命名空间提出之前，不同的组件很容易碰到命名的冲突，例如 `Request` 、`Response` 等常见的命名。PHP 在 5.3 后提出了命名空间用来解决组件之间的命名冲突问题，主要参考了文件系统的设计：

* 同一个目录下不允许有相同的文件名  - 同一个命名空间下不允许有相同的类；
* 不同的目录可以有同名文件 - 不同的命名空间可以有相同的类；

## 定义

使用 `namespace` 关键字来定义一个命名空间。其中，顶层命名空间通常为厂商名，不同开发者的厂商命名空间是唯一的。命名空间不需要与文件目录一一对应，但是最好遵守 [PSR-4](https://learnku.com/docs/psr/psr-4-autoloader/1608) 规范。

```php
<?php

namespace Symfony\Component\HttpFoundation;

class Request {

}
```

命名空间必须在所有代码之前声明，唯一的例外就是 `declare` 关键字。

```php
<?php

declare(strict_types=1);

namespace App;
```

命名空间内可包含任意 PHP 代码，但是仅对类(包括抽象类和 Trait)、接口、函数和常量这四种类型生效。

```php
<?php
namespace MyProject;

const CONNECT_OK = 1;
class FOO {}
interface Foo{}
function foo() {}
```

## 使用

使用 `use` 关键字来引入命名空间

```php
<?php

namespace App;

use Symfony\Component\HttpFoundation\Request;
use Foo\Bar;

class Test {
	public function run() 
	{
		$bar = new Bar();
	}
}
```

定义和使用推荐遵循 [PSR-2](https://learnku.com/docs/psr/psr-2-coding-style-guide/1606) 的规范

* `namespace` 之后必须存在一个空行；
* 所有 `use` 声明必须位于 `namespace` 声明之后;
* 每条 `use` 声明必须只有一个 `use` 关键字。
`use` 语句块之后必须存在一个空行。

当 `use` 引入的类出现同名时，可使用 `as` 来定义别名

```php
<?php

namespace App;

use Foo\Bar as BaseBar;

class Bar extends BaseBar {

}
```

## 限定符

除了使用 `use` 外，还可以直接使用 `\` 限定符来进行解析，规则很简单：如果含有 `\` 前缀则代表从全局命名空间开始解析，否则则代表从当前命名空间开始解析。

```php
<?php

namespace App;

\Foo\Bar\foo();  // 解析成 \Foo\Bar\foo();
Foo\Bar\foo();  // 解析成 App\Foo\Bar\foo();
```

此规则也适用于函数、常量等

```php
$a = \strlen('hi'); // 调用全局函数 strlen
$b = \INI_ALL; // 访问全局常量 INI_ALL
$c = new \Exception('error'); // 实例化全局类 Exception
```

有两个需要特别注意的地方：

对于函数和常量而言，如果当前命名空间不存在，则会自动去全局命名空间去寻找，因此可省略 `\` 前缀。对于类而言，如果当前命名空间解析不到，不会去全局空间寻找，因此，不可省略 `\`

```php
$a = strlen('hi');
$b = INI_ALL;
$c = new Exception('error'); // 错误
$c = new Exception('error'); // 正确
```

当动态调用命名空间时，该命名空间始终会被当成是全局命名空间，因此可以省略前缀  `\`

```php
$class1 = 'Foo\Bar';
$object1 = new $class1;  // 始终被解析成 \Foo\Bar
```

## 在内部访问命名空间

PHP 支持两种抽象的访问当前命名空间内部元素的方法，`__NAMESPACE__` 魔术常量和 `namespace` 关键字。

`__NAMESPACE__` 常量的值是包含当前命名空间名称的字符串，如果是在全局命名空间，则返回空字符串。

```php
<?php
namespace MyProject;

function get($classname)
{
    $a = __NAMESPACE__ . '\\' . $classname;
    return new $a;
}
```

关键字 `namespace` 可用来显式访问当前命名空间或子命名空间中的元素。它等价于类中的 `self` 操作符

```php
namespace App;

use blah\blah as mine; 

blah\mine(); // App\blah\mine()
namespace\blah\mine(); // App\blah\mine()

namespace\func(); // App\func()
namespace\sub\func(); // App\sub\func()
namespace\cname::method(); // App\cname::method()
$a = new namespace\sub\cname(); // App\sub\cname
$b = namespace\CONSTANT; // App\CONSTANT
```

## 转义 `\` 符号

此外，推荐对所有的 `\` 进行转义，避免出现不可预期的后果

```php
$class = "dangerous\name"; // \n 被解析成换行符
$obj = new $class;

$class = 'dangerous\name'; // 正确，但是不推荐
$class = 'dangerous\\name'; // 推荐
$class = "dangerous\\name"; // 推荐
```

参考资料

* [PHP: 命名空间 - Manual](https://www.php.net/manual/zh/language.namespaces.php)
* [Modern PHP（中文版）](https://book.douban.com/subject/26635862/)
* [PSR-1 基础编码规范](https://learnku.com/docs/psr/basic-coding-standard/1605)
* [PSR-4 自动加载规范](https://learnku.com/docs/psr/psr-4-autoloader/1608)
* [PHP: The Right Way](https://phptherightway.com/#use_the_current_stable_version)



