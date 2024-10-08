# MyBatis

# dependencies

1. mysql-connector-j
2. mybatis
3. junit (optional)
4. lombok (optional)

# configurations

Can be finded from [MyBatis Documentation](https://mybatis.org/mybatis-3/getting-started.html)

like:

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "https://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>
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
    <mapper resource="org/mybatis/example/BlogMapper.xml"/>
  </mappers>
</configuration>
```

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

# Mapper interface

In basic jdbc, we need to create DAO interface and the implements of such interface to access to database, but in MyBatis, the mapper interface can be seem as the combaination of those two class because MyBatis supports to **Interface oriented programming**, so we dont need to create the object of interface, and thats the reason why we dont need to write class to achive interface. It is more easier to code.

# Files structure

- java
    - pojo
        - User (Java Bean)
    - mapper
        - UserMapper (Interface)
- test

# Mapping File

1. make sure the file's name is same as table. For example, t_user vs UserMapper.

2. the file's name use XMapper.xml

3. each mapping file corresponds to its table.

4. putting mapping file in resources directory.

5. the content of mapping file's example can be found in [documentation](https://mybatis.org/mybatis-3/getting-started.html). 

6. the namespace in xml must same as the full path of the correspond interface

7. the id in xml must same as the method in the interface.

# 2 ways to get values from variable in MyBatis (!Important!)

1. `${}` <- **字符串拼接** 存在 sql 注入，单引号需要手动拼接，like `where username = '${username}'`

2. `#{}` <- **占位符赋值** recommend

3. if have more than 2 formal variable, MyBatis will put those formal variable into a map automatically, which means need to use default key like `#{arg0} #{arg1}` or `#{param1} #{param2}` to visit values.

*You can also create map by yourself to set keys' name.* Like:

```
map = new HashMap();
map.put("username", root);
map.put("password", root);
mapper.select(map);

select * from table where username = #{username}
```

4. if inputing a pojo like `User user`, can also use `#{}` to access object's values.