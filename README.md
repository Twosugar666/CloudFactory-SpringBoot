# CloudFactory-SpringBoot
东北大学软件工程大二暑期实训项目——基于SpringBoot与MybatisPlus的云工厂管理系统
# 1.**架构**

**Java后台：**SpringBoot + Mybatis-Plus

**前端页面：**Vue + ElementUI + Jquery

# 2.**开发环境**

**数据库：**Mysql 8.0

**Java：**Jdk1.8

**网站：**Chrome浏览器，IE 9+

# 3.**代码包简介**

**Src：**源代码

**Pom.xml:** 项目maven依赖

# 4.**Java源代码**

### 4.1 **Src/main/java:**

#####   Java源代码

![image-20210718114600369](C:\Users\86130\AppData\Roaming\Typora\typora-user-images\image-20210718114600369.png)

### 4.2 **Src/main/java/com/example/Application.java：**

##### Springboot启动类

```java
package com.example;

import lombok.extern.slf4j.Slf4j;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@Slf4j
@MapperScan("com.example.mapper")
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
        log.warn("请访问：http://localhost:9999/page/end/login.html");
    }
}
```

### 4.3 **Src/main/java/com/example/controller:**

#####  Controller层，定义接口路由，前后台数据交互的入口。

![image-20210718114642351](C:\Users\86130\AppData\Roaming\Typora\typora-user-images\image-20210718114642351.png)

### 4.4 **Src/main/java/com/example/service：**

##### Service层，处理一些业务，例如从数据库查询账号密码，判断是否存在等，可以理解成对数据库查询结果的一个处理类，由于系统业务比较简单，这里直接继承了MybatisPlus的ServiceImpl，可以使用MybatisPlus自带的增删改查方法。

![image-20210718114703889](C:\Users\86130\AppData\Roaming\Typora\typora-user-images\image-20210718114703889.png)

### 4.5 **Src/main/java/com/example/mapper：**

##### 数据接口层，负责操作数据库，定义一些操作数据库的接口，同样的，我们使用MybatisPlus，这里就直接使用它自带的增删改查方法即可。

![image-20210718114826739](C:\Users\86130\AppData\Roaming\Typora\typora-user-images\image-20210718114826739.png)

### 4.6 **Src/main/java/com/example/entity:** 

##### 实体类，这个类定义了Java的实体对象和数据库表字段的一一映射关系，这样我们就可以使用Java对象来操作数据库了。

![image-20210718114850846](C:\Users\86130\AppData\Roaming\Typora\typora-user-images\image-20210718114850846.png)

### 4.7 **Src/main/java/com/example/common/WebMvcConfig.java:**

#####  定义拦截器，拦截登录请求

```java
package com.example.common;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

/**
 * 拦截器配置
 **/
@Configuration
public class WebMvcConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(authInterceptor())
//                .addPathPatterns("/api/**")
                .addPathPatterns("/page/end/**")
                .excludePathPatterns("/page/end/login.html", "/page/end/register.html", "/page/end/auth.html", "/page/end/person.html");
//                .excludePathPatterns("/api/user/login", "/api/user/register");
    }

    @Bean
    public AuthInterceptor authInterceptor() {
        return new AuthInterceptor();
    }
}
```

### 4.8 **Src/main/java/com/example/common/AuthInterceptor.java：**

##### 拦截器的具体操作

```java
package com.example.common;

import com.example.entity.Permission;
import com.example.entity.User;
import com.example.service.UserService;
import org.springframework.web.servlet.HandlerInterceptor;

import javax.annotation.Resource;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.List;

/**
 * 拦截器
 */
public class AuthInterceptor implements HandlerInterceptor {

    @Resource
    private UserService userService;

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws IOException {
        String servletPath = request.getServletPath();
        User user = (User) request.getSession().getAttribute("user");
        if (user == null) {
            response.sendRedirect("/page/end/login.html");
            return false;
        }
//        List<Permission> permissions = userService.getPermissions(user.getId());
//        if (permissions.stream().noneMatch(p -> servletPath.contains(p.getFlag())) && !servletPath.contains("index")) {
//            response.sendRedirect("/page/end/auth.html");
//            return false;
//        }
        return true;
    }

}
```

### 4.9 **Src/main/java/com/example/common/CorsConfig.java：**

##### 配置跨域

```java
package com.example.common;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.UrlBasedCorsConfigurationSource;
import org.springframework.web.filter.CorsFilter;

@Configuration
public class CorsConfig {

    // 当前跨域请求最大有效时长。这里默认1天
    private static final long MAX_AGE = 24 * 60 * 60;

    private CorsConfiguration buildConfig() {
        CorsConfiguration corsConfiguration = new CorsConfiguration();
        corsConfiguration.addAllowedOrigin("*"); // 1 设置访问源地址
        corsConfiguration.addAllowedHeader("*"); // 2 设置访问源请求头
        corsConfiguration.addAllowedMethod("*"); // 3 设置访问源请求方法
        corsConfiguration.setMaxAge(MAX_AGE);
        return corsConfiguration;
    }

    @Bean
    public CorsFilter corsFilter() {
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", buildConfig()); // 4 对接口配置跨域设置
        return new CorsFilter(source);
    }
}
```

### 4.10 **Src/main/java/com/example/common/MybatisPlusConfig.java:**

#####  Mybatis-Plus配置

```java
package com.example.common;
import com.baomidou.mybatisplus.extension.plugins.PaginationInterceptor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
/**
 *  mybatis-plus 分页插件
 */
@Configuration
public class MybatisPlusConfig {

    /**
     * 分页插件
     */
    @Bean
    public PaginationInterceptor paginationInterceptor() {
        return new PaginationInterceptor();
    }
}
```

### 4.11 **Src/main/java/com/example/common/Result.java：**

##### Controller统一返回的Json包装类

```java
package com.example.common;
public class Result<T> {
    private String code;
    private String msg;
    private T data;
    public String getCode() {
        return code;
    }
    public void setCode(String code) {
        this.code = code;
    }
    public String getMsg() {
        return msg;
    }
    public void setMsg(String msg) {
        this.msg = msg;
    }
    public T getData() {
        return data;
    }
    public void setData(T data) {
        this.data = data;
    }
    public Result() {
    }
    public Result(T data) {
        this.data = data;
    }
    public static Result success() {
        Result result = new Result<>();
        result.setCode("0");
        result.setMsg("成功");
        return result;
    }
    public static <T> Result<T> success(T data) {
        Result<T> result = new Result<>(data);
        result.setCode("0");
        result.setMsg("成功");
        return result;
    }
    public static Result error(String code, String msg) {
        Result result = new Result();
        result.setCode(code);
        result.setMsg(msg);
        return result;
    }
}
```

# 5.**配置文件**

##### Application.yml: 配置项目端口、数据库

```yml
server:
  port: 9999
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    username: root
    password: 622001
    url: jdbc:mysql://localhost:3306/x-admin?useUnicode=true&characterEncoding=utf-8&allowMultiQueries=true&useSSL=false&serverTimezone=GMT%2b8
    type: com.alibaba.druid.pool.DruidDataSource
#  jpa:
#    generate-ddl: false
#    hibernate:
#      ddl-auto: none
  servlet:
    multipart:
      max-file-size: 100MB
      max-request-size: 100MB

mybatis:
  mapper-locations: classpath:mapper/*.xml

logging:
  level:
    com:
      example:
        mapper:
          debug

```

# 6.**页面**

![image-20210718113807918](C:\Users\86130\AppData\Roaming\Typora\typora-user-images\image-20210718113807918.png)

**Resources/static/page/end:** 后台页面

![image-20210718115142885](C:\Users\86130\AppData\Roaming\Typora\typora-user-images\image-20210718115142885.png)

**Resources/static/page/front：**前台页面

![image-20210718115154883](C:\Users\86130\AppData\Roaming\Typora\typora-user-images\image-20210718115154883.png)
