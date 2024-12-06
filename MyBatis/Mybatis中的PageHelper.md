# **MyBatis 分页插件 PageHelper 详解**

PageHelper 是 MyBatis 社区提供的强大分页插件，它可以通过拦截 SQL 的方式实现分页功能，大幅减少手动编写分页代码的复杂性。

---

## **1. 什么是 PageHelper**

PageHelper 是一个 MyBatis 分页插件，能够自动处理分页查询的逻辑。通过调用其方法，开发者无需关心具体的分页 SQL 语法，只需要传递分页参数即可。

---

## **2. 引入 PageHelper**

### **2.1 添加依赖**

在项目的 `pom.xml` 中引入 PageHelper 的依赖：

```xml
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper</artifactId>
    <version>5.3.3</version>
</dependency>
```

### **2.2 配置插件**

在 MyBatis 全局配置文件中注册 PageHelper 插件。

#### Spring 配置示例

如果使用 Spring，推荐通过配置类的方式集成 PageHelper：

```java
import com.github.pagehelper.PageInterceptor;
import org.apache.ibatis.session.SqlSessionFactory;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.Properties;

@Configuration
public class MyBatisConfig {
    @Bean
    public PageInterceptor pageInterceptor() {
        PageInterceptor pageInterceptor = new PageInterceptor();
        Properties properties = new Properties();
        properties.setProperty("helperDialect", "mysql"); // 配置数据库方言
        properties.setProperty("reasonable", "true");    // 启用合理化分页
        properties.setProperty("supportMethodsArguments", "true");
        properties.setProperty("params", "count=countSql");
        pageInterceptor.setProperties(properties);
        return pageInterceptor;
    }
}
```

然后将该插件注册到 MyBatis 的 `SqlSessionFactory` 中。

---

## **3. 使用 PageHelper**

PageHelper 的核心方法是 `PageHelper.startPage()`，它通过拦截器拦截 SQL，在执行查询之前对 SQL 添加分页信息。

### **3.1 基本使用**

#### **示例代码**

```java
import com.github.pagehelper.PageHelper;
import com.github.pagehelper.PageInfo;
import java.util.List;

public class UserService {
    public PageInfo<User> findUsersByPage(int pageNum, int pageSize) {
        // 设置分页参数
        PageHelper.startPage(pageNum, pageSize);
        
        // 查询用户数据
        List<User> users = userMapper.findAllUsers();
        
        // 包装分页结果
        return new PageInfo<>(users);
    }
}
```

#### **Mapper 方法**

```java
public interface UserMapper {
    @Select("SELECT * FROM users")
    List<User> findAllUsers();
}
```

---

### **3.2 分页查询结果**

PageHelper 使用 `PageInfo` 对象来封装分页结果，提供丰富的分页信息，包括总记录数、总页数、当前页码等。

#### **PageInfo 的常用字段**

- `getList()`：当前页的数据列表。
- `getTotal()`：总记录数。
- `getPages()`：总页数。
- `getPageNum()`：当前页码。
- `getPageSize()`：每页显示的记录数。

#### **示例**

```java
PageInfo<User> pageInfo = userService.findUsersByPage(1, 10);
System.out.println("总记录数：" + pageInfo.getTotal());
System.out.println("总页数：" + pageInfo.getPages());
System.out.println("当前页码：" + pageInfo.getPageNum());
System.out.println("每页记录数：" + pageInfo.getPageSize());
pageInfo.getList().forEach(System.out::println); // 打印当前页的数据
```

---

### **3.3 分页参数说明**

#### **核心方法：**

```java
PageHelper.startPage(int pageNum, int pageSize);
```

|参数|含义|
|---|---|
|`pageNum`|当前页码，从 1 开始|
|`pageSize`|每页显示的记录数|

#### **合理化分页**

- 开启合理化分页后，如果 `pageNum < 1`，则会自动调整为 `1`；如果 `pageNum > 总页数`，则调整为最后一页。
- 配置方法：`reasonable=true`

---

## **4. 高级功能**

### **4.1 排序功能**

可以在分页时同时指定排序规则：

```java
PageHelper.startPage(pageNum, pageSize, "id DESC, name ASC");
```

### **4.2 使用分页插件支持的参数**

PageHelper 支持通过方法参数动态传递分页信息：

```java
PageHelper.startPage(pageNum, pageSize)
           .setOrderBy("id DESC");
```

---

## **5. 与其他框架的整合**

### **5.1 与 Spring 整合**

结合 Spring，推荐使用 `PageInfo` 作为返回结果，将分页数据返回给前端。

#### **示例 Controller**

```java
@RestController
@RequestMapping("/users")
public class UserController {
    @Autowired
    private UserService userService;

    @GetMapping
    public PageInfo<User> getUsers(@RequestParam int pageNum, @RequestParam int pageSize) {
        return userService.findUsersByPage(pageNum, pageSize);
    }
}
```

### **5.2 与前端整合**

前端常见的分页框架（如 Bootstrap Table、Element UI）可以通过接收分页数据实现分页展示。可以将 `PageInfo` 数据直接转换为 JSON 格式返回。

---

## **6. 注意事项**

1. **分页结果的影响范围：**
    
    - `PageHelper.startPage()` 的分页设置只对其后第一个查询生效，必须在调用查询方法前使用。
2. **查询结果类型：**
    
    - 查询方法返回的结果通常是一个 `List`，需要将其封装到 `PageInfo` 中获取完整的分页信息。
3. **性能优化：**
    
    - 分页查询时尽量避免查询过大的数据量。可以结合数据库索引和条件优化分页 SQL。
4. **MyBatis 配置：**
    
    - 确保全局配置中已启用分页插件。

---

## **7. 总结**

PageHelper 提供了一种高效、简单的分页处理方式，省去了手动编写分页 SQL 的繁琐步骤。通过灵活的配置和强大的功能，PageHelper 能够满足绝大多数场景的分页需求。同时，它与 Spring 和其他框架良好兼容，使分页功能的实现更加优雅。