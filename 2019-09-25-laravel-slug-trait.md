# Laravel 自动维护 slug 字段

Slug 是 URL 的一部分，使 URL 根据更具有可读性，例如，这篇文章的英文标题为 `Laravel Slug Trait`，对应生成的 `slug` 为 `laravel-slug-trait`，那么，如何在 Laravel 中自动维护 slug 字段呢，以下是简单的实现。

## 定义迁移字段

通常，我们这样定义 `slug` 字段

```php
$table->string('slug')->unique()->comment('url slug');
```

可将其进行封装

```php
// /app/Providers/AppServiceProvider.php
use Illuminate\Database\Schema\Blueprint;

Blueprint::macro('sluggable', function ($column = 'slug') {
    return static::string($column)->unique()->commment('url slug');
});
```

快速定义

```php
$table->sluggable();
```

## 定义 Trait

`Laravel` 自带模型事件以及实用函数即可实现该功能

```php
<?php

namespace App\Models\Concerns;

use Illuminate\Support\Str;

trait Sluggable 
{	
	public static function bootSluggable()
	{
		static::saving(function ($model) {
			$model->slug = Str::slug(
				$model->sluggable()
			);
		});
	}


	/**
	 * Slug 字段
	 * 
	 * @return string
	 */
    abstract public function sluggable(): string;

}
```

## 使用

```php
<?php

namespace App\Models;

class Post extends Model
{	
	use Concerns\Sluggable;
	
	/**
	 * 文章标题作为 Slug 字段
	 * 
	 * @return string
	 */
	public function sluggable() : string
	{
		return 'title';
	}

	/**
	 * 指定路由模型绑定的的键为 slug
	 * 
	 * @return string
	 */
	public function getRouteKeyName()
	{
	    return 'slug';
	}
}
```

