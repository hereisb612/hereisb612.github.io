# Spring

## 容器

组件：具有一定功能的对象

容器：管理组件（创建、获取、保存、销毁）

> 控制反转是一种思想，而依赖注入是 Spring 实现这种思想的方式

IOC：Inversion of control 控制反转
- 控制：资源的控制权（资源的创建、获取、销毁等）
- 反转：从 Programmer 控制转交给 Spring

DI：Dependency Injection 依赖注入
- 依赖：组件之间的依赖关系，如 News Controller 依赖于 NewsServices
- 注入：通过 setter、构造器等方式自动注入（赋值）



## Basic Spring Environment Creating In Idea

1. create a parent project, and make it packing to pom
2. create new model by using SpringBoot option



## 主程序类、主入口类

```java
/**
 * 主入口类、主程序类：
 * run spring application and return a ConfigurableApplicationContext,
 * which is an interface that extends from ApplicationContext.
 */

@SpringBootApplication
public class Spring01IocApplication {

    public static void main(String[] args) {

        // 1. ApplicationContext: spring 的应用上下文，即 spring 的容器
        ConfigurableApplicationContext ioc = SpringApplication.run(Spring01IocApplication.class, args);
        System.out.println("ioc is: " + ioc);

        // 2. 获取容器中所有组件的名字，可验证容器中有哪些组件
        String[] beanDefinitionNames = ioc.getBeanDefinitionNames();
        for (String name : beanDefinitionNames) {
            System.out.println("name = " + name);
        }
    }
```



## 注册组件的方式

``````java
@SpringBootApplication
public class example {

    public static void main(String[] args) {
        ...
    }
  
    // important! 此处的并不是 Person 的构造方法，只是一个普通的方法名，所以不会造成递归或循环调用的问题
    @Bean
    public Person person() {
        Person p = new Person();
        p.setName("Ethan");
        p.setAge(23);
        p.setGender("Male");
        return p;
    }
}
``````



## 组件的命名规则

- if do not set bean's name mannually, methods' name is bean's name. Like in above code, the bean's name is person in default.
- The way to set bean's name is that key in name in `@Bean(name)`, for example:

``````java
    @Bean("example_name")
    public Person person() {
        Person p = new Person();
        p.setName("Ethan");
        p.setAge(23);
        p.setGender("Male");
        return p;
    }
``````



## 组件特性

- 名字、类型、对象、作用域
- 组件名要求全局唯一
  - 在使用 `@Bean("name")`  方式注册组件时，注解的重名不会被编译器检查到，在代码级别是可以显示为给两个组件注册了同样的名字的
  - 但在实际的 Spring 执行过程中，只有先声明的组件会成功注册进容器
- 容器启动过程中就会创建组件对象
- 组件是**单实例的**



## 获取组件的方式

#### 通过组件的名字获取

``````java
Person example = (Person) ioc.getBean("example");
``````

#### 通过组件的类型获取

```java
Person bean = ioc.getBean(Person.class);
```

#### 通过组件的名字及类型获取

```java
Person person = ioc.getBean("example", Person.class);
```

#### 通过组件类型，获取到该类型的所有组件

```java
Map<String, Person> beansOfType = ioc.getBeansOfType(Person.class);
beansOfType.forEach((k, v) -> {
    System.out.println(v);
});
```



## MVC 分层模型对应注解

- @Configuration for Configuration Class
- @Controller for Controller Class
- @Service for Service Class
- @Repository for DAO Class
- @Component for Other Classes



## Configuration

- 组件：框架的底层配置
  - 配置文件：指定配置的文件
  - **配置类**：分类管理组件的配置，配置类也是容器中的一种组件
- 配置类的实现方式
  1. new config directory
  2. new configClass and use @Configuration to declare it
  3. then move things like components into this ConfigClass to register them

```java
@Configuration
public class PersonConfig {
    @Bean("example")
    public Person person() {
        Person p = new Person();
        p.setName("Ethan");
        p.setAge(23);
        p.setGender("Male");
        return p;
    }
}
```

*ConfigClass is also a component placed in the container.*



## @bean in ConfigurationClass VS @Component

- 使用 `@Configuration` 修饰的类为配置类，其中通常包含多个通过 `@bean` 所注册的方法，称为工厂方法。
  -  `@bean` 方法可以分类放在对应的配置类中，也可以定义在主入口类中

- 配置类的奥义是：Spring 会保证**在多次调用 `@bean` 工厂方法时**，返回的始终是**同一个对象**，不会创建新的对象。以此来贯彻 Spring 的单例哲学。
- 相反，多次调用 `@Component` 修饰的类中的方法是，则会**创建多个对象**。



## @ComponentScan

默认情况下，分层注解能起作用的前提是，这些组件必须在主程序所在的包 及其子包结构下

如果想要扫描其它包下的组件，则需要在主入口类使用 `@ComponentScan(basePackages = "..")` 注解，在主入口类声明组件扫描的范围。一般声明到稍大一级的范围，以开启组件的批量扫描

但更建议按照规范编程，将 Controller 等包规范放在和主程序类平级的位置



## 