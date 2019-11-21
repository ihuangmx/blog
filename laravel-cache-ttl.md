# 缓存时间单位的使用

Laravel 5.8 将缓存项的时间单位从分钟调整到了秒，主要是为了遵循 `PSR-16` 的相关规范：

> 缓存类**必须**实现 `Psr\SimpleCache\CacheInterface` 接口，并且**必须**以整数秒作为缓存有效时长的最小粒度。其中，有效时长(`TTL`)是指一个缓存项从存储到过期的时间长度。`TTL` 一般用以秒为单位的整数或者 `DateInterval` 实例对象表示。

`5.8` 

```php
// 30 分钟
Cache::put('foo', 'bar', 30);
```

`5.8`

```php
// 30 秒
Cache::put('foo', 'bar', 30);
```

在 `Laravel` 的项目中，我们`强烈推荐`使用 `Carbon` 相关运算来代表 `TTL`，能够大大提高代码的可读性

```php
// 30 秒，可读性不强
Cache::put('foo', 'bar', now()->addSeconds(30));
// 一天，可读性强
Cache::put('foo', 'bar', now()->addDay());
```