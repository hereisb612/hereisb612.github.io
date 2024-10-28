# Spring 事务

## 声明式事务 vs 编程式事务

### 编程式

通过代码的方式告知 JVM 需要做什么。需要自行编程实现

#### 优点

排错简单

#### 缺点

代码量大

### 声明式

通过注解等方式告知框架需要做什么，框架将会完成余下工作

**Spring** 中的事务管理是**声明式**的。

#### 优点

代码量小，可读性高

#### 缺点

封装太多，所以通过 debug 排错时需要深入底层，排错困难



## Module 准备

1. create module in SpringBoot Starter
2. because need to use Mysql, need to import JDBC & MySql driver from springboot starter
3. can also use parent module to manage dependencies by using label modules



## Basic Database Operation

Spring 提供了简易的数据库连接方法，在 application.properties 中，使用 spring.datasource 即可完成数据库配置信息的配置。

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/spring_tx
spring.datasource.username=root
spring.datasource.password=password
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
```

### DataSource

当完成 datasource 的配置之后，Spring 会自动在容器中生成一个 `DataSource` 数据源对象，使用自动装配注入后即可使用。

```java
@SpringBootTest
class Spring03TxApplicationTests {
    @Autowired
    DataSource dataSource;

    @Test
    public void test1() throws SQLException {
        Connection connection = dataSource.getConnection();
        System.out.println(connection);
    }
}
```

*output*:	**Hikari**ProxyConnection@885876140 wrapping com.mysql.cj.jdbc.ConnectionImpl@35c9a231

Spring 默认使用市面最快的 Hikari 数据源。

### JdbcTemplate 

与此同时，Spring 还提供了专门用于数据库增删改查的 `JdbcTemplate` 对象，其中内置一系列 api 可供调用，依旧是自动装配即可用。

```java
@Component
public class BookDao {
    @Autowired
    private JdbcTemplate jdbcTemplate;

    public Book getBookById(Integer id) {
        String sql = "select * from book where id = ?";
        return jdbcTemplate.queryForObject(sql, new BeanPropertyRowMapper<>(Book.class), id);
    }
}
```

```java
@SpringBootTest
class Spring03TxApplicationTests {
    @Autowired
    DataSource dataSource;
    @Autowired
    BookDao bookDao;

    @Test
    public void test1() throws SQLException {
        Connection connection = dataSource.getConnection();
        System.out.println(connection);
    }

    @Test
    public void test2() throws SQLException {
        Book bookById = bookDao.getBookById(1);
        System.out.println(bookById);
    }
}
```

*output*:	Book(id=1, bookName=剑指Java, price=100.00, stock=100)

#### queryForObject()

```java
public <T> T queryForObject(String sql, RowMapper<T> rowMapper, @Nullable Object... args) throws DataAccessException{}
```

有三个参数：

1. sql 可选 **有/无占位符**版本 的，如果用占位符，可用第三个可变参数形参传入。

2. RowMapper 是一个接口，用于描述查询结果与 Javabean 的对应关系。在其下有一个实现类为 **BeanPropertyRowMapper**，用于将 封装好的 JavaBean 对象的每个属性 与 数据库一行中的每一列 完成映射，其参数传入 JavaBean.class 即可。

3. 可变参数列表用于给占位符赋值。

#### update()

##### 添加

```java
public class BookDao {
    @Autowired
    private JdbcTemplate jdbcTemplate;

    public void addBook(Book book){
        String sql = "insert into book(bookName, price, stock) values(?,?,?)";
        jdbcTemplate.update(sql, book.getBookName(), book.getPrice(), book.getStock());
    }
}
```

##### 修改

```java
@Component
public class BookDao {
    @Autowired
    private JdbcTemplate jdbcTemplate;

    public void updateBookDivideStock(Integer id, Integer stock) {
        String sql = "update book set stock = stock - ? where id = ?";
        jdbcTemplate.update(sql, stock, id);
    }
}
```

##### 删除

```java
@Component
public class BookDao {
    @Autowired
    private JdbcTemplate jdbcTemplate;

		public void deleteBookById(Integer id) {
        String sql = "delete from book where id = ?";
        jdbcTemplate.update(sql, id);
    }
}
```



## 无事务参与的交易

```java
@Service
public class UserServiceImpl implements UserService {
    @Autowired
    BookDao bookDao;
    @Autowired
    AccountDao accountDao;

    @Override
    public void checkOut(String username, Integer bookId, Integer boughtBookNumber) {
        // 1. 查询图书信息
        Book book = bookDao.getBookById(bookId);
        // 2. 计算价格
        BigDecimal price = book.getPrice();
        BigDecimal howMuch = new BigDecimal(boughtBookNumber).multiply(price);
        // 3. 扣钱
        accountDao.updateAccountDividedBalanceByUsername(username, howMuch);
        // 3. 减库存
        bookDao.updateBookDivideStock(bookId,boughtBookNumber);
    }
}
```

1、2、3 步应连续完成，若其中有多线程执行，可能会导致数据错误。



## @Transactional

1. 首先，想要开启事务功能，需要使用 @EnableTransactionManagement 注解，来**开启基于注解的自动化事务管理**。

   可以标记在主程序上。

   ```java
   @SpringBootApplication
   @EnableTransactionManagement
   public class Spring03TxApplication {
       public static void main(String[] args) {
           SpringApplication.run(Spring03TxApplication.class, args);
       }
   }
   ```

2. 为想要开启事务的方法上标记 @Transactional

   ```java
   @Override
   @Transactional
   public void checkOut(String username, Integer bookId, Integer boughtBookNumber) {
       // 1. 查询图书信息
       Book book = bookDao.getBookById(bookId);
       // 2. 减余额
       BigDecimal price = book.getPrice();
       BigDecimal howMuch = new BigDecimal(boughtBookNumber).multiply(price);
       accountDao.updateAccountDividedBalanceByUsername(username, howMuch);
       // 3. 减库存
       bookDao.updateBookDivideStock(bookId,boughtBookNumber);
   }
   ```

   此时事务管理即开启开启，可除 0 或 throw Exception 测试。



