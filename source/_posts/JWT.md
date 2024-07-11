# JWT

## 什么是JWT

​	JWT(JSON WEB TOKEN)是一种开放标准（RFC 7519），它定义了一种紧凑的、自包含的方式，用于在各方之间以 JSON 对象安全地传输信息。JWT 可以对声明进行签名，从而确保数据的真实性和完整性，并且可以加密以保护数据的机密性。JWT 通常用于身份验证和授权场景。

## JWT的组成

- **Header（头部）**

  ```java
  {
    "alg": "HS256",//加密算法
    "typ": "JWT"//令牌类型
  }
  
  ```

- **Payload（载荷）**

  ```java
  {
    "sub": "1234567890",
    "name": "John Doe",
    "admin": true
  }
  ```

- **Signature（签名）**

  ```JAVA
  HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload), secret);
  ```

  ​	将header和payload加密后用 "."连接就是Signature(签名)

## 生成JWT

```java
public static String createToken(){
    private static String signature_key = "123456";//为安全考虑 通常不是这么写的
        JwtBuilder jwtBuilder = Jwts.builder();
        String JWTToken = jwtBuilder
                //header
                .setHeaderParam("typ", "JWT")
                .setHeaderParam("alg", "HS256")
                //payload
                .claim("user", "admin")
                .claim("roles", "admin")
                .setSubject("admin")
                .setExpiration(new java.util.Date(System.currentTimeMillis() +3000 ))
                //signature
                .signWith(io.jsonwebtoken.SignatureAlgorithm.HS256, signature_key)
                .compact();//将整个 JWT 令牌序列化为字符串
        return JWTToken;
    }
```



## 校验JWT

​	如果token有错误或者过期了就会报错

```java
 public static boolean checkToken(String token){
     private static String signature_key = "123456";//为安全考虑 通常不是这么写的
        try{
            Claims claims = Jwts.parser().setSigningKey(signature_key).parseClaimsJws(token).getBody();
            Date expiration = claims.getExpiration();
            if (expiration.before(new Date())) {
                // JWT过期，需要重置token
                // TODO: 重置token的逻辑
                System.out.println("token expired");
                return false;
            }
            return true;
        } catch (ExpiredJwtException e) {
            System.out.println("Token is expired");
        } catch (UnsupportedJwtException e) {
            System.out.println("Unsupported JWT token");
        } catch (MalformedJwtException e) {
            System.out.println("Malformed JWT token");
        } catch (SignatureException e) {
            System.out.println("Invalid JWT signature");
        } catch (IllegalArgumentException e) {
            System.out.println("Illegal argument token");
        }
        return false;
    }
```

## 实例

```java
package com.example.demo.util;
import io.jsonwebtoken.*;

import java.util.Date;
public class jwt {
    private static String signature_key = "123456";//为安全考虑 通常不是这么写的
    public static String createToken(){
        JwtBuilder jwtBuilder = Jwts.builder();
        String JWTToken = jwtBuilder
                //header
                .setHeaderParam("typ", "JWT")
                .setHeaderParam("alg", "HS256")
                //payload
                .claim("user", "admin")
                .claim("roles", "admin")
                .setSubject("admin")
                .setExpiration(new java.util.Date(System.currentTimeMillis() +3000 ))
                //signature
                .signWith(io.jsonwebtoken.SignatureAlgorithm.HS256, signature_key)
                .compact();//将整个 JWT 令牌序列化为字符串
        return JWTToken;
    }
    public static boolean checkToken(String token){
        try{
            Claims claims = Jwts.parser().setSigningKey(signature_key).parseClaimsJws(token).getBody();
            Date expiration = claims.getExpiration();
            if (expiration.before(new Date())) {
                // JWT过期，需要重置token
                // TODO: 重置token的逻辑
                System.out.println("token expired");
                return false;
            }
            return true;
        } catch (ExpiredJwtException e) {
            System.out.println("Token is expired");
        } catch (UnsupportedJwtException e) {
            System.out.println("Unsupported JWT token");
        } catch (MalformedJwtException e) {
            System.out.println("Malformed JWT token");
        } catch (SignatureException e) {
            System.out.println("Invalid JWT signature");
        } catch (IllegalArgumentException e) {
            System.out.println("Illegal argument token");
        }
        return false;
    }
    public static void main(String[] args) throws InterruptedException {
        String token = createToken();
        System.out.println(token);
        System.out.println(checkToken(token));
        Thread.sleep(3000);
        System.out.println(checkToken(token));
    }
}
```

这个程序的输出的结果为

```java
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoiYWRtaW4iLCJyb2xlcyI6ImFkbWluIiwic3ViIjoiYWRtaW4iLCJleHAiOjE3MjA2OTA4MDl9.S7gZajm8EtGZYc_BYKVOFIbktkalg_KnFQHbbAvW9TY//生成的token
true//校验结果
Token is expired//token过期了
false//校验结果
```

讲上方生成的token复制到[JSON Web Tokens - jwt.io](https://jwt.io/)可以查看解密后的token

## 登录业务

1. 用户在客户端输入账号密码登录
2. 服务端校验密码
3. 密码正确，服务端生成一个token返回给客户端，客户端会将token存储下来（LocalStorage或者Session）
4. 客户端在后面的请求中都会携带token,服务器每次处理请求都会先校验一次token再返回客户端的请求