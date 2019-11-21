# 确保网站的内容不会嵌入到其他站点中

HTTP 响应头部中，有一个字段，叫做 `X-Frame-Options`，该字段可以用来指示是否允许自己的网站被嵌入到其他网站的 `<iframe>` 或者 `<object>` 标签中。该头部有三个值

* `DENY` - 始终不允许嵌入，即使是同一个域名
* `SAMEORIGIN` - 只能在相同域名中嵌入
* `ALLOW-FROM uri` - 设置允许的域

通常，可以在 HTTP 代理中进行配置，比如 `nginx`

```
add_header X-Frame-Options SAMEORIGIN;
```

Laravel 自带了用来「只允许同域名嵌入」的中间件，我们只需要在 `/app/Http/Kernel.php` 中添加即可

```php
// /app/Http/Kernel.php
protected $middleware = [
    \Illuminate\Http\Middleware\FrameGuard::class,
];
```

该中间件的实现如下

```php
<?php

namespace Illuminate\Http\Middleware;

use Closure;

class FrameGuard
{
    /**
     * Handle the given request and get the response.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @return \Symfony\Component\HttpFoundation\Response
     */
    public function handle($request, Closure $next)
    {
        $response = $next($request);

        $response->headers->set('X-Frame-Options', 'SAMEORIGIN', false);

        return $response;
    }
}
```




