# MyBatis

## dependencies

1. mysql-connector-j
2. mybatis
3. junit (optional)
4. lombok (optional)

## configurations

Can be finded from [MyBatis Documentation](https://mybatis.org/mybatis-3/getting-started.html)

like:

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>
    
    <properties resource="db.properties"/>
    
    <typeAliases>
        <package name="com.forty2.mybatis.pojo"/>
    </typeAliases>

    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${driver}"/>
                <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
            </dataSource>
        </environment>
    </environments>

    <mappers>
        <package name="com.forty2.mybatis.mapper" /> 
    </mappers>
</configuration>
```

** resources 文件夹下只能建立 directory，which means that need to use com\forty2\pojo instead of com.forty2.pojo. this 2 kind of name is different in Finder. the former one is tree-directory and the later one is only name.**

By the way, ${driver} like variables, can use

 `<properties > resource="db.properties"/>` 

 to include the properties from the files-name.properties.

 The content of .properties files seems like:

 ```
driver=com.mysql.cj.jdbc.Driver
url=jdbc:mysql://localhost:3306/mybatis
username=root
password=******
```

**need to regist the mapping files in the config file by using mapper label**

## Mapper interface

In basic jdbc, we need to create DAO interface and the implements of such interface to access to database, but in MyBatis, the mapper interface can be seem as the combaination of those two class because MyBatis supports to **Interface oriented programming**, so we dont need to create the object of interface, and thats the reason why we dont need to write class to achive interface. It is more easier to code.

## Files structure

- java
    - pojo
        - User (Java Bean)
    - mapper
        - UserMapper (Interface)
- test

## Mapping File

1. make sure the file's name is same as table. For example, t_user vs UserMapper.

2. the file's name use XMapper.xml

3. each mapping file corresponds to its table.

4. putting mapping file in resources directory.

5. the content of mapping file's example can be found in [documentation](https://mybatis.org/mybatis-3/getting-started.html). 

6. the namespace in xml must same as the full path of the correspond interface

7. the id in xml must same as the method in the interface.

## 2 ways to get values from variable in MyBatis (!Important!)

1. `${}` <- **字符串拼接** 存在 sql 注入，单引号需要手动拼接，like `where username = '${username}'`

2. `#{}` <- **占位符赋值** recommend

3. if have more than 2 formal variable, MyBatis will put those formal variable into a map automatically, which means need to use default key like `#{arg0} #{arg1}` or `#{param1} #{param2}` to visit values. **(but can use @Param to change default key name from arg0 to others.)**

*You can also create map by yourself to set keys' name.* Like:

```
map = new HashMap();
map.put("username", root);
map.put("password", root);
mapper.select(map);

select * from table where username = #{username}
```

4. if inputing a pojo like `User user`, can also use `#{}` to access object's values.

5. Can return `Map`, `List<Map>` and so on..

6. 模糊查询无法使用 #{}，应当使用 ${}

7. if have more than one rows are returned, can use @Mapkey() to set the key.

## Mutil-table

1. if a field's name differs in pojo & database, can use `as` to alias.

2. in mabatis-config.xml, can use `<settings> to set <mapUnderscoretoCamelCase>`

## resultMap

```
    <resultMap id="empResultMap" type="Emp">
        <id property="eid" column="eid"/>
        <result property="empName" column="emp_name"/>
        <result property="age" column="age"/>
        <result property="sex" column="sex"/>
        <result property="email" column="email"/>
        <result property="did" column="did"/>
    </resultMap>

    <select id="findAll" resultMap="empResultMap">
        select * from t_emp
    </select>
```

`<id>` : for primary key
`<result>` : for others

`<type>` : identify which pojo

`<property>` : from pojo
`<column>` : from database

use `<resultMap>` can match pojo & select even if the name is different.

## 多对一

> 对一对应对象，对多对应方法

为多的一方设置一的属性

- 级连
- association
- 分步查询

### 级连

for example:

```
class Emp:
    private Dept dept; // every employee belongs different department. N : 1
```

```
    <!--    Emp findAllInfoById(Integer id);-->
    <resultMap id="empAllInfoMap" type="Emp">
        <id property="eid" column="eid"/>
        <result property="empName" column="emp_name"/>
        <result property="age" column="age"/>
        <result property="sex" column="sex"/>
        <result property="email" column="email"/>
        <result property="did" column="did"/>
        <result property="dept.did" column="did"/>
        <result property="dept.deptName" column="dept_name"/>
    </resultMap>

    <select id="findAllInfoById" resultMap="empAllInfoMap">
        select *
        from t_emp left join t_dept
        on t_emp.did = t_dept.did
        where eid = #{eid}
    </select>
```

### association

*OR can use `<association>` to replace `dept.xx`, it is more concise*

### 分步查询

```
17:52:13.539 [main] DEBUG com.forty2.mybatis.mapper.EmpMapper.getAllInfoStepOne -- ==>  Preparing: select * from t_emp where eid = ?
17:52:13.554 [main] DEBUG com.forty2.mybatis.mapper.EmpMapper.getAllInfoStepOne -- ==> Parameters: 2(Integer)
17:52:13.575 [main] DEBUG com.forty2.mybatis.mapper.DeptMapper.getAllInfoStepTwo -- ====>  Preparing: select * from t_dept where did = ?
17:52:13.575 [main] DEBUG com.forty2.mybatis.mapper.DeptMapper.getAllInfoStepTwo -- ====> Parameters: 2(Integer)
17:52:13.576 [main] DEBUG com.forty2.mybatis.mapper.DeptMapper.getAllInfoStepTwo -- <====      Total: 1
17:52:13.577 [main] DEBUG com.forty2.mybatis.mapper.EmpMapper.getAllInfoStepOne -- <==      Total: 1
```

have selectOne & selectTwo. Use `association` to call selectTwo after selectOne was operated. ** !key point is `association`! **

```
<!--    Emp getAllInfoStepOne(Integer eid);-->

    <resultMap id="empAllInfoStep1" type="Emp">
        <id property="eid" column="eid"/>
        <result property="empName" column="emp_name"/>
        <result property="age" column="age"/>
        <result property="sex" column="sex"/>
        <result property="email" column="email"/>
        <result property="did" column="did"/>

        <association property="dept"
                     select="com.forty2.mybatis.mapper.DeptMapper.getAllInfoStepTwo"
                     column="did"/>
    </resultMap>

    <select id="getAllInfoStepOne" resultMap="empAllInfoStep1">
        select * from t_emp where eid = #{eid}
    </select>
```

Select label should use the unique name. 唯一标识？全类名。

#### 分布查询：延迟加载（懒加载）

如 emp 有 empID 和 Dept dept。

开启延迟加载后，如访问 empID 则直接查询，不需要进行第二步查询来查询 Dept 信息；等访问 Dept 信息时再查询 Dept。

默认关闭，需要在 mybatis-config 中打开。

```
<settings>
  <lazyLoadingEnabled> & <aggressiveLazyLoding>
</settings>
```

can be found in Documentations.

lazyLoadingEnabled: 延迟加载的全局开关，默认关闭
aggressiveLazyLoding: 默认开启，开启时无论调用什么方法都会加载所有属性，which means 只查询 empID 也会加载 dept，使得延迟加载失效

** so which means if wanna enable this function, need to change these two options simultaneously.**

## 一对多

> 对一对应对象，对多对应方法

通过 `<collection>` 解决。
1. ofType 用来声明集合内的 object 的属性。而后手写映射即可。如下代码示。
2. 也可以如上分布查询一样，在 collection 内用 property select column 来完成一个分布查询。


```
<!--    Dept getDeptAndEmp(@Param("did") Integer did);-->
    <resultMap id="deptAndEmpResultMap" type="Dept">
        <id property="did" column="did"/>
        <result property="deptName" column="dept_name"/>
        <collection property="emps" ofType="Emp">
            <id property="eid" column="eid"/>
            <result property="empName" column="emp_name"/>
            <result property="age" column="age"/>
            <result property="sex" column="sex"/>
            <result property="email" column="email"/>
            <result property="did" column="did"/>
        </collection>
    </resultMap>

    <select id="getDeptAndEmp" resultMap="deptAndEmpResultMap">
        select * from t_dept left join t_emp on t_dept.did = t_emp.did where t_dept.did = #{did}
    </select>
```

## 动态 SQL [Documentations](https://mybatis.org/mybatis-3/dynamic-sql.html)

provided by Mybatis. 是一种动态拼接字符串的技术，来解决字符串拼接操作的痛点。

### if

```
<!--    List<Emp> getEmpByCondition(Emp emp);-->
    <select id="getEmpByCondition" resultType="Emp">
        select * from t_emp where 1=1
        <if test="eid != null and eid != ''">
            and eid = #{eid}
        </if>
        <if test="empName != null and empName != ''">
            and emp_name = #{empName}
        </if>
        <if test="age != null and age != ''">
            and age = #{age}
        </if>
    </select>
```

不符合 test 条件的 if 不会被拼接进 sql 语句里

for example like if set age = null, preparedSql is like `Preparing: select * from t_emp where 1=1 and eid = ? and emp_name = ?`, `and age = #{age}` wont be concat to the whole sql statement.

> the reason why `where 1=1` 

but there are a simple question, if the first `if` unmet the test field, it will disappear, but the following statement are all starting with `and`. The way to fix it is, using a 1=1 statement after where, make all the if statement start with `and`.

### where

can also use `<where></where>` to surround `<if>`

```
<!--    List<Emp> getEmpByCondition(Emp emp);-->
    <select id="getEmpByCondition" resultType="Emp">
        select * from t_emp
        <where>
        <if test="eid != null and eid != ''">
            and eid = #{eid}
        </if>
        <if test="empName != null and empName != ''">
            and emp_name = #{empName}
        </if>
        <if test="age != null and age != ''">
            and age = #{age}
        </if>
        <where>
    </select>

    where 同时能自动去掉前面多余的 and & or，但后面的不能自动去除 like `eid = #{eid} and`
```

### trim 

have 4 fields:
    - suffix|prefix: 在 trim 标签中的内容的前面或后面加上指定内容
    - suffixOverrides|prefixOverrides: 在 trim 标签中的内容的前面或后面去掉指定内容

### choose..when..otherwise

> same as if.. / else if.. / else
> in my view i think it is more likely switch case but automatically added break on each lazyLoadingEnabled

### for each

collection: identify List or Set needed to be used
item: types of collection
separator: used to separate, often `,`
open: after where
close: ending

### sql 片段

> a simple way to insert fields

```
    <sql id="demo">eid, ename</sql>

    <select>
        select <include id="demo"> from t_emp
    </select>
```