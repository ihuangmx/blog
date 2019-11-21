# 深入浅出 Json Web Token

## 原理

Web 应用可分为客户端和服务端，这两者之间经常需要进行身份认证。

![img](imgs/jwt-web-and-server-auth.jpeg)

由于 HTTP 是无状态协议，不能保存认证后的用户状态，因此，每一次发送请求都需要重复的进行认证。

<img src="imgs/http-stateless-protocol.png" alt="image-20191113154353888" style="zoom:75%;" />

为了解决该问题，通常会使用 Cookie 来管理 Session，来实现用户状态管理。

<img src="imgs/http-session-cookie-auth.png" alt="image-20191113153610704" style="zoom:67%;" />

服务器需要记录 Session 信息，当存在多台服务器时，这些服务器就需要共享 Session。例如，将 Session 数据进行持久化存储，每台服务器都从持久层中获取 Session。这种方案也有风险，当持久层突然失效时，所有登录信息都将失效。

![4gKshCJIgU](imgs/session-persistence.png)


由此可见，服务端保存认证状态是一件棘手的事，既然如此，何不将数据全都保存在客户端呢？Json Web Token 就是基于该思路提出的一种认证方案。

我们从 JSON Web Token 的命名就可以得出一些信息

* JSON - 数据是 JSON 格式的
* Web -  是用于 Web 的
* Token - Token 是令牌的意思，代表了 JWT 是一种认证的凭证

![Image result for Json Web Token](imgs/json-web-token.png)

JWT 的认证过程如下：

1. 客户端发送登录信息（用户 ID，密码）
2. 服务端基于密钥生成 JWT，返回给客户端
3. 客户端在接下来的请求中将 Token 放在头部中一起发送给服务端
4. 服务端对 JWT 进行验证

![JWT Diagram](imgs/jwt-diagram.png)

生成的 JWT 的格式如下

![image-20191113172725564](imgs/jwt-string.png)

 JWT 可分为三部分，用 `.` 符号隔开

* Header 头部
* Payload 负载
* Signature  签名

Header 用于携带一些元数据，比如加密算法与类型

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

Payload  携带令牌的具体内容，常用的内容如下

- iss (issuer) 签发人
- exp (expiration time) 过期时间
- sub (subject) 主题
- aud (audience) 受众
- nbf (Not Before) 生效时间
- iat (Issued At) 签发时间
- jti (JWT ID) 编号

可根据需要自行选择，也可以自己定义

```json
{
  "sub": "1",
  "name": "Mind Geek",
  "admin": true
}
```

Signature 是加密 Header 和 Payload 后得到的签名，防止数据篡改，加密公式如下

```
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  mind-geek-jwt
)
```

解读

* HMACSHA256 是 Header 指定的算法
* `mind-geek-jwt` 则是服务端定义的密钥
* JWT 作为一个令牌（token），有些场合可能会放到 URL 中。Base64 有三个字符 `+`、`/` 和 `=`，在 URL 里面有特殊含义，所以要被替换 `=` 被省略、`+` 替换成 `-`，`/` 替换成 `_` 。这就是 Base64URL 算法。


## 示例

### 加密

定义 Header

```php
$header = [
  'alg' => 'HS256',
  'typ' => 'JWT',
];
```

定义 Payload

```php
$payload = [
  'sub'   => 1,
  'name'  => 'Mind Geek',
  'admin' => true,
];
```

定义密钥

```php
$secret = 'mind-geek-jwt';
```


将  Header 和 Payload 数组转化成 Json，再使用 Base64URL 算法进行编码。

```php
function base64url(string $string)
{
  return str_replace('=', '', strtr(base64_encode($string), '+/', '-_'));
}

// Header
$base64Header = base64url(json_encode($header));

// Payload
$base64Payload = base64url(json_encode($payload));
```

将编码后的 Header 和 Payload 用 `.` 拼接起来

```php
$encryp = $base64Header.".".$base64Payload;
```

对其进行加密，加密后再进行 Base64URL 编码，得到 Signature

```php
$signature = hash_hmac('sha256', $encryp, $secret, true);
$base64Signature = base64url($signature);
```

拼接之后就是 JWT 字符串了

```php
$token = $base64Header.".".$base64Payload.".".$base64Signature;
```

### 验证

获取客户端发送的 JWT

```php
$token = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOjEsIm5hbWUiOiJNaW5kIEdlZWsiLCJhZG1pbiI6dHJ1ZX0.0_dneYOin4yWRYlD-KmfvGEY6AhjA_zDyyvPhgYq2sU';
```

解析 JWT，得到编码过后的  Header、Payload 以及 Signature

```php
list($base64Header, $base64Payload, $base64Signature) = explode('.', $token);
```

利用得到的 `Header` 和 `Payload` ，以及服务端保存的密钥来计算出签名

```php
$encryp = $base64Header.".".$base64Payload;
$signature = hash_hmac('sha256', $encryp, $secret, true);
$computedBase64Signature = base64url($signature);
```

将计算出来的签名与客户端发送的签名进行对比

```php
if($computedBase64Signature === $base64Signature){
    echo "认证成功!";
}
```

参考链接：

* [JSON Web Token 入门教程 - 阮一峰的网络日志](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html)
* [JSON Web Tokens - jwt.io](https://jwt.io/)
* [图解HTTP (豆瓣)](https://book.douban.com/subject/25863515/)
* [理解 cookie、session、token、jwt | PHP 技术论坛](https://learnku.com/articles/30051)