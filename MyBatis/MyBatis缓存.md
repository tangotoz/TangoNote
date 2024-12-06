# **MyBatis 缓存详解**

MyBatis 提供了强大的缓存机制，用于提高查询性能和减少数据库访问频率。它支持两种类型的缓存：一级缓存和二级缓存。此外，MyBatis 还可以整合第三方缓存（如 EHCache）以进一步增强缓存能力。

---

## **1. 一级缓存**

### **1.1 定义**

一级缓存是 MyBatis 默认开启的缓存机制，与 SQL 会话（`SqlSession`）绑定。它的作用域为单个会话，在同一个 `SqlSession` 中，如果执行相同的查询条件，MyBatis 会直接从缓存中返回结果，而不是重新查询数据库。

### **1.2 特点**

- 默认开启，无需配置。
- 缓存的数据只在当前会话（`SqlSession`）内有效。
- 会话关闭后，一级缓存也会随之清空。

### **1.3 工作原理**

1. 在同一个 `SqlSession` 中执行查询时，MyBatis 会将查询结果存储在一级缓存中。
2. 如果再次执行相同的查询条件，MyBatis 会从缓存中取出结果，而不会访问数据库。
3. 如果进行增删改操作，一级缓存会被清空，确保数据的一致性。

### **1.4 示例**

#### **示例代码：**

```java
SqlSession sqlSession = sqlSessionFactory.openSession();
User user1 = sqlSession.selectOne("com.example.mapper.UserMapper.findUserById", 1);
User user2 = sqlSession.selectOne("com.example.mapper.UserMapper.findUserById", 1);
System.out.println(user1 == user2); // true，来自一级缓存
sqlSession.close();
```

#### **一级缓存失效的情况：**

1. 执行 `INSERT`、`UPDATE` 或 `DELETE` 操作，一级缓存会被清空。
2. 执行不同的查询条件。
3. `SqlSession` 被关闭或清空缓存（调用 `clearCache()` 方法）。

---

## **2. 二级缓存**

### **2.1 定义**

二级缓存是作用于命名空间（Mapper）的缓存。它允许多个 `SqlSession` 共享缓存数据，从而进一步提高性能。需要显式配置才能启用。

### **2.2 特点**

- 二级缓存是跨会话的缓存，同一命名空间的多个 `SqlSession` 实例可以共享缓存数据。
- 需要手动配置并开启。
- 二级缓存存储在内存中，可以通过第三方缓存实现持久化。

### **2.3 启用二级缓存**

#### **配置步骤：**

1. **在全局配置文件中启用二级缓存：**

```xml
<settings>
    <setting name="cacheEnabled" value="true"/>
</settings>
```

2. **在 Mapper 配置文件中声明使用二级缓存：**

```xml
<cache/>
```

3. **实体类实现序列化接口：** 二级缓存会将查询结果序列化后存储，因此需要让实体类实现 `java.io.Serializable` 接口。

#### **完整示例：**

##### **Mapper XML 配置文件（`UserMapper.xml`）：**

```xml
<mapper namespace="com.example.mapper.UserMapper">
    <!-- 启用二级缓存 -->
    <cache/>

    <select id="findUserById" resultType="com.example.User">
        SELECT * FROM users WHERE id = #{id}
    </select>
</mapper>
```

##### **Java 测试代码：**

```java
SqlSession sqlSession1 = sqlSessionFactory.openSession();
User user1 = sqlSession1.selectOne("com.example.mapper.UserMapper.findUserById", 1);
sqlSession1.close();

SqlSession sqlSession2 = sqlSessionFactory.openSession();
User user2 = sqlSession2.selectOne("com.example.mapper.UserMapper.findUserById", 1);
sqlSession2.close();

System.out.println(user1 == user2); // true，来自二级缓存
```

---

## **3. 整合第三方缓存（以 EHCache 为例）**

### **3.1 什么是 EHCache**

EHCache 是一个强大的分布式缓存框架，它可以与 MyBatis 集成，作为 MyBatis 的二级缓存提供者。通过 EHCache，可以实现更高效的缓存管理和更丰富的功能（如持久化、分布式缓存等）。

### **3.2 整合步骤**

#### **1. 引入依赖**

在 `pom.xml` 中添加 EHCache 和 MyBatis 的整合依赖：

```xml
<dependencies>
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis-ehcache</artifactId>
        <version>1.1.0</version>
    </dependency>
    <dependency>
        <groupId>net.sf.ehcache</groupId>
        <artifactId>ehcache</artifactId>
        <version>2.10.9.2</version>
    </dependency>
</dependencies>
```

#### **2. 配置 EHCache**

在 `resources` 目录下创建 `ehcache.xml` 文件：

```xml
<ehcache>
    <cache name="com.example.mapper.UserMapper"
           maxEntriesLocalHeap="1000"
           eternal="false"
           timeToIdleSeconds="3600"
           timeToLiveSeconds="3600"
           memoryStoreEvictionPolicy="LRU">
    </cache>
</ehcache>
```

#### **3. 修改 Mapper 配置**

在 Mapper XML 文件中指定使用 EHCache：

```xml
<cache type="org.mybatis.caches.ehcache.EhcacheCache"/>
```

#### **4. 测试代码：**

```java
SqlSession sqlSession1 = sqlSessionFactory.openSession();
User user1 = sqlSession1.selectOne("com.example.mapper.UserMapper.findUserById", 1);
sqlSession1.close();

SqlSession sqlSession2 = sqlSessionFactory.openSession();
User user2 = sqlSession2.selectOne("com.example.mapper.UserMapper.findUserById", 1);
sqlSession2.close();

System.out.println(user1 == user2); // true，来自 EHCache
```

---

## **4. 一级缓存与二级缓存的对比**

|特性|一级缓存|二级缓存|
|---|---|---|
|**作用域**|单个 `SqlSession`|跨 `SqlSession`，作用于命名空间|
|**默认状态**|默认开启|默认关闭，需要显式配置|
|**存储位置**|内存（当前会话内存）|内存，可结合第三方缓存实现持久化|
|**数据共享**|不同 `SqlSession` 不共享|相同命名空间的 `SqlSession` 共享|
|**缓存失效**|增删改操作、会话关闭清空|增删改操作清空对应命名空间的数据|
|**实现难度**|简单，无需额外配置|需要额外配置|

---

## **5. 缓存的优缺点**

### **优点**

- 减少数据库访问次数，提高查询性能。
- 提升系统的响应速度。
- 提供可扩展的缓存管理（如结合第三方缓存）。

### **缺点**

- 占用内存资源，可能增加 GC 压力。
- 缓存失效机制需要合理设计，否则可能导致数据不一致。
- 二级缓存的序列化和反序列化可能增加 CPU 开销。

---

## **6. 总结**

MyBatis 的缓存机制是优化性能的重要工具，其中一级缓存适用于简单场景，默认启用，使用成本低；二级缓存需要显式配置，并且可以与第三方缓存工具（如 EHCache）集成，从而实现更高级的缓存功能。在实际开发中，选择适合的缓存策略并合理配置缓存机制，可以有效提升应用性能，但也需要注意缓存的同步和一致性问题。