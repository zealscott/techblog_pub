<p align="center">
  <b>使用注解用Hibernate自动创建多对多数据表</b>
</p>


之前我们用hibernate连接已经创建的数据表，并避免了直接写SQL语言。

同样，对于多对多的关系映射，我们可以直接在程序中定义这种对象的关系，然后JPA会自动帮助我们创建表和相关关系。

## 创建项目

- 与之前一样，创建Hibernate项目。
- 需要额外注意，JDK9以上需要添加JPA额外的包才不会报错，具体请[参考这里](https://blog.csdn.net/hadues/article/details/79188793)。
- 同时，将Postgres的JAR包加入lib。

### 设置xml

- 设置`hibernate.cfg.xml`，注意，由于我们对象对应的表在该数据库中不存在，因此要添加自动创建表的设置

- ```xml
  <?xml version='1.0' encoding='utf-8'?>
  <!DOCTYPE hibernate-configuration PUBLIC
          "-//Hibernate/Hibernate Configuration DTD//EN"
          "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">
  <hibernate-configuration>
      <session-factory>
          <property name="connection.driver_class">org.postgresql.Driver</property>
          <!-- hibernate.connection.url : 连接数据库的地址,路径 -->
          <property name="hibernate.connection.url">jdbc:postgresql://localhost:5432/stu</property>
          <property name="hibernate.connection.username">postgres</property>
          <property name="hibernate.connection.password">xxx</property>
          <!-- show_sql: 操作数据库时,会 向控制台打印sql语句 -->
          <property name="show_sql">true</property>
          <!-- format_sql: 打印sql语句前,会将sql语句先格式化  -->

          <!--自动创建表-->
          <property name="hibernate.hbm2ddl.auto">update</property>

          <!-- Set "true" to show SQL statements -->
          <property name="hibernate.show_sql">true</property>

          <!-- mapping class using annotation -->
          <mapping class="Entity.Student"></mapping>
          <mapping class="Entity.Teacher"></mapping>

          <!-- DB schema will be updated if needed -->
          <!-- <property name="hbm2ddl.auto">update</property> -->
      </session-factory>
  </hibernate-configuration>
  ```

## 项目结构

![hibernate10](/images/hibernate10.png)

我们以老师和学生多对多的关系举例

### 学生

```java
package Entity;

import javax.persistence.*;
import java.io.Serializable;

import java.util.Date;
import java.util.Set;

@Entity
//@Table(name = "t_student")
//@SequenceGenerator(name = "SEQ_STUDENT", sequenceName = "SEQ_STUDENT")

public class Student implements Serializable {
    private static final long serialVersionUID = 2524659555729848644L;
    private Long id;
    private String name;
    private Date birthday;
    private int sex;
    private String address;
    private Set<Teacher> teacherList;

    private Integer a;


    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id", nullable = false, precision = 22, scale = 0)
    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    @Column(name = "name")
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Temporal(TemporalType.DATE)
    @Column(name = "birthday")
    public Date getBirthday() {
        return birthday;
    }

    public void setBirthday(Date birthday) {
        this.birthday = birthday;
    }

    @Column(name = "sex")
    public int getSex() {
        return sex;
    }

    public void setSex(int sex) {
        this.sex = sex;
    }

    @Column(name = "address")
    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    @ManyToMany(cascade = CascadeType.ALL)
    @JoinTable(name = "t_teacher_student",
            joinColumns = @JoinColumn(name = "stu_id",referencedColumnName="id"),
            inverseJoinColumns = @JoinColumn(name = "teacher_id",referencedColumnName ="id"))
    public Set<Teacher> getTeacherList() {
        return teacherList;
    }

    public void setTeacherList(Set<Teacher> teacherList) {
        this.teacherList = teacherList;
    }
}
```

### 老师

```java
package Entity;
import Entity.Student;

import javax.persistence.*;
import java.io.Serializable;
import java.util.Set;

@Entity
//@Table(name = "t_teacher")
//@SequenceGenerator(name = "SEQ_TEACHER", sequenceName = "SEQ_TEACHER")

public class Teacher implements Serializable {
    private static final long serialVersionUID = 2297316923535111793L;
    private Long id;
    private String name;
    private int sex;
    private Set<Student> studentList;

    @Id
//    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "SEQ_TEACHER")
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id", nullable = false, precision = 22, scale = 0)
    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    @Column(name = "name")
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Column(name = "sex")
    public int getSex() {
        return sex;
    }

    public void setSex(int sex) {
        this.sex = sex;
    }

    @ManyToMany(mappedBy = "teacherList", cascade = CascadeType.ALL)
    public Set<Student> getStudentList() {
        return studentList;
    }

    public void setStudentList(Set<Student> studentList) {
        this.studentList = studentList;
    }
}
```

### Main

```java
import Entity.Student;
import Entity.Teacher;
import org.hibernate.HibernateException;
import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.cfg.Configuration;

import java.util.HashSet;
import java.util.Set;

public class Main {


    public static void main(final String[] args) throws Exception {
        //创建配置对象(读取配置文档)
        Configuration config = new Configuration().configure();
        //创建会话工厂对象
        SessionFactory sessionFactory = config.buildSessionFactory();
        //会话对象
        Session session = sessionFactory.openSession();
        //这是开启Session的操作
        session.beginTransaction();
        Student s = new Student();
        s.setName("小猪");
        Teacher t = new Teacher();
        t.setName("小李");
        Set<Teacher> t_set = new HashSet<Teacher>();
        t_set.add(t);
        s.setTeacherList(t_set);
        session.save(s);
        //此处才是真正与数据库交互的语句
        session.getTransaction().commit();

        session.close();

    }
}
```

## 运行

运行后，打印SQL语句，发现hibernate帮我们创建了表，并添加了数据

![hibernate11](/images/hibernate11.png)

在shell中查看数据：

![hibernate12](/images/hibernate12.png)

## 参考

1. [hibernate多对多关系](https://blog.csdn.net/yudar1024/article/details/43025367)

2. [理解JPA注解@GeneratedValue](https://blog.csdn.net/canot/article/details/51455967)