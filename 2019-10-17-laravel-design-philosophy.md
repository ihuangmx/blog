# Laravel 的设计哲学

## 单一职责

`UserController` 的 `index` 方法从数据库中获取全部用户，并返回渲染后的视图。

```php
class UserController extends Controller
{
    public function index()
    {
        $users = User::all();
        
        return view('users.index', compact('users'));
    }
}
```

为了提高应用效率，用户数据可能会保存在 Redis 中

```php
public function index()
{   
    // 因为数据获取的不同而修改了代码
    $users = Redis::get('users')
    
    return view('users.index', compact('users'));
}
```

该例子违反了类的 **单一职责**。控制器应当作为 **请求和响应的中介**，不应当因为其他理由而修改代码。然而，我们却因为数据获取的不同（与控制器的职责无关）而修改了代码。

## 仓库模式

对于控制器而言，并不需要知道数据是从 DB 还是从 Redis 中获取，只需要知道如何获取就行。数据的获取交给专门的仓库类处理即可。因此，分别定义一个 DB 仓库和一个 Redis 仓库来进一步划分职责。

DB 仓库

```php
<?php

namespace App\Repositories;

use App\User;

class DbUserRepository
{
    public function all(): array
    {
        return User::all()->toArray();
    }
}
```

Redis 仓库

```php
<?php

namespace App\Repositories;

class RedisUserRepository
{
    public function all(): array
    {
        return Redis::get('users');
    }
}
```

控制器不再负责数据的处理

```php
class UserController extends Controller
{
    private $users;

    public function __construct( )
    {
        $this->users = new DbUserRepository;
        // $this->users = new RedisRepository;
    }

    public function index()
    {   
        $users = $this->users->all();
        return $users;
    }
}
```

## 控制反转

虽然获取数据的职责委托给了仓库类，但是该例子仍然存在问题。我们直接在控制器的构造函数中 **主动声明需要依赖的对象**，这种在类中声明依赖对象的行为，也可以称为 **依赖正转**。

```php
public function __construct( )
{
    $this->users = new DbUserRepository;
    // $this->users = new RedisRepository;
}
```
依赖正转的不合理之处在哪里呢？位于高层的控制器依赖于具体的底层数据获取服务，当底层发生变动时，就需要对应的修改高层的内部结构。

我们对依赖关系进一步分析，可知控制器关注的并不是具体如何获取数据，控制器关注的是「数据的可获取性」这一抽象。因此，我们应当将依赖关系进行反转，将对依赖的具体声明职责转移到外部，让控制器仅依赖于抽象层（数据的可获取性）。这种解决方式称之为 **控制反转** 或 **依赖倒置**。通过控制反转，高层不再依赖于具体的底层，仅仅是依赖于抽象层，高层和底层实现了解耦。

## 依赖注入

懂得控制反转的含义后，就可以进一步实现控制反转了。实现控制反转的方式不止一种，其中最为常用的方式就是通过 **依赖注入**的方式。具体实现如下。

首先，用接口来表示「数据的可获取性」这一抽象

```php
<?php

namespace App\Repositories;

interface UserRepositoryInterface
{
    public function all(): array;
}
```

`UserController` 依赖的是「数据的可获取性」，不依赖于具体的实现

```php
class UserController extends Controller
{
    private $users;

    public function __construct(UserRepositoryInterface $users)
    {
        $this->users = $users;
    }
}
```

具体的实现交给对应的仓库类即可

```php
class DbUserRepository implements UserRepositoryInterface {}
class RedisRepository implements UserRepositoryInterface { }
```

根据自己的需要注入对应的服务，这样就实现了依赖注入。

```php
$userRepository = new DbUserRepository;
$userController = new UserController($userRepository)
```

总的来说，依赖注入由四部分构成

* 被使用的服务 - `DbUserRepository` 或者 `RedisRepository` 等
* 依赖某种服务的客户端 - `UserController`
* 声明客户端如何依赖服务的接口 - `UserRepositoryInterface`
* 依赖注入器，用于决定注入哪项服务给客户端

在上例中，我们的依赖注入器只是简单的手工注入，对于 Laravel 而言，依赖注入器则是通过服务容器来进行。

## 服务容器

Laravel 的服务容器是一个用于管理类的依赖和执行依赖注入的强大工具，主要由「服务绑定」和「服务解析」两部分构成，以下是一个简单的服务容器的实现

```php
namespace App\Services;

use Exception;

class Container 
{
    protected static $container = [];

    /**
     * 绑定服务
     * 
     * @param  服务名称 $name 
     * @param  Callable $resolver
     * @return void
     */
    public static function bind($name, Callable $resolver)
    {   
        static::$container[$name] = $resolver;
    }

    /**
     * 解析服务
     * 
     * @param  服务名称 $name
     * @return mix
     */
    public static function make($name)
    {
        if(isset(static::$container[$name])){
            $resolver = static::$container[$name];
            return $resolver();
        }
        
        throw new Exception("不存在该绑定");
   }
    
}
```

绑定服务

```php
App\Services\Container::bind('UserRepository', function(){
    return new App\Repositories\DbUserRepository;
});
```

解析服务

```php
$userRepository = App\Services\Container::make('UserRepository');
$userController = new UserController($userRepository)
```

Laravel 的服务容器的功能则更加的强大，比如，可以将接口与具体的实现进行绑定，通常在 **服务提供者** 中使用服务容器来进行绑定

```php
public function register()
{
    $this->app->singleton(UserRepositoryInterface::class, function ($app) {
        return new UserRepository;
    });
}
```

这样的话，我们就可以根据配置来进行灵活的切换，不需要手工的进行依赖注入。

## 自动解析依赖

Laravel 的服务容器最强大的地方在于可以通过反射来自动解析类的依赖，也就是说，大多数类可以自动解析，不需要在服务提供者中进行绑定。例如，我们在路由中只需要指定对应的控制器及方法，并不需要手动去实例化控制器

```php
Route::get('users', 'UserController@index');
```

`UserController` 除了依赖 `UserRepositoryInterface` 外，可能还会依赖于 `Request`，Laravel 是如何自动解析这些依赖并实例化控制器的呢，大致过程如下：

1. 服务容器中是否存在一个 `UserController` 的解析器？答案是否。
2. 通过反射检查下 `UserController` 的依赖。
3. 检测到 `UserController` 依赖于 `UserRepositoryInterface`，递归的对依赖进行处理，解析出 `UserRepositoryInterface`，其他依赖同理。
4. 最后，使用 `ReflectionClass->newInstanceArgs()` 方法来实例化 `UserController`

对应的源码

```php

/**
 * Resolve the given type from the container.
 *
 * @param  string  $abstract
 * @param  array  $parameters
 * @param  bool   $raiseEvents
 * @return mixed
 *
 * @throws \Illuminate\Contracts\Container\BindingResolutionException
 */
protected function resolve($abstract, $parameters = [], $raiseEvents = true)
{
    $abstract = $this->getAlias($abstract);

    // 获取该类的相关依赖绑定
    $needsContextualBuild = ! empty($parameters) || ! is_null(
        $this->getContextualConcrete($abstract)
    );

    // 单例模式直接返回，无需重新实例化
    if (isset($this->instances[$abstract]) && ! $needsContextualBuild) {
        return $this->instances[$abstract];
    }

    $this->with[] = $parameters;

    $concrete = $this->getConcrete($abstract);

    // 嵌套的解析依赖，构建服务
    if ($this->isBuildable($concrete, $abstract)) {
        $object = $this->build($concrete);
    } else {
        $object = $this->make($concrete);
    }

    // If we defined any extenders for this type, we'll need to spin through them
    // and apply them to the object being built. This allows for the extension
    // of services, such as changing configuration or decorating the object.
    foreach ($this->getExtenders($abstract) as $extender) {
        $object = $extender($object, $this);
    }

    // If the requested type is registered as a singleton we'll want to cache off
    // the instances in "memory" so we can return it later without creating an
    // entirely new instance of an object on each subsequent request for it.
    if ($this->isShared($abstract) && ! $needsContextualBuild) {
        $this->instances[$abstract] = $object;
    }

    if ($raiseEvents) {
        $this->fireResolvingCallbacks($abstract, $object);
    }

    // Before returning, we will also set the resolved flag to "true" and pop off
    // the parameter overrides for this build. After those two things are done
    // we will be ready to return back the fully constructed class instance.
    $this->resolved[$abstract] = true;

    array_pop($this->with);

    return $object;
}
```



