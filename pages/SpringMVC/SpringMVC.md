# SpringMVC[^1]

## Definition

Spring Web MVC is the original web framework built on the Servlet API and has been included in the Spring Framework from the very beginning. The formal name, "Spring Web MVC," comes from the name of its source module **spring-webmvc**, but it is more commonly known as "Spring MVC".[^2]

Web 应用的核心就是 **处理 HTTP 请求响应**。

在前后端分离的开发环境下，交互是通过字符串完成的。对象会序列化成字符串传输，传输到前端后再反序列化成响应的内容。



## 基本前后端交互

```java
@Controller
public class HelloWorldController {

    @ResponseBody
    @RequestMapping("/hello")
    public String helloWorld() {
        return "Hello SpringMVC, 你好～";
    }
}
```

1. 声明 controller
2. 将 url 和对应的方法做 mapping
3. return 的默认返回值是做页面跳转，可以使用  @ResponseBody 转换成响应体

4. 启动主程序，spring 会自动搞定 tomcat、servlet 等一系列需要的内容

   output:

   ```java
    :: Spring Boot ::                (v3.3.5)
   
    : Starting Springmvc01HelloworldApplication using Java 17.0.12 with PID 13206 
    : No active profile set, falling back to 1 default profile: "default"
    : Tomcat initialized with port 8080 (http)
    : Starting service [Tomcat]
    : Starting Servlet engine: [Apache Tomcat/10.1.31]
    : Initializing Spring embedded WebApplicationContext
    : Root WebApplicationContext: initialization completed in 305 ms
    : Tomcat started on port 8080 (http) with context path '/'
    : Started Springmvc01HelloworldApplication in 0.583 seconds (process running for 0.779)
    : Initializing Spring DispatcherServlet 'dispatcherServlet'
    : Initializing Servlet 'dispatcherServlet'
    : Completed initialization in 1 ms
   ```

5. 访问 localhost:8080/mapping 即可



## @ResponseBody

声明方法中的 return 从做页面跳转变为返回数据（响应体）



## @RestController

```java
@Controller
@ResponseBody
public @interface RestController {
    @AliasFor(
        annotation = Controller.class
    )
    String value() default "";
}
```

@RestController 注解标记于类上，是前后端分离开发的**标准注解**。

声明此类为 Controller 且其中均为 @ResponseBody



## @RequestMapping

将某一个方法和对应的路径做绑定，接收到路径就启动对应的方法执行。

### 通配符

在路径位置允许通配符的出现。通配符有**精确优先原则**。

**?**：匹配任意**一个字符**

*****：匹配任意**多个字符**

******：匹配任意**多层路径**。只能存在于路径最后

### 请求限定

```java
RequestMethod[] method() default {}; // 限定请求方式

String[] params() default {}; // 限定请求参数

String[] headers() default {}; // 限定请求头

String[] consumes() default {}; // 限定请求内容类型

String[] produces() default {}; // 指定给浏览器响应内容的类型
```

#### method()

```
public enum RequestMethod {
    GET,
    HEAD,
    POST,
    PUT,
    PATCH,
    DELETE,
    OPTIONS,
    TRACE;
}
```

##### Postman

为了测试不同的请求方式，可以使用 [postman](https://www.postman.com/downloads/) 来完成。

#### params()

have 2 situations. 

if write params = "age", means must include field age;

if write params = "age=18", means that must contain "age=18".

Can also write like "age!=18".

```java
@RequestMapping(value = "/params", params = "age")
public String testParams() {
    return "testParams";
}

@RequestMapping(value = "/paramsAndValue", params = "age=18")
public String testParamsAndValue() {
    return "testParamsAndValue";
}
```

#### headers()

请求头中必须包含 key 为 haha 的 K-V 对

```java
@RequestMapping(value = "/test3",headers = "haha")
public String test3() {
    return "test3";
}
```

#### consumes()

限定**请求的数据类型**必须为 json

```java
@RequestMapping(value = "/test4", consumes = "application/json")
public String test4() {
    return "test4";
}
```

#### produces()

指定给浏览器**响应体内的内容类型**。

如指定为 "text/html;charset=utf-8"，则浏览器会将接收到的响应体的内容按照 html 格式解析。

```java
@RequestMapping(value = "/test5", produces = "text/html;charset=utf-8")
public String test5() {
    return "<h1>Test5</h1>";
}
```



## HTTP



























































































































































































































































































































------



## 引用

遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接和本声明。

[^1]: https://www.bilibili.com/video/BV14WtLeDEit/?p=33&share_source=copy_web&vd_source=732a79db14c78dbec659a1afbe66586e

[^2]: https://docs.spring.io/spring-framework/reference/web/webmvc.html