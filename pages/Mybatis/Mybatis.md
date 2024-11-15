# Mybatis[^1]

## Mybatis VS JDBC

在 jdbc 的情况下，dao 需要准备接口和实现类。但在 Mybatis 中，只需要准备接口和接口的映射文件，省略了实现类的过程。

映射文件需要准备在 resources 目录下，可以通过 **mybatisx** 插件自动完成配置文件的生成。



## Mybatis Hello World 

1. add basic dependencies from spring boot starter

   - Mybatis framework
   - Mysql Driver
   - Lombok (optional)
   - Spring web (optional)

2. link to dabatase in application.properties

3. prepare pojo and dao

4. write dao interface, and use **@Mapper** annotation to tell Spring that this interface is used by Mabatis to operate database

   ```java
   @Mapper
   public interface EmpMapper {
       Emp getEmpById(Integer id);
   }
   ```

5. use MabatisX to generate counterpart mapper into resources/mapper, like:

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
   <mapper namespace="com.forty2.training.mybatis.helloworld.dao.EmpMapper">
       
   </mapper>
   ```

   namespace is the full name of interface, used to bind xml and interfaces.

6. In application.properties, tell Mybatis the location of xml files.

   ```java
   mybatis.mapper-locations=classpath:mapper/**.xml
   ```

7. open `mybatis.configuration.map-underscore-to-camel-case=true`

### basic Operation

#### select

```xml
<!-- Emp getEmpById(Integer id); -->
<select id="getEmpById" resultType="com.forty2.training.mybatis.helloworld.pojo.Emp">
    select *
    from t_emp
    where id = #{id}
</select>
```

in this case, id indicates the method in the interface, resultType means the type of result, which should be filled by full class name.

```xml
<!--     List<Emp> getAllEmps(); -->
<select id="getAllEmps" resultType="com.forty2.training.mybatis.helloworld.pojo.Emp">
    select * from t_emp
</select>
```

返回值是集合，resultType 依旧写集合中元素的类型 

#### update

```xml
<!-- void updateEmp(Emp emp); -->
<update id="updateEmp">
    update t_emp
    set emp_name   = #{empName},
        age        = #{age},
        emp_salary = #{empSalary}
    where id = #{id}
</update>
```

#### insert

```xml
<!-- void addEmp(Emp emp); -->
<insert id="addEmp">
    insert into t_emp(emp_name, age, emp_salary)
        value (#{empName}, #{age}, #{empSalary})
</insert>
```

#### delete

```xml
<!-- void deleteEmpById(Integer id); -->
<delete id="deleteEmpById">
    delete
    from t_emp
    where id = #{id}
</delete>
```



## open Logging

if wanna see sql statement in terminal, can set in application.properties

```properties
logging.level.com.forty2.training.mybatis.helloworld.mapper = debug
```

**logging.level** used to set different log levels, and after this to **set the target package**.

In this case, log info above debug level from com.forty2.training.mybatis.helloworld.mapper will be shown in the terminal.

``` 
2024-11-12T16:26:43.278+08:00 DEBUG 27702 --- [mybatis-01-helloworld] [           main] c.f.t.m.h.m.EmpMapper.deleteEmpById      : ==>  Preparing: delete from t_emp where id = ?
2024-11-12T16:26:43.289+08:00 DEBUG 27702 --- [mybatis-01-helloworld] [           main] c.f.t.m.h.m.EmpMapper.deleteEmpById      : ==> Parameters: 3(Integer)
2024-11-12T16:26:43.290+08:00 DEBUG 27702 --- [mybatis-01-helloworld] [           main] c.f.t.m.h.m.EmpMapper.deleteEmpById      : <==    Updates: 0
```



## 自增 id 回填

```xml
<!-- void addEmp(Emp emp); -->
<insert id="addEmp" useGeneratedKeys="true" keyProperty="id">
    insert into t_emp(emp_name, age, emp_salary)
        value (#{empName}, #{age}, #{empSalary})
</insert>
```

useGeneratedKeys 声明本次自增的 id 将是数据库自动生成的。

keyProperty 指明取回此自增 id 时装入传入对象的哪个属性。

```java
@Test
public void test2() {
    Emp emp = new Emp();
    emp.setEmpName("xu");
    emp.setAge(33);
    emp.setEmpSalary(8800.00);

    empMapper.addEmp(emp);
    System.out.println(emp.getId());
}
```

output: 5



## 参数传递

### #{}

底层使用 preparedStatement 完成，先生成占位符，后填充。无注入风险。

```
Preparing: select * from t_emp where id = ?
Parameters: 1(Integer)
Total: 1
```

### ${}

底层使用字符串直接拼接成 sql 语句，有注入风险

``````
Preparing: select * from t_emp where id = 1
Parameters: 
Total: 1
``````



### 最佳实践

为每一个接口中的形参使用 @Param 指定别名，此别名将供 xml 中 #{别名} 使用。

```java
getEmployee(@Param("id") int id, @Param("name") String name);
```

xml 中映射为 #{id}，此处 id 取自 @Param 的参数。



## 表名拼接

```xml
<!-- Emp getEmpByIdWithTableName(Integer id, String tableName); -->
<select id="getEmpByIdWithTableName" resultType="com.forty2.training.mybatis.helloworld.pojo.Emp">
    select * from #{tableName} where id = #{id}
</select>
```

此种方法无法将表名使用 preparedStatement 填充，**表名位置需要使用 ${} 拼接**。



## 结果封装

> 返回对象或普通类型：resultType = 全类名
>
> 返回集合：resultType = 集合中元素的全类名

### Basic

List、基本数据类型、pojo 等用**全类名**书写。基本数据类型 Mybatis 提供了简化版。

### Map

**Map 的 resultType 依旧是集合内元素对象的全类名**，key 可以通过 @MapKey("") 指定。

#### 错误

如果如下写，resultType="java.util.Map"，虽然能成功获取到 Emp 对象，但在获取 class 时会发现是 HashMap。

```xml
<!-- @MapKey("id")
     Map<Integer, Emp> getAllEmpsMap(); -->
<select id="getAllEmpsMap" resultType="java.util.Map">
    select * from t_emp
</select>
```

```java
@Test
public void test7() {
    Map<Integer, Emp> allEmpsMap = empMapper.getAllEmpsMap();
    System.out.println(allEmpsMap.get(2).getClass());
}
```

output: java.lang.ClassCastException: class **java.util.HashMap** cannot be cast to class com.forty2.training.mybatis.helloworld.pojo.Emp 

#### 正确

所以正确的写法是，即使返回值是装进 **Map 里**，依旧使用集合内**元素的类型作为 resultType**，并使用 @MapKey 指定 k 值，Mybatis 自动填充 v 值。

```java
<!-- @MapKey("id")
     Map<Integer, Emp> getAllEmpsMap(); -->
<select id="getAllEmpsMap" resultType="com.forty2.training.mybatis.helloworld.pojo.Emp">
    select * from t_emp
</select>
```

```java
@Test
public void test7() {
    Map<Integer, Emp> allEmpsMap = empMapper.getAllEmpsMap();
    System.out.println(allEmpsMap.get(2).getClass());
}
```



## ResultMap

### 场景

Pojo 属性和数据库中的字段无法对应时，封装为 null

解决方法：

1. 在 sql 中使用别名
2. 使用驼峰命名的映射
3. 使用 ResultMap（自定义结果集）

### 使用

定义 resultMap。第一个 id 用于映射，type 为管理的 pojo 的类型。第二个 id 声明主键的映射规则，result 声明数据库中列与 pojo 的属性的映射规则。

而后在 select 标签的 resultMap 处完成 id 映射即可。

```java
<resultMap id="EmpRM" type="com.forty2.training.mybatis.helloworld.pojo.Emp">
    <id column="id" property="id"/>
    <result column="emp_name" property="empName"/>
    <result column="age" property="age"/>
    <result column="emp_salary" property="empSalary"/>
</resultMap>

<select id="getEmpById" resultMap="EmpRM">
    select *
    from t_emp
    where id = #{id}
</select>
```

### 最佳实践

1. 开启驼峰命名
2. 对于驼峰命名搞不定的复杂关系，用 resultMap 来完成自定义映射



## 关联查询

### association 一对一

指定自定义对象的封装规则。一般用于联合查询中**一对一关系**的封装，比如一个订单只能对应一个客户。

每行 new 一个新对象映射好返回。

- javaType 指定关联的 bean 的类型
- select 指定分步查询调用的方法
- 指定分步查询传递的参数列

#### 案例

按照 id 查询订单及下单的客户信息。一对一关系。

1. 向其中一方加入另一方的主键。如，给 order 中加入属性 customerId。

2. 定义 resultMap 的映射规则，将另一个对象的属性和 sql 中返回的某些列映射。其中，用 association 标签嵌套映射主 pojo 中的对象，property 为对应的属性名，javaType 说明类型，供反射使用。反射会调用空参构造器，然后按照子标签一一赋值。

   ```xml
   <resultMap id="OrderRM" type="com.forty2.training.mybatis.helloworld.pojo.Order">
       <id column="id" property="id"/>
       <result column="address" property="address"/>
       <result column="amount" property="amount"/>
       <result column="customer_id" property="customerId"/>
       <association property="customer" javaType="com.forty2.training.mybatis.helloworld.pojo.Customer">
           <id column="c_id" property="id"/>
           <result column="customer_name" property="customerName"/>
           <result column="phone" property="phone"/>
       </association>
   </resultMap>
   ```

### Collection 一对多

指定自定义对象的封装规则。一般用于联合查询中**一对多关系**的封装，比如一个用户对应着多个订单。

数据库的查询结果可能有很多行，但前面的 customer 信息都是相同的，区别只是后面不同的 order。所以 Mybatis 在查到第二行时，就不会再创建新的 customer 对象装结果，而是在发现 collection 标签之后，自动把接下来每行里的 order 信息 new 新对象封装好，依次放进 List orders 里。这就是 collection 标签的意义。

- ofType 指定集合中 bean 的类型

#### 案例

按照 id 查询客服以及客户下单的所有信息。一对多关系，一个客户可能有很多订单。

1. 向客户中添加一个 List 用于存放所有 Order。

2. 定义 resultMap，使用 Collection。

   ```xml
   <resultMap id="CusRM" type="com.forty2.training.mybatis.helloworld.pojo.Customer">
       <id column="c_id" property="id"/>
       <result column="customer_name" property="customerName"/>
       <result column="phone" property="phone"/>
       <collection property="orders" ofType="com.forty2.training.mybatis.helloworld.pojo.Order">
           <id column="id" property="id"/>
           <result column="address" property="address"/>
           <result column="amount" property="amount"/>
           <result column="customer_id" property="customerId"/>
       </collection>
   </resultMap>
   
   <select id="getCustomerByIdWithOrders" resultMap="CusRM">
       select c.id c_id,
              c.customer_name,
              c.phone,
              o.*
       from t_customer c
                left join t_order o on c.id = o.customer_id
       where c.id = #{id}
   </select>
   ```

### 分步查询

在 association 和 collection 封装过程中，可以使用 select + column 指定分步查询逻辑

- select 指定分步查询调用的方法
- column 指定分步查询传递的参数
  - 传递单个：直接写列名，表示将这列的值作为参数传递给下一个查询
  - 传递多个：column="{prop1=col1, prop2=col2}"，下一个查询使用 prop1、prop2 取值

#### 案例

按照 id 查询客户 以及 他下的所有订单

##### 原生写法

1. 分别查询客户信息、该客户 id 下的订单信息

   ```java
   @Mapper
   public interface OrderCustomerStepMapper {
       Customer getCustomerById(Long id);
   
       List<Order> getOrdersByCustomerId(Long customerId);
   }
   ```

   ```xml
   <select id="getCustomerById" resultType="com.forty2.training.mybatis.helloworld.pojo.Customer">
       select *
       from t_customer
       where id = #{id}
   </select>
   
   <select id="getOrdersByCustomerId" resultType="com.forty2.training.mybatis.helloworld.pojo.Order">
       select *
       from t_order
       where customer_id = #{customerId}
   </select>
   ```

2. 获得客户信息、该客户的所有订单，在 test 或 service 中手动封装

   ```java
   @Test
   public void test1() {
       Customer customer = orderCustomerStepMapper.getCustomerById(1L);
       List<Order> orders = orderCustomerStepMapper.getOrdersByCustomerId(customer.getId());
       customer.setOrders(orders);
       System.out.println(customer);
   }
   ```

此处可见，是我们手动调用两次方法完成的。Mybatis 支持利用自动分布查询机制，将手动调用变为自动调用。

##### 自动分步查询

```java
Customer getCustomerByIdAndOrderStep(Long id);
```

```xml
  <resultMap id="CustomerOrderStepRM" type="com.forty2.training.mybatis.helloworld.pojo.Customer">
      <id column="id" property="id"/>
      <result column="customer_name" property="customerName"/>
      <result column="phone" property="phone"/>
      <collection property="orders"
                  ofType="com.forty2.training.mybatis.helloworld.pojo.Order" select="com.forty2.training.mybatis.helloworld.mapper.OrderCustomerStepMapper.getOrdersByCustomerId" column="id"
      >
        
      </collection>
  </resultMap>

  <select id="getCustomerByIdAndOrderStep" resultMap="CustomerOrderStepRM">
      select *
      from t_customer
      where id = #{id}
  </select>
```

因为 Customer 中有 List orders，所以直接 select * 即可，自动分步查询可将第二步查回的 order 数组封装给 Customer 中的 List。

当 collection 有 select 参数时，means 封装 orders 属性时，该属性应当是一个集合。但这个集合需要用另一个 sql 方法来查询，另一个 sql 的返回值作为该集合的值。

collection 下的参数分别为：

1. property 用于对应 Customer 的属性；
2. ofType 指明集合的元素类型；
3. select 指明第二步自动查询时调用哪个方法。在方法执行完成时，会自动把结果封装进 collection 对应的 property 中。
4. column 用于传参，其内容的含义是 将第一次查询结果中的哪一列 作为参数 传递给第二次查询。





































## 引用

遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接和本声明。

[^1]: https://www.bilibili.com/video/BV14WtLeDEit/?p=33&share_source=copy_web&vd_source=732a79db14c78dbec659a1afbe66586e
