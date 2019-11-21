# PHP 异常处理

## 错误与异常

在 PHP 7 之前，错误处理和错误处理是分开的。

```php
try {
	$a = 5 % 0;
} catch (Exception $e) {
	echo $e->getMessage();
	$a = -1;  // 通过异常来处理 $a 为 0 的情况，但是实际上，捕获不到该异常
}

echo $a;  // 无法执行
```

PHP 7 开始，错误也可以像异常那样抛出。

```php
try {
	$a = 5 % 0;
	// 注意，DivisionByZeroError 错误只能捕捉到 % 运算，无法捕捉 / 运算
} catch (DivisionByZeroError $e) {
	echo $e->getMessage();
	$a = -1;  
}

echo $a; // -1
```

## 相关类

PHP 中的全部异常和错误

```
Throwable - 接口
	Error - 所有错误的基类
	   ArithmeticError
	      DivisionByZeroError
	   AssertionError
	   CompileError
	      ParseError
	   TypeError
	      ArgumentCountError
	Exception - 所有异常的基类
	   ClosedGeneratorException
	   DOMException
	   ErrorException
	   IntlException
	   JsonException
	   LogicException
	      BadFunctionCallException
	         BadMethodCallException
	      DomainException
	      InvalidArgumentException
	      LengthException
	      OutOfRangeException
	   PharException
	   ReflectionException
	   RuntimeException
	      OutOfBoundsException
	      OverflowException
	      PDOException
	      RangeException
	      UnderflowException
	      UnexpectedValueException
	   SodiumException
```

Throwable 接口

```php
Throwable {
	abstract public getMessage ( void ) : string
	abstract public getCode ( void ) : int
	abstract public getFile ( void ) : string
	abstract public getLine ( void ) : int
	abstract public getTrace ( void ) : array
	abstract public getTraceAsString ( void ) : string
	abstract public getPrevious ( void ) : Throwable
	abstract public __toString ( void ) : string
}
```

Error 基类

```php
Error implements Throwable {

	protected string $message ;
	protected int $code ;
	protected string $file ;
	protected int $line ;

	public __construct ([ string $message = "" [, int $code = 0 [, Throwable $previous = NULL ]]] )
	final public getMessage ( void ) : string
	final public getPrevious ( void ) : Throwable
	final public getCode ( void ) : mixed
	final public getFile ( void ) : string
	final public getLine ( void ) : int
	final public getTrace ( void ) : array
	final public getTraceAsString ( void ) : string
	public __toString ( void ) : string
	final private __clone ( void ) : void
}
```

Exception 基类

```php
Exception {

	protected string $message ;
	protected int $code ;
	protected string $file ;
	protected int $line ;

	public __construct ([ string $message = "" [, int $code = 0 [, Throwable $previous = NULL ]]] )
	final public getMessage ( void ) : string
	final public getPrevious ( void ) : Throwable
	final public getCode ( void ) : int
	final public getFile ( void ) : string
	final public getLine ( void ) : int
	final public getTrace ( void ) : array
	final public getTraceAsString ( void ) : string
	public __toString ( void ) : string
	final private __clone ( void ) : void
}
```

除此之外，用户也可以自定义异常

```php
class ModelNotFoundException extends \RuntimeException
{

    protected $model;

    public function setModel($model)
    {
        $this->model = $model;
        $this->message = "No query results for model [{$model}]";
        return $this;
    }
}

// test
throw (new ModelNotFoundException)->setModel(get_class($this->model));
```

用户只能通过继承 `Error` 或 `Exception` 来实现 `Throwable` 接口。

```php
//  Class MyException cannot implement interface Throwable, extend Exception or Error instead
class MyException implements Throwable {}

// 正确
class MyException extends Exception implements Throwable {}
```

也可以这么操作

```php
interface MyThrowable extends Throwable {
    public function myCustomMethod();
}

class MyException extends Exception implements MyThrowable {
    public function myCustomMethod()
    {
        // implement custom method code
    }
}
```

## 使用场景

以下三种场景需要用到异常处理机制：

对程序的悲观预测。认为自己的代码无法处理各种不可预见的情况。例如，程序员悲观的认为自己的这段代码在高并发条件下可能产生死锁，那么他就会悲观地抛出异常，然后在死锁时进行捕获，对异常进行细致的处理。

程序的需要和对业务的关注。异常机制认为，数据一致性很重要，在数据一致性可能被破坏时，就需要异常机制进行事后补救。

```php
try {

} cache(Exception $e){
	// 执行一些补救措施
}
```

语言级别的健壮性要求。通过精确控制运行时的流程，在程序中断时，有预见地用 `try` 缩小可能出错的范围，及时捕获异常发生并做出相应补救，以使逻辑流程仍然回到正常轨道上。

## 使用

set_exception_handler
