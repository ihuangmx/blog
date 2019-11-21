# Laravel 模型路由自动解析

`5.8` 之前，模型和策略之间的关系需要显示的进行注册

```php
// /app/Providers/AuthServiceProvider.php
protected $policies = [
    'App\User' => 'App\Policies\UserPolicy',
];
```

`5.8` 引入模型策略自动发现机制，但需要遵循一定的规范，即策略类都必须位于模型类所在目录的 `Policies` 目录中

```
App\User 对应 App\Policies\UserPolicy
App\Models\User 对应 App\Models\Policies\UserPolicy
```

如果不符合该规范，将无法进行模型策略自动发现，这时候就需要自定义解析规则，例如，要实现 `App\Policies` 对应 `App\Models` ，需要自定义 `Gate::guessPolicyNamesUsing` 

```php
// /app/Providers/AuthServiceProvider.php

use Illuminate\Support\Facades\Gate;

public function boot()
{
    $this->registerPolicies();

    Gate::guessPolicyNamesUsing(function($modelName){
    
        $name = class_basename($modelName).'Policy';
        
        return "App\\Policies\\{$name}";
    });
}
```