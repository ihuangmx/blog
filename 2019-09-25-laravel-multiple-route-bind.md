# Laravel 多键路由绑定

Laravel 可以通过两种方式进行路由模型绑定。

在模型中指定路由的键

```php
/**
 * 获取该模型的路由的自定义键名。
 *
 * @return string
 */
public function getRouteKeyName()
{
    return 'slug';
}
```

在服务提供者中手动进行绑定

```php
Route::bind('post', function ($value) {
    return App\Post::where('slug', $value)->first() ?? abort(404);
 });
}
```

如何同时支持 `id` 和 `slug` 的路由模型绑定呢？同样有两种方式实现。

一是在模型重写 `resolveRouteBinding` 方法

```php
/**
 * Retrieve the model for a bound value.
 *
 * @param  mixed  $value
 * @return \Illuminate\Database\Eloquent\Model|null
 */
public function resolveRouteBinding($value)
{
    return $this->where($this->getRouteKeyName(), $value)->first();
}
```

重写

```php
/**
 * Retrieve the model for a bound value.
 *
 * @param  mixed  $value
 * @return \Illuminate\Database\Eloquent\Model|null
 */
public function resolveRouteBinding($value)
{
    if( is_numeric($value) ){
        return self::findOrFail($value);
    } else {
        return self::whereSlug($value)->firstOrFail();
    }
}
```

也可以在服务提供者中进行绑定

```php
Route::bind('post', function ($value) {
    if( is_numeric($value) ){
        return App\Post::findOrFail($value);
    } else {
        return App\Post::whereSlug($value)->firstOrFail();
    }
 });
}
```




