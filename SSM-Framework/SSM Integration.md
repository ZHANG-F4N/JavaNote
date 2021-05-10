# SSM Integration

[TOC]

---

集成开发环境：`IDEA`

数据库：`Mysql 5.7.32`

服务器：`Tomcat 9.0.45`

项目管理工具：`Maven：3.8.1`

JDK：`JDK 1.8`

---



# 1、数据库建表

```mysql
CREATE DATABASE `ssmbuild`;

USE `ssmbuild`;

CREATE TABLE `books`(
`bookID` INT(10) NOT NULL AUTO_INCREMENT COMMENT '书id',
`bookName` VARCHAR(100) NOT NULL COMMENT '书名',
`bookCounts` INT(11) NOT NULL COMMENT '数量',
`detail` VARCHAR(200) NOT NULL COMMENT '描述',
KEY `bookID`(`bookID`)
)ENGINE=INNODB DEFAULT CHARSET=utf8;

INSERT INTO `books`(`bookID`,`bookName`,`bookCounts`,`detail`) VALUES
(1,'Java',1,'从入门到放弃'),
(2,'MySQL',10,'从删库到跑路'),
(3,'Linux',5,'从进门到进牢');
```

# 2、项目pom.xml配置

- 创建普通Maven项目

- pom.xml中配置依赖

  ```xml
  <!--添加依赖
      junit   数据库驱动   数据库连接池  servlet JSP
      mybatis mybatis-spring  spring lombok
  -->
  <dependencies>
      <dependency>
          <groupId>junit</groupId>
          <artifactId>junit</artifactId>
          <version>4.13</version>
      </dependency>
      <dependency>
          <groupId>mysql</groupId>
          <artifactId>mysql-connector-java</artifactId>
          <version>5.1.47</version>
      </dependency>
      <!--数据库连接池 : c3p0 -->
      <dependency>
          <groupId>com.mchange</groupId>
          <artifactId>c3p0</artifactId>
          <version>0.9.5.5</version>
      </dependency>
      <dependency>
          <groupId>javax.servlet</groupId>
          <artifactId>servlet-api</artifactId>
          <version>2.5</version>
      </dependency>
      <dependency>
          <groupId>javax.servlet.jsp</groupId>
          <artifactId>jsp-api</artifactId>
          <version>2.2</version>
      </dependency>
      <dependency>
          <groupId>javax.servlet</groupId>
          <artifactId>jstl</artifactId>
          <version>1.2</version>
      </dependency>
      <dependency>
          <groupId>org.mybatis</groupId>
          <artifactId>mybatis</artifactId>
          <version>3.5.2</version>
      </dependency>
      <dependency>
          <groupId>org.mybatis</groupId>
          <artifactId>mybatis-spring</artifactId>
          <version>2.0.2</version>
      </dependency>
      <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-webmvc</artifactId>
          <version>5.3.6</version>
      </dependency>
      <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-jdbc</artifactId>
          <version>5.3.6</version>
      </dependency>
      <dependency>
          <groupId>org.projectlombok</groupId>
          <artifactId>lombok</artifactId>
          <version>1.18.12</version>
      </dependency>
  </dependencies>
  ```

- pom.xml中配置静态资源导出

  ```xml
  <build>
      <resources>
          <resource>
              <directory>src/main/resources</directory>
              <excludes>
                  <exclude>**/*.properties</exclude>
                  <exclude>**/*.xml</exclude>
              </excludes>
              <filtering>false</filtering>
          </resource>
          <resource>
              <directory>src/main/java</directory>
              <includes>
                  <include>**/*.properties</include>
                  <include>**/*.xml</include>
              </includes>
              <filtering>false</filtering>
          </resource>
      </resources>
  </build>
  ```





# 3、resources中创建配置和资源文件

![image-20210510112906346](SSM Integration.assets/image-20210510112906346.png)

- 创建数据库连接配置文件database.properties

  ```properties
  jdbc.driver=com.mysql.jdbc.Driver
  #Mysql8.0+需要在url中增加时区配置 &serverTimezone=Asia/Shanghai
  jdbc.url=jdbc:mysql://localhost:3306/ssmbuild?useSSL=true&useUnicode=true&characterEncoding=utf8
  jdbc.username=root
  jdbc.password=123456
  ```

- 创建mybatis的配置文件mybatis-config.xml

  ```xml
  <?xml version="1.0" encoding="UTF-8" ?>
  <!DOCTYPE configuration
          PUBLIC "-//mybatis.prg//DTD Config 3.0//EN"
          "http://mybatis.org/dtd/mybatis-3-config.dtd">
  <configuration>
  </configuration>
  ```

- 创建spring的配置文件applicationContext.xml

  ```xml
  <?xml version="1.0" encoding="UTF-8" ?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://www.springframework.org/schema/beans
          http:http://www.springframework.org/schema/beans/spring-beans.xsd">
  
  </beans>
  ```



# 4、创建实体类及接口，实现SQL



- 创建实体类`Books`

  ```java
  @Data
  @AllArgsConstructor
  @NoArgsConstructor
  public class Books {
      private int bookID;
      private String bookName;
      private int bookCounts;
      private String detail;
  
  }
  ```

- 编写dao层的`BookMapper`接口及`BookMapper.xml`实现

  ```java
  public interface BookMapper {
      //增加一本书
      int addBook(Books book);
      //删除一本书
      int deleteBookByID(@Param("bookId") int ID);
      //更新一本书
      int updateBook(Books book);
      //查询一本书
      Books queryBookByID(@Param("bookId") int ID);
      //查询全部的书
      //也可以使用注解写SQL,但有的复杂Sql不好写
      //@Select("select * from  ssmbuild.books")
      List<Books> queryAllBooks();
  }
  ```

  

  ```xml-dtd
  <?xml version="1.0" encoding="UTF-8" ?>
  <!DOCTYPE mapper
          PUBLIC "-//mybatis.prg//DTD Config 3.0//EN"
          "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
  <mapper namespace="com.zhang.dao.BookMapper">
  
      <insert id="addBook" parameterType="Books">
          insert into ssmbuild.books( bookName, bookCounts, detail)
          VALUES (#{bookName},#{bookCounts},#{detail});
      </insert>
      <delete id="deleteBookByID" parameterType="int">
          delete from ssmbuild.books where bookID=#{bookId};
      </delete>
      <update id="updateBook" parameterType="Books">
          update ssmbuild.books
          set bookName=#{bookName},
              bookCounts=#{bookCounts},
              detail=#{detail}
          where bookID=#{bookID}
      </update>
      <select id="queryBookByID" resultType="Books">
          select *
          from ssmbuild.books where bookID=#{bookId};
      </select>
      <select id="queryAllBooks" resultType="Books">
          select *
          from ssmbuild.books;
      </select>
  </mapper>
  ```

  写Sql时的自动补齐可以在IDEA中设置，`File | Settings | Languages & Frameworks | SQL Dialects`

- 在`mybatis-config.xml`中注册写好的`BookMapper.xml`

  ```xml
  <!--配置数据源,交给Spring做,-->
  <typeAliases>
      <package name="com.zhang.pojo"/>
  </typeAliases>
  <mappers>
      <!--dao里面的xml与接口名一致的话,可以使用class绑定
          如果不一致,使用resource绑定-->
      <mapper class="com.zhang.dao.BookMapper"></mapper>
  </mappers>
  ```



# 5、整合Spring

因为IDEA设置配置文件时,自动将配置文件关联了起来，不然需要手动将各个配置文件import到一个文件下。

![image-20210510161534973](SSM Integration.assets/image-20210510161534973.png)

- 创建`spring-dao.xml`，将`dao`层配置进来

  ```xml
  <?xml version="1.0" encoding="UTF-8" ?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns:context="http://www.springframework.org/schema/context"
         xsi:schemaLocation="http://www.springframework.org/schema/beans
          http:http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">
  
      <!--1.关联数据库配置文件-->
      <context:property-placeholder location="classpath:database.properties"/>
  
      <!--2.数据库连接池
          dbcp:半自动化操作,不能自动连接          c3p0:自动操作(自动化操作,可以自动设置到对象中)
          hikari:Spring2.0+自带              druid:很多公司在用
          我们用 c3p0
          -->
      <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
          <property name="driverClass" value="${jdbc.driver}"/>
          <property name="jdbcUrl" value="${jdbc.url}"/>
          <property name="user" value="${jdbc.username}"/>
          <property name="password" value="${jdbc.password}"/>
          <!--c3p0的私有属性-->
          <property name="maxPoolSize" value="30"/>
          <property name="minPoolSize" value="10"/>
          <!--关闭后不自动commit-->
          <property name="autoCommitOnClose" value="false"/>
          <!--获取连接超时时间-->
          <property name="checkoutTimeout" value="10000"/>
          <!--获取连接失败重连次数-->
          <property name="acquireRetryAttempts" value="2"/>
      </bean>
  
      <!--3. sqlSessionFactory-->
      <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
          <property name="dataSource" ref="dataSource"/>
          <!--绑定Mybatis的配置文件-->
          <property name="configLocation" value="classpath:mybatis-config.xml"/>
      </bean>
  
      <!--配置Dao接口,扫描包,动态实现了Dao接口,可以注入到Spring容器中-->
      <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
          <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/><!--和上面的sqlSessionFactory同名-->
          <!--要扫描的包-->
          <property name="basePackage" value="com.zhang.dao"/>
      </bean>
  </beans>
  ```

- 创建`spring-service.xml`，将`service`层配置进来

  ```xml
  <?xml version="1.0" encoding="UTF-8" ?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns:context="http://www.springframework.org/schema/context"
         xsi:schemaLocation="http://www.springframework.org/schema/beans
          http:http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">
  
      <!--1.扫描service下的包-->
      <context:component-scan base-package="com.zhang.service"/>
  
      <!--2.将我们的所有业务类,注入到Spring,可以通过配置或者注解实现-->
      <bean id="BookServiceImpl" class="com.zhang.service.BookServiceImpl">
          <!--因为IDEA设置配置文件时,自动将配置文件关联了起来
              会有自动补全,不然需要手动将各个配置文件import到一个文件下-->
          <property name="bookMapper" ref="bookMapper"/>
      </bean>
  
      <!--3.声明式事务配置-->
      <bean id="transcationManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
          <property name="dataSource" ref="dataSource"/>
      </bean>
      <!--4.可选aop事务支持-->
  </beans>
  ```



# 6、整合SpringMVC

- 添加WEB框架支持

- 配置`src\main\resources\spring-mvc.xml`并添加入主配置文件。

  ```xml
  <?xml version="1.0" encoding="UTF-8" ?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns:context="http://www.springframework.org/schema/context"
         xmlns:mvc="http://www.springframework.org/schema/mvc"
         xsi:schemaLocation="http://www.springframework.org/schema/beans
          http:http://www.springframework.org/schema/beans/spring-beans.xsd
          http://www.springframework.org/schema/context
          https://www.springframework.org/schema/context/spring-context.xsd
          http://www.springframework.org/schema/mvc
          http://www.springframework.org/schema/cache/spring-mvc.xsd">
  
      <!--1.注解驱动-->
      <mvc:annotation-driven/>
      <!--2.静态资源过滤-->
      <mvc:default-servlet-handler/>
      <!--3.扫描包 controller-->
      <context:component-scan base-package="com.zhang.controller"/>
      <!--4.视图解析器-->
      <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
          <property name="prefix" value="/WEB-INF/jsp/"/>
          <property name="suffix" value=".jsp"/>
      </bean>
  
  </beans>
  ```

- 配置`web\WEB-INF\web.xml`

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
           version="4.0">
      <!--DispatcherServlet-->
      <servlet>
          <servlet-name>springmvc</servlet-name>
          <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
          <init-param>
              <param-name>contextConfigLocation</param-name>
              <param-value>classpath:spring-mvc.xml</param-value>
          </init-param>
      </servlet>
      <servlet-mapping>
          <servlet-name>springmvc</servlet-name>
          <url-pattern>/</url-pattern>
      </servlet-mapping>
      <!--乱码过滤-->
      <filter>
          <filter-name>encodingFilter</filter-name>
          <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
          <init-param>
              <param-name>encoding</param-name>
              <param-value>utf-8</param-value>
          </init-param>
      </filter>
      <filter-mapping>
          <filter-name>encodingFilter</filter-name>
          <url-pattern>/*</url-pattern>
      </filter-mapping>
      <!--Session-->
      <session-config>
          <session-timeout>15</session-timeout>
      </session-config>
  </web-app>
  ```



- 完整的项目结构

  ![image-20210510170718840](SSM Integration.assets/image-20210510170718840.png)



# 7、实现第一个业务-查询

- 创建`src\main\java\com\zhang\controller\BookController.java`，实现业务逻辑，调用`Service`层。

  ```java
  @Controller
  @RequestMapping("/book")
  public class BookController {
      //Controller层调Service层
      @Autowired
      @Qualifier("BookServiceImpl")
      private BookService bookService;
  
      //查询全部数据,并返回到一个书籍的展示页面
      @RequestMapping("/allbook")
      public String bookList(Model model){
          List<Books> books = bookService.queryAllBooks();
          model.addAttribute("list",books);
          return "allBook";
      }
  
  }
  ```

- 创建JSP页面。

- 配置Tomcat，添加lib

- 运行














