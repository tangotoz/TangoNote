要通过 **IntelliJ IDEA** 使用 **Maven** 搭建一个简单的 **MyBatis** 项目，并仅引入 **MyBatis**、**MySQL JDBC** 和 **JUnit 3** 这三个 JAR 包，你可以按照以下步骤操作：

### **1. 创建 Maven 项目**

首先，打开 **IntelliJ IDEA** 并创建一个新的 Maven 项目：

1. 打开 IntelliJ IDEA，选择 **File > New > Project**。
2. 选择 **Maven**，点击 **Next**。
3. 在 **GroupId** 和 **ArtifactId** 中填写你的项目名称，例如：
    - **GroupId**：`com.example`
    - **ArtifactId**：`mybatis-demo`
4. 点击 **Finish** 完成创建。

### **2. 配置 `pom.xml`**

接下来，编辑 **pom.xml** 文件，添加所需的依赖项。将以下内容添加到 `<dependencies>` 标签内：

```xml
<?xml version="1.0" encoding="UTF-8"?>  
<project xmlns="http://maven.apache.org/POM/4.0.0"  
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">  
    <modelVersion>4.0.0</modelVersion>  
  
    <groupId>com.tango</groupId>  
    <artifactId>MyBatis-demo1</artifactId>  
    <version>1.0-SNAPSHOT</version>  
  
  
  
  
    <dependencies>        <dependency>            <groupId>mysql</groupId>  
            <artifactId>mysql-connector-java</artifactId>  
            <version>8.0.28</version>  
        </dependency>  
        <dependency>            <groupId>org.mybatis</groupId>  
            <artifactId>mybatis</artifactId>  
            <version>3.5.16</version>  
        </dependency>  
        <dependency>            <groupId>junit</groupId>  
            <artifactId>junit</artifactId>  
            <version>4.13.2</version>  
        </dependency>    </dependencies>  
</project>
```

这些依赖项分别是：

- **MyBatis**：用于数据库持久化操作。
- **MySQL JDBC**：用于连接 MySQL 数据库。
- **JUnit 4**：用于单元测试。

### **3. 配置 MyBatis 配置文件**

在 `src/main/resources` 目录下，创建一个 MyBatis 配置文件 `mybatis-config.xml`，并配置数据库连接和 Mapper 映射器：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!-- 配置全局属性 -->
    <settings>
        <setting name="logImpl" value="SLF4J"/>
        <setting name="jdbcTypeForNull" value="NULL"/>
    </settings>

    <!-- 数据源配置 -->
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/mydatabase"/>
                <property name="username" value="root"/>
                <property name="password" value="password"/>
            </dataSource>
        </environment>
    </environments>

    <!-- 配置 Mapper 映射器 -->
    <mappers>
        <mapper resource="com/example/mapper/UserMapper.xml"/>
    </mappers>
</configuration>
```

### **4. 创建 Mapper 接口和 XML 文件**

在 `src/main/java/com/example/mapper` 目录下，创建一个 **Mapper** 接口 `UserMapper.java`，并定义 SQL 查询：

```java
package com.example.mapper;

import com.example.model.User;
import org.apache.ibatis.annotations.Select;

public interface UserMapper {
    @Select("SELECT * FROM users WHERE id = #{id}")
    User getUserById(int id);
}
```

然后，在 `src/main/resources/com/example/mapper` 目录下，创建一个 `UserMapper.xml` 文件来映射 SQL 查询：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.mapper.UserMapper">

    <!-- 映射 SQL 查询 -->
    <select id="getUserById" resultType="com.example.model.User">
        SELECT * FROM users WHERE id = #{id}
    </select>

</mapper>
```

### **5. 创建 Model 类**

在 `src/main/java/com/example/model` 目录下，创建一个 **User.java** 类，表示数据库中的 `users` 表：

```java
package com.example.model;
@Data
public class User {
    private int id;
    private String name;
    private String email;
}
```

### **6. 使用 MyBatis 执行 SQL 查询**

在 `src/main/java/com/example` 目录下创建一个 **Main.java** 类来执行 MyBatis 查询：

```java
package com.example;

import com.example.mapper.UserMapper;
import com.example.model.User;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import java.io.InputStream;

public class Main {
    public static void main(String[] args) throws Exception {
        // 加载 MyBatis 配置文件
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        // 创建 SqlSession
        try (SqlSession session = sqlSessionFactory.openSession()) {
            // 获取 Mapper
            UserMapper userMapper = session.getMapper(UserMapper.class);

            // 执行查询
            User user = userMapper.getUserById(1);
            System.out.println(user.getName());
        }
    }
}
```

### **7. 创建 JUnit 测试**

在 `src/test/java/com/example` 目录下，创建一个 **UserMapperTest.java** 测试类，使用 JUnit 3 编写单元测试：

```java
package com.example;

import com.example.mapper.UserMapper;
import com.example.model.User;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import junit.framework.TestCase;

import java.io.InputStream;

public class UserMapperTest extends TestCase {
    private SqlSessionFactory sqlSessionFactory;

    @Override
    protected void setUp() throws Exception {
        // 加载 MyBatis 配置文件
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
    }

    public void testGetUserById() {
        try (SqlSession session = sqlSessionFactory.openSession()) {
            UserMapper userMapper = session.getMapper(UserMapper.class);
            User user = userMapper.getUserById(1);
            assertNotNull("User should not be null", user);
            assertEquals("Expected user name", "John Doe", user.getName());
        }
    }
}
```

### **8. 运行和测试**

- 运行 `Main` 类，确保数据库中有一个名为 `users` 的表，包含适当的数据。
- 运行 `UserMapperTest` 类，验证 MyBatis 查询是否正常工作。

### 9. log4j的日志功能
要在现有的 MyBatis 项目中加入 **Log4j** 日志功能，步骤如下：
首先，在 `pom.xml` 文件中添加 Log4j 的依赖：

```xml
<dependencies>
    <!-- 其他依赖项 -->
    
    <!-- 添加 Log4j 1.x 依赖 -->
    <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>1.2.17</version>
    </dependency>
</dependencies>
```


在 `src/main/resources` 目录下，创建一个 **log4j.xml** 文件，配置日志记录器：


```xml
<?xml version="1.0" encoding="UTF-8"?>
<log4j:configuration xmlns:log4j="http://jakarta.apache.org/log4j/">

    <!-- 设置根日志记录器 -->
    <root>
        <level value="INFO"/>
        <appender-ref ref="console"/>
    </root>

    <!-- 设置控制台输出 -->
    <appender name="console" class="org.apache.log4j.ConsoleAppender">
        <layout class="org.apache.log4j.PatternLayout">
            <param name="ConversionPattern" value="%d{yyyy-MM-dd HH:mm:ss} - %p - %m%n"/>
        </layout>
    </appender>

    <!-- 配置 MyBatis 日志 -->
    <logger name="org.apache.ibatis">
        <level value="DEBUG"/>
    </logger>

</log4j:configuration>
```

在需要记录日志的地方，添加 Log4j 日志功能。例如，在 `Main.java` 中：

```java
package com.example;

import com.example.mapper.UserMapper;
import com.example.model.User;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.apache.log4j.Logger;

import java.io.InputStream;

public class Main {
    private static final Logger logger = Logger.getLogger(Main.class);  // 创建日志记录器

    public static void main(String[] args) throws Exception {
        // 加载 MyBatis 配置文件
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        // 创建 SqlSession
        try (SqlSession session = sqlSessionFactory.openSession()) {
            // 获取 Mapper
            UserMapper userMapper = session.getMapper(UserMapper.class);

            // 执行查询
            User user = userMapper.getUserById(1);
            if (user != null) {
                logger.info("Found user: " + user.getName());  // 记录信息日志
            } else {
                logger.warn("User not found!");  // 记录警告日志
            }
        } catch (Exception e) {
            logger.error("Error occurred: ", e);  // 记录错误日志
        }
    }
}
```

- 运行 `Main` 类时，你将在控制台看到日志输出。例如：
    - 查询成功时，输出日志：`INFO - Found user: John Doe`
    - 查询失败时，输出警告日志：`WARN - User not found!`
    - 如果出现异常，日志将记录错误信息。

通过以上步骤，你已成功将 **Log4j** 集成到 MyBatis 项目中，并通过 `log4j.properties` 配置文件设置了日志输出级别和格式。此配置允许你在执行 SQL 查询时输出详细的日志信息。

### **总结**

以上步骤通过 Maven 搭建了一个 MyBatis 项目，使用了 **MyBatis**、**MySQL JDBC** 和 **JUnit 3** 三个 JAR 包。关键步骤包括：

1. 配置 `pom.xml` 文件，添加 MyBatis、MySQL 和 JUnit 依赖。
2. 创建 MyBatis 配置文件和 SQL 映射文件。
3. 编写 Mapper 接口和 Model 类。
4. 使用 JUnit 编写测试，确保 MyBatis 查询正常。
5. 使用log4j完成日志打印。