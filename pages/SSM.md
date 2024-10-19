# Spring

## 容器

### 容器 - 组件注册

组件：具有一定功能的对象

容器：管理组件（创建、获取、保存、销毁）

> 控制反转是一种思想，而依赖注入是 Spring 实现这种思想的方式

IOC：Inversion of control 控制反转

- 控制：资源的控制权（资源的创建、获取、销毁等）
- 反转：从 Programmer 控制转交给 Spring

DI：Dependency Injection 依赖注入

- 依赖：组件之间的依赖关系，如 News Controller 依赖于 NewsServices
- 注入：通过 setter、构造器等方式自动注入（赋值）



#### Basic Spring Environment Creating In Idea

1. create a parent project, and make it packing to pom
2. create new model by using SpringBoot option



#### 主程序类、主入口类

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



#### 注册组件的方式

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



#### 组件的命名规则

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



#### 组件特性

- 名字、类型、对象、作用域
- 组件名要求全局唯一
  - 在使用 `@Bean("name")`  方式注册组件时，注解的重名不会被编译器检查到，在代码级别是可以显示为给两个组件注册了同样的名字的
  - 但在实际的 Spring 执行过程中，只有先声明的组件会成功注册进容器
- 容器启动过程中就会创建组件对象
- 组件是**单实例的**



#### 获取组件的方式

##### 通过组件的名字获取

``````java
Person example = (Person) ioc.getBean("example");
``````

##### 通过组件的类型获取

```java
Person bean = ioc.getBean(Person.class);
```

##### 通过组件的名字及类型获取

```java
Person person = ioc.getBean("example", Person.class);
```

##### 通过组件类型，获取到该类型的所有组件

```java
Map<String, Person> beansOfType = ioc.getBeansOfType(Person.class);
beansOfType.forEach((k, v) -> {
    System.out.println(v);
});
```



#### MVC 分层模型对应注解

- @Configuration for Configuration Class
- @Controller for Controller Class
- @Service for Service Class
- @Repository for DAO Class
- @Component for Other Classes



#### Configuration

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



#### @bean in ConfigurationClass VS @Component

- 使用 `@Configuration` 修饰的类是配置类，通常包含通过 `@Bean` 注解注册的方法，称为工厂方法。

  - Spring 会**保证**当配置类中的 `@Bean` 方法被多次调用时，返回的是**同一个对象**，这体现了 Spring 的单例哲学。
  - Spring 会为 `@Configuration` 类生成 CGLIB 代理，**确保** `@Bean` 方法的返回值是**单例**。

- `@Component` 类在**默认**情况下（单例模式）也是**单例**的。Spring 容器在启动时会自动装配这些组件，并在需要时返回**相同的实例**。

  - 只有在设置为 `prototype` 作用域时，每次调用 `@Component` 的方法才会创建新的对象。

  

#### @ComponentScan

默认情况下，分层注解能起作用的前提是，这些组件必须在主程序所在的包 及其子包结构下

如果想要扫描其它包下的组件，则需要在主入口类使用 `@ComponentScan(basePackages = "..")` 注解，在主入口类声明组件扫描的范围。一般声明到稍大一级的范围，以开启组件的批量扫描

但更建议按照规范编程，将 Controller 等包规范放在和主程序类平级的位置



#### 第三方组件导入容器

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



#### @Scope

>  用于调整组件的作用域，默认是 `@Scope("singleton")` 单实例



1.  @Scope("prototype")，调整组件为 非单实例
2.  @Scope("singleton")，调整组件为 单实例
3.  @Scope("request")，调整组件为 同一个请求单实例
4.  @Scope("session")，调整组件为 同一次会话单实例



容器创建的过程中，就把所有单实例的组件创建完成并加入容器

当组件设置为非单实例（多例）时，容器启动时不会创建非单实例组件的对象，相反，什么时候获取什么时候创建



#### @Lazy

> 用于 `@Scope("singleton")` 时，因为单实例时，所有组件是在容器初始化时创建好的，所以对于有懒加载需求的组件，可以使用 `@Lazy` 来设置。



#### @FactoryBean<>

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



#### @Conditional

> 此注解可以令组件在满足某些条件时才被创建并加载进容器

1. condition 是个接口，应当有一个类实现 Condition 接口，并在该接口的方法内进行条件的判断
2. Conditional 内的参数为实现了接口的 `ExampleCondition.class`
3. `ioc.getEnvironment()` 可以获得有关于环境变量的集合
4. 环境变量可以在 Run/Debug Configuration 中的 Environment variables 中手动设定，如 `os=mac`
5. 如没有环境变量的设置栏，可以 Run/Debug Configuration - Build And Run - Modify Options 中添加
6. `@Conditional` can be used for both Class and Method. If both Class and Method have `@Conditional`, the annotation on class is priorer than methods.



#### Conditional 派生注解

有一系列派生注解可以使用

使用派生注解可以免去手写 ExampleCondition 类并手动实现 Condition-matches 方法的过程



---



### 容器 - 注入

#### @Autowired

> 自动装配。
>
> 原理是 Spring 调用容器的 getBean() 获得符合条件的组件，自动装配。



自动装配流程：

1. 按照类型找
   1. 有且只有一个组件，则注入，不考虑组件名
   2. 找到多个组件，则按照组件名继续匹配
      1. 如果找到同名组件，直接注入
      2. 如果找不到同名组件，则报错



@Autowired 的适用方式

```java
@Autowired
Person bill; // 注入一个

@Autowired
List<Person> people; // 注入查询到的多个

@Autowired
Map<String, Person> peopleMap; // 使用 Map，注入查询到的多个。Key 为组件名，Value 为组件

@Autowired
ApplicationContext ioc; // 注入 ioc 容器本体
```



#### @Qualify & @Primary

> **Error**: Field person in UserService required a single bean, but 3 were found.
>
>  **Solution**: Consider marking one of the beans as `@Primary`, updating the consumer to accept multiple beans, or using `@Qualifier ` to identify the bean that should be consumed



当使用 `@Autowired` 查询到多个组件导致无法自动注入时，可以使用 `@Qualify(Component's name)` 精确指定需要注入的组件

亦可以在组件上使用 `@Primary` 标记为默认组件。当使用 `@Autowired` 查询到多个组件导致无法自动注入时，若这些组件中某一个被标注为 `@Primary` 则自动注入该组件。

**important**：同类型组件有多个，且使用 `@Primary` 标记了一个默认组件后，自动注入的将一直是该默认组件，**不再根据组件名进行匹配**。若需要注入非默认组件，可以使用 `@Qualify` 进行声明。



#### @Resource

> import jakarta.annotation.Resource;
>
> jakarta 前身是 JavaX，是 JavaEE 的标准组织，具有更强的通用性



@Resource 也能实现自动注入，其与 @Autowired 的最大区别是，一个是 Java 标准下的，一个是 Spring 下的。这意味着所有具有容器功能的接口都支持 @Resource，然而 @Autowired 只有 Spring 下的框架支持。



@Autowired(required = false) 允许组件不存在时，令需要注入的属性为空，但 @Resource 没有类似的操作。



#### 构造器注入

```java
@Repository
@Data
@ToString
public class UserDao {

    Person person;

    @Autowired
    public UserDao(Person person){
        this.person = person;
    }
}
```

可以使用 @Autowired 对 **构造器的形参** 进行自动注入。

**Very Recommend**。因为使用构造器注入，即使脱离了 Spring 环境手动 new 对象，依旧可以在 new 的时候手动为形参传参。而使用 @Autowired 在属性上时，则无法在手动创建对象时为属性赋值。



#### Setter 方法注入

```java
  @Autowired
  public void setPerson(Person person) {
      this.person = person;
  }
```

可以使用 @Autowired 对 **setter方法的形参** 进行自动注入。



在 JavaBean 规范中，属性通常由 `getX()` 和 `setX()` 方法定义，而不一定与类中的某个具体变量名直接关联。Spring 在创建对象的过程中，如果发现某类型有对应的 `get/set` 方法，则会将该方法对应的变量视为属性，并自动调用 `set` 方法进行依赖注入。



```java
@Repository
@Data
@ToString
public class UserDao {

    Person dog;

    @Autowired
    public void setPerson(Person person) {
        this.dog = person;
    }
}
```

此代码依旧能完成 Person dog 属性的注入，因为 @Autowired 实际注入的是形参位置的 Person.class，再通过 Person.class 朝 this.dog 赋值。所以 person 和 dog 即使不重名也能完成注入。



#### xxxAware 感知接口

> Aware 接口下有一系列子接口，用于一系列内容。可以通过这些感知接口自带的 set 方法获取内容给属性。
>
> 感知接口自带的 set 方法无需 @Autowired，Spring 就会自动注入这些方法的形参，将想要感知的内容传送来。

```java
@Service
public class xxAwareService implements EnvironmentAware{
    Environment environment;

    @Override
    public void setEnvironment(Environment environment) {
        this.environment = environment;
    }

    public String typeOfEnvironment(){
        return environment.getProperty("spring.application.name");
    }
}
```



#### @Value

@Autowired 是用来自动注入**组件**的，但一些基本数据类型，如 `String name` 是无法通过 @Autowired 完成注入的。

如果想给基本数据类型完成注入，则需要使用 @Value，如：

1. 直接用字面量给属性赋值

   ```java
   @Value("name")
   private String name;
   ```

2. 从配置文件中动态取出某一项的值

   ```java
   @Value("${dog.age}")
   private int age;
   ```

   配置文件（如 application.properties）

   ```properties
   dog.age=7
   ```

3. 在 `@Value("#{SpEL}")` 中写 SpEL (Spring Expression Language) Spring 表达式语言

   ```java
   @Value("#{10*${dog.age}}")
   private String color;
   ```

   赋 uuid 的场景：

   ```java
   @Value("#{T(java.util.UUID).randomUUID().toString()}") 
   private String uuid;
   ```

   此处的 T(type) 是 SpEL 的语法，用来表示 Java.lang.Class 类的实例，可以视作 Java 中直接写类名。

   只有 Java.lang 下的类可以省略包名。

   此写法一般用于引用常量及静态方法（因其可直接被类名调用）[^1]































## 引用

遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接和本声明。

[^1]: https://blog.csdn.net/ZBZBZB12138/article/details/122597702
