### **MyBatis 中的 `resultMap` 使用详解**

`resultMap` 是 MyBatis 的核心功能之一，用于将查询结果映射到自定义 Java 对象。通过 `resultMap`，我们可以灵活地解决字段名与属性名不一致、多表关联查询映射等问题。

---

### **1. 为什么使用 `resultMap`**

在 MyBatis 中，默认会将查询结果映射为 Java 对象。如果字段名和属性名相同，可以直接使用 `resultType` 映射；但在以下场景中，`resultType` 就显得力不从心了：

- 数据库字段名和 Java 对象的属性名不一致。
- 查询结果需要映射到嵌套对象或关联对象中。
- 查询返回的数据需要做额外的处理。

此时，`resultMap` 提供了强大的自定义映射能力。

---

### **2. `resultMap` 的基本语法**

#### **定义 `resultMap`**

`resultMap` 是在 Mapper XML 文件中定义的，用于描述查询结果如何映射到目标对象。

##### **语法示例：**

```xml
<resultMap id="userResultMap" type="com.example.User">
    <id column="user_id" property="id"/>
    <result column="user_name" property="name"/>
    <result column="user_age" property="age"/>
</resultMap>
```

##### **说明：**

- `id`：指定主键的映射关系。
- `result`：指定普通字段的映射关系。
- `column`：数据库字段名。
- `property`：Java 对象的属性名。
- `type`：指定结果映射的目标 Java 类。

---

### **3. 使用 `resultMap`**

#### **在查询中引用 `resultMap`**

在定义 `resultMap` 后，可以在查询语句中通过 `resultMap` 来指定结果的映射方式。

##### **示例：**

```xml
<select id="getUserById" resultMap="userResultMap">
    SELECT user_id, user_name, user_age FROM users WHERE user_id = #{id}
</select>
```

#### **Java 调用：**

```java
User user = userMapper.getUserById(1);
```

---

### **4. 高级功能**

#### **4.1 嵌套映射（嵌套对象）**

如果查询结果中包含另一个对象的字段，可以通过嵌套映射来处理。

##### **场景：**

用户表 `users` 和地址表 `addresses`，每个用户对应一个地址。

##### **示例：**

**SQL 查询：**

```sql
SELECT u.user_id, u.user_name, a.address_id, a.address_detail 
FROM users u
JOIN addresses a ON u.address_id = a.address_id;
```

**定义 `resultMap`：**

```xml
<resultMap id="userResultMap" type="com.example.User">
    <id column="user_id" property="id"/>
    <result column="user_name" property="name"/>
    <association property="address" javaType="com.example.Address">
        <id column="address_id" property="id"/>
        <result column="address_detail" property="detail"/>
    </association>
</resultMap>
```

**查询配置：**

```xml
<select id="getUserWithAddress" resultMap="userResultMap">
    SELECT u.user_id, u.user_name, a.address_id, a.address_detail 
    FROM users u
    JOIN addresses a ON u.address_id = a.address_id
    WHERE u.user_id = #{id};
</select>
```

#### **4.2 嵌套集合**

如果一个用户拥有多个订单，可以通过 `collection` 映射到集合类型。

##### **示例：**

**SQL 查询：**

```sql
SELECT u.user_id, u.user_name, o.order_id, o.order_detail 
FROM users u
JOIN orders o ON u.user_id = o.user_id;
```

**定义 `resultMap`：**

```xml
<resultMap id="userWithOrdersResultMap" type="com.example.User">
    <id column="user_id" property="id"/>
    <result column="user_name" property="name"/>
    <collection property="orders" ofType="com.example.Order">
        <id column="order_id" property="id"/>
        <result column="order_detail" property="detail"/>
    </collection>
</resultMap>
```

**查询配置：**

```xml
<select id="getUserWithOrders" resultMap="userWithOrdersResultMap">
    SELECT u.user_id, u.user_name, o.order_id, o.order_detail 
    FROM users u
    JOIN orders o ON u.user_id = o.user_id
    WHERE u.user_id = #{id};
</select>
```

---

### **5. 常见问题**

#### **5.1 字段名与属性名不一致**

解决方法：通过 `resultMap` 显式映射。

```xml
<result column="user_name" property="name"/>
```

#### **5.2 嵌套对象和集合属性为 `null` 的问题**

确保查询结果中返回的字段能够正确匹配嵌套对象的主键或外键，否则 MyBatis 会自动忽略该嵌套对象的初始化。

---

### **6. 总结**

`resultMap` 提供了强大的结果映射能力，适用于复杂场景。以下是其核心功能：

|功能|标签|描述|
|---|---|---|
|**简单字段映射**|`<id>` `<result>`|将字段映射到 Java 属性|
|**嵌套对象映射**|`<association>`|将字段映射到 Java 中的另一个对象|
|**嵌套集合映射**|`<collection>`|将字段映射到 Java 中的集合类型|
|**多表查询映射**|`<resultMap>`|通过多表查询将结果映射到复杂对象或嵌套结构中|

通过合理使用 `resultMap`，可以极大简化复杂 SQL 的结果处理逻辑，同时增强代码的可读性和维护性。