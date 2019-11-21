# Symfony 编码规范

Symfony 的编码规范建立在 `PSR-1`, `PSR-2` and `PSR-4` 的基础上。

## 工具

使用 `php-cs-fixer` 工具来自动检查编码规范

```bash
$ cd your-project/
$ php php-cs-fixer.phar fix -v
```
## 结构

每个逗号分隔符之后添加一个空格

```php
$arr = [1, 2, 3];
```

所有的二元运算符左右都添加一个空格，`.` 运算符除外

```php
$sum = $a + $b;
$name = $firstName.$lastName;
```

一元运算符紧跟关联的变量，不需要添加空格

```php
if( !$name )
```

始终使用全等，除非你需要进行一些类型转换(type juggling)

```php
if($rst === true)
```

使用 `==`， `!=`，`===`，和 `!==` 等判断时，需要将不变的量放在左边，避免意外情况出现

```php
if(10 === $number)
```

多行数组的最后一项始终添加括号

```php
$arr = [
	[1,2],
	[1,2],
];
```

代码块中除了 `return` 语句之外还包含其他代码，那么 `return` 语句之前保留一个空行

```php
// 添加空行
function sum($a, $b)
{
	$sum = $a + $b;

	return $sum;
}

// 只有 return 语句，无需添加空行
function sum($a, $b)
{
	return $a + $b;
}
```

虽然 `return;` 与 `return null` 在语法上没有什么区别。但是在使用上要根据具体的场景来：如果函数不需要返回值，则使用 `return;` 如果需要返回值但是返回值为 `null`，就使用 `return null`。

```php
function foo() : void
{
	return;
}
```

控制语句不能省略花括号

```php
if($a === 1){
	return true;
}
```

一个文件中只包括一个类。除非是那些不需要在外部实例化的私有的帮助类。


将类的继承和实现的接口声明都写在同一行。

```php
class User extends Model implements AuthenticatableContract, AuthorizableContract, CanResetPasswordContract
```

属性要在方法之前声明

```php
class Foo
{
	private $bar;

	public function foo()
	{
		
	}
}
```

方法默认按照可访问性从高到低排列。除此之外，构造函数  `__construct`、测试方法 `setUp()`、 `tearDown()` 始终放在最前面，以提高可读性

```php
public foo() {}
protected bar() {}
private barz() {}
```

无论方法中包括多少个参数，都应当在同一行声明

```php
public function __construct($command, string $cwd = null, array $env = null, $input = null, ?float $timeout = 60)
{

}
```

类的实例化始终使用括号

```php
$foo = new Foo();
```

异常和错误消息字符串必须使用 `sprintf` 来进行拼接；

```php
throw new CommandNotFoundException(sprintf('Command "%s" does not exist.', $name));
```

当错误类型为 `E_USER_DEPRECATED` 时，需要添加 `@` 

```php
@trigger_error("foo", E_USER_DEPRECATED);
```

数组访问器之间不要存在空格

```php
$arr[0][1];
```

引用的类如果不属于全局命名空间，则每个类都需要使用 `use`

```php
use Symfony\Component\Console\Exception\InvalidArgumentException;
use Symfony\Component\Console\Exception\LogicException;
```

注释文档中 `@param` 或 `@return` 如果包括 `null` 类型，要将 `null` 放在最后面

```php
/**
 * @param int|null
 * @return string|null
 */
```

## 命名约定

使用驼峰法 (camelCase) 命名变量、方法或函数

```php
$acceptableContentTypes
hasSession();
```

使用蛇形 (snake_case) 命名法来命名配置参数以及 `Twig` 模板变量

```php
framework.csrf_protection
http_status_code
```

为所有的类添加命名空间，类名始终使用大写驼峰 (UpperCamelCase)

```php
<?php

namespace App;

class FooBar {

}
```

抽象类添加 `abstract` 前缀，除了 PHPUnit 的 TestCase 抽象类。

```php
abstract class Helper 
{

}
```

接口添加 `Interface` 后缀

```php
interface ExceptionInterface
{
}
```

Trait 添加 `Trait` 后缀

```php
trait FooTrait
{
}
```

异常添加 `Exception` 后缀

```php
class CommandNotFoundException 
{

}
```

使用大写驼峰命名 PHP 文件，小写蛇形命名模板引擎或 web 资源文件

```
FooBar.php
section_layout.html.twig
index.scss
```

文档注释或者类型转换中，使用 `bool` 而不是 `boolean` 或 `Boolean`，使用 `int` 而不是 `integer`，使用 `float` 而不是 `double` 或 `real`

```php
(int) $a;
(bool) $a;
(float) $a;
```

## 服务

这部分主要是介绍 `symfony` 框架的 `services` 使用规范

* 服务的名称使用类的完全限定名，如 `App\EventSubscriber\UserSubscriber`;
* 存在多个同名服务时，主要的服务使用完全限定名，其他服务使用小写下划线命名。也可以用 `.` 符号进行分组，例如 `something.service_name`, `fos_user.something.service_name`;
* 参数名使用小写，环境变量名使用 `%env(VARIABLE_NAME)% `
* 公共的服务添加类关联，例如关联 `Symfony\Component\Something\ClassName` 为 `something.service_name`


## 文档

为所有的类、方法以及函数添加 `PHPDoc` 注释。

根据类型来进行分组，不同类型之间空一行

```php
 /**
 * @FooBar(1)
 * @FooBar(2)
 *
 * @var int
 */
private $var;
```

无返回值时，省略 `@return` 注释

```php
/**
 * Configures the current command.
 */
protected function configure()
{
}
```

不要使用 `@package` 和 `@subpackage` 标记。

块注释时即使只有一个标签，也不要写在同一行，比如 `/** {@inheritdoc} */`

```php
/**
 * @inheritdoc
 */
function testInheritDocValid1($test)
{
}
```

当添加一个新的类或者大量改动一个已存在类时，也许要添加一个 `@author` 标记来留下个人的联系方式。

```php
/**
 * @author Jérôme Tamarelle <jerome@tamarelle.net>
 */
```

## 示例

官方给的示例

```php
/*
 * This file is part of the Symfony package.
 *
 * (c) Fabien Potencier <fabien@symfony.com>
 *
 * For the full copyright and license information, please view the LICENSE
 * file that was distributed with this source code.
 */

namespace Acme;

/**
 * Coding standards demonstration.
 */
class FooBar
{
    const SOME_CONST = 42;

    /**
     * @var string
     */
    private $fooBar;

    /**
     * @param string $dummy Some argument description
     */
    public function __construct($dummy)
    {
        $this->fooBar = $this->transformText($dummy);
    }

    /**
     * @return string
     *
     * @deprecated
     */
    public function someDeprecatedMethod()
    {
        @trigger_error(sprintf('The %s() method is deprecated since vendor-name/package-name 2.8 and will be removed in 3.0. Use Acme\Baz::someMethod() instead.', __METHOD__), E_USER_DEPRECATED);

        return Baz::someMethod();
    }

    /**
     * Transforms the input given as first argument.
     *
     * @param bool|string $dummy   Some argument description
     * @param array       $options An options collection to be used within the transformation
     *
     * @return string|null The transformed input
     *
     * @throws \RuntimeException When an invalid option is provided
     */
    private function transformText($dummy, array $options = [])
    {
        $defaultOptions = [
            'some_default' => 'values',
            'another_default' => 'more values',
        ];

        foreach ($options as $option) {
            if (!in_array($option, $defaultOptions)) {
                throw new \RuntimeException(sprintf('Unrecognized option "%s"', $option));
            }
        }

        $mergedOptions = array_merge(
            $defaultOptions,
            $options
        );

        if (true === $dummy) {
            return null;
        }

        if ('string' === $dummy) {
            if ('values' === $mergedOptions['some_default']) {
                return substr($dummy, 0, 5);
            }

            return ucwords($dummy);
        }
    }

    /**
     * Performs some basic check for a given value.
     *
     * @param mixed $value     Some value to check against
     * @param bool  $theSwitch Some switch to control the method's flow
     *
     * @return bool|void The resultant check if $theSwitch isn't false, void otherwise
     */
    private function reverseBoolean($value = null, $theSwitch = false)
    {
        if (!$theSwitch) {
            return;
        }

        return !$value;
    }
}
```