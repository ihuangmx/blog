# 自定义视图组件

Laravel 的[视图合成器](https://learnku.com/docs/laravel/6.x/views/5141#sharing-data-with-all-views)可将数据与指定视图绑定在一起，避免了重复编写代码。

```php
View::composer('profile', 'App\Http\View\Composers\ProfileComposer');
```

由于数据的生成和渲染是分开进行的，理解起来不够直观。因此，可以采用视图组件的方式将两者进行封装。

```php
<?php

namespace App\ViewComponents;

use Illuminate\Contracts\Support\Htmlable;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\View;

class ExampleComponent implements Htmlable
{
    private $color;

    private $request;

    public function __construct(Request $request, string $color)
    {
        $this->color = $color;
        $this->request = $request;
    }

    public function toHtml()
    {   
        return View::make('example')
            ->with('color', $this->color)
            ->render();
    }
}
```

在视图中使用

```php
{{ app()->makeWith(App\ViewComponents\ExampleComponent::class,['color' => 'green'])->toHtml() }}
```

封装指令

```php	
Blade::directive('render', function ($expression) {
    list($class, $params) = explode(',', $expression, 2);
    $class = "App\\ViewComponents\\".trim($class, '\'" ');
    return "<?php echo app()->makeWith('$class', $params)->toHtml(); ?>";
});
```

使用指令

```php
@render('ExampleComponent', ['color' => 'green'])
```

参考资料

* [spatie/laravel-view-components: A better way to connect data with view rendering in Laravel](https://github.com/spatie/laravel-view-components)
* [Introducing View Components in Laravel, an alternative to View Composers - Laravel News](https://laravel-news.com/introducing-view-components-on-laravel-an-alternative-to-view-composers)


