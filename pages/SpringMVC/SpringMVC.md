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



## 请求处理

### 分别收集请求参数

```java
@RestController
public class IndexDemoController {

    @RequestMapping("/handle01")
    /*
    http://localhost:8080/handle01?
    username=1&
    password=2&
    cellphone=1&
    agreement=on
     */
    public String handle01(String username,String password,String cellphone, String agreement) {
        System.out.println(username);
        System.out.println(password);
        System.out.println(cellphone);
        System.out.println(agreement);
        return "ok!";
    }
}
```

请求为注释内的内容。requestMapping 的地址为 form 的 action。

普通变量的键值对可以自动封装。

**默认情况下，请求中的每个参数都会携带内容，如果内容为空，包装类型则自动封装为 null，基本类型自动为默认值。**

#### **@RequestParam("name")** 

在直接写变量进行自动封装的时候，要求**变量名为请求中 key 的名字**。如果不匹配，可以用 **@RequestParam("name")** 进行映射。

在使用 **@RequestParam("name")**  时，要求该注解修饰的参数必须有对应的值，如果请求中不携带则会报错。在此情况下，可以使用 required=false 来使其可以不携带参数，不携带参数时的规则与默认相同，即如果内容为空，包装类型则自动封装为 null，基本类型自动为默认值。

同时，@RequestParam 注解还有 defaultValue 属性，如果没有携带参数则使用 defaultValue。在有defaultValue 的情况下，只有无 defaultValue 且无 required=false 才会报错。

### Pojo 收集请求参数

```java
@RequestMapping("handle03")
public String handle03(Person person) {
    System.out.println(person);
    return "ok!";
}
```

如果目标方法的参数是一个 pojo，SpringMVC 会利用反射自动映射封装。

如果请求参数没带（url 中无键值对），则会封装为 null。如果url 中的值为空，则封装为空字符串。

**没带参数指 url 中无键值对**

pojo 的 defaultValue 可以直接在 pojo 中 `private String username="defaultValue";`

### 获取请求头

```java
@RequestMapping("handle04")
public String handle04(@RequestHeader("host") String host) {
    System.out.println(host);
    return "OK";
}
```

使用 @RequestHeader("key") 注解即可获得请求头中的内容。

### 获取 cookie

```java
@RequestMapping("handle05")
public String handle05(@CookieValue("cookie1") String cookie1) {
    System.out.println(cookie1);
    return "OK";
}
```

使用 @CookieValue("key") 获取 cookie 的值，同一个请求中可以有多个 cookie 被获取。

### pojo 级连封装复杂属性

> http://localhost:8080/handle06?username=zhangsan&password=psw&cellphone=173&
> address.province=hebei&address.city=normal&address.area=uni&sex=男&
> hobby=足球&hobby=篮球&grade=四年级&agreement=on

对于这种复杂属性的封装，Spring 依旧可以完成 pojo 自动注入。pojo 的属性与请求参数类型对应即可。

```java
package com.forty2.training.springmvc.bean;

import lombok.Data;

@Data
public class Person {
    private String username;
    private String password;
    private String cellphone;
    private String agreement;
    private Address address;
    private String[] hobby;
    private String grade;
}

@Data
class Address {
    private String province;
    private String city;
    private String area;
}

```

```java
public String handle06(Person person) {
    System.out.println(person);
    return "ok";
}
```

*output:* Person(username=zhangsan, password=psw, cellphone=173, agreement=on, address=Address(province=hebei, city=normal, area=uni), hobby=[足球, 篮球], grade=四年级)

### 使用 RequestBody 封装 json 对象































































































































































































































































































------



## 引用

遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接和本声明。

[^1]: https://www.bilibili.com/video/BV14WtLeDEit/?p=33&share_source=copy_web&vd_source=732a79db14c78dbec659a1afbe66586e

[^2]: https://docs.spring.io/spring-framework/reference/web/webmvc.html