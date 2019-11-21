# Laravel 按照模块的方式组织路由文件

Laravel 的路由文件默认分为 `web.php` 和 `api.php` 两个，正常来说，网站的非 `api` 路由都会写在 `web.php` 中，但是随着项目的复杂度增加，`web.php` 文件会变得越来越臃肿，这时候，可以考虑下对路由文件进行分割了。

我们以后台管理的路由为例进行介绍。首先，创建目录  `routes/admin`，用于存放后台的路由，我们的设想是以「模块」的形式来组织路由文件，即 `admin` 目录下存放各个模块的路由，例如

* `good.php` - 用于存放商品有关的路由，如商品、商品分类、商品标签等
* `crm.php` - 用于存放往来单位有关的路由，如客户、联系人、供应商等

那么，如何让这些路由文件生效呢，只需要在 `RouteServiceProvider` 手动加载路由文件即可

```php
// /app/Providers/RouteServiceProvider.php

public function map()
{
    $this->mapAdminRoutes();
}

protected function mapAdminRoutes()
{
    Route::middleware('web')
         ->namespace($this->namespace)
         ->group(function($route){
            collect(
                glob(base_path('routes/admin') . '/*.php')
            )
            ->each(function($fileName){
                require $fileName;
            });
         });
}
```

在生产环境中，运行命令

```bash
$ php artisan route:cache
```

该命令会将所有的路由缓存为单个文件 `/bootstrap/cache/routes.php`，因此不需要担心模块化分割路由文件导致的性能下降问题。

