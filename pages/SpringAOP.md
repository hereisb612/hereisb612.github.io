# Spring AOP[^1]

## 计算器的日志场景

- 设计：编写一个计算器接口及其实现类，实现基础的四则运算功能
- 需求：记录相关日志，如运算前的参数及运算后的结果
- 实现方式:
  - 静态代理
  - 动态代理
  - **AOP**

### Basic code

> 在此实现中如果想要添加日志功能，只能使用硬编码方式，即将日志内容在核心业务逻辑中 sout。
>
> 硬编码方式没有区分 通用逻辑 与 专用逻辑，耦合度高，维护难度极高。

#### interface

```java
public interface MathCalculator {
    int add(int a, int b);
    int subtract(int a, int b);
    int multiply(int a, int b);
    int divide(int a, int b);
}
```

#### implements

```java
@Component
public class CalculatorImpl implements MathCalculator {
    @Override
    public int add(int a, int b) {
        return a + b;
    }

    @Override
    public int subtract(int a, int b) {
        return a - b;
    }

    @Override
    public int multiply(int a, int b) {
        return a * b;
    }

    @Override
    public int divide(int a, int b) {
        return a / b;
    }
}
```

#### 测试类

```java
@SpringBootTest
public class CalculatorTest {
    @Autowired
    MathCalculator mathCalculator; // 多态

    @Test
    public void test01() {
        System.out.println(mathCalculator.add(1, 2));
    }
}
```

应使用多态。

在设计模式中有依赖倒置原则，which means 应当依赖接口而不是实现类。以这种设计方式设计的程序能保证实现类的随意切换。因为在实际程序中实现类是经常变化的，而接口大部分时间是定死的。



### 静态代理

> 代理的对象是**目标对象接口的子类型**。
>
> 代理对象本身并不是目标对象，而是将目标对象作为自己的属性，调用目标对象的方法来完成核心功能的执行。
>
> **在定义期间就指定了互相代理关系**，所以称为静态代理。
>
> 在编码时介入。

#### 优缺点

##### 优点：

同一种接口的所有对象都能代理。

##### 缺点：

代理对象需要与目标对象实现相同的接口，当目标对象（接口）增加方法时，代理对象也要跟着维护，所以好用的范围很小，只能负责部分接口的代理。

例如，静态代理在 DAO 的场景里就很难用。每当 DAO 接口增加一个方法时，就需要维护该方法对应的静态代理对象，非常繁琐。

#### 静态代理类的实现

定义一个代理对象用于包装该组件，以后业务的执行从代理开始，由代理完成日志等功能后再行调用组件。

```java
@Data
public class CalculatorStaticProxy implements MathCalculator { // implements same interface

    private MathCalculator target; // 多态，目标对象。在定义期间就决定好了代理的关系。

    public CalculatorStaticProxy(MathCalculator mathCalculator) {
        this.target = mathCalculator;
    }

    @Override
    public int add(int a, int b) {
        System.out.println("【日志】add 开始，传入参数为：" + a + " " + b);
        int result = target.add(a, b);
        System.out.print("【日志】： add 结束，值为：");
        return result;
    }
}
```

1. implements 和核心实现类相同的接口，以保证代理能覆盖所有定义好的功能。
2. 使用该接口**多态**的接受实现类。以后任意对于 MathCalculator 接口的实现，均可以使用该静态代理来代理，以实现日志打印功能。
3. 利用构造器获得一个实现类
4. 在静态代理类实现的方法中添加日志功能，核心的加减乘除代码通过调用实现类内的代码来实现

#### 测试

```java
public class MathTest {
    @Test
    public void test1() {
        System.out.println(new CalculatorStaticProxy(new CalculatorImpl()).add(1, 2)); // 应用多态
    }
}
```

```java
【日志】add 开始，传入参数为：1 2
【日志】： add 结束，值为：3

Process finished with exit code 0
```



### 动态代理 - Java 原生

> 目标对象在执行期间会被动态拦截，插入指定逻辑。与静态代理的区别是，运行期间才会确定代理关系，所以其能代理世间万物。

```java
@Test
public void test2() {
    CalculatorImpl target = new CalculatorImpl();

    MathCalculator proxyInstance = (MathCalculator) Proxy.newProxyInstance(
            target.getClass().getClassLoader(),
            target.getClass().getInterfaces(),
            (proxy, method, args) -> {
                System.out.println("before..");
                Object result = method.invoke(target, args);// 目标方法执行
                return result;
            });

    System.out.println(proxyInstance.add(1, 2));
}
}
```

Proxy.newProxyInstance() 是 Java 原生支持的动态代理对象。其有三个属性：

#### ClassLoader loader

是目标对象的类加载器，该类加载器是反射机制要求的

#### Class<?>[] interfaces

是目标对象所 implements 的接口，代理对象也将实现这些接口。

正因为如此 `MathCalculator proxyInstance = (MathCalculator) Proxy.newProxyInstance()`  才可以将代理对象基于多态放心的强制类型转换成目标对象的接口。而当强转完成时，就可以在代理对象身上调用接口中所定义的所有方法了。基于这样的操作，就实现了使用代理对象来执行目标对象功能的需求。

#### InvocationHandler h

函数式接口，可用 lambda 表达式。可以在目标方法真正执行前进行拦截。

其中有一个没有方法体的函数 `public Object invoke(Object proxy, Method method, Object[] args) throws Throwable;` 需要被重写，而该方法体内定义的操作则是代理对象被调用时需要完成的所有操作。

proxy: 代理对象本身，此处是 `proxyInstance`

method: 目标对象需要被调用的方法。实际上就是 `proxyInstance.add(1, 2);` 中在代理对象身上调用的方法 `add`

args: method 所需的参数

#### InvocationHandler 详解

```java
new InvocationHandler(){
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("before..");
        int result = method.invoke(target, args);
      
     
        return result;
    }
}
```

重写的方法体中可以写需要代理对象执行的逻辑。

其核心部分是 ` int result = method.invoke(target, args); return result;` 

method 是需要被执行的**方法**的反射对象，invoke() 是 reflect 包里的方法，target 是通过反射执行方法时实际拥有此方法的**对象**，args 是方法所需要的**参数**。至此，对象、方法、参数、调用动作齐全，与基本的反射无异。

而后的 return 是把目标对象的返回值**返回给代理对象**，获得到目标方法的返回值，代理对象才可以完成返回值的中间传递。

#### 动态代理的工具化抽取

```java
public class DynamicProxy {
    public static Object getDynamicProxy(Object target) {
        return Proxy.newProxyInstance(
                target.getClass().getClassLoader(),
                target.getClass().getInterfaces(),
                ((proxy, method, args) -> {
                    System.out.println("-- dynamicProxy, Before --");
                    return method.invoke(target, args);
                }));
    }
}
```

抽取成静态方法，传入任意类型的目标对象，return 任意类型的返回值，即可完成万能的动态代理对象。

如有日志需求，可以对 method、args 等对象取值来优化输出完成日志功能。

#### 日志工具类

进一步抽取出日志内容为一个工具类

```java
public class LogUtils {
    public static void logStart(String name, Object... args) {
        System.out.println("【日志】：【" + name + "】开始；参数：" + Arrays.toString(args));
    }

    public static void logEnd(String name) {
        System.out.println("【日志】：【" + name + "】结束");
    }

    public static void logException(String name, Throwable e) {
        System.out.println("【日志】：【" + name + "】异常；异常信息：" + e);
    }

    public static void logReturn(String name,Object result) {
        System.out.println("【日志】：【" + name + "】返回；返回值：" + result);
    }
}
```

优化动态代理的日志功能，提供开始、结束、异常、返回的日志通知

```java
public class DynamicProxy {
    public static Object getDynamicProxy(Object target) {
        return Proxy.newProxyInstance(
                target.getClass().getClassLoader(),
                target.getClass().getInterfaces(),
                (proxy, method, args) -> {
                    Object result = null;
                    try {
                        LogUtils.logStart(method.getName(), args);
                        result = method.invoke(target, args);
                    } catch (Exception e) {
                        LogUtils.logException(method.getName(), e);
                    } finally {
                        LogUtils.logEnd(method.getName());
                    }
                    LogUtils.logReturn(method.getName(), result);
                    return result;
                });
    }
}
```

#### 缺点

JDK 自带的动态代理要求目标对象**必须实现接口**，因为需要转换成对应的接口类型来获得可供调用的方法列表进行方法的调用，其也只能代理接口规定的方法。这意味着如果接口中没有目标对象的某个方法，则无法通过动态代理完成该方法的代理



## What is AOP

AOP: Aspect Oriented Programming 面向切面编程

Spring 将自动为目标对象产生一个代理对象，此代理对象的作用就是自动侦测代码执行到的时机，把切面类中定义的通知方法织入代码逻辑中。



![AOPwords](../imgs/AOPwords.svg)



### 专业术语

**时机**：每个方法都有四个时机，分别是方法开始、方法返回、方法异常、方法结束。正常的方法执行都是从头到尾一串下来的。

**横切关注点**：由于所有方法都有这四个时机，所以将不同方法的相同时机横向来看，即为横切关注点。

**通知方法**：每个横切关注点所在的时机到了后需要执行的方法称为通知方法。按照时机的不同可以分为 *前置通知*，*返回通知*，*异常通知*，*结束通知*

**切面类**：各通知方法所在的类

**连接点**：时机和横切关注点之间线段的交点称为连接点，每一个连接点实际上意味着一个通知可以发生的位置。

**切入点**：可以挑选**感兴趣**的连接点来设置其对应的通知方法执行，如图蓝色示。被选中的、希望在此连接点**将通知方法插入执行的位置**称为切入点。也可以称切入点为 真正会执行通知方法的点。

**织入**：代理对象将从切面类中拿到对应的通知方法，动态的发现目标对象执行到哪一个连接点，如果该连接点是感兴趣的连接点（切点），就将相应的通知方法动态的织入目标对象的代码逻辑中。

**切入点表达式**：选出自己感兴趣的切入点的方式



## 计算器案例的 AOP 实现

1. 引入 aop 的 dependency。copy starter 后面加 aop 即可。
   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-aop</artifactId>
   </dependency>
   ```

2. 编写切面 Aspect 及通知方法

   ```java
   @Aspect
   @Component
   public class LogAspect {
       public void logStart(){
           System.out.println("LogAspect.logStart");
       }
   }
   ```

3. 指定切入点表达式，告诉 Spring 以下通知方法**何时何地**运行

   1. 何时？

      1. @Before: 方法执行前运行
      2. @After: 方法执行后运行
      3. @AfterReturning: 方法执行 return 后运行
      4. @AfterThrowing: 方法抛出异常后运行

   2. 何地？切入点表达式约束。

      1. execution(方法的全签名)

         全签名：[public] **int** [com.forty2.training.spring.aop.calculator.MathCalculator].**add(int [i], int [j])** [throws Exception]

         省略后：int add(int, int)

   ```java
   @Aspect
   @Component
   public class LogAspect {
       @Before("execution(int *(int, int))")		// 全体方法有效
       public void logStart(){
           System.out.println("LogAspect.logStart");
       }
   
       @After("execution(int add(int, int))")		// 指定方法有效
       public void logEnd(){
           System.out.println("LogAspect.logEnd");
       }
   }
   ```

4. 测试



## 表达式通配符

1. \* 标识任意字符，可以放在 返回值、方法名、包的全类名路径中

   eg: * *(int)	* *(\*, int)

2. .. 放在参数位置表示任意类型的多个参数，放在包的全类名中表示多个层级

   eg: * *(..)	  * *(.., int)

3. 常用写法

   某接口的所有方法 `int com.forty2.training.spring.aop.calculator.MathCalculator.*(..)`

   对类型约束的要精确，如果一不小心切入到底层的方法，可能项目启动会失败



## 切入点表达式[^2]

Spring AOP supports the following AspectJ pointcut designators (PCD) for use in pointcut expressions:

- **`execution`: For matching method execution join points. This is the primary pointcut designator to use when working with Spring AOP.**

  ```java
  @Before("execution(int com.forty2.training.spring.aop.calculator.MathCalculator.*(..))")
  public void logStart() {
      System.out.println("LogAspect.logStart");
  }
  ```

- `within`: Limits matching to join points within certain types (the execution of a method declared within a matching type when using Spring AOP).

- `this`: Limits matching to join points (the execution of methods when using Spring AOP) where the bean reference (Spring AOP proxy) is an instance of the given type.

- `target`: Limits matching to join points (the execution of methods when using Spring AOP) where the target object (application object being proxied) is an instance of the given type.

- `args`: Limits matching to join points (the execution of methods when using Spring AOP) where the arguments are instances of the given types.

- `@target`: Limits matching to join points (the execution of methods when using Spring AOP) where the class of the executing object has an annotation of the given type.

- `@args`: Limits matching to join points (the execution of methods when using Spring AOP) where the runtime type of the actual arguments passed have annotations of the given types.

- `@within`: Limits matching to join points within types that have the given annotation (the execution of methods declared in types with the given annotation when using Spring AOP).

- **`@annotation`: Limits matching to join points where the subject of the join point (the method being run in Spring AOP) has the given annotation.**

  ```java
  @Before("@annotation(com.forty2.training.spring.aop.annotation.MyAnnotation)")
  public void forAtAnnotationTest() {
      System.out.println("LogAspect.forAtAnnotationTest");
  }
  ```

  经一通测试，似乎 annotation 要标记在实现类上？

  

## 代理对象 Spring VS JDK

获取实现类的 class 对象

```java
@SpringBootTest
public class AOPTest {
    @Autowired
    MathCalculator mathCalculator;

    @Test
    public void test1() {
        System.out.println(mathCalculator.getClass());
    }
}
```

### 开启 AOP 

在切面类通过 @Component 放入容器中时，MathCalculator 的实现类为

class com.forty2.training.spring.aop.calculator.impl.**CalculatorImpl$$SpringCGLIB$$0**

### 关闭 AOP

在切面类没有通过 @Component 放入容器中时，MathCalculator 的实现类为

class com.forty2.training.spring.aop.calculator.impl.**CalculatorImpl**

### 原因

在有切面存在时，容器中放入的**被切入的组件（目标对象）**将是由 Spring 底层的 **SpringCGLIB** 产生的代理对象。

JDK 原生的 Proxy 动态代理的要求是目标对象必须实现接口，因为要将动态代理对象生成为接口类型才能多态的装回，并通过接口获得到目标对象所拥有的方法，顺利完成调用。而 SpringCGLIB 则没有必须实现接口的要求，可以为万物产生代理。

### 结论

以后只要发现某个组件被切面切入了，那么容器中的该组件就不再是原生的该组件了，而是被 Spring 动态代理过的代理对象。



## 增强器链

### what is

切面当中的所有通知方法被称作增强器，这些增强器被组织成一个链路放到集合中。

目标方法真正执行前后会去增强器链中执行那些需要被执行的方法。

### AOP 原理

Spring 会为每个被切面切入的组件通过 Spring CGLIB 创建代理对象，该代理对象中保存了切面类里面所有通知方法构成的增强器链，目标方法执行时会先去增强器链中拿到需要提前执行的通知方法去执行，以此实现 AOP

### 通知方法的执行顺序

正常链路：前置通知 -> 目标方法 -> 返回通知 -> 后置通知

异常链路：前置通知 -> 目标方法 -> 异常通知 -> 后置通知



## AOP 细节

### JoinPoint 连接点信息

可以通过参数列表传入 `JoinPoint`，本连接点里包装了当前目标方法的所有信息

```java
@After("execution(* *(int, int))")
public void logEnd(JoinPoint joinPoint) {
    MethodSignature signature = (MethodSignature) joinPoint.getSignature();
    String name = signature.getName();
    System.out.println(name);

    Object[] args = joinPoint.getArgs();
    System.out.println(Arrays.toString(args));
}
```

getSignature() 可以获得 Signature 类型的方法全签名，强转成 MethodSignature caz MethodSignature extends from Signature, have more effective methods could be used.

can also use getArgs() on JoinPoint to get all of args of target methods.



### 获得目标方法返回值

```java
@AfterReturning(value = "execution(* *(int, int))",
returning = "result")
public void logReturn(Object result) {
    System.out.println("result: " + result);
}
```

using returning to **bind** formal variable with the returning value of target method.



### 获得目标方法异常信息

```java
@AfterThrowing(value = "execution(* *(int, int))",
        throwing = "e")
public void logException(Exception e) {
    System.out.println(e.getMessage() + "eee");
}
```

using throwing to bind Exception Object.



### @PointCut 抽取切入点表达式

可以随意定义一个方法，将该方法用 @Pointcut(execution()) 修饰。

此时，该方法就可以作为一个引用，完成对切入点表达式的抽取，方便统一管理。

```java
@Aspect
@Component
public class LogAspect {
    
    @Pointcut("execution(int com.forty2.training.spring.aop.calculator.MathCalculator.*(..))")
    public void pointCut() {}
    
    @Before("pointCut()")
    public void logStart() {
        System.out.println("LogAspect.logStart");
    }
}
```



### 多切面的执行顺序

#### 切面执行原理

![AspectOrder](../imgs/AspectOrder.svg)



#### @Order demo

```java
@Aspect
@Component
@Order(1)
public class LogAspect {

    @Pointcut("execution(int com.forty2.training.spring.aop.calculator.MathCalculator.*(..))")
    public void pointCut() {}

    @Before("pointCut()")
    public void logStart() {
        System.out.println("LogAspect.logStart");
    }

    @After("pointCut()")
    public void logEnd() {
        System.out.println("LogAspect.logEnd");
    }
}
```













































































































------



## 引用

遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接和本声明。

[^1]: https://www.bilibili.com/video/BV14WtLeDEit/?p=33&share_source=copy_web&vd_source=732a79db14c78dbec659a1afbe66586e

[^2]: https://docs.spring.io/spring-framework/reference/core/aop/ataspectj/pointcuts.html
