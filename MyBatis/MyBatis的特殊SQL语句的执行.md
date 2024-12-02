下面是关于 MyBatis 特殊 SQL 语句执行的笔记，不涉及动态 SQL 的内容：

---

### **1. 模糊查询**

**场景：**  
模糊查询通常用于通过 `LIKE` 关键字进行匹配查询，适用于通过部分字符串进行查找。

#### **示例：**

##### **Mapper 方法：**

```java
List<User> getUsersByName(String name);
```

##### **XML 配置：**

```xml
<select id="getUsersByName" resultType="com.example.User">
    SELECT * FROM users WHERE name LIKE CONCAT('%', #{name}, '%');
</select>
```

**工作原理：**

- `#{name}` 为传入的参数，`CONCAT('%', #{name}, '%')` 用于在查询时拼接 `%` 符号，表示模糊匹配前后部分。
- 查询结果将返回 `name` 列中包含输入名称的所有记录。

---

### **2. 批量删除**

**场景：**  
批量删除常用于删除多个记录，避免使用循环逐个删除，影响性能。MyBatis 支持通过 `IN` 子句进行批量删除。

#### **示例：**

##### **Mapper 方法：**

```java
void deleteUsersByIds(String ids);
```

##### **XML 配置：**

```xml
<delete id="deleteUsersByIds">
    DELETE FROM users WHERE id IN (${ids})
</delete>
```

**工作原理：**

- 使用 `IN` 子句来批量删除记录，`foreach` 标签用于遍历传入的 `ids` 列表，并生成 SQL 查询。
- 生成的 SQL 语句类似于：`DELETE FROM users WHERE id IN (1, 2, 3)`。

---

### **3. 动态设置表名

**场景：**  
有时候在某些情况下，需要根据不同的条件查询不同的表。在 MyBatis 中，如果不使用动态 SQL，可以通过传递不同的表名来实现此需求。

#### **示例：**

##### **Mapper 方法：**

```java
List<User> getUsersFromTable(String tableName);
```

##### **XML 配置：**

```xml
<select id="getUsersFromTable" resultType="com.example.User">
    SELECT * FROM ${tableName} WHERE age > #{age}
</select>
```

**工作原理：**

- 使用 `${}` 语法动态替换表名。需要注意的是，这种方式直接拼接表名时，要确保输入的表名来自可信的来源，防止 SQL 注入问题。
- 该方法允许在查询时根据条件切换不同的表，但不推荐直接使用动态 SQL（如 `WHERE` 条件变化等），此处仅替换表名。

---

### **4. 获取自增主键**

**场景：**  
在执行插入操作时，数据库的自增主键是一个常见需求。MyBatis 可以通过配置在插入时自动获取自增主键的值。

#### **示例：**

##### **Mapper 方法：**

```java
void insertUser(User user);
```

##### **XML 配置：**

```xml
<insert id="insertUser" useGeneratedKeys="true" keyProperty="id">
    INSERT INTO users (name, age) VALUES (#{name}, #{age});
</insert>
```

**工作原理：**

- `useGeneratedKeys="true"` 表示启用自增主键返回功能。
- `keyProperty="id"` 表示将自增主键值返回并设置到 `User` 实体的 `id` 属性上。
- 插入操作后，`User` 实体的 `id` 属性将自动填充为数据库生成的自增主键。

---

### **总结**

|操作类别|适用场景|MyBatis 配置方式|
|---|---|---|
|**模糊查询**|根据部分匹配条件查找数据|使用 `LIKE` 和 `CONCAT` 进行模糊匹配查询|
|**批量删除**|删除多个记录|使用 `IN` 子句和 `foreach` 动态生成批量删除条件|
|**动态设置表名**|根据不同条件查询不同的表|使用 `${}` 语法动态替换表名，需注意防止 SQL 注入|
|**获取自增主键**|插入数据后获取数据库自增主键|使用 `useGeneratedKeys="true"` 和 `keyProperty` 获取自增主键值|

---

这些 SQL 操作在 MyBatis 中非常常见，通过灵活配置，可以高效地处理多种数据库操作，满足实际开发需求。