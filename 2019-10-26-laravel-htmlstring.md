# Laravel 小知识点之 HtmlString 类

Laravel 中实现了一个非常简单的 `HtmlString` 类

```php
<?php

namespace Illuminate\Support;

use Illuminate\Contracts\Support\Htmlable;

class HtmlString implements Htmlable
{
    /**
     * The HTML string.
     *
     * @var string
     */
    protected $html;

    /**
     * Create a new HTML string instance.
     *
     * @param  string  $html
     * @return void
     */
    public function __construct($html)
    {
        $this->html = $html;
    }

    /**
     * Get the HTML string.
     *
     * @return string
     */
    public function toHtml()
    {
        return $this->html;
    }

    /**
     * Get the HTML string.
     *
     * @return string
     */
    public function __toString()
    {
        return $this->toHtml();
    }
}
```

该类实现了 `Htmlable` 接口

```php
<?php

namespace Illuminate\Contracts\Support;

interface Htmlable
{
    /**
     * Get content as a string of HTML.
     *
     * @return string
     */
    public function toHtml();
}
```

我们来看看 `HtmlString` 类的应用

```php
if (! function_exists('method_field')) {
    /**
     * Generate a form field to spoof the HTTP verb used by forms.
     *
     * @param  string  $method
     * @return \Illuminate\Support\HtmlString
     */
    function method_field($method)
    {
        return new HtmlString('<input type="hidden" name="_method" value="'.$method.'">');
    }
}
```

那么，该类有什么用呢？

* 代表对象实现了 `Htmlable` 接口，代表了可转化成 `HTML`，在 `blade` 中可以直接使用，不需要进行额外处理；
* `__toString` 方法的使用让我们在模板中可以优雅的使用 `HtmlString` 对象，例如直接使用`{!! $htmlStringObj !!}` 而不是 `{!! $htmlStringObj->toHtml() !!}`

