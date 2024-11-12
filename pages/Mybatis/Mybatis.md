# Mybatis[^1]

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

### Mybatis VS JDBC

在 jdbc 的情况下，dao 需要准备接口和实现类。但在 Mybatis 中，只需要准备接口和接口的映射文件，省略了实现类的过程。

映射文件需要准备在 resources 目录下，可以通过 **mybatisx** 插件自动完成配置文件的生成。

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

### open Logging

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

### 自增 id 回填

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





































































## 引用

遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接和本声明。

[^1]: https://www.bilibili.com/video/BV14WtLeDEit/?p=33&share_source=copy_web&vd_source=732a79db14c78dbec659a1afbe66586e
