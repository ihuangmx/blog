# Laravel 生命周期之创建应用实例

`/public/index.php` 为应用的入口

首先，定义了一个常量，开发者可用该常量来测量 Laravel 框架的启动时间

```php
define('LARAVEL_START', microtime(true));
```

Composer 自动加载器, Laravel 没有任何关系

```php
require __DIR__.'/../vendor/autoload.php';
```

加载 `bootstrap/app.php` ，返回一个 `$app` 实例

```php
$app = require_once __DIR__.'/../bootstrap/app.php';
```

首先是创建实例

```php
$app = new Illuminate\Foundation\Application(
    $_ENV['APP_BASE_PATH'] ?? dirname(__DIR__)
);
```

`Application` 继承了 `Container` 类，`Container` 是服务容器的实现代码，看看构造函数

```php
public function __construct($basePath = null)
{
    if ($basePath) {
        $this->setBasePath($basePath);
    }

    $this->registerBaseBindings();
    $this->registerBaseServiceProviders();
    $this->registerCoreContainerAliases();
}
```

将所有的相关路径注册到应用中

```php
$this->setBasePath($basePath);

    protected function bindPathsInContainer()
    {
        $this->instance('path', $this->path());
        $this->instance('path.base', $this->basePath());
        $this->instance('path.lang', $this->langPath());
        $this->instance('path.config', $this->configPath());
        $this->instance('path.public', $this->publicPath());
        $this->instance('path.storage', $this->storagePath());
        $this->instance('path.database', $this->databasePath());
        $this->instance('path.resources', $this->resourcePath());
        $this->instance('path.bootstrap', $this->bootstrapPath());
    }
```

注册后，可进行解析

```php
app()->make('path.lang')
```

```
#instances: array:9 [
    "path" => "/Users/zen/xm5m/app/minghe-tag/app"
    "path.base" => "/Users/zen/xm5m/app/minghe-tag"
    "path.lang" => "/Users/zen/xm5m/app/minghe-tag/resources/lang"
    "path.config" => "/Users/zen/xm5m/app/minghe-tag/config"
    "path.public" => "/Users/zen/xm5m/app/minghe-tag/public"
    "path.storage" => "/Users/zen/xm5m/app/minghe-tag/storage"
    "path.database" => "/Users/zen/xm5m/app/minghe-tag/database"
    "path.resources" => "/Users/zen/xm5m/app/minghe-tag/resources"
    "path.bootstrap" => "/Users/zen/xm5m/app/minghe-tag/bootstrap"
  ]
```

注册容器别名

```php
$this->registerBaseBindings();
```


```php
protected function registerBaseBindings()
{	
	// 将自己的实例挂在到静态类上
    static::setInstance($this);

    // 绑定别名 app('app')
    $this->instance('app', $this);

    // app('Illuminate\Container\Container')
    $this->instance(Container::class, $this);
    $this->singleton(Mix::class);

    $this->instance(PackageManifest::class, new PackageManifest(
        new Filesystem, $this->basePath(), $this->getCachedPackagesPath()
    ));
}
```

注册核心服务提供者

```php
protected function registerBaseServiceProviders()
{
    $this->register(new EventServiceProvider($this));
    $this->register(new LogServiceProvider($this));
    $this->register(new RoutingServiceProvider($this));
}
```

`register` 方法一览

```php
    /**
     * Register a service provider with the application.
     *
     * @param  \Illuminate\Support\ServiceProvider|string  $provider
     * @param  bool   $force
     * @return \Illuminate\Support\ServiceProvider
     */
    public function register($provider, $force = false)
    {
        if (($registered = $this->getProvider($provider)) && ! $force) {
            return $registered;
        }

        // If the given "provider" is a string, we will resolve it, passing in the
        // application instance automatically for the developer. This is simply
        // a more convenient way of specifying your service provider classes.
        if (is_string($provider)) {
            $provider = $this->resolveProvider($provider);
        }

        $provider->register();

        // If there are bindings / singletons set as properties on the provider we
        // will spin through them and register them with the application, which
        // serves as a convenience layer while registering a lot of bindings.
        if (property_exists($provider, 'bindings')) {
            foreach ($provider->bindings as $key => $value) {
                $this->bind($key, $value);
            }
        }

        if (property_exists($provider, 'singletons')) {
            foreach ($provider->singletons as $key => $value) {
                $this->singleton($key, $value);
            }
        }

        $this->markAsRegistered($provider);

        // If the application has already booted, we will call this boot method on
        // the provider class so it has an opportunity to do its boot logic and
        // will be ready for any usage by this developer's application logic.
        if ($this->isBooted()) {
            $this->bootProvider($provider);
        }

        return $provider;
    }
```

