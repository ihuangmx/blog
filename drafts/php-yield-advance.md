# 生成器（高级篇）

### 发送值

可在一个表达式上下文中使用 `yield`

```php
<?php

function myGenerator()
{
    $foo = yield 'foo';
    $bar = yield 'key' => 'bar';
}

$gen = myGenerator();
$gen->current(); // foo
$gen->current(); // bar
$gen->key(); // key
```

通常与 `send` 方法结合使用，用于给生成器发送值
`Generator::send` 比较难以理解，官方说明如下

> Sends the given value to the generator as the result of the current yield expression and resumes execution of the generator.

也就是说，该方法做了两件事

1. 将 `send` 传递的值作为当前 `yield` 表达式的结果
2. 执行下一个生成器

```php
<?php

function myGenerator()
{   
    $first = yield "foo";
    $second = yield $first;
}

$gen = myGenerator();

// 当前生成器是 $first = yield "foo";
$gen->current(); // foo

$gen->send('bar'); 
// 将 bar 传递给当前生成器，所以 $first 现在的值变成了 bar
// 继续执行下一个生成器 $second = yield $first，所以 $second 的值变成了 bar

$gen->current(); // bar
```

通过发送值来中断生成器

```php
<?php

function nums()
{
    for ($i = 0; $i < 5; ++$i) {

        $cmd = yield $i;
       
        if ($cmd == 'stop') {
            return;
        }
    }
}

$gen = nums();
foreach($gen as $v)
{
    if($v == 3) {
        $gen->send('stop');
        // 传入 stop
    }
   
    echo "{$v} ";
}
```

