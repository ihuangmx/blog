# Laravel Api 的简单实现

Laravel 的控制器，只需要简单封装，就能够进行 Api 开发

1. 在控制器中注入一个转化层，用于处理要返回的数据；
2. 定义一个基类控制器 `ApiController`，用于返回数据；

## 定义控制器基类

ApiController 的简单示例

```php
<?php

namespace App\Http\Controllers;


class ApiController extends Controller
{   

    /**
     * 返回码
     * @var integer
     */
    private $statusCode = 200;

    public function __construct($statusCode)
    {
        $this->statusCode = $statusCode;
    }

    /**
     * 获取返回码
     * 
     * @return int
     */
    public function getStatusCode()
    {
        return $this->statusCode;
    }

    /**
     * 设置返回码
     * 
     * @param int $statusCode
     * 
     * @return $this
     */
    public function setStatusCode($statusCode)
    {
        $this->statusCode = $statusCode;
        return $this;
    }

    /**
     * 响应
     * 
     * @param  $data
     * 
     * @return Illuminate\Http\JsonResponse
     */
    public function response($data)
    {   
        return response()->json($data);
    }

    /**
     * 错误响应
     * @param  string $message 
     * 
     * @return   Illuminate\Http\JsonResponse       
     */
    public function responseErrors($message = 'Not Found')
    {
        return $this->response([
            'status'      => 'failed',
            'status_code' => $this->getStatusCode(),
            'massage'     => $message
        ]);
    }

    /**
     * 成功响应数据
     * 
     * @param  string $message 
     * 
     * @return Illuminate\Http\JsonResponse         
     */
    public function responseSuccess($data, $message = 'Success')
    {
        return $this->response([
            'status'      => 'success',
            'status_code' => $this->getStatusCode(),
            'massage'     => $message,
            'data'        => $data
        ]);
    }

    /**
     * 成功响应
     * 
     * @param  string $message 
     * 
     * @return Illuminate\Http\JsonResponse         
     */
    public function responseOk($message ='Ok')
    {
        return $this->response([
            'status'      => 'success',
            'status_code' => $this->getStatusCode(),
            'massage'     => $message
        ]);
    }
}
```

## 定义转换层

定义一个抽象的转化层，用于格式化要返回的数据

```php
<?php
namespace App\Transformers;

abstract class Transformer
{   
    /**
     * 转换多条数据
     * 
     * @param  array $items 
     * 
     * @return array
     */
    public function transforms(array $items)
    {
        return array_map([$this, 'transform'], $items);
    }

    abstract public function transform(array $item);
}
```

常规的转化层类

```php
<?php
namespace App\Transformers;

class LessonsTransformer extends Transformer
{
    /**
     * 处理单挑数据
     * 
     * @param  array  $lesson 
     * @return array        
     */
      public function transform(array $lesson)
      {
          return [
            'title'   => $lesson['title'],
            'content' => $lesson['body'],
            'is_free' => (bool) $lesson['free'],
          ];
      }
}
```

## 使用

```php
<?php

namespace App\Http\Controllers;

use App\Lesson;
use App\Transformers\LessonsTransformer;
use Illuminate\Http\Request;

class LessonController extends ApiController
{
    private $transform;

    public function __construct(LessonsTransformer $transform)
    {
        $this->transform = $transform;
    }


    /**
     * Display a listing of the resource.
     *
     * @return \Illuminate\Http\Response
     */
    public function index()
    {
        $lessons = Lesson::all();
        return $this->responseSuccess(
        	$this->transform->transforms($lessons->toArray())
        );
    }


    /**
     * Display the specified resource.
     *
     * @param  \App\Lesson  $lesson
     * @return \Illuminate\Http\Response
     */
    public function show(Lesson $lesson)
    {
        return $this->responseSuccess(
        	$this->transform->transform($lesson->toArray())
        );
    }

}

```
