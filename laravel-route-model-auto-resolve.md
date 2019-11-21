# 模型路由自动解析

`5.8` 之前，Laravel 的模型和策略之间的关系需要进行显示的绑定

```php
// /app/Providers/AuthServiceProvider.php
protected $policies = [
    'App\User' => 'App\Policies\UserPolicy',
];
```

`5.8` 引入模型策略自动发现机制，但需要遵循一定的规范，即策略类都必须位于模型类所在目录的 `Policies` 目录中。

示例：

* `App\User` 的策略将被解析成  `App\Policies\UserPolicy`
* `App\Models\User` 的策略将被解析成 `App\Models\Policies\UserPolicy`


如果不符合该规范，将无法进行模型策略自动发现。例如，模型的目录为 `App\Models`，策略的目录为 `App\Policies`，自动发现机制将无法生效。此时可以手动定义自动发现规则

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