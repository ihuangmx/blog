# Laravel 大型项目架构（2） - 处理数据

## 数据是起点

当使用 Laravel 进行开发时，首先考虑的往往是 Model，可以说，数据是应用的起点，几乎所有的业务流程都需要与数据进行交互。因此，需要对数据进行结构化的处理，使其变得更加的安全和可靠。

## 弱语言类型的不足

对于 PHP 这种弱类型语言而言，变量只是充当一个占位符，不能保证其类型的正确性。当然，对于 PHP 这种与 HTTP 请求打交道的语言而言，这样设计是有意义的，因为 HTTP 传送过来的都是一个个的字符串。

示例 - 变量类型是松散的

```php
$foo = "bar";
$foo = 123;  // 随意修改变量类型
```

类型提示可以规范数据，但是过于死板，因为有时候我们在函数内部仍然可以转化类型，比如

```php
declare(strict_types=1);

function find(int $id)
{
}

User::find('1'); // 类型提示导致报错
```

而且类型提示只能保证当前数据的类型，对于未来的数据却毫无保证。这是因为 PHP 只有在运行的时候检查其变量类型，但是为时已晚，程序可能会因此而终止。

对于 Laravel 的请求数据而言，我们无法从中了解哪些数据是可用的。

```php
function store(CustomerRequest $request, Customer $customer) 
{
    $validated = $request->validated();  // 什么玩意？
    
    $customer->name = $validated['name'];
    $customer->email = $validated['email'];
}
```

因此，我们需要对数据进行结构化处理，这里需要用到 Data Transfer Objects（DTO）。DTO 用来在应用层和展现层之间传输数据。看上去要为每个服务创建一个 DTO 是一个费时又增加性能开销的时，但是 DTO 却能够大大减少团队成员的认知负荷。

PHP 7.4 及以上版本允许定义属性类型。

```php
class CustomerData
{
    public string $name;
    public string $email;
    public Carbon $birth_date;
}

function store(CustomerRequest $request, Customer $customer) 
{
    $validated = CustomerData::fromRequest($request);
    
    $customer->name = $validated->name;
    $customer->email = $validated->email;
    $customer->birth_date = $validated->birth_date;
    
    // …
}
```

## 创建 DTO

使用 DTO 工厂

```php
class CustomerDataFactory
{
    public function fromRequest( CustomerRequest $request): CustomerData 
    {
        return new CustomerData([
            'name' => $request->get('name'),
            'email' => $request->get('email'),
            'birth_date' => Carbon::make( $request->get('birth_date')),
        ]);
    }
}
```

`fromRequest`

```php
use Spatie\DataTransferObject\DataTransferObject;

class CustomerData extends DataTransferObject
{

    public static function fromRequest( CustomerRequest $request): self 
    {
        return new self([
            'name' => $request->get('name'),
            'email' => $request->get('email'),
            'birth_date' => Carbon::make( $request->get('birth_date')),
        ]);
    }
}
```



为了弥补弱类型的不足，

即使使用了 Strict Type 或类型提示，PHP 语言依旧没有摆脱弱类型的局限。类型系统可以有效的弥补该不足。类型系统规定了变量是否可以改变类型。

Type System 

定义了变量是否可以改变它的类型。


工厂

class CustomerDataFactory
{
    public function fromRequest(
       CustomerRequest $request
    ): CustomerData {
        return new CustomerData([
            'name' => $request->get('name'),
            'email' => $request->get('email'),
            'birth_date' => Carbon::make(
                $request->get('birth_date')
            ),
        ]);
    }
}

数据

use Spatie\DataTransferObject\DataTransferObject;

class CustomerData extends DataTransferObject
{
    // …
    
    public static function fromRequest(
        CustomerRequest $request
    ): self {
        return new self([
            'name' => $request->get('name'),
            'email' => $request->get('email'),
            'birth_date' => Carbon::make(
                $request->get('birth_date')
            ),
        ]);
    }
}