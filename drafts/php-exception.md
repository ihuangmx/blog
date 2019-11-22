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

// 使用
throw (new ModelNotFoundException)->setModel(get_class($this->model));
```

自定义异常时，应当以具体错误来命名，而不是从发布者的角度来命名

```php
// 不好的命名
CouldNotCopyFileException

// 好的命名
FileNotFoundException
```

注意，用户无法实现 `Throwable` 接口，只能通过继承 `Error` 或 `Exception` 来实现。

```php
//  报错：Class MyException cannot implement interface Throwable, extend Exception or Error instead
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

异常将错误处理与常规代码进行分离，能够让业务流程更加清晰。

```php
try {
	// 业务流程
} catch (FileNotFoundException $e) {
	
} catch (FileTypeException $e) {
	
} catch (Exception $e) {
	
}
```

如果没有使用异常，那么在进行业务流程时就必须考虑各种错误处理，会让代码的逻辑变得混乱

```php
// 业务流程
// 错误处理
// 业务流程
// 错误处理
```

在很多情况下，方法检测到错误却没有足够的信息来进行处理，这时候应该抛出抛出异常，让客户端来处理异常，有利于保持职责的单一性。

```php
public function foo()
{
	if(条件不满足){
		// 抛出异常，不进行处理，让客户端进行处理
	}
}
```

异常机制有利于保持数据一致性很重要，在数据一致性可能被破坏时，就需要使用异常机制进行事后补救。

```php
try {

} cache(Exception $e){
	// 执行一些补救措施
} 
```

异常是层层抛出的，客户端只需要在合适的时候处理特定的异常即可，不需要一次性处理所有异常。

```php
// 只处理 404 异常
public function actionType($username)
{
    try {
        $user = $client->get(sprintf("/api/user/%s", $username));
    } catch (RequestException $e) {
        if (404 === $e->getResponse()->getStatusCode()) {
            return "create";
        }

        throw $e;
    }

    return "update";
}
```

为了不泄露敏感信息，需要一个全局异常处理程序。

```php
set_exception_handler(function($exception){
    echo '发生了未知错误';
    log($exception->getMessage());
});
```

