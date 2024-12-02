MyBatis 提供了多种方式来接受查询结果，具体取决于查询的需求和返回的类型。以下是通过不同数据类型接受查询结果的几种常见方式：

---

### **1. 通过实体类接收查询结果**

**场景：**  
当查询的结果直接对应到一个实体类时，MyBatis 会将查询结果的列映射到实体类的属性上，常用于单表查询。

**示例：**

#### **Mapper 方法：**

```java
User getUserById(int id);
```

#### **XML 配置：**

```xml
<select id="getUserById" parameterType="int" resultType="com.example.User">
    SELECT * FROM users WHERE id = #{id};
</select>
```

**工作原理：**  
MyBatis 会根据查询的列名和实体类的属性名进行映射（大小写不敏感，默认使用驼峰命名规则）。查询结果会被自动封装成 `User` 实例。

---

### **2. 通过 `List` 集合接收查询结果**

**场景：**  
当查询结果是多行数据时，通常会返回一个集合，例如 `List` 或 `Set`，每一行数据会映射为一个实体类对象。

**示例：**

#### **Mapper 方法：**

```java
List<User> getAllUsers();
```

#### **XML 配置：**

```xml
<select id="getAllUsers" resultType="com.example.User">
    SELECT * FROM users;
</select>
```

**工作原理：**  
查询结果的每一行都会被映射为 `User` 实体类对象，最终返回一个 `List<User>`。对于多行结果，MyBatis 会自动将每一行封装成实体类并添加到集合中。

---

### **3. 通过数字（如 `Integer`，`Long`）接收查询结果**

**场景：**  
当查询结果是单一值（如统计数据、最大值、最小值等）时，可以直接返回基本数据类型的包装类（如 `Integer`、`Long`、`Double` 等）。

**示例：**

#### **Mapper 方法：**

```java
Integer getUserCount();
```

#### **XML 配置：**

```xml
<select id="getUserCount" resultType="java.lang.Integer">
    SELECT COUNT(*) FROM users;
</select>
```

**工作原理：**  
查询结果是一个单一的数字（例如 `COUNT`、`SUM` 等聚合函数的返回值），MyBatis 会将其直接映射为相应的包装类（如 `Integer`、`Long`）并返回。

---

### **4. 通过 `Map` 集合接收查询结果**

**场景：**  
当查询的结果列不固定或需要自定义列名映射时，可以使用 `Map` 来接收查询结果。这种方式适用于不直接对应某个实体类的复杂查询，或者查询的列名称动态变化的情况。

**示例：**

#### **Mapper 方法：**

```java
Map<String, Object> getUserInfoById(int id);
```

#### **XML 配置：**

```xml
<select id="getUserInfoById" parameterType="int" resultMap="userInfoMap">
    SELECT id, name, age FROM users WHERE id = #{id};
</select>
```

#### **`resultMap` 配置：**

```xml
<resultMap id="userInfoMap" type="java.util.HashMap">
    <result property="id" column="id"/>
    <result property="name" column="name"/>
    <result property="age" column="age"/>
</resultMap>
```

**工作原理：**  
MyBatis 会将查询结果中的列名作为 `Map` 的键，列值作为对应的值。在这个例子中，查询结果会被存储在 `Map<String, Object>` 中，键为列名（例如 `"id"`、`"name"`），值为对应的列值。

---

### **总结：**

|接收方式|适用场景|主要特点|
|---|---|---|
|**实体类接收**|查询结果与实体类一一对应时，适用于单行或多行查询|每行结果映射为实体类对象，属性与列名对应|
|**List 接收**|查询多行数据时，通常使用集合接收|返回结果为 `List` 或 `Set`，每个元素为实体类对象|
|**数字接收**|查询单一数字值（如 `COUNT`, `SUM`）时|直接返回包装类（如 `Integer`, `Long`）|
|**Map 接收**|查询列名不固定或列数不固定时|每行结果映射为 `Map`，键为列名，值为列值|

MyBatis 通过这些方式能够灵活地处理各种查询需求，确保了查询的高效性和灵活性。