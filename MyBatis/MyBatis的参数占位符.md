在 **MyBatis** 中，`#{}` 和 `${}` 是两种常用的参数占位符，它们的用法和处理机制不同，适用的场景也各有特点。以下是它们的详细对比和用法说明：

---

### **1. `#{}`：安全的预编译占位符**

- **作用**：`#{}` 是 MyBatis 的参数占位符，会将参数值安全地绑定到 SQL 中，使用预编译机制防止 SQL 注入。
- **特点**：
    1. 参数会通过 **PreparedStatement** 的 `?` 占位符机制进行处理。
    2. MyBatis 会自动将参数值进行类型转换和转义，适用于任何类型的参数。
    3. 安全性高，推荐在大多数场景中使用。

#### **示例：**

```java
User getUserById(int id);
```

**对应的 XML 配置：**

```xml
<select id="getUserById" parameterType="int" resultType="com.example.model.User">
    SELECT * FROM users WHERE id = #{id}
</select>
```

**生成的 SQL：**

```sql
SELECT * FROM users WHERE id = ?
```

- 参数 `id` 会在执行时通过 `PreparedStatement` 替换为对应值。

---

### **2. `${}`：直接文本替换**

- **作用**：`${}` 是 MyBatis 的字符串替换占位符，直接将参数值插入到 SQL 中，不经过预编译。
- **特点**：
    1. 参数值会被直接拼接到 SQL 字符串中，适用于动态生成字段名或表名等 SQL 片段的场景。
    2. 存在 **SQL 注入** 风险，不推荐用于传递用户输入的参数值。
    3. 需要手动确保参数值的安全性。

#### **示例：**

```java
List<User> getUsersByColumn(@Param("column") String column, @Param("value") String value);
```

**对应的 XML 配置：**

```xml
<select id="getUsersByColumn" resultType="com.example.model.User">
    SELECT * FROM users WHERE ${column} = #{value}
</select>
```

**传入参数：**

```java
column = "name";
value = "John";
```

**生成的 SQL：**

```sql
SELECT * FROM users WHERE name = ?
```

- `${column}` 会被直接替换为 `"name"`，而 `#{value}` 会通过预编译的方式绑定值。

---

### **3. 区别对比**

|**特性**|`#{}`|`${}`|
|---|---|---|
|**参数替换方式**|通过预编译机制，将参数值转换为 `?` 绑定到 SQL|将参数值直接拼接到 SQL 中|
|**是否安全**|安全，防止 SQL 注入|不安全，容易引发 SQL 注入|
|**适用场景**|用户输入值、需要防止注入的参数、常规查询字段或条件|动态生成 SQL 的字段名、表名等（开发者需确保安全）|
|**是否支持动态生成表名或字段名**|不支持|支持（仅限字符串动态拼接）|
|**性能**|较好，因使用预编译提高了查询效率|略差，因动态拼接每次执行都需重新解析 SQL|

---

### **4. 综合示例**

假设需要根据动态字段名查询用户信息：

**Mapper 接口方法：**

```java
List<User> getUsersByDynamicField(@Param("field") String field, @Param("value") String value);
```

**XML 配置：**

```xml
<select id="getUsersByDynamicField" resultType="com.example.model.User">
    SELECT * FROM users WHERE ${field} = #{value}
</select>
```

**传入参数：**

```java
field = "email";
value = "user@example.com";
```

**生成的 SQL：**

```sql
SELECT * FROM users WHERE email = ?
```

---

### **5. 注意事项**

- **`#{}` 推荐用于传递所有用户输入的值**，因为它能够防止 SQL 注入。
- **`${}` 仅在动态生成 SQL 结构（如表名或列名）时使用**，必须确保参数值的安全性。
- 使用 `${}` 时，需避免直接使用用户输入的值来拼接 SQL，否则可能导致安全隐患。

---

### **6. 总结**

- **`#{}`**：默认选择，用于参数值绑定，安全且高效。
- **`${}`**：仅限特殊场景（如动态 SQL 生成），需慎用并确保安全性。



在 MyBatis 中，如果方法参数有多个，MyBatis 会根据特定的规则和配置来处理这些参数。以下是 MyBatis 如何处理多个参数的细节。

---

### **1. 默认方式（索引引用）**

如果没有使用 `@Param` 注解或参数不是对象类型，MyBatis 会将多个参数放入一个 **`Map`** 中，并按以下规则进行处理：

- 参数的键名为 `param1`、`param2`、`param3`...，按参数的顺序依次命名。
- 同时，会按参数的位置索引从 `0` 开始生成键名，如 `0`、`1`、`2` 等。
- 你可以通过键名 `paramN` 或数字索引来访问参数。

#### **示例：**

**Mapper 接口：**

```java
User getUserByIdAndName(int id, String name);
```

**对应的 XML 配置：**

```xml
<select id="getUserByIdAndName" resultType="com.example.model.User">
    SELECT * FROM users
    WHERE id = #{param1} AND name = #{param2}
</select>
```

**传入参数：**

```java
getUserByIdAndName(1, "John");
```

**MyBatis 内部 Map 结构：**

```java
{
  "param1": 1,
  "param2": "John",
  "0": 1,
  "1": "John"
}
```

**生成的 SQL：**

```sql
SELECT * FROM users WHERE id = 1 AND name = 'John';
```

#### **注意：**

这种方式虽然能正常工作，但键名（`param1`、`param2` 等）对参数意义不明确，可读性较差。

---

### **2. 使用 `@Param` 注解**

为了提高可读性和便于维护，可以使用 `@Param` 注解为参数指定一个明确的名称。MyBatis 会将这些参数存储在一个 Map 中，键名为注解中指定的名称。

#### **示例：**

**Mapper 接口：**

```java
User getUserByIdAndName(@Param("id") int id, @Param("name") String name);
```

**对应的 XML 配置：**

```xml
<select id="getUserByIdAndName" resultType="com.example.model.User">
    SELECT * FROM users
    WHERE id = #{id} AND name = #{name}
</select>
```

**传入参数：**

```java
getUserByIdAndName(1, "John");
```

**MyBatis 内部 Map 结构：**

```java
{
  "id": 1,
  "name": "John"
}
```

**生成的 SQL：**

```sql
SELECT * FROM users WHERE id = 1 AND name = 'John';
```

#### **优势：**

1. 参数名称更加明确，代码可读性提高。
2. XML 中的 SQL 能够直观地映射到参数名。

---

### **3. 使用 Java 对象传递参数**

如果方法的多个参数封装在一个对象中，MyBatis 会直接使用对象的属性来引用参数值。

#### **示例：**

**定义一个参数对象：**

```java
public class UserQuery {
    private int id;
    private String name;

    // Getter 和 Setter 方法
}
```

**Mapper 接口：**

```java
User getUser(UserQuery query);
```

**对应的 XML 配置：**

```xml
<select id="getUser" resultType="com.example.model.User">
    SELECT * FROM users
    WHERE id = #{id} AND name = #{name}
</select>
```

**传入参数：**

```java
UserQuery query = new UserQuery();
query.setId(1);
query.setName("John");
getUser(query);
```

**MyBatis 内部处理：**

- MyBatis 会通过反射机制，直接访问 `query` 对象的 `id` 和 `name` 属性。

**生成的 SQL：**

```sql
SELECT * FROM users WHERE id = 1 AND name = 'John';
```

#### **优势：**

1. 适用于参数较多的场景，避免多个参数的复杂管理。
2. 参数更易于扩展和维护。

---

### **4. 参数类型为 `Map`**

开发者可以直接使用 `Map` 作为方法参数，手动管理键值对，MyBatis 会按 Map 的键名引用参数。

#### **示例：**

**Mapper 接口：**

```java
User getUserByIdAndName(Map<String, Object> params);
```

**对应的 XML 配置：**

```xml
<select id="getUserByIdAndName" resultType="com.example.model.User">
    SELECT * FROM users
    WHERE id = #{id} AND name = #{name}
</select>
```

**传入参数：**

```java
Map<String, Object> params = new HashMap<>();
params.put("id", 1);
params.put("name", "John");
getUserByIdAndName(params);
```

**生成的 SQL：**

```sql
SELECT * FROM users WHERE id = 1 AND name = 'John';
```

#### **优势：**

- 灵活性高，适合动态查询。
- 参数名和值完全由开发者定义。

---

### **5. 复杂场景：动态 SQL**

如果需要动态传递多个参数，可以结合 MyBatis 的动态 SQL（如 `<if>`、`<foreach>` 等）来实现。

#### **示例：**

**动态查询：**

```xml
<select id="getUsers" resultType="com.example.model.User">
    SELECT * FROM users
    WHERE 1=1
    <if test="id != null">
        AND id = #{id}
    </if>
    <if test="name != null">
        AND name = #{name}
    </if>
</select>
```

**Mapper 接口：**

```java
List<User> getUsers(@Param("id") Integer id, @Param("name") String name);
```

**传入参数：**

```java
getUsers(1, null);
```

**生成的 SQL：**

```sql
SELECT * FROM users WHERE 1=1 AND id = 1;
```

---

### **总结**

|**方式**|**处理方式**|**适用场景**|
|---|---|---|
|**默认方式（索引引用）**|参数按 `paramN` 或索引值 `0`、`1` 传递|参数较少，且不需要显式命名时使用|
|**`@Param` 注解命名**|通过注解显式指定参数名称|参数较多，或需要明确命名以提高可读性|
|**对象封装**|使用对象的属性来传递参数|参数较多或参数属于同一业务对象的场景|
|**`Map` 传参**|开发者手动管理参数键值对|动态参数和灵活查询场景|
|**动态 SQL**|使用动态 SQL 标签（如 `<if>`、`<foreach>`）组合条件|参数数量或查询条件不固定的复杂场景|

一般情况下，推荐使用 **`@Param`** 或 **对象封装**，代码更直观且易于维护。