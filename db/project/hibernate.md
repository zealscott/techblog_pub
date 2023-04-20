
## 创建项目

- 创建一个`HibernateTest`项目
  
- ![hibernate1](/images/hibernate1.png)
  
- 在依赖中添加postgresql的JAR包

- 修改`hibernate.cfg.xml`

  - ```xml
    <?xml version='1.0' encoding='utf-8'?>
    <!DOCTYPE hibernate-configuration PUBLIC
            "-//Hibernate/Hibernate Configuration DTD//EN"
            "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">
    <hibernate-configuration>
        <session-factory>
            <property name="connection.driver_class">org.postgresql.Driver</property>
            <!-- hibernate.connection.url : 连接数据库的地址,路径 -->
            <property name="hibernate.connection.url">jdbc:postgresql://localhost:5432/Game2</property>
            <!-- hibernate.connection.username : 连接数据库的用户名 -->
            <property name="hibernate.connection.username">postgres</property>
            <!-- hibernate.connection.password : 连接数据库的密码 -->
            <property name="hibernate.connection.password">1234</property>
            <!-- show_sql: 操作数据库时,会 向控制台打印sql语句 -->
            <property name="show_sql">true</property>
            <!-- format_sql: 打印sql语句前,会将sql语句先格式化  -->

            <!-- Set "true" to show SQL statements -->
            <property name="hibernate.show_sql">true</property>

             <!-- DB schema will be updated if needed -->
            <!-- <property name="hbm2ddl.auto">update</property> -->
        </session-factory>
    </hibernate-configuration>
    ```


- 这里连接了`Game2`的数据库


## 连接数据库

- 添加PostgreSQL

![hibernate2](/images/hibernate2.png)

- 写入用户名和密码，注意，红框中写数据库名字
  - ![hibernate3](/images/hibernate3.png)
- 在持久化中，添加我们需要的表：
  - ![hibernate4](/images/hibernate4.png)
- 这里需要指定Package，指定我们之前创建的entity
  - ![hibernat5](/images/hibernate5.png)
- 这样，我们会发现其自动帮我们新建了一个Trade的实体类：
  - ![hibernate6](/images/hibernate6.png)
- 但注意，没有帮我们添加构造函数，需要自己添加。
- 同时，在cfg文件中，发现其自动添加了mapping关系：
  - ![hibernate7](/images/hibernate8.png)

## 运行

- 这里需要额外注意的是，hibernate默认不区分大小写，如果表名或者column有大小写区分，需要用转义符自己更改

创建`HibernateSessionFactory`：



```java
      import entity.PlayerEntity;
      import org.hibernate.Session;
      import org.hibernate.SessionFactory;
      import org.hibernate.Transaction;
      import org.hibernate.boot.MetadataSources;
      import org.hibernate.boot.registry.StandardServiceRegistry;
      import org.hibernate.boot.registry.StandardServiceRegistryBuilder;
      import org.hibernate.cfg.Configuration;

      import java.util.List;

      public class HibernateSessionFactory {

      public static void main(String[] args) {
          //创建配置对象(读取配置文档)
          Configuration config = new Configuration().configure();
          //创建会话工厂对象
          SessionFactory sessionFactory = config.buildSessionFactory();
          //会话对象
          Session session = sessionFactory.openSession();
          //这是开启Session的操作
          session.beginTransaction();

          PlayerEntity player1 = new PlayerEntity();
          player1.setCoin(10);
          player1.setUserName("scott");

          //这正是把数据放入一级缓存session中的操作
          session.save(player1);
          //此处才是真正与数据库交互的语句
          session.getTransaction().commit();

          session.close();
      }
  }
```

这样直接运行会报错：

>  java.lang.ClassNotFoundException: javax.xml.bind.JAXBException

原因是JAXB API是java EE 的API，因此在java SE 9.0 中不再包含这个 Jar 包。java 9 中引入了模块的概念，默认情况下，Java SE中将不再包含java EE 的Jar包。而在 java 6/7 / 8 时关于这个API 都是捆绑在一起的。

因此，需要手动添加以下Jar包到lib：

- [javax.activation-1.2.0.jar](http://search.maven.org/remotecontent?filepath=com/sun/activation/javax.activation/1.2.0/javax.activation-1.2.0.jar)
- [jaxb-api-2.3.0.jar](http://search.maven.org/remotecontent?filepath=javax/xml/bind/jaxb-api/2.3.0/jaxb-api-2.3.0.jar)
- [jaxb-core-2.3.0.jar](http://search.maven.org/remotecontent?filepath=com/sun/xml/bind/jaxb-core/2.3.0/jaxb-core-2.3.0.jar)
- [jaxb-impl-2.3.0.jar](http://search.maven.org/remotecontent?filepath=com/sun/xml/bind/jaxb-impl/2.3.0/jaxb-impl-2.3.0.jar)

这样运行后，在数据库查看：

![hibernate9](/images/hibernate9.png)

成功！

## Debug

1. 在shell中，如果选择的column有大小写区分，需要添加**双引号**
2. 在shell中，如果需要添加字符串，需要添加**单引号**
3. [Hibernate5中表字段大小写探讨](https://blog.csdn.net/wm6752062/article/details/78417227)
4. [JAXBE错误](https://blog.csdn.net/hadues/article/details/79188793)
5. [Hibernate增删查改操作](https://blog.csdn.net/linhaiyun_ytdx/article/details/54946714)

