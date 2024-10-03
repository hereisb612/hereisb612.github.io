# dependencies

1. mysql-connector-j
2. mybatis
3. junit (optional)

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
