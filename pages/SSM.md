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

- 使用 `@Configuration` 修饰的类是配置类，通常包含通过 `@Bean` 注解注册的方法，称为工厂方法。
  - Spring 会**保证**当配置类中的 `@Bean` 方法被多次调用时，返回的是**同一个对象**，这体现了 Spring 的单例哲学。
  - Spring 会为 `@Configuration` 类生成 CGLIB 代理，**确保** `@Bean` 方法的返回值是**单例**。

- `@Component` 类在**默认**情况下（单例模式）也是**单例**的。Spring 容器在启动时会自动装配这些组件，并在需要时返回**相同的实例**。
  
  - 只有在设置为 `prototype` 作用域时，每次调用 `@Component` 的方法才会创建新的对象。
  
  

## @ComponentScan

默认情况下，分层注解能起作用的前提是，这些组件必须在主程序所在的包 及其子包结构下

如果想要扫描其它包下的组件，则需要在主入口类使用 `@ComponentScan(basePackages = "..")` 注解，在主入口类声明组件扫描的范围。一般声明到稍大一级的范围，以开启组件的批量扫描

但更建议按照规范编程，将 Controller 等包规范放在和主程序类平级的位置



## 第三方组件导入容器

> 因为第三方组件的源码通常无法修改，所以无法通过标记分层注解的方式来导入容器
>
> 故可用：

1. 使用配置类，在配置类中写好工厂方法，手动加入容器中，like:

```java
@Configuration
public class ExampleConfig{
  @Bean
  public A a(){
    return new A();
  }
}
```

2. 在主入口类或容器中任何一个组件上使用 `@Import(Example.class)` 注解，来导入组件，仅需导入一次
3. 更推荐使用一个单独的配置类，如 `AppConfig` 来管理与整个程序相关的注解



## @Scope

>  用于调整组件的作用域，默认是 `@Scope("singleton")` 单实例



1.  @Scope("prototype")，调整组件为 非单实例
2.  @Scope("singleton")，调整组件为 单实例
3.  @Scope("request")，调整组件为 同一个请求单实例
4.  @Scope("session")，调整组件为 同一次会话单实例



容器创建的过程中，就把所有单实例的组件创建完成并加入容器

当组件设置为非单实例（多例）时，容器启动时不会创建非单实例组件的对象，相反，什么时候获取什么时候创建



## @Lazy

> 用于 `@Scope("singleton")` 时，因为单实例时，所有组件是在容器初始化时创建好的，所以对于有懒加载需求的组件，可以使用 `@Lazy` 来设置。



## @FactoryBean<>

> 如果制造某些对象的过程比较复杂，不是用注解可以直接搞定的，则可以使用 FactoryBean，在 getObject() 中写复杂逻辑



- FactoryBean<> is a interface, and which class implements this interface, which class converts to a bean factory automatically.

- the class who implements FactoryBean<> need to be added `@Component` annotation to be putted into container.

- This interface has 3 methods need to override.

  - ```java
    public Car getObject() throws Exception {}
    
    // factory will use this method to create object and put it into container automatically.
    ```

  - ```java
    public Class<?> getObjectType() {}
    
    // use this method to return the type of object for 多态
    ```

  - ```java
    public boolean isSingleton() {}
    
    // make it return true & Singleton by default.
    ```



