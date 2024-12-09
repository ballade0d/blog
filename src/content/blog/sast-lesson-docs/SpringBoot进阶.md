---
title: "Web组 第八次授课"
description: "SpringBoot进阶"
date: "Dec 8 2024"
---

# SpringBoot 进阶

## JWT

JWT（JSON Web Token）是一种开放标准（RFC 7519），用于在网络应用环境间安全地传递信息。信息以 JSON 对象的形式进行编码，经过签名保证安全性，通常用于身份验证和信息交换。

![jwt](/jwt.png)

一个 JWT 实际上是一个字符串，它由三部分组成，每部分之间用点（`.`）分隔，然后经过 Base64 处理：

- **Header（头部）**
- **Payload（负载）**
- **Signature（签名）**

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWUsImlhdCI6MTUxNjIzOTAyMn0.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

#### 1. Header

头部通常包括两部分：令牌的类型（即 JWT）和所使用的签名算法（例如 HMAC SHA256 或 RSA）。

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

#### 2. Payload

负载部分包含所要传递的信息，这些信息称为 claims（声明）。声明是关于实体（通常是用户）和其他数据的语句。有三种类型的声明：注册声明、公共声明和私有声明。

```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true,
  "iat": 1516239022
}
```

#### 3. Signature

要创建签名部分，必须使用头部指定的算法（例如 HMAC SHA256）对 header 和 payload 进行编码，并使用一个密钥。签名用于确认消息在传输过程中未被篡改、并验证发送方的身份。

## 拦截器

![interceptor](/interceptor.png)

### 定义拦截器

```java
import org.springframework.web.servlet.HandlerInterceptor;

@Component
public class MyInterceptor extends HandlerInterceptor	{

  @Override
	public boolean preHandle(@NonNull HttpServletRequest request, @NonNull HttpServletResponse response, @NonNull Object handler) {
      // 接收到请求时触发
	}

  @Override
	public void afterCompletion(@NonNull HttpServletRequest request, @NonNull HttpServletResponse response,
                                @NonNull Object handler, @Nullable Exception ex) {
        // 请求处理后触发
	}
}
```

### 注册拦截器

实现 WebMvcConfigurer 类，通过重写 addInterceptors 方法添加拦截器

```java
@Configuration
public class InterceptorConfig implements WebMvcConfigurer {

  @Resource
  private MyInterceptor interceptor;

	@Override
	public void addInterceptors(InterceptorRegistry registry) {
    registry.addInterceptor(interceptor).excludePathPatterns("/login");
  }
}
```

### 在拦截器中处理 JWT

一般来说，JWT 参数添加在请求头中

![request](/request.png)

```java
@Override
public boolean preHandle(@NonNull HttpServletRequest request, @NonNull HttpServletResponse response, @NonNull Object handler) {
  // 获取请求头的Authorization字段内容
	String header = request.getHeader("Authorization");
  if (header == null || !header.startsWith("Bearer ")) {
		throw new IllegalStateException("Please login first");
	}
	String token = header.substring(7);
  // 处理token
	return true;
}
```

### JWT 工具类

```java
import com.auth0.jwt.JWT;
import com.auth0.jwt.JWTVerifier;
import com.auth0.jwt.algorithms.Algorithm;
import com.auth0.jwt.exceptions.JWTVerificationException;
import com.auth0.jwt.interfaces.DecodedJWT;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

import java.time.Duration;
import java.time.Instant;
import java.util.Map;

@Component
public class JwtUtil {

    private final Algorithm algorithm;
    private final JWTVerifier verifier;
    private final Duration defaultExpireTime;

    public JwtUtil(@Value("${app.security.jwt.secret}") String secret,
                   @Value("${app.security.jwt.expiresIn}") Long expiresInMinutes) {
        this.algorithm = Algorithm.HMAC512(secret);
        this.verifier = JWT.require(algorithm).build();
        this.defaultExpireTime = Duration.ofMinutes(expiresInMinutes);
    }

    public DecodedJWT verify(String token) {
        try {
            return verifier.verify(token);
        } catch (JWTVerificationException e) {
            throw new IllegalArgumentException("Invalid JWT token");
        }
    }

    public String generateAccessToken(String id) {
        return generateAccessToken(id, defaultExpireTime);
    }

    public String generateAccessToken(String id, Duration expireTime) {
        Instant now = Instant.now();
        return JWT.create()
                .withSubject(id)
                .withIssuedAt(now)
                .withExpiresAt(now.plus(expireTime))
                .sign(algorithm);
    }

    public String generateAccessTokenWithClaims(String id, Map<String, String> claims) {
        Instant now = Instant.now();
        var jwtBuilder = JWT.create()
                .withSubject(id)
                .withIssuedAt(now)
                .withExpiresAt(now.plus(defaultExpireTime));
        claims.forEach(jwtBuilder::withClaim);
        return jwtBuilder.sign(algorithm);
    }

    public boolean isTokenExpired(DecodedJWT decodedJWT) {
        Instant expiresAt = decodedJWT.getExpiresAt().toInstant();
        return Instant.now().isAfter(expiresAt);
    }
}
```

## MyBatis

MyBatis 是一个持久层框架。它将 SQL 语句从 Java 代码中分离出来，并通过 XML 配置文件或注解的方式进行管理。

### 基本概念

- **\<mapper>**：定义了命名空间以及 SQL 语句和映射定义。
- **\<select>、\<insert>、\<update>、\<delete>**：定义具体的 SQL 语句。通过`id`属性标识，对应 mapper 接口的方法名。
- **\<resultMap>**：定义了如何从数据库结果集中映射数据为 Java 对象。
- **#{id}**：表示参数占位符。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="fun.sast.sasttodo.mapper.TaskMapper">
    <!-- 结果映射定义 -->
    <resultMap id="taskResultMap" type="fun.sast.sasttodo.entity.Task">
        <id property="id" column="id" />
        <result property="title" column="title" />
      ...
    </resultMap>

    <!-- 查询操作 -->
    <select id="selectTask" resultMap="taskResultMap">
        SELECT id, title FROM task WHERE id = #{id}
    </select>

    <!-- 插入操作 -->
    <insert id="insertTask">
        INSERT INTO task (id, title) VALUES (#{id}, #{title})
    </insert>

    <!-- 更新操作 -->
    <update id="updateTask">
        UPDATE task SET title = #{title} WHERE id = #{id}
    </update>

    <!-- 删除操作 -->
    <delete id="deleteTask">
        DELETE FROM task WHERE id = #{id}
    </delete>
</mapper>
```
