# 前端分页参数最大限制调整

### 问题描述
前端请求参数的分页大小默认是由限制的，早期版本限制最大`400`，`1.4.0.RELEASE`之后修改为限制`1000`，如果默认限制无法满足请求，可以通过下面的解决方案自定义限制。


### 解决方案
最大限制是在Spring的参数解析中限制的

```java
io.choerodon.mybatis.spring.resolver.PageRequestHandlerMethodArgumentResolver#setMaxPageSize
```

可以通过在该Bean的构建后调用`setMaxPageSize`方法自定义最大限制，例如：

```java
@Component
public class MaxPageModifier implements BeanPostProcessor {

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) {
        if (bean instanceof PageRequestHandlerMethodArgumentResolver) {
		    ((PageRequestHandlerMethodArgumentResolver) bean).setMaxPageSize(2000);
		}
        return bean;
    }
}
```

---

# 查询时不想分页怎么控制

### 问题描述

在使用Mapper开发组件时，有分页控制的API，在某些情况下不想分页，需要怎么处理？

### 解决方案

在分页参数`page`，传入`-1`时，则不进行分页查询。

---

# 组件概述

> 组件编码 `hzero-starter-mybatis-mapper`

## 一、简介

## 1.1 概述

增强ORM框架Mybatis的数据库DML处理能力，支持分页、数据多语言、基于对象的SQL编写，数据防篡改等功能。

## 1.2 组件坐标

```xml
<dependency>
    <groupId>org.hzero.starter</groupId>
    <artifactId>hzero-starter-mybatis-mapper</artifactId>
</dependency>
```

## 1.3 特性
* 基于猪齿鱼choerodon-starter-mybatis-mapper组件拓展
* 支持复杂条件查询
* 扩展多语言支持
* 添加数据防篡改功能
* 添加数据加密存储功能
* 添加唯一校验功能

## 1.4 支持数据库

Mybatis增强组件基本支持所有常见的数据库类型，例如：MySQL、Oracle、Microsoft SQL Server、PostgreSQL、Zenith等。

除常见数据库外，还支持一些国产数据库，如：DM DBMS(达梦)、KingbaseES(人大金仓)

---

# 自定义主键策略

# 自定义主键策略

## 雪花ID

 **雪花ID是Twitter推出的分布式全局唯一ID的一套解决方案，相比于UUID，雪花ID有以下优点：**

- 按照时间有序生成
- 长度最长为19位的数字（Long）
- 效率较高，在整个分布式系统内不会产生碰撞

### 雪花ID结构

```text
0 - 0000000000 0000000000 0000000000 0000000000 0 - 00000 - 00000 - 000000000000
```

- 第 `1 ` 位：符号位。由于ID一般采用整数，所以首位一般固定`0`；
- 后 `41` 位：时间戳位。雪花ID需要有一个固定的起始时间，这里记录的是毫秒级的当前时间戳差值，最多可以使用越 69 年，(1L << 41) / (1000L * 60 * 60 * 24 * 365) = 69；
- 后 `10` 位：机器标识位：可以部署`1024`个节点，包含`5`位(`32`个)数据中心ID（dataCenterId）和`5`位(`32`个)工作机器ID（workerId）；
- 后 `12` 位：序列位：毫秒内的计算。`12`位的计数顺序号支持每个节点，每毫秒生成`4096`个序号（也就是理论上每个分布式节点在`1`毫秒内可以生成`4096`个ID）；
- 总 `64` 位：合并组成一个`Long`数字。

### 配置属性

```yaml
mybatis:
  configuration:
    key-generator: snowflake
    snowflake:
      start-timestamp: 1577808000000
      meta-provider: redis
      meta-provider-redis-db: 1
      meta-provider-redis-refresh-interval: 540000
      meta-provider-redis-expire: 600000
      data-center-id: 1
      worker-id: 1
	  bit-timestamp: 41
	  bit-data-center-id: 5
	  bit-worker-id: 5
	  bit-sequence: 12
```
- `key-generator`：主键生成策略，`snowflake`表示使用雪花ID
- `start-timestamp`：雪花ID起始时间，默认`1577808000000`，也就是`2020/01/01 00:00:00`
- `meta-provider`：雪花ID元数据生成支持，自动生成`dataCenterId`和`workerId`，默认`redis`，可选值：
  - `none`：简易模式，`dataCenterId`使用当前服务名的`hash`值的绝对值然后和`32`求余，`workerId`取`[0-32)`之间的随机整数，节点之间可能会重复
  - `redis`：Redis模式，自动注册`dataCenterId`和`workerId`，支持最多`32`个服务，每个服务`32`个实例，超出限制会注册报错，节点之间不会重复
  - `eureka`：Eureka模式，在服务注册时自动生成`dataCenterId`和`workerId`，多实例之间不会重复。使用该模式时需要将hzero-register服务eureka.client.fetch-registry配置置为true。
- `meta-provider-redis-db`：仅在Redis模式下生效，默认`1`，Redis模式自动注册的DB
- `meta-provider-redis-refresh-interval`：仅在Redis模式下生效，默认`540000`，也就是`9`分钟，Redis模式自动注册刷新间隔
- `meta-provider-redis-expire`：仅在Redis模式下生效，默认`600000`，也就是`10`分钟，Redis模式自动注册的失效时间，请保证过期时间大于刷新间隔，否则设置无效
- `data-center-id`：数据中心ID，可以手工指定`[0-31)`，如果手工指定了数据中心ID，`provider`就会使用手工指定的值
- `worker-id`：工作机器ID，可以指定`[0-31)`，如果手工指定了工作机器ID，`provider`就会使用手工指定的值
- `bit-*`：代表对应的段的数据长度，某些项目可能服务特别多，超过`32`个导致雪花ID数据中心位不够用，此时可以扩展设置为`6`增加到`64`个，但是相应的`worker-id`或者`sequence`需要减掉一位来保证数据不会溢出，这个需要项目自己权衡，大多数情况下`32`个机器ID可能比较多，减掉一位变成`4`（16个实例）也能满足需求。**（需要注意的是怎么组合都是允许的，但是最终需要满足`timestamp + data-center-id + worker-id + sequence = 63`）**

**注意：**一般来说只需要指定`key-generator: snowflake`就可以了，其他的配置都有默认值，按需设置

### 常见问题

**Q：开启雪花ID之后前端传给后端的ID与后端传递到前端的ID不一致。**
A：雪花ID最终生成的数值是一个长整型数值，最大值为`2^63 - 1`，但是由于JS遵循`IEEE 754`规范，采用双精度存储，在值大于`2^53`次方之后会有精度丢失的问题，解决这个问题可以结合主键加密工具，将雪花ID转成加密字符串或者纯数字的字符串。
[主键加密参考文档](https://open.hand-china.com/document-center/doc/component/134/16966?doc_id=220145&doc_code=5261 "主键加密参考文档")（文档中心搜索主键加密组件）

---

# 数据唯一校验

# 数据唯一校验

## 功能说明
* 在开发过程中，经常需要到传到后端的数据在数据库中做唯一校验，该功能是对该需求的封装，旨在简化开发过程中重复的工作，降低开发量。

## 使用说明
1. 声明唯一校验字段：在`Entity`需要校验唯一的字段上添加注解`@org.hzero.mybatis.annotation.Unique`
2. 调用校验方法：`org.hzero.mybatis.helper.UniqueHelper#valid(T)`方法，该方法返回一个**布尔值**，如果返回`true`表示校验通过，返回`false`表示数据已存在。

```java
Assert.isTrue(UniqueHelper.valid(bank), BaseConstants.ErrorCode.DATA_EXISTS);
```


## 多唯一索引
<div style="text-align: left"><img src="https://hsop-doc.su.bcebos.com/doc_classify/0/HSOP-BAIDU/ce3291754a394772a262ce506a0d87fd@wecom-temp-22acd815a6742163a1e8403e3165268b.png" alt="" width="auto" height="auto"/></div>
代码示例

```java
@VersionAudit
@ModifyAudit
@JsonInclude(value = JsonInclude.Include.NON_NULL)
@Table(name = "demo_index")
public class DemoIndex extends AuditDomain {

    @Id
    @GeneratedValue
    private Long id;
	
    @Unique("indexOne")
    @Unique("indexTwo")
    private String indexOne;
	
    @Unique("indexOne")
    @Unique("indexTwo")
    private String indexTwo;
	//get/set方法
}
```

或者

```java
@VersionAudit
@ModifyAudit
@JsonInclude(value = JsonInclude.Include.NON_NULL)
@Table(name = "demo_index")
public class DemoIndex extends AuditDomain {

    @Id
    @GeneratedValue
    private Long id;
	
    @Unique.List({@Unique("indexOne"), @Unique("indexTwo")})
    private String indexOne;
	
    @Unique.List({@Unique("indexOne"), @Unique("indexTwo")})
    private String indexTwo;
	//get/set方法
}
```


---

# 租户条件过滤

# 租户条件过滤

## 功能说明
* 为了防止Saas模式下的租户功能越权（查询到不属于自己租户的数据），在没有租户参数进行数据过滤控制的情况下，增加了后端通用过滤规则。

## 使用说明
1.注解模式，此模式针对使用平台封装好的查询方法生效。

1.1 在Controller类方法上添加注解`@org.hzero.mybatis.annotation.TenantLimitedRequest`注解，默认SQL拼装为`IN`模式，如：

```SQL
WHERE 1 = 1
  AND tenant_id IN (可访问租户ID列表)
```

1.2 设置注解属性`TenantLimitedRequest(equal=true)`，SQL拼装为`=`模式，如：

```SQL
WHERE 1 = 1
  AND tenant_id = (当前租户ID)
```

2. 针对自定义Mapper的SQL语句，注解方式不支持自动改写，需要在`Mapper`中使用bind的方式进行引用

2.1 获取可访问租户列表函数，示例：

```XML
<bind name="__tenantIds" value="@org.hzero.mybatis.helper.TenantLimitedHelper@tenantIds()" />
```

引入后，在使用到的地方进行使用，如：

```XML
<if test="__tenantIds != null and !__tenantIds.isEmpty()">
  and tenant_id in 
  <foreach collection="__tenantIds" item="__tenantId" separator="," open="(" close=")">
    #{__tenantId}
  </foreach>
</if>
```

2.2 获取当前租户函数，示例：

```XML
<bind name="__tenantId" value="@org.hzero.mybatis.helper.TenantLimitedHelper@tenantId()" />
```

引入后，在使用到的地方进行使用，如：

```XML
<if test="__tenantId != null">
  and tenant_id = #{__tenantId}
</if>
```


---

# 数据加密存储

# 数据加密存储

## 功能说明
* 有一些保密性比较强的信息需要加密之后保存到数据库，例如配置的用户的邮箱密码，某些其他三方服务的密钥等，在数据库做加密存储主要是防止数据库信息被盗取导致信息泄露。

> 加密使用AES加密

## 使用说明

1. 在数据库对应的实体类只需要加密的字段上添加`@org.hzero.mybatis.annotation.DataSecurity`注解。
2. 在新增/更新数据之前，调用`org.hzero.mybatis.helper.DataSecurityHelper#open`方法开启数据加密。
3. 在查询数据之前，调用`org.hzero.mybatis.helper.DataSecurityHelper#open`方法开启加密。

## 配置说明

在`hzero.mybatis-mapper.data-security`前缀下有若干个配置选项：
  
* `default-open`：是否默认打开数据加解密，**默认关闭**，设置为`true`之后，不需要调用`DataSecurityHelper#open()`即可自动打开，**但是相应的，如果需要禁用数据加解密，则需要显式调用`DataSecurityHelper#close()`，并且会影响性能**。
  
* `isolation-level`：隔离级别，表示设置的启用/禁用（调用`DataSecurityHelper#open/close()`）影响的范围，默认`once`，表示一次查询或修改后失效(不包含`count`查询); `thread`表示当前线程内一直生效。
  
* `security-key`：自定义AES密钥
  
* `as-default-key`：如果配置了自定义密钥，是否将该密钥左右其他服务的默认密钥，默认`false`，也就是说只要有任一一个服务配置了自定义密钥，并且开启了该配置，其他服务在没有自定义密钥的情况下，会将当前服务的密钥作为默认密钥使用。

## 注意事项

* 默认情况下，使用字段加密功能需要显式的开启`DataSecurityHelper.open()`，开启方法只对当前线程有效【推荐】。如果想全局开启，可以通过配置`hzero.mybatis-mapper.data-security.default-open:true`进行处理，全局的配置会对应用性能有一定的影响，需谨慎使用。

* 在使用组件提供的`batchXxx`等批量操作的方法时，因为`DataSecurityHelper#open()`默认仅对一次操作有效，`batchXxx`操作本质也是循环多次操作，因为仅会对第一次操作有效。如需解决这个问题，一般有两种做法：1、建议不要使用batch方法，开发者自己循环调用单条操作方法，在调用时申明开启`DataSecurityHelper#open()`，这种显式控制便于管控【推荐】。2、通过调整默认隔离级别配置`hzero.mybatis-mapper.data-security.isolation-level:thread`进行处理，这个配置属于全局配置，对应用性能有一定影响，需谨慎使用。

* 加密字段如需要作为查询条件，需要进行加密后在数据库进行查询，并且不支持模糊查询。如使用`DataSecurityHelper.encrypt(param)`方法进行加密。

---

# 数据防篡改

# 数据防篡改

## 功能说明
* 数据从后端传输到前端之后，在进行更新操作时，经常需要对主键字段做校验，防止恶意篡改主键导致后端数据被破坏，数据防篡改功能将数据主键进行加密，进行数据更新时，对加密信息做校验用来验证主键信息有没有被篡改。**多语言支持也是基于数据防篡改实现的。**

## 使用说明
1. 如果是和数据库对应的实体类，只需要继承`io.choerodon.mybatis.domain.AuditDomain`即可，**如果是自定义的`VO/DTO`，也可以继承`AuditDomain`类，或者实现`org.hzero.mybatis.domian.SecurityToken`接口，接口中的`set_token`方法用于往`VO/DTO`中保存加密信息，`get_token`方法用于获取`VO/DTO`中保存的加密信息，`associateEntityClass`方法用于将`VO/DTO`与数据库对应的实体类关联在一起，需要注意在`VO/DTO`中必须和实体类主键属性**
2. 在更新数据前调用`org.hzero.mybatis.helper.SecurityTokenHelper#validToken(..)`方法校验主键有没有被篡改。

```java
class Entity extends io.choerodon.mybatis.domain.AuditDomain {
    @Id
    private long id;
    // other field ...
    // getter and setter ...
}

class EntityDTO implements org.hzero.mybatis.domian.SecurityToken {
    private long id;
    // other field ...
    // getter and setter ...
    private String _token;
    @Override
    public void set_token(String _token) {
        this._token = _token;
    }
    @Override
    public String get_token() {
        return this._token;
    }
    @Override
    public Class<? extends SecurityToken> associateEntityClass() {
        return Entity.class;
    }
}

// controller
// GET
public ResponseEntity<List<EntityDTO>> selectEntity(...){
    List<EntityDTO> list = // select ...
    return Results.success(list)
}
// PUT
public ResponseEntity<Entity> updateEntity(Entity entity){
    org.hzero.mybatis.helper.SecurityTokenHelper.validToken(entity);
    // update ...
}
```


---

# 多语言支持

# 多语言支持

## 功能说明
* 在业务处理中，经常会有有一些数据需要做多语言支持，根据用户选择的语言来动态切换显示内容。使用多语言组件时，提供的查询方法会自动join多语言表，不必开发人员再去手写SQL，自己创建的mapper接口需要自行join多语言表。

> 注意：多语言的主表命名的时候不能以`_b`结尾，`_b`结尾的表是H0平台保留的规范。

## 使用说明
1. 创建表时创建对应的多语言表，多语言表的表明需要在原表明的基础上增加`_tl`,**所以在设计多语言表的时候主表表名长度不要超过23，以免多语言表字段长度超出限制**，多语言表中需要包含对应表的主键，以及需要多语言的列，再加上`lang varchar(30)`字段。
2. 在数据库对应的多语言java实体类上继承`io.choerodon.mybatis.domain.AuditDomain`类，添加`@io.choerodon.mybatis.annotation.MultiLanguage`注解，在对应的多语言列上添加`@io.choerodon.mybatis.annotation.MultiLanguageField`注解
3. 新增/更新数据时，实体类json中需要添加多语言map，结构示例：

```java
{
    // other field ...
    _tls:{
    roleName : {
        zh_CN : '管理员',
        en_GB : 'Admin'
    },
    description : {
        zh_CN : '管理员',
        en_GB : 'administrator'
    }
}
}
```
* 如果因为一些特殊需求，需要在新增或者删除时关闭多语言支持，可以调用`org.hzero.mybatis.helper.MultiLanguageHelper#close`方法临时关闭多语言支持，**在一次mybatis操作之后，恢复启用状态，只在当前线程内生效**。
* 自行创建的mapper方法，在关联多语言时，可以在xml中添加`<bind name="lang" value="@io.choerodon.mybatis.helper.LanguageHelper@language()"/>`标签来获取当前用户的语言。

```sql
<select id="selectEntity" resultType="c.x.Entity">
    <bind name="lang" value="@io.choerodon.mybatis.helper.LanguageHelper@language()"/>
    SELECT
        t.id,
        ttl.multi_lang
        -- other column ..
    FROM table1 t
        JOIN table1_tl ttl ON t.id = ttl.id AND ttl.lang = #{lang}
    WHERE
        -- condition
</select>
```


---

# CRUD支持

# CRUD支持

## 新增支持

* `int insert(T record)` : 插入一条记录
* `int insertSelective(T record)` ： 插入一条记录，Bean中null的字段不会被插入
* `int insertOptional(T record)` : 插入一条记录，指定插入的列，插入前调用 `io.choerodon.mybatis.helper.OptionalHelper#optional(java.util.List<java.lang.String>)`方法
* `int insertList(List<T> recordList)` : 批量插入，如果主键名称不叫 `id`，需要再mapper中重新覆写该方法，在注解中声明主键的名称。

```java
@Options(useGeneratedKeys = true, keyProperty = "主键名称")
@InsertProvider(type = SpecialProvider.class, method = "dynamicSql")
int insertList(List<T> recordList);
```

## 更新支持

* `int updateByPrimaryKey(T record)` : 根据主键更新实体全部字段，null值会被更新
* `int updateByPrimaryKeySelective(T record)` ： 根据主键更新属性不为null的值
* `int updateOptional(T record)` : 更新一条记录，指定更新的列，更新前调用 `io.choerodon.mybatis.helper.OptionalHelper#optional(java.util.List<java.lang.String>)`方法

## 删除支持

* `int delete(T record)` : 根据实体属性作为条件进行删除，查询条件使用等号
* `int deleteByPrimaryKey(Object key)` ： 根据主键字段进行删除，方法参数必须包含完整的主键属性

> **注意：ORACLE数据下CLOB类型字段不能作为筛选条件，注意规避**

## 查询支持

* `List<T> select(T record)` : 根据实体中的属性值进行查询，查询条件使用等号
* `List<T> selectAll()` : 查询**全表结果**，慎用
* `T selectByPrimaryKey(Object key)` : 根据主键字段进行查询，方法参数必须包含完整的主键属性，查询条件使用等号
* `List<T> selectByIds(String ids)` : 根据主键字符串进行查询，类中只有存在一个带有 `@Id`注解的字段，多个主键使用 `,`分割
* `int selectCount(T record)` : 根据实体中的属性查询总数，查询条件使用等号
* `T selectOne(T record)` : 根据实体中的属性进行查询，只能有一个返回值，有多个结果是抛出异常，查询条件使用等号
* `List<T> selectByCondition(Object condition)` : 根据Condition条件进行查询
* `int selectCountByCondition(Object condition)` : 根据Condition条件进行查询总数

```java
mapper.selectByCondition(
    org.hzero.mybatis.domian.Condition.builder(Entity.class)
    .andWhere(
        org.hzero.mybatis.util.Sqls.custom()
            .andEqualTo(FIELD1, VALUE1)
            .andLike(FIELD2, VALUE2)
    ).build()
);
```

* `List<T> selectOptional(T condition, Criteria criteria)`：复杂查询，`condition`参数是查询条件，`criteria`可以指定查询的列以及一些其他的查询参数，以及排序等。该方法支持多表关联查询，使用关联查询时需要在与其他表的关联字段上添加 `@org.hzero.mybatis.common.query.JoinTable`注解，该注解指定关联的表以及关联方式和关联字段，然后需要在 `Entity`中新建关联表中的字段，并且添加 `@org.hzero.mybatis.common.query.JoinColumn`注解，**如果字段作为查询条件，必须添加 `@Where`注解**。

```java
// Entity
@Table(name = "hmsg_user_receive_config")
class UserReceiveConfig extends AuditDomain {
    // other field ...
    @Where
    private Long userId;
    @ApiModelProperty(value = "hmsg_receive_config .receiver_code", required = true)
    @JoinTable(name = "receiveConfigJoin", target = ReceiveConfig.class, on = @JoinOn(joinField = ReceiveConfig.FIELD_RECEIVE_CODE))
    private String receiveCode;
    @Transient
    @JoinColumn(joinName = "receiveConfigJoin", field = ReceiveConfig.FIELD_DEFAULT_RECEIVE_TYPE)
    private String defaultReceiveType;
    // getter and setter ...
}

// Entity
@Table(name = "hmsg_receive_config")
class ReceiveConfig extends AuditDomain {
    // other field ...
    private String receiveCode;
    private String defaultReceiveType;
    // getter and setter ...
}

// Repository
userReceiveConfigRepository
    .selectOptional(new UserReceiveConfig().setUserId(1L), 
            new Criteria().select("userReceiveId", "receiveCode", "receiveType", "defaultReceiveType", "userId"));
```

```sql
-- Result SQL
SELECT 
    A.object_version_number, 
    A.user_receive_id, 
    A.receive_code, 
    A.receive_type, 
    B.default_receive_type AS default_receive_type, 
    A.user_id 
FROM 
    hmsg_user_receive_config A 
    INNER JOIN hmsg_receive_config B 
        ON A.receive_code = B.receive_code 
WHERE 
    (A.user_id = ?)
```

### 分页查询

```java
import io.choerodon.mybatis.pagehelper.PageHelper;
import io.choerodon.mybatis.pagehelper.domain.PageRequest;

//分页并排序
Page<OrderUser> pageAndSort = PageHelper.doPageAndSort(pageRequest, () -> orderUserRepository.selectAll());
//仅分页
Page<OrderUser>  page= PageHelper.doPage(pageRequest, () -> orderUserRepository.selectAll());
//仅排序
List<OrderUser> sort = PageHelper.doSort(pageRequest.getSort(), () -> orderUserRepository.selectAll());
```


---

# 常用注解详解

# Mybatis增强的注解

## 用于多语言控制

### @MultiLanguage

**作用域：**类声明  

**用途：**多语言特性注解.

```java
@MultiLanguage
public class SecGrp extends AuditDomain {

}
```

注意：

1.创建表时创建对应的多语言表，多语言表的表明需要在原表明的基础上增加_tl

2.新增/更新数据时，实体类json中需要添加多语言map

```
{
    // other field ...
    _tls:{
    roleName : {
        zh_CN : '管理员',
        en_GB : 'Admin'
    },
    description : {
        zh_CN : '管理员',
        en_GB : 'administrator'
    }
}
}
```

### @MultiLanguageField

**作用域：**字段声明

**用途：**多语言字段注解，配合MultiLanguage使用

```java
@ApiModel("表单配置行")
@VersionAudit
@ModifyAudit
@MultiLanguage
@JsonInclude(value = JsonInclude.Include.NON_NULL)
@Table(name = "hpfm_form_line")
public class FormLine extends AuditDomain {
    @ApiModelProperty(value = "配置项说明")
    @Length(max = 480)
    @MultiLanguageField
    private String itemDescription;
}
```

## 用于控制

### @ModifyAudit

**作用域：**类声明

**用途：**加在类上代表开启审计监控

```java
@ModifyAudit
public class OrderUser extends AuditDomain {

}
```

### @VersionAudit

**作用域：**类声明  

**用途：**是否开启版本监控，加在类上表是开启版本控制可以配合@ModifyAudit使用

```java
@VersionAudit
@ModifyAudit
@Table(name = "hpfm_lov_view_header")
@JsonInclude(JsonInclude.Include.NON_NULL)
@ApiModel("值集视图头")
@MultiLanguage
public class LovViewHeader extends AuditDomain {

}
```

### @SnowflakeExclude

**作用域：**字段声明  

**用途：**加在字段上表示此字段不受雪花id算法影响

```java
@Id
@GeneratedValue
@SnowflakeExclude
@ApiModelProperty("租户ID")
private Long tenantId;
```

### @InterceptorOrder

**作用域：**类、接口（包括注释类型）、枚举、方法、字段声明  

**用途：**设置拦截器的顺序

```java
@InterceptorOrder(499)
public class DataChangeInterceptor implements Interceptor {

}
```

属性说明

```
value 拦截器的优先级 
Ordered.HIGHEST_PRECEDENCE(-2147483648)
Ordered.LOWEST_PRECEDENCE(2147483647)
有两种预制的优先级，数越小，优先级越高，默认为LOWEST_PRECEDENCE
```

### @CrossSchema

**作用域：**方法的声明  

**用途：**用于跨 Schema 查询多个租户的信息

属性说明

```
value 租户ID
```

### @TenantLimitedRequest

**作用域：**用于方法的声明

**用途：**租户限制的请求

属性说明

```
equal 是否使用等于条件 默认值为false 
true 为 TENANT_ID = ?
false 为 TENANT_ID IN (?)
```

## 用于数据控制

### @ColumnType

**作用域：**用于字段声明

**用途：**针对列的复杂属性配置

```java
@ColumnType(isBlob = true ,jdbcType = JdbcType.VARCHAR)
private String name;
```

属性说明

```text
isBlob 是否为 BLOB 字段 默认为false
JdbcType jdbc中的类型 默认为UNDEFINED
jdbcType 可选参数
		ARRAY(2003),
    BIT(-7),
    TINYINT(-6),
    SMALLINT(5),
    INTEGER(4),
    BIGINT(-5),
    FLOAT(6),
    REAL(7),
    DOUBLE(8),
    NUMERIC(2),
    DECIMAL(3),
    CHAR(1),
    VARCHAR(12),
    LONGVARCHAR(-1),
    DATE(91),
    TIME(92),
    TIMESTAMP(93),
    BINARY(-2),
    VARBINARY(-3),
    LONGVARBINARY(-4),
    NULL(0),
    OTHER(1111),
    BLOB(2004),
    CLOB(2005),
    BOOLEAN(16),
    CURSOR(-10),
    UNDEFINED(-2147482648),
    NVARCHAR(-9),
    NCHAR(-15),
    NCLOB(2011),
    STRUCT(2002),
    JAVA_OBJECT(2000),
    DISTINCT(2001),
    REF(2006),
    DATALINK(70),
    ROWID(-8),
    LONGNVARCHAR(-16),
    SQLXML(2009),
    DATETIMEOFFSET(-155),
    TIME_WITH_TIMEZONE(2013),
    TIMESTAMP_WITH_TIMEZONE(2014)
```

### @Unique

**作用域：**用于字段声明  

**用途：**唯一性校验

```java
@Unique
@Pattern(regexp = Regexs.CODE_UPPER)
private String configCode;
```

### @DataSecurity

**作用域：**用于字段声明  

**用途：**数据安全注解，使用该字段注解的字段会被插入到数据库之前加密，读取时解密，需要使用BaseMapper

```java
@DataSecurity
private String passwordEncrypted;
```

属性说明

```
always 添加注解的字段是否总是加密，默认开启，如果关闭，则跟随租户级配置
```

### @PageableDefault

**作用域：**用于行参声明 

**用途：**声明分页+排序的默认值

注意：如果加了排序需要使用PageHelper.doPageAndSort(PageRequest pageRequest, Select select)，如果仅仅只是分页用PageHelper.doPage(PageRequest pageRequest, Select select)

```java
public ResponseEntity<List<OrderUser>> export(@PathVariable Long organizationId, ExportParam exportParam, HttpServletResponse response,  @PageableDefault(size = 10,page = 0 , sort = "age" , direction = Sort.Direction.DESC) PageRequest pageRequest) {
    return Results.success(orderUserService.exportAll(pageRequest));
}
```

属性说明：

```text
value 分页条数 默认值10 与size相似，建议使用size

size 分页条数 默认值10

page 分页页数 默认值0

sort 排序字段 默认不开启排序

direction 排序顺序 默认ASC
```

### @SortDefault

**作用域：**用于行参声明  

**用途：**仅仅声明排序的默认值

注意：使用PageHelper.doPageAndSort(PageRequest pageRequest, Select select)调用

```java
public ResponseEntity<List<OrderUser>> export(@PathVariable Long organizationId, ExportParam exportParam, HttpServletResponse response,  @SortDefault(sort = "age" , direction = Sort.Direction.DESC) PageRequest pageRequest) {
    return Results.success(orderUserService.exportAll(pageRequest));
}
```

属性说明

```
value 分页条数 默认不开启排序 与sort相似

sort 排序字段 默认不开启排序

direction 排序顺序 默认ASC
```

## 用于sql编写

### @Where

**作用域：**用于字段声明  

**用途：**标明此字段是一个等值的查询条件

```java
@ApiModel("订单用户")
@VersionAudit
@ModifyAudit
@JsonInclude(value = JsonInclude.Include.NON_NULL)
@Table(name = "demo_order_user")
@ExcelSheet(zh = "订单用户")
public class OrderUser extends AuditDomain {

    @ApiModelProperty(" 用户ID，主键，与order_adress的userId关联")
    @Id
    @GeneratedValue
    private Long userId;

    @ApiModelProperty(value = "姓名", required = true)
    @NotNull
    @ExcelColumn(zh = "姓名")
    @Where
    private String name;

    @ApiModelProperty(value = "手机号")
    @ExcelColumn(zh = "手机号")
    private String phone;

    @ApiModelProperty(value = "年龄")
    @ExcelColumn(zh = "年龄")
    private int age;
    
    //get/set方法
    }
```
where的in和between的示例
需要声明一个List类型的字段，用于存储查询的字段值，并使用bindColumn绑定数据库的字段
注意：1、声明的字段因为非数据库字段需要使用@Transient声明 2、Between内只可以存放2个数据，否则会报错
```
    @ApiModelProperty(value = "年龄")
    @ExcelColumn(zh = "年龄")
    private Integer age;

    @Where(comparison = Comparison.IN,bindColumn = "age")
//    @Where(comparison = Comparison.BETWEEN,bindColumn = "age")
    @Transient
    private List<Integer> ages;
```

使用xxxRepository调用selectOptional(T condition, Criteria criteria)

condition为加入的条件参数，可new个对象放入进行传参

criteria为标准查询的字段，里面提供了select，unSelect，sort等多种方法，此处的select方法表示查询的参数



```java
@Override
public List<OrderUser> complexSelect() {
    OrderUser orderUser=new OrderUser();
    orderUser.setName("张三");
    return orderUserRepository.selectOptional(
            orderUser,
            new Criteria().select( "name", "phone", "age"));
}
```

等价于sql

```sql
select name,phone,age 
from demo_order_user 
where name="张三";
```

### @JoinTable @JoinOn @JoinColumn

**作用域：**三者皆用于字段声明  

@JoinTable作用：用于声明关联的表

属性说明

```
name 自定义的连接的表的别名
target 关联的表的实体
on 使用@JoinOn 声明此字段与关联表中的什么字段进行关联
```

@JoinOn作用：用于声明关联的字段

属性说明

```
joinField 关联的字段
joinExpression 关联的表达式
```

@JoinColumn 作用：用于声明从别的表中关联过来的字段

属性说明

```
joinName @JoinTable中自定义的关联表的名字，用于确定字段值来自于哪张表
field 声明从关联表中的何字段取值
expression 表达式 或者值用表达式取值
```

```java
@ApiModel("订单用户")
@VersionAudit
@ModifyAudit
@JsonInclude(value = JsonInclude.Include.NON_NULL)
@Table(name = "demo_order_user")
@ExcelSheet(zh = "订单用户")
public class OrderUser extends AuditDomain {

    @ApiModelProperty(" 用户ID，主键，与order_adress的userId关联")
    @Id
    @GeneratedValue
    @JoinTable(name = "demoAddress", target = UserAddress.class, on = @JoinOn(joinField = "userId"))
    private Long userId;

    @ApiModelProperty(value = "姓名", required = true)
    @NotNull
    @ExcelColumn(zh = "姓名")
    @Where
    private String name;

    @ApiModelProperty(value = "手机号")
    @ExcelColumn(zh = "手机号")
    private String phone;

    @ApiModelProperty(value = "年龄")
    @ExcelColumn(zh = "年龄")
    private int age;

    @ApiModelProperty(value = "地址")
    @Transient
    @JoinColumn(joinName = "demoAddress", field = "address")
    private String address;
 
  //get/set方法
}
```

```java
@ApiModel("用户地址")
@VersionAudit
@ModifyAudit
@JsonInclude(value = JsonInclude.Include.NON_NULL)
@Table(name = "demo_order_address")
public class UserAddress {

    @ApiModelProperty(" 地址ID，主键")
    @Id
    @GeneratedValue
    private Long addressId;

    @ApiModelProperty(value = "用户Id，与demo_order_user的userId关联", required = true)
    @NotNull
    private Long userId;

    @ApiModelProperty(value = "用户的地址", required = true)
    @NotNull
    private String address;

//get/set方法
}
```

```java
@Override
public List<OrderUser> complexSelect() {
    OrderUser orderUser=new OrderUser();
    orderUser.setName("张三");
    return orderUserRepository.selectOptional(
            orderUser,
            new Criteria().select( "name", "phone", "age","address"));
}
```

**等价于sql**

```sql
select A.name , A.phone , A.age , demoAddress.address
from demo_order_user A
inner join demo_order_address demoAddress on A.user_id = demoAddress.user_id
where A.name = "张三";
```

### @JoinTables

**作用域：**用于字段声明  

**用途：**使一个字段同时关联多个表

属性说明

```
value 放置多个@JoinTable的集合
```

```java
    @ApiModelProperty(" 用户ID，主键，与order_adress的userId关联")
    @Id
    @GeneratedValue
    @JoinTables(
            {
                    @JoinTable(name = "demoAddress", target = UserAddress.class, on = @JoinOn(joinField = "userId")),
                    @JoinTable(name = "demoHouse", target = UserHouse.class, on = @JoinOn(joinField = "userId"))
            }
            )
    private Long userId;
```

其他与@JoinTable，@JoinOn，@JoinColumn一致

**等价于sql**

```sql
select A.name , A.phone , A.age , demoAddress.address
from demo_order_user A
inner join demo_order_address demoAddress on A.user_id = demoAddress.user_id
inner join demo_user_house demoHouse on A.user_id = demoHouse.user_id
where A.name = "张三";
```

## 其他

### @EmbeddedField

**作用域：**用于字段声明  

**用途：**抽象不同实体类共同属性

```
  public class Customer extends AuditDomain {
    private Long id;
    @EmbeddedField
    private Address address;
 
    // 其它属性与方法略
  }
 
  public class Address {
    private String area;
    private String street;
    private String postCode;
  }
 
  相当于
 
  public class Customer extends AuditDomain {
    private Long id;
    private String area;
    private String street;
    private String postCode;
 
    // 其它属性与方法略
  }
```

---

# 自定义sql拦截

# 自定义sql拦截

## sql解析器

HZERO的sql解析借助了开源组件[JSqlParser](https://github.com/JSQLParser/JSqlParser)的能力，将SQL转换为Java类的可遍历层次结构。支持Oracle，SqlServer，MySQL，PostgreSQL等常用数据库。某些数据库方言无法解析，但不会影响sql执行。

> HZERO的某些功能借助了sql解析器的能力，如：数据权限、单据权限等，若sql无法被解析，则这些功能都会失效。建议开发人员在编写sql时，尽量避免sql方言的使用

## 自定义sql拦截器

平台对sql拦截器做了进一步的封装，Mybatis增强组件提供了接口`org.hzero.mybatis.parser.SqlInterceptor`，仅需继承该结构，重写需要的方法即可

例：

```
@Component
public class CustomSqlInterceptor implements SqlInterceptor {

    @Override
    public boolean select() {
        return true;
    }

    @Override
    public boolean insert() {
        return true;
    }

    @Override
    public boolean update() {
        return true;
    }
}
```

### 接口方法说明

SqlInterceptor 接口提供的方法都有默认实现，按需要重写即可
```
    /**
     * 在拦截之前执行
     */
    default void before() {
    }

    /**
     * 在拦截之后执行
     */
    default void after() {
    }

    /**
     * @return 是否拦截 SELECT 语句
     */
    default boolean select() {
        return false;
    }

    /**
     * @return 是否拦截 INSERT 语句
     */
    default boolean insert() {
        return false;
    }

    /**
     * @return 是否拦截 UPDATE 语句
     */
    default boolean update() {
        return false;
    }

    /**
     * @return 是否拦截 DELETE 语句
     */
    default boolean delete() {
        return false;
    }

    /**
     * @return 拦截器抛出异常时是否中断SQL调用
     */
    default boolean interrupted() {
        return false;
    }

    /**
     * @return SQL 解析拦截器顺序
     */
    default int order() {
        return 0;
    }

    /**
     * 改写SQL
     *
     * @param statement   statement
     * @param serviceName 服务名称
     * @param sqlId       MyBatis SQL ID
     * @param args        Sql执行参数
     * @param userDetails 用户信息
     * @return 改写后的对象
     */
    default Statement handleStatement(Statement statement, String serviceName, String sqlId, Map args, CustomUserDetails userDetails) {
        // 查询
        if (select() && statement instanceof Select) {
            statement = handleSelect((Select) statement, serviceName, sqlId, args, userDetails);
        } else if (insert() && statement instanceof Insert) {
            statement = handleInsert((Insert) statement, serviceName, sqlId, args, userDetails);
        } else if (update() && statement instanceof Update) {
            statement = handleUpdate((Update) statement, serviceName, sqlId, args, userDetails);
        } else if (delete() && statement instanceof Delete) {
            statement = handleDelete((Delete) statement, serviceName, sqlId, args, userDetails);
        }
        return statement;
    }
	
	...
```

---

# 覆盖Mapper

## 场景

在实际项目中，经常会遇到定制化二开标准功能的情况，比如标准功能的查询结果增加字段等，由于默认的Mybatis机制，不允许出现重复的SQL ID，没有同路径文件覆盖机制，因此平台 `hzero-starter-mybatis-mapper`组件在 `1.7.0+`版本后增加了拓展支持，允许使用者通过同路径mapper.xml文件的方式对某个SQL ID进行覆盖操作。

## 用法

* 添加应用配置
  在所属服务应用的 `application.yml`配置文件中增加如下配置：

  ```
  mybatis:
    configuration:
      enableMapperOverride: true
  ```
* 编写Mapper.xml文件进行覆盖

<p align="left"><img alt="" src="https://hsop-doc.su.bcebos.com/doc_classify/0/HSOP-BAIDU/89a83121076442c38c1c3ea60efea638@image.png" width="auto" height="auto"></p>

注意，如果使用了引用sql，需要将引用的sql也放在自定义mapper.xml中，否则会覆盖失败


---

# 字段自动填充

## 应用场景

在使用平台组件标准的新增、更新方法对开启了审计监控的类进行操作时，默认提供了创建人、创建时间、更新人、更新时间等字段的自动赋值。但在实际项目中，可能也会需要在插入或更新时对某些字段进行自动赋值的操作，如用户操作的数据自动赋值租户id为用户的所属id，`1.8.3+`版本后增加了拓展支持，允许使用者通过一定的配置使得在使用通用的新增、更新方法的情况下，能为某些字段自动填充值。

## 使用步骤

1. 在要应用自动填充功能的服务应用下的 `application.yml`配置文件中增加如下配置以开启功能：

   ```yml
   hzero:
     mybatis-mapper:
       fill:
         // 是否启用自动填充功能，默认不开启
         enable: true
   ```
2. 在要应用自动填充功能的实体字段下增加注解 ``@AutoFill``

<p align="left"><img alt="image-20220329153039745.png" src="https://hsop-image.su.bcebos.com/doc_classify/0/HSOP-BAIDU/c9e307b012524667819b02e385b3e800@image-20220329153039745.png" width="auto" height="auto"></p>

3. 实现接口 ``AutoFillValueProvider``，并重写其方法以修改自动填充的属性值（该接口的具体说明可参考下面的 ``AutoFillValueProvider接口``章节）

<p align="left"><img alt="image-20220329153058267.png" src="https://hsop-image.su.bcebos.com/doc_classify/0/HSOP-BAIDU/600927ee4444441f9b18b143dbe42542@image-20220329153058267.png" width="auto" height="auto"></p>

4. 使用平台组件的标准新增、更新接口对数据进行操作，便会自动填充指定的值到数据里。

## 组件说明

### @AutoFill注解

**作用域：**字段声明

**用途：**自动填充注解，使用该字段注解的字段才能设置自动填充的值

```java
@Target({ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
public @interface AutoFill {
    /**
     * 是否为原值优先
     * 为true的时候，只有当传入的数据为空的时候才会应用自动填充的数据
     */
    boolean propertyFirst() default false;
    /**
     * 更新时是否忽略该字段
     * 为true的时候使用默认方法更新时不会更新该字段
     */
    boolean ignoreUpdate() default false;
    /**
     * 新增时是否忽略该字段
     * 为true的时候使用默认方法新增时不会设置该字段值
     */
    boolean ignoreInsert() default false;
}
```

*两个ignore选项设置true的时候，不是代表其对应列最后的值为原值，而是在执行的sql中忽略掉该列*

### AutoFillValueProvider接口

```java
public interface AutoFillValueProvider {
    /**
     * 是否支持，默认支持所有实体类
     * @param metaObject 传入的参数（批量插入时这里传入的是列表的第一条数据）
     * @return 是否支持
     */
    default boolean isSupport(final MetaObject metaObject, SqlCommandType sqlCommandType) {
        return true;
    }

    /**
     * 处理新增时默认填充值的map，实现该方法以设置想要填充的数据值
     * key为要填充的字段属性，value为默认填充的值
     * value为空时，填充值也为空
     * 若有多个实现，冲突时将会按顺序靠后的值设置
     * @param metaObject 元对象 （批量插入时这里传入的是列表的第一条数据）
     * @param fillValueMap 默认填充值的map
     */
    void processFillValueMap(final MetaObject metaObject, Map<String, Object> fillValueMap);
}
```

### 相关配置

```
hzero:
  mybatis-mapper:
    fill:
      // 是否启用自动填充功能，默认不开启
      enable: false
      // 是否回写自动填充的值，默认开启，开启后会将原对象对应的值更改为自动填充的属性值
      write-back: true
```


---

# 覆盖默认分页SQL

## 场景说明

在我们编写SQL解决一些复杂查询时，由于默认的分页条数查询会沿用标准查询的SQL语句进行改写，可能会因为功能查询的复杂SQL语句在改写成COUNT查询时的性能损耗。

因此为了提升性能，我们而已针对分页使用的COUNT条数的SQL进行单独编写，使用与主SQL一致的查询条件和关键表，排除一些不需要用于汇总条数的其他关系表，通过精简SQL的方式提升性能。

## 简单示例

下面简单介绍一下使用示例：

1、功能查询的主SQL ID如：`selectUsers`

2、需要覆盖默认的分页COUNT查询的逻辑时，只需要编写一个对应的SQL语句即可，如上述SQL ID对应的分页SQL ID 为：`selectUsers_COUNT`


---

# 查询时报JSqlparser解析错误

### 问题描述

查询数据时报错Encountered unexpected token: "link" "LINK"

![](https://hsop-doc.su.bcebos.com/doc_classify/0/HSOP-BAIDU/7f497585242c4d9198ea78be6b1464e3@image.png)

报此错是因为sql中使用了JSqlparser的关键字，"LINK"仅为其中一个关键字，部分情况下是不影响使用的

### 解决方案

在执行sql前关闭解析，便可跳过该报错，调用方法如下：

org.hzero.mybatis.parser.SqlParserHelper#close


---

