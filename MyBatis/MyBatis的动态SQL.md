### **MyBatis 动态 SQL 详解**

动态 SQL 是指在运行时根据条件动态构建 SQL 语句。MyBatis 提供了一些内置的标签和功能，使得我们可以根据不同的条件动态生成 SQL 语句。动态 SQL 的使用可以大大提高查询的灵活性，减少不必要的查询，并提升代码的可维护性。

### **1. 为什么需要动态 SQL？**

在实际开发中，查询条件经常会根据不同的业务需求有所变化。如果不使用动态 SQL，我们可能会写多个方法来处理不同条件的查询，这会导致代码重复和维护困难。动态 SQL 允许我们在同一个 SQL 语句中根据不同的条件生成不同的查询，从而提高代码的灵活性和复用性。

#### **常见场景：**

- 根据用户输入的查询条件动态生成 WHERE 子句。
- 根据是否传入某些条件决定是否加入某个查询条件。
- 实现复杂的 `JOIN`、`IN` 等查询操作。

---

### **2. MyBatis 动态 SQL 的实现方式**

MyBatis 提供了多个标签来构建动态 SQL，主要包括以下几种：

#### **2.1 `<if>` 标签**

`<if>` 标签用于根据某个条件判断是否将某段 SQL 语句添加到最终的 SQL 中。通常用于条件查询。

##### **语法：**

```xml
<if test="condition">
    SQL 语句
</if>
```

##### **示例：**

```xml
<select id="findUser" resultType="User">
    SELECT * FROM users
    <where>
        <if test="name != null">AND name = #{name}</if>
        <if test="age != null">AND age = #{age}</if>
    </where>
</select>
```

**说明：**

- `test` 属性接受一个条件表达式，只有当 `test` 的条件为 `true` 时，才会将 `<if>` 中的 SQL 添加到最终的查询语句中。
- `where` 标签会自动为第一个查询条件前添加 `WHERE`，并且会自动去除多余的 `AND`。

---

#### **2.2 `<choose>` 标签**

`<choose>` 标签类似于 Java 中的 `if-else` 语句，它可以根据条件选择不同的 SQL 语句。当有多个条件时，可以使用 `<choose>` 来指定只有一个条件满足时的执行路径。

##### **语法：**

```xml
<choose>
    <when test="condition1">
        SQL 语句1
    </when>
    <when test="condition2">
        SQL 语句2
    </when>
    <otherwise>
        SQL 语句3
    </otherwise>
</choose>
```

##### **示例：**

```xml
<select id="findUser" resultType="User">
    SELECT * FROM users
    <where>
        <choose>
            <when test="name != null">AND name = #{name}</when>
            <when test="age != null">AND age = #{age}</when>
            <otherwise>AND status = 'ACTIVE'</otherwise>
        </choose>
    </where>
</select>
```

**说明：**

- `when` 标签表示条件成立时执行的 SQL。
- `otherwise` 标签表示所有条件都不成立时执行的 SQL，相当于 `else` 部分。

---

#### **2.3 `<foreach>` 标签**

`<foreach>` 标签用于遍历集合（如列表、数组等），并根据集合的内容动态生成 SQL 语句。常用于处理 `IN` 查询、批量更新或批量删除等场景。

##### **语法：**

```xml
<foreach collection="collection" item="item" separator="," open="(" close=")">
    SQL 语句
</foreach>
```

##### **示例：**

```xml
<select id="findUsersByIds" resultType="User">
    SELECT * FROM users WHERE id IN 
    <foreach collection="ids" item="id" open="(" close=")" separator=",">
        #{id}
    </foreach>
</select>
```

**说明：**

- `collection`：传入的集合（如 List）。
- `item`：集合中的每个元素的名称。
- `separator`：元素之间的分隔符，通常是 `,`。
- `open` 和 `close`：用于包装 SQL 语句，常用于生成 `(` 和 `)`。

---

#### **2.4 `<set>` 标签**

`<set>` 标签用于动态更新时，自动处理 SQL 中的 `SET` 子句。在更新操作中，它会自动去除多余的 `AND` 或 `,`，并且只会在实际有更新字段时生成相应的 `SET` 子句。

##### **语法：**

```xml
<set>
    <if test="field != null">column = #{field},</if>
</set>
```

##### **示例：**

```xml
<update id="updateUser">
    UPDATE users
    <set>
        <if test="name != null">name = #{name},</if>
        <if test="age != null">age = #{age},</if>
        <if test="email != null">email = #{email},</if>
    </set>
    WHERE id = #{id}
</update>
```

**说明：**

- `set` 会自动去除最后的 `,`，避免生成不合法的 SQL。
- `if` 标签中的条件控制是否更新某一字段。

---

#### **2.5 `<trim>` 标签**

`<trim>` 标签用于去除 SQL 语句中的前缀或后缀，通常与 `<if>` 标签一起使用，以便处理不需要的 SQL 片段（例如，去除多余的 `AND` 或 `OR`）。

##### **语法：**

```xml
<trim prefix="PREFIX" suffix="SUFFIX" suffixOverrides="SUFFIX_OVERRIDES">
    SQL 语句
</trim>
```

##### **示例：**

```xml
<select id="findUser" resultType="User">
    SELECT * FROM users
    <trim prefix="WHERE" suffixOverrides="AND">
        <if test="name != null">AND name = #{name}</if>
        <if test="age != null">AND age = #{age}</if>
    </trim>
</select>
```

**说明：**

- `prefix` 和 `suffix` 分别用于添加前缀和后缀。
- `suffixOverrides` 用于去除末尾的某些字符（如 `AND`）。

---

### **3. 动态 SQL 的实际应用**

#### **3.1 动态查询条件**

当我们需要根据不同的查询条件动态生成 SQL 时，MyBatis 的动态 SQL 非常有用。例如，一个查询用户的方法，可以根据不同的条件生成不同的 WHERE 子句：

##### **示例：**

```xml
<select id="findUser" resultType="User">
    SELECT * FROM users
    <where>
        <if test="name != null">AND name = #{name}</if>
        <if test="age != null">AND age = #{age}</if>
        <if test="status != null">AND status = #{status}</if>
    </where>
</select>
```

#### **3.2 分页查询**

分页查询是 MyBatis 中常见的场景。通过动态 SQL，我们可以灵活地为查询语句添加分页参数。

##### **示例：**

```xml
<select id="getUsersByPage" resultType="User">
    SELECT * FROM users
    <where>
        <if test="name != null">AND name LIKE #{name}</if>
        <if test="age != null">AND age = #{age}</if>
    </where>
    LIMIT #{offset}, #{limit}
</select>
```

---

### **4. 动态 SQL 的优势**

|优势|说明|
|---|---|
|**减少代码冗余**|动态 SQL 可以避免为每种不同的查询条件写多个方法，减少了代码重复。|
|**灵活性高**|可以根据不同的查询条件生成不同的 SQL 查询，适应复杂的查询需求。|
|**提高可维护性**|动态 SQL 通过使用条件语句，使得 SQL 语句变得更加简洁、清晰，容易维护。|
|**性能优化**|通过动态控制 WHERE 子句，避免了不必要的查询条件，提升了查询性能。|

---

### **5. 总结**

MyBatis 的动态 SQL 是非常强大和灵活的功能，它通过各种标签（如 `<if>`、`<choose>`、`<foreach>` 等）来动态构建 SQL 语句。动态 SQL 可以帮助我们减少代码冗余、提高 SQL 查询的灵活性，同时还能提升查询性能，特别是在复杂查询和大数据量查询的场景中。

掌握动态 SQL 的使用，可以显著提高 MyBatis 项目的开发效率和可维护性。