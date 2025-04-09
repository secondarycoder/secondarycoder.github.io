# 辅助开发工具包中拼音处理工具类解析错误处理

### 问题描述

**H0提供的拼音处理工具包：**

org.hzero.core.util.PinyinUtils.*getPinyin()*

*使用时，发现汉字《* 余 *》解析出的结果是 * |tu|yu|。多了 tu 的读音。

### 解决方案

因为拼音工具类中使用的是第三方工具包，无法直接修复，但是可以通过配置纠错文件来解决这个问题

方案如下：

第一步：定义文件，可作为纠错的文件

<p align="left"><img alt="" src="https://hsop-doc.su.bcebos.com/doc_classify/0/HSOP-BAIDU/654139f1593a41c5a299aabe8cc77ff9@wecom-temp-433929-00c084fe3379af8700a6a19621e81337.png" width="auto" height="auto"></p>

第二步：在调翻译工具接口之前制定纠错文件
MultiPinyinConfig.multiPinyinPath = "src/main/resources/error_correction_multi_pinyin.txt";

第三步：测试效果

<p align="left"><img alt="" src="https://hsop-doc.su.bcebos.com/doc_classify/0/HSOP-BAIDU/7656f5368d67428faeed227948eb6307@wecom-temp-312100-16b1486d10a4230275c168d4eaa09312.png" width="auto" height="auto"></p>

方案原理：

<p align="left"><img alt="" src="https://hsop-doc.su.bcebos.com/doc_classify/0/HSOP-BAIDU/31bf5c91d179466195d7dd517cbecd4c@wecom-temp-110771-e77b34e30f94162d95ecf3c2adc38433.png" width="auto" height="auto"></p>


---

# 组件概述

> 组件编码 `hzero-starter-core`   

## 一、简介

## 1.1 概述

该组件主要针对技术开发中进行技术支持，提供一些通用的技术开发支持功能，减少重复造轮子。定义了基础实现类，异常封装，常用工具等。

## 1.2 组件坐标  

```xml
<dependency>
     <groupId>org.hzero.starter</groupId>
    <artifactId>hzero-starter-core</artifactId>
</dependency>
```



---

# 开发者指南


* 基础常量

* 基础类

* 异常处理类

* 响应结果处理

* 辅助类工具

* 后端多语言消息

* 通用线程池

* 动态职麦链

* 观察者事件总线

* 服务信息配置

* 加密解密工具

* 常用工具类

* 数据结构

* JSON相关




---

# 基础常量


## 概述

hzero-starter-core 基础包中提供了很多常用的常量类和常量定义，在开发中就无需重复定义常量。

常用的如用户、语言常量，符号常量，日期格式常量，错误编码常量，请求头常量等等。

## 常用常量

>注意：本节相关常量都定义在 **org.hzero.core.base.BaseConstants** 这个类下。

### 1、用户相关常量

```java
public interface BaseConstants {
    /**
     * 默认租户ID
     */
    Long DEFAULT_TENANT_ID = 0L;
    /**
     * 匿名用户ID
     */
    Long ANONYMOUS_USER_ID = 0L;
    /**
     * 匿名用户名
     */
    String ANONYMOUS_USER_NAME = "ANONYMOUS";
    /**
     * 默认用户类型：子账户默认有平台用户(P)和C端用户(C)两种类型
     */
    String DEFAULT_USER_TYPE = "P";
}
```

### 2、语言时区相关常量

```java
public interface BaseConstants {
    /**
     * 默认语言
     */
    Locale DEFAULT_LOCALE = Locale.CHINA;
    /**
     * 默认语言
     */
    String DEFAULT_LOCALE_STR = Locale.CHINA.toString();
    /**
     * 默认编码
     */
    String DEFAULT_CHARSET = "UTF-8";
    /**
     * 默认国际管码
     */
    String DEFAULT_CROWN_CODE = "+86";
    /**
     * 默认时区
     */
    String DEFAULT_TIME_ZONE = "GMT+8";
}
```

### 3、分页相关常量

```java
public interface BaseConstants {
    /**
     * 默认页码
     */
    String PAGE = "0";
    /**
     * 默认页面大小
     */
    String SIZE = "10";
    /**
     * 默认页码
     */
    int PAGE_NUM = 0;
    /**
     * 默认页面大小
     */
    int PAGE_SIZE = 10;
    /**
     * 默认页码字段名
     */
    String PAGE_FIELD_NAME = "page";
    /**
     * 默认页面大小字段名
     */
    String SIZE_FIELD_NAME = "size";
}
```

### 4、字段常量

```java
public interface BaseConstants {
    /**
     * body
     */
    String FIELD_BODY = "body";
    /**
     * KEY content
     */
    String FIELD_CONTENT = "content";
    /**
     * KEY message
     */
    String FIELD_MSG = "message";
    /**
     * KEY failed
     */
    String FIELD_FAILED = "failed";
    /**
     * KEY success
     */
    String FIELD_SUCCESS = "success";
    /**
     * KEY errorMsg
     */
    String FIELD_ERROR_MSG = "errorMsg";
}
```

**场景1**：可以在 @ProcessLovValue 注解是使用 FIELD_BODY 常量

```
@ProcessLovValue(targetField = BaseConstants.FIELD_BODY)
public ResponseEntity<Page<DocType>> queryDocTypeByTenant() {
 
}
```

**场景2**：封装返回结果时，可以使用 FIELD_SUCCESS、FIELD_ERROR_MSG 等常量。

```
HashMap<String, String> result = new HashMap<>();
result.put(BaseConstants.FIELD_SUCCESS, "success");
result.put(BaseConstants.FIELD_ERROR_MSG, "error_message");
```


### 5、其它常量

```java
public interface BaseConstants {
    /**
     * -1
     */
    int NEGATIVE_ONE = -1;
    /**
     * 默认环境
     */
    String DEFAULT_ENV = "dev";

    /**
     * ObjectMapper
     */
    ObjectMapper MAPPER = new ObjectMapper();

    /**
     * 路径分隔符：|
     */
    String PATH_SEPARATOR = Symbol.VERTICAL_BAR;
}
```

**场景1**：使用 ObjectMapper 序列化或反序列化

```java
public void demo() {
    User user = new User();
    // 序列化
    String json = null;
    try {
        json = BaseConstants.MAPPER.writeValueAsString(user);
        // 反序列化
        user = BaseConstants.MAPPER.readValue(json, User.class);
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

**场景2**：路径分隔符

在一些树形结构的数据中，我们一般可以添加一个 level_path 的字段来展示树形的层级关系，这时可以使用 `BaseConstants.PATH_SEPARATOR` 作为路径分隔符。

例如菜单的路径：

```txt
hzero.dev
hzero.dev|hzero.dev.auth.ca
hzero.dev|hzero.dev.auth.ca|hzero.dev.auth.ca.ps.default
```

## 格式常量

> 注意：本节的常量都在 **org.hzero.core.base.BaseConstants.Pattern** 这个类下。

### 1、常规日期时间格式

```java
public interface BaseConstants {

	interface Pattern {
        /**
         * yyyy-MM-dd
         */
        String DATE = "yyyy-MM-dd";
        /**
         * yyyy-MM-dd HH:mm:ss
         */
        String DATETIME = "yyyy-MM-dd HH:mm:ss";
        /**
         * yyyy-MM-dd HH:mm
         */
        String DATETIME_MM = "yyyy-MM-dd HH:mm";
        /**
         * yyyy-MM-dd HH:mm:ss.SSS
         */
        String DATETIME_SSS = "yyyy-MM-dd HH:mm:ss.SSS";
        /**
         * HH:mm
         */
        String TIME = "HH:mm";
        /**
         * HH:mm:ss
         */
        String TIME_SS = "HH:mm:ss";
    }
}
```

**场景1**：格式化时间和解析字符串

```java
// SimpleDateFormat 是非线程安全的，可以定义在 ThreadLocal 中
final ThreadLocal<SimpleDateFormat> local = ThreadLocal.withInitial(
        () -> new SimpleDateFormat(BaseConstants.Pattern.DATETIME));

public void demo() throws ParseException {
    Date date = new Date();
    // Date 格式化成 字符串
    String dateStr = local.get().format(date);
    // 字符串格式化成 Date
    date = local.get().parse(dateStr);
}
```

### 2、系统时间格式

```java
public interface BaseConstants {

	interface Pattern {
        /**
         * yyyy/MM/dd
         */
        String SYS_DATE = "yyyy/MM/dd";
        /**
         * yyyy/MM/dd HH:mm:ss
         */
        String SYS_DATETIME = "yyyy/MM/dd HH:mm:ss";
        /**
         * yyyy/MM/dd HH:mm
         */
        String SYS_DATETIME_MM = "yyyy/MM/dd HH:mm";
        /**
         * yyyy/MM/dd HH:mm:ss.SSS
         */
        String SYS_DATETIME_SSS = "yyyy/MM/dd HH:mm:ss.SSS";
    }
}
```

### 3、无连接符格式

```java
public interface BaseConstants {

	interface Pattern {
        /**
         * yyyyMMdd
         */
        String NONE_DATE = "yyyyMMdd";
        /**
         * yyyyMMddHHmmss
         */
        String NONE_DATETIME = "yyyyMMddHHmmss";
        /**
         * yyyyMMddHHmm
         */
        String NONE_DATETIME_MM = "yyyyMMddHHmm";
        /**
         * yyyyMMddHHmmssSSS
         */
        String NONE_DATETIME_SSS = "yyyyMMddHHmmssSSS";

        /**
         * EEE MMM dd HH:mm:ss 'CST' yyyy
         */
        String CST_DATETIME = "EEE MMM dd HH:mm:ss 'CST' yyyy";
    }
}
```

### 4、数字格式

```java
public interface BaseConstants {

	interface Pattern {
        /**
         * 无小数位 0
         */
        String NONE_DECIMAL = "0";
        /**
         * 一位小数 0.0
         */
        String ONE_DECIMAL = "0.0";
        /**
         * 两位小数 0.00
         */
        String TWO_DECIMAL = "0.00";
        /**
         * 千分位表示 无小数 #,##0
         */
        String TB_NONE_DECIMAL = "#,##0";
        /**
         * 千分位表示 一位小数 #,##0.0
         */
        String TB_ONE_DECIMAL = "#,##0.0";
        /**
         * 千分位表示 两位小数 #,##0.00
         */
        String TB_TWO_DECIMAL = "#,##0.00";
    }
}
```

**场景1**：格式化小数位

```java
public void demo() {
    // 格式化保留两位小数
    DecimalFormat format = new DecimalFormat(BaseConstants.Pattern.TWO_DECIMAL);

    BigDecimal num = new BigDecimal("10.0001");
    String numStr = format.format(num); // 10.00
}
```

## 是/否标识符

> 注意：本节的常量都在 **org.hzero.core.base.BaseConstants.Flag** 这个类下。

业务中经常会用到 是/否、启用/禁用 这样的状态字段，一般我们会使用 1 表示 是，0 表示 否。Flag 常量就提供了这两个常量。

```java
public interface BaseConstants {

    interface Flag {
        /**
         * 1
         */
        Integer YES = 1;
        /**
         * 0
         */
        Integer NO = 0;
    }
}
```

## 数字常量

> 注意：本节的常量都在 **org.hzero.core.base.BaseConstants.Digital** 这个类下。


```java
public interface BaseConstants {

    interface Digital {
        int NEGATIVE_ONE = -1;
        int ZERO = 0;
        int ONE = 1;
        int TWO = 2;
        int FOUR = 4;
        int EIGHT = 8;
        int SIXTEEN = 16;
    }
}
```

## 常用错误编码

> 注意：本节的常量都在 **org.hzero.core.base.BaseConstants.ErrorCode** 这个类下。

```java

public interface BaseConstants {

    interface ErrorCode {
        /**
         * 数据校验不通过
         */
        String DATA_INVALID = "error.data_invalid";
        /**
         * 资源不存在
         */
        String NOT_FOUND = "error.not_found";
        /**
         * 程序出现错误，请联系管理员
         */
        String ERROR = "error.error";
        /**
         * 网络异常，请稍后重试
         */
        String ERROR_NET = "error.network";
        /**
         * 记录不存在或版本不一致
         */
        String OPTIMISTIC_LOCK = "error.optimistic_lock";
        /**
         * 数据已存在，请不要重复提交
         */
        String DATA_EXISTS = "error.data_exists";
        /**
         * 数据不存在
         */
        String DATA_NOT_EXISTS = "error.data_not_exists";
        /**
         * 资源禁止访问
         */
        String FORBIDDEN = "error.forbidden";
        /**
         * 数据库异常：编码重复
         */
        String ERROR_CODE_REPEAT = "error.code_repeat";
        /**
         * 数据库异常：编号重复
         */
        String ERROR_NUMBER_REPEAT = "error.number_repeat";
        /**
         * SQL执行异常
         */
        String ERROR_SQL_EXCEPTION = "error.sql_exception";
        /**
         * 请登录后再进行操作！
         */
        String NOT_LOGIN = "error.not_login";
        /**
         * 不能为空
         */
        String NOT_NULL = "error.not_null";
        /**
         * 响应超时
         */
        String TIMEOUT = "error.timeout";
        /**
         * 服务器繁忙，请稍后重试
         */
        String SERVER_BUSY = "error.serverBusy";
    }
}
```

**场景1**：系统中对数据校验或一些异常时，指定异常编码

```java
public void demo(User params)  {
    // 校验数据异常
    Assert.notNull(params, BaseConstants.ErrorCode.DATA_INVALID);
    // 数据不存在异常
    Assert.notNull(params, BaseConstants.ErrorCode.DATA_NOT_EXISTS);
    // 编码重复异常
    if ("admin".equals(params.getLoginName())) {
        throw new CommonException(BaseConstants.ErrorCode.ERROR_NUMBER_REPEAT);
    }
}
```

## 符号常量

> 注意：本节的常量都在 **org.hzero.core.base.BaseConstants.Symbol** 这个类下。

```java
public interface BaseConstants {

    interface Symbol {
        /**
         * 感叹号：!
         */
        String SIGH = "!";
        /**
         * 符号：@
         */
        String AT = "@";
        /**
         * 井号：#
         */
        String WELL = "#";
        /**
         * 美元符：$
         */
        String DOLLAR = "$";
        /**
         * 人民币符号：￥
         */
        String RMB = "￥";
        /**
         * 空格：
         */
        String SPACE = " ";
        /**
         * 换行符：\r\n
         */
        String LB = System.getProperty("line.separator");
        /**
         * 百分号：%
         */
        String PERCENTAGE = "%";
        /**
         * 符号：&amp;
         */
        String AND = "&";
        /**
         * 星号
         */
        String STAR = "*";
        /**
         * 中横线：-
         */
        String MIDDLE_LINE = "-";
        /**
         * 下划线：_
         */
        String LOWER_LINE = "_";
        /**
         * 等号：=
         */
        String EQUAL = "=";
        /**
         * 加号：+
         */
        String PLUS = "+";
        /**
         * 冒号：:
         */
        String COLON = ":";
        /**
         * 分号：;
         */
        String SEMICOLON = ";";
        /**
         * 逗号：,
         */
        String COMMA = ",";
        /**
         * 点号：.
         */
        String POINT = ".";
        /**
         * 斜杠：/
         */
        String SLASH = "/";
        /**
         * 竖杠：|
         */
        String VERTICAL_BAR = "|";
        /**
         * 双斜杠：//
         */
        String DOUBLE_SLASH = "//";
        /**
         * 反斜杠
         */
        String BACKSLASH = "\\";
        /**
         * 问号：?
         */
        String QUESTION = "?";
        /**
         * 左花括号：{
         */
        String LEFT_BIG_BRACE = "{";
        /**
         * 右花括号：}
         */
        String RIGHT_BIG_BRACE = "}";
        /**
         * 左中括号：[
         */
        String LEFT_MIDDLE_BRACE = "[";
        /**
         * 右中括号：[
         */
        String RIGHT_MIDDLE_BRACE = "]";
        /**
         * 反引号：`
         */
        String BACKQUOTE = "`";
    }
}
```

**场景1**：开发中需要一些特殊符号时

```JAVA
public void demo(User params)  {
    String key = "hiam:rule" + BaseConstants.Symbol.AT + "user";
    String money = "10.00" + BaseConstants.Symbol.RMB;
}
```

## 请求头常量

> 注意：本节的常量都在 **org.hzero.core.base.BaseHeaders** 这个类下。

以 `H-` 开头的请求头为 HZERO 自定义的请求头，其它的为标准请求头。

```java
public class BaseHeaders {
    /**
     * 资源根路径
     */
    public static final String H_ROOT_PATH = "H-Root-Path";
    /**
     * 请求来源，在 gateway 固定的源
     */
    public static final String H_REQUEST_FROM = "H-Request-From";
    /**
     * 请求头：菜单ID
     */
    public static final String H_MENU_ID = "H-Menu-Id";
    /**
     * 请求头：请求ID
     */
    public static final String H_REQUEST_ID = "H-Request-Id";
    /**
     * 请求头：租户ID
     */
    public static final String H_TENANT_ID = "H-Tenant-Id";

    public static final String X_FORWARDED_FOR = "X-Forwarded-For";

    public static final String X_REAL_IP = "X-Real-IP";

    public static final String COOKIE = "Cookie";
    
    public static final String AUTHENTICATION = "Authentication";
}
```

## Token相关常量

> 注意：本节的常量都在 **org.hzero.core.base.TokenConstants** 这个类下。

```java
public final class TokenConstants {
    public static final String HEADER_JWT = "Jwt_Token";
    public static final String HEADER_BEARER = "Bearer";
    public static final String HEADER_AUTH = getAuthHeaderName();
    public static final String ACCESS_TOKEN = "access_token";
    public static final String JWT_TOKEN = "jwt_token";
}
```

## 角色编码常量

HZERO定义了几个基础的角色，角色的编码是固定的，如平台超级管理员、租户超级管理员、游客角色等，如有需要，可以使用这些常量。

> 位置：org.hzero.common.HZeroConstant.RoleCode

```
public class HZeroConstant {
    /**
     * 角色
     */
    public static class RoleCode {
        /**
         * 全局层角色编码
         */
        public static final String SITE = "role/site/default/administrator";
        /**
         * 租户层角色编码
         */
        public static final String TENANT = "role/organization/default/administrator";
        /**
         * 小项目层角色编码
         */
        public static final String PROJECT = "role/project/default/administrator";
        /**
         * 访客角色编码
         */
        public static final String GUEST = "role/site/default/guest";

        /**
         * 租户层角色模板
         */
        public static final String TENANT_TEMPLATE = "role/organization/default/template/administrator";
    }
}
```
















---

# 常用基础类


## 概述

hzero-starter-core 包和 hzero-starter-mybatis-helper 包下提供了编写DDD各层代码的基础类或接口，为了完整性，就放到一块进行说明。

通常我们开发的代码是DDD的四层结构，关于DDD的代码结构请参考：[应用分层](https://open.hand-china.com/document-center/doc/product/10137/10227?doc_id=32614)

这些基础类提供了一些通用的方法，可以提高我们开发的速度。通常我们在开发相关组件时需要实现或继承这些基础接口，例如 BaseController、BaseRepository 等。

## API接口 BaseController

在编写 Controller 接口时，需要继承 `org.hzero.core.base.BaseController` 类。

BaseController 提供了常用的参数校验方法，用于对参数对象中标记了 @NotNull、@Size、@Length 等校验注解的字段进行基础校验。

```java
/**
 * 验证单个对象
 *
 * @param object 验证对象
 * @param groups 验证分组
 * @param <T>    Bean 泛型
 */
<T> void validObject(T object, Class... groups)

/**
 * 验证单个对象
 *
 * @param object  验证对象
 * @param groups  验证分组
 * @param process 异常信息处理
 * @param <T>     Bean 泛型
 */
<T> void validObject(T object, ValidUtils.ValidationResult process, Class... groups)

/**
 * 迭代验证集合元素，使用默认异常信息处理
 *
 * @param collection Bean集合
 * @param groups     验证组
 * @param <T>        Bean 泛型
 */
<T> void validList(Collection<T> collection, Class... groups)

/**
 * 迭代验证集合元素，使用默认异常信息处理
 *
 * @param collection Bean集合
 * @param groups     验证组
 * @param process    异常信息处理
 * @param <T>        Bean 泛型
 */
<T> void validList(Collection<T> collection, ValidUtils.ValidationResult process, Class... groups)
```

**场景1**：校验对象

```java
@RestController
@RequestMapping(value = "/v1/clients")
public class ClientSiteController extends BaseController {

    @Permission(level = ResourceLevel.SITE)
    @ApiOperation(value = "创建客户端")
    @PostMapping
    public ResponseEntity<Client> create(@RequestBody Client client) {
        // 数据对象
        super.validObject(client);
        return Results.success();
    }
}
```

**场景2**：分组校验

针对一个实体中的字段，可能在不同场景下，需要分开校验，这时可以使用分组校验功能。

1、首先在实体中定义分组的接口，可根据业务场景定义，例如 Group1、Group2。
2、然后在校验注解中配置 **groups** 参数，指定不同的分组。

```java
public class Client extends AuditDomain {
    @Id
    @GeneratedValue
    private Long id;
    @NotNull(groups = {Group1.class, Group2.class})
    private String name;
    @NotNull
    private Long organizationId;
    @NotNull(groups = Group1.class)
    private String resourceIds;
    @NotNull(groups = Group2.class)
    private String secret;

    public interface Group1 {
    }

    public interface Group2 {
    }
}
```

3、在 validObject、validList 校验注解的第二个参数指定校验的分组。

```java
@RequestMapping(value = "/v1/clients")
public class ClientSiteController extends BaseController {

    @PostMapping("/group1")
    public ResponseEntity<Client> group1(@RequestBody Client client) {
        // 校验对象中字段中有 Group1 的校验注解
        super.validObject(client, Client.Group1.class);
        return Results.success();
    }

    @PostMapping("/group2")
    public ResponseEntity<Client> group2(@RequestBody Client client) {
        // 校验对象中字段中有 Group2 的校验注解
        super.validObject(client, Client.Group2.class);
        return Results.success();
    }

    @PostMapping("/all")
    public ResponseEntity<Client> all(@RequestBody Client client) {
        // 不指定校验分组，就校验所有字段
        super.validObject(client);
        return Results.success();
    }
}
```

## 应用服务 BaseService 

在编写应用成Service类时，可以实现 `io.choerodon.mybatis.service.BaseService` 接口，并继承实现类 `io.choerodon.mybatis.service.BaseServiceImpl`。

但对于应用层Service来说，并非必须要实现 BaseService 接口。BaseService 提供了一些基础的数据增删改查操作，其基于 BaseMapper 实现，所以这些功能一般也可以用 Repository 来实现。

```java
public interface BaseService<T> {

    // 查询一个
    T selectOne(T record);
    // 根据主键查询
    T selectByPrimaryKey(Object key);
    // 查询集合
    List<T> select(T record);
    // 查询所有
    List<T> selectAll();
    // 分页查询
    Page<T> pageAll(int page, int size);
    // 根据条件分页查询
    Page<T> page(T record, int page, int size);
    // 查询数量
    int selectCount(T record);
    // 判断是否存在主键相同的记录
    boolean existsWithPrimaryKey(Object key);

    // 插入数据
    int insert(T record);
    // 插入数据（排除非空字段）
    int insertSelective(T record);
    // 插入部分字段
    int insertOptional(T record, String... optionals);

    // 删除数据
    int delete(T record);
    // 根据主键删除
    int deleteByPrimaryKey(Object key);

    // 根据主键更新（会更新值为NULL的字段）
    int updateByPrimaryKey(T record);
    // 根据主键更新（不会更新值为NULL的字段）
    int updateByPrimaryKeySelective(T record);
    // 更新部分字段
    int updateOptional(T record, String... optionals);
}
```

**场景1**：查询数量，插入数据，根据指定字段更新数据

```java
@Service
public class ClientServiceImpl extends BaseServiceImpl<Client> 
				implements ClientService, BaseService<Client> {

    @Override
    public Client create(Client client) {
        Client param = new Client();
        param.setName(client.getName());
        // 根据名称查询数量
        int count = super.selectCount(param);
        Assert.isTrue(count == 0, BaseConstants.ErrorCode.DATA_EXISTS);

        // 插入数据
        super.insert(client);

        return client;
    }

    @Override
    public void update(Client client) {
        boolean exists = super.existsWithPrimaryKey(client);
        Assert.isTrue(!exists, BaseConstants.ErrorCode.DATA_NOT_EXISTS);

        // 更新指定字段
        super.updateOptional(client,
                Client.FIELD_RESOURCE_IDS,
                Client.FIELD_SECRET,
                Client.FIELD_SCOPE,
                Client.FIELD_WEB_SERVER_REDIRECT_URI
        );
    }
}
```

## 资源库 BaseRepository

编写资源库Repository时，可以实现 `org.hzero.mybatis.base.BaseRepository` 接口，继承 `org.hzero.mybatis.base.impl.BaseRepositoryImpl` 实现类。

BaseRepository 提供了比 BaseService 更丰富的增删改查操作，一般来说我们应该使用 Repository 来增删改查数据。

```java
public interface BaseRepository<T> extends BaseService<T> {

    /**
     * 根据key和value查询
     *
     * @param key   字段名称
     * @param value 值
     * @return 查询结果
     */
    List<T> select(String key, Object value);
    /**
     * 根据主键字符串进行查询，类中只有存在一个带有@Id注解的字段
     *
     * @param ids 如 "1,2,3,4"
     * @return 返回列表
     */
    List<T> selectByIds(String ids);
    /**
     * 根据Condition条件进行查询
     *
     * @param condition 查询条件
     * @return 返回列表
     */
    List<T> selectByCondition(Condition condition);
    /**
     * 根据Condition条件进行查询总数
     *
     * @param condition 查询条件
     * @return 数量
     */
    int selectCountByCondition(Condition condition);
    /**
     * 根据指定的字段查询
     *
     * @param condition 查询条件
     * @param criteria  查询条件
     * @return 查询结果
     */
    T selectOneOptional(T condition, Criteria criteria);
    /**
     * 根据指定的字段查询
     *
     * @param condition 查询条件
     * @param criteria  查询条件
     * @return 查询结果
     */
    List<T> selectOptional(T condition, Criteria criteria);
    /**
     * 分页排序查询
     *
     * @param pageRequest 分页排序条件
     * @param queryParam  查询条件
     * @return
     */
    Page<T> pageAndSort(PageRequest pageRequest, T queryParam);
    /**
     * 分页排序查询
     *
     * @param pageRequest 分页排序条件
     * @param key         field名
     * @param value       值
     * @return 查询结果
     */
    Page<T> pageAndSort(PageRequest pageRequest, String key, Object value);


    /**
     * 按照主键更新记录，更新所有字段
     *
     * @param record 更新的记录内容
     * @return 更新记录数量，必定大于1
     * @throws OptimisticLockException <strong>当更新记录数量为0并且记录继承自AuditDomain时</strong>，抛出异常记录不存在或者版本不一致
     */
    @Override
    int updateByPrimaryKey(T record);
    /**
     * 按照主键更新记录，更新非null字段
     *
     * @param record 更新的记录内容
     * @return 更新记录数量，必定大于1
     * @throws OptimisticLockException <strong>当更新记录数量为0并且记录继承自AuditDomain时</strong>，抛出异常记录不存在或者版本不一致
     */
    @Override
    int updateByPrimaryKeySelective(T record);
    /**
     * 批量选择更新
     *
     * @param list      准备更新的list
     * @param optionals 更新字段
     * @return 更新之后的list
     */
    List<T> batchUpdateOptional(List<T> list, String... optionals);
    /**
     * 按主键批量更新
     *
     * @param list 准备更新的list
     * @return 更新之后的list
     */
    List<T> batchUpdateByPrimaryKey(List<T> list);
    /**
     * 按主键选择性批量更新
     *
     * @param list 准备更新的list
     * @return 更新之后的list
     */
    List<T> batchUpdateByPrimaryKeySelective(List<T> list);


    /**
     * 批量插入
     *
     * @param list 准备插入的list
     * @return 插入之后的list
     */
    List<T> batchInsert(List<T> list);
    /**
     * 批量插入
     *
     * @param list 准备插入的list
     * @return 插入之后的list
     */
    List<T> batchInsertSelective(List<T> list);
    /**
     * 批量保存，根据数据的 __status 判断 新增、删除、修改
     *
     * @param list 准备保存的list
     * @return 保存数量
     */
    int batchSave(List<T> list);
    /**
     * 批量保存，根据数据的 __status 判断 新增、删除、修改
     *
     * @param list 准备保存的list
     * @return 保存数量
     */
    int batchSaveSelective(List<T> list);


    /**
     * 批量删除
     *
     * @param list 准备删除的list
     * @return 删除数量
     */
    int batchDelete(List<T> list);
    /**
     * 按主键批量删除
     *
     * @param list 准备删除的list
     * @return 删除数量
     */
    int batchDeleteByPrimaryKey(List<T> list);
    /**
     * 根据指定的字段查询
     *
     * @param condition 查询条件
     * @param criteria  查询条件
     * @return 更新记录数
     */
    int deleteOptional(T condition, Criteria criteria);
}
```

**场景1**：注入应用服务

1、创建Repository

```java
// 资源库接口
public interface ClientRepository extends BaseRepository<Client> {
}

// 资源库实现类
@Component
public class ClientRepositoryImpl extends BaseRepositoryImpl<Client> implements ClientRepository {

}
```

2、在应用服务中注入 Repository，调用 Repository 完成数据操作

```java
@Service
public class ClientServiceImpl implements ClientService {

    @Autowired
    private ClientRepository clientRepository;

    @Override
    public Client create(Client client) {
        // 根据条件查询数量
        int count = clientRepository.selectCountByCondition(Condition.builder(Client.class)
                .andWhere(Sqls.custom().andEqualTo(Client.FIELD_NAME, client.getName()))
                .build());
        Assert.isTrue(count == 0, BaseConstants.ErrorCode.DATA_EXISTS);

        // 插入数据
        clientRepository.insertSelective(client);
        return client;
    }

}
```

**场景2**：设置数据状态，批量保存

```java
@Service
public class ClientServiceImpl implements ClientService {

    @Autowired
    private ClientRepository clientRepository;

    public List<Client> batchSave() {
        Client c1 = new Client();
        c1.set_status(AuditDomain.RecordStatus.create); // 创建

        Client c2 = new Client();
        c2.set_status(AuditDomain.RecordStatus.update); // 更新

        Client c3 = new Client();
        c3.set_status(AuditDomain.RecordStatus.delete); // 删除

        List<Client> clients = Lists.newArrayList(c1, c2, c3);
        // 根据状态批量保存
        clientRepository.batchSave(clients);

        return clients;
    }
}
```

## 数据库映射 BaseMapper

最后一步，需要编写底层操作数据库的 Mapper，需要开发一个接口继承 `io.choerodon.mybatis.common.BaseMapper`。

BaseMapper 也提供了基础的增删改查操作，BaseRepository 也是基于 BaseMapper 实现的。自定义 Mapper 一般会对应一个 Mapper.xml 文件，一般是为了开发业务相关的SQL。如果要使用基础的增删改查，可直接使用 BaseRepository 即可。

```java
public interface ClientMapper extends BaseMapper<Client> {

}
```

## 获取接口代理 AopProxy

`AopProxy` 接口提供的 self() 方法能够获取接口自身的代理对象，常用在一个事务方法里调用当前类的其它事务方法，如果不使用代理对象调用方法，本质使用的是原始对象（this），因而可能导致事务或AOP拦截不生效。

> 注意：请明确是否有必要使用 self() 来调用当前对象的其它方法，self() 调用时实际上是两个分开的事务，可能无法一起回滚。

```java
@Service
public class UserServiceImpl implements UserService, AopProxy<UserServiceImpl> {

    @Override
    @Transactional(rollbackFor = Exception.class)
    public User createUser(User user) {
        userCreateService.createUser(user);
        // self() 返回当前代理对象，createOtherInfo 会单独开一个事务，被AOP拦截处理
        self().createOtherInfo(user);
        // this 是当前对象
        //this.createOtherInfo(user);
        return user;
    }

    @Override
    @Transactional(rollbackFor = Exception.class)
    public void createOtherInfo(User user) {
        // 其它数据库操作
    }
}
```



---

# 异常处理类


## 概述

`io.choerodon.core.exception` 和 `org.hzero.core.exception` 两个包下提供了一些常用的常量类封装，以及通用的异常拦截处理器。

## 异常类

**1、基础异常**

* BaseException：基础异常抽象类，在标准异常上增加了 code、parameters 等参数。自定义受检异常可以继承 BaseException。

* BaseRuntimeException：基础运行时异常。自定义运行时异常类可以继承 BaseRuntimeException。

* ExceptionResponse：异常信息封装对象，如果要返回详细的异常信息，可以自己将异常信息封装到 ExceptionResponse 中。

**2、常用异常**

* CommonException：通用业务异常，最常用的一个业务异常类

* NotFoundException：资源不存在异常

* AlreadyExistedException：对象已存在异常

* EmptyParamException：空参数异常

* IllegalArgumentException：非法参数异常

* NotExistedException：数据不存在异常

* CheckedException：通用受检异常

* IllegalOperationException：非法操作异常

* NotLoginException：未登录异常

## 异常拦截处理器

### 1、基础异常处理器

`BaseExceptionHandler` 是一个全局的统一异常拦截处理器，避免直接将应用中的异常栈返回给用户。

例如：拦截了 RuntimeException、Exception，并返回 “程序出现错误，请联系管理员”。

```java
@ControllerAdvice
public class BaseExceptionHandler {
    private static final Logger LOGGER = LoggerFactory.getLogger(BaseExceptionHandler.class);

    @Value("${spring.profiles.active:" + DEFAULT_ENV + "}")
    private String env;

    /**
     * 拦截 {@link RuntimeException} / {@link Exception} 异常信息，返回 “程序出现错误，请联系管理员” 信息
     *
     * @param exception 异常
     * @return ExceptionResponse
     */
    @ExceptionHandler({RuntimeException.class, Exception.class})
    public ResponseEntity<ExceptionResponse> process(HttpServletRequest request, Exception exception) {
        LOGGER.error(exceptionMessage("Unknown exception", request, null), exception);
        ExceptionResponse er = new ExceptionResponse(BaseConstants.ErrorCode.ERROR);
        setDevException(er, exception);
        return new ResponseEntity<>(er, HttpStatus.OK);
    }

}
```

> 在开发环境中，为了方便排查问题，如果我们想将异常栈返回到前端，可以指定 `spring.profiles.active=dev`。


### 2、自定义异常处理器

BaseExceptionHandler 中拦截了前面提到过的一些异常以及 RuntimeException 和 Exception。自定义的异常，将会进入 RuntimeException 和 Exception 的拦截方法。

如果想定制拦截方法并返回自定义的消息，可以按如下步骤自定义异常拦截器。

**1、拦截器切面类**

定义一个 XxxExceptionHandler 处理器类，添加注解 `@ControllerAdvice`。如果要覆盖标准的处理器，可以用 `@Order` 提高自定义处理器的优先级。

```java
@Order(0)
@ControllerAdvice
public class SecurityTokenExceptionHandler {
    
}
```

**2、添加处理器方法**

在处理器类中添加一个 process 方法，方法入参为要拦截的异常类，同时在方法上添加注解 `@ExceptionHandler(XxxException.class)` 指定要拦截的异常类。

方法实现中，一般需要根据异常级别打印出日志和异常栈，然后将异常信息封装到 ExceptionResponse 中返回，并指定响应状态码。

```java
@Order(0)
@ControllerAdvice
public class SecurityTokenExceptionHandler {
    private static final Logger LOGGER = LoggerFactory.getLogger(SecurityTokenExceptionHandler.class);
    private static final String EXCEPTION_CODE = "error.invalid.security.token";
    
    /**
     * 拦截 {@link SecurityTokenException} 异常信息，直接返回封装的异常消息
     */
    @ExceptionHandler(SecurityTokenException.class)
    public ResponseEntity<ExceptionResponse> process(SecurityTokenException exception) {
        LOGGER.warn(exception.getMessage(), exception);
        return new ResponseEntity<>(new ExceptionResponse(EXCEPTION_CODE), HttpStatus.OK);
    }
}
```
















---

# 响应结果处理


## 概述

hzero-starter-core 提供了返回结果封装工具，以及远程调用结果处理工具，下面看下如何使用它们。

## 响应结果判断依据

HZERO 响应结果判断一般有两个维度:

* 首先是判断响应状态码，例如 200 代表成功，500 代表服务端异常，具体可以参考 [HTTP响应设计](https://open.hand-china.com/document-center/doc/product/10137/10227?doc_id=32621#HTTP%E5%93%8D%E5%BA%94%E8%AE%BE%E8%AE%A1)

* 如果状态码返回 200，表示正常返回数据了，但业务不一定执行成功。还会判断返回数据中 `failed` 字段，没有 failed 字段或 `failed=false` 表示业务执行成功；如果 `failed=true` 表示业务执行失败，这时一般还会返回 `message` 错误消息字段。

Controller 中的方法一般会响应 `ResponseEntity`，它的 status 字段表示响应状态码。

```java
@GetMapping("/v1/roles/{roleId}")
public ResponseEntity<Role> queryRole(@PathVariable Long roleId) {
    Role role = roleRepository.selectRoleDetails(roleId);
    //return ResponseEntity.ok(role);
    return ResponseEntity.status(HttpStatus.OK).body(role).build();
}
```

## 封装返回结果 Results

在 Controller 方法中返回结果时，一般要返回 ResponseEntity，这时可能需要自己去构建 ResponseEntity 对象。

例如下面的代码，请求成功或失败，都需要自己构建 ResponseEntity。

```java
@PostMapping("/v1/roles")
public ResponseEntity<?> create(@RequestBody Role role) {
    try {
        roleService.createRole(role);
        // 响应成功
        return ResponseEntity.ok(role);
    } catch (Exception e) {
        // 失败
        ExceptionResponse response = new ExceptionResponse(e.getMessage());
        //return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).build();
        return ResponseEntity.ok(response);
    }
}
```

Results 工具则提供了一些常用的方法来封装返回结果。

```java
public class Results {
    /**
     * 请求成功
     * 
     * @param data 返回值
     * @return HttpStatus 200
     */
    public static <T> ResponseEntity<T> success(T data);

    /**
     * 请求成功, 已创建
     * 
     * @param data 返回值
     * @return HttpStatus 201
     */
    public static <T> ResponseEntity<T> created(T data);

    /**
     * 请求成功,无返回值
     * 
     * @return HttpStatus 204
     */
    public static <T> ResponseEntity<T> success();

    /**
     * 请求失败 请求参数错误
     * 
     * @return HttpStatus 400
     */
    public static <T> ResponseEntity<T> invalid();

    /**
     * 请求失败 请求参数错误
     * 
     * @param data 返回值
     * @return HttpStatus 400
     */
    public static <T> ResponseEntity<T> invalid(T data);

    /**
     * 请求失败 服务端错误
     * 
     * @return HttpStatus 500
     */
    public static <T> ResponseEntity<T> error();

    /**
     * 请求失败 服务端错误
     * 
     * @param data 返回值
     * @return HttpStatus 500
     */
    public static <T> ResponseEntity<T> error(T data);

    /**
     * 自定义Http返回
     * 
     * @param code HttpStatus code
     * @return ResponseEntity
     */
    public static <T> ResponseEntity<T> newResult(int code);

    /**
     * 自定义Http返回
     * 
     * @param code HttpStatus code
     * @param data 返回值
     * @return ResponseEntity
     */
    public static <T> ResponseEntity<T> newResult(int code, T data);

}
```

就可以使用 Results 来快速返回 ResponseEntity 对象了。

```java
@PostMapping("/v1/roles")
public ResponseEntity<?> create(@RequestBody Role role) {
    try {
        roleService.createRole(role);
        // 响应成功
        return Results.success(role);
    } catch (Exception e) {
        // 失败
        ExceptionResponse response = new ExceptionResponse(e.getMessage());
        return Results.error(response);
    }
}
```


## 请求结果解析 ResponseUtils

ResponseUtils 是一个响应处理类，通常用于处理 Feign 调用返回结果，由于 Feign 调用结果可能错误，返回对象并不一定是指定的泛型，所以通用的处理方式是指定返回类型为 String，然后使用 ResponseUtils 来处理返回结果。下面来看一个示例。

1、首先可能会开发一个 Feign 调用的接口，但是返回类型是 String。

```java
@FeignClient(value = HZeroService.Iam.NAME)
public interface UserFeignClient {

	@PostMapping("/v1/users")
	ResponseEntity<String> createUser(UserDTO user);
}

public class UserDTO {
	private Long id;
	private String email;

	// getter/setter
}
```

2、处理Feign调用结果

```java
public class UserService {

    private UserFeignClient userFeignClient;

    public void demo() {
        UserDTO dto = new UserDTO();
        dto.setEmail("demo@hand-china.com");

        // Feign 调用
        ResponseEntity<String> result = userFeignClient.createUser(dto);

        // 处理方式一：判断结果处理
        if (ResponseUtils.isFailed(result)) {
            // 获取异常信息
            ExceptionResponse response = ResponseUtils.getResponse(result, ExceptionResponse.class);
            throw new CommonException(response.getMessage());
        }
        // 成功，获取返回用户信息
        UserDTO user1 = ResponseUtils.getResponse(result, UserDTO.class);


        // 处理方式二，基于 lambada 表达式处理
        // 第三个参数：非2xx错误处理；第四个参数：200，但是failed=true处理
        UserDTO user2 = ResponseUtils.getResponse(result, UserDTO.class, (httpStatus, response) -> {
            throw new CommonException(response);
        }, (exceptionResponse) -> {
            throw new CommonException(exceptionResponse.getMessage());
        });

    }
}
```





















---

# 常用辅助类

## 概述

hzero-starter-core 下提供了一些 Helper开发辅助类，如获取当前登录用户，获取语言等等。

## 用户 DetailsHelper

### 1、获取当前登录用户

DetailsHelper 主要用于在程序中获取当前登录用户 CustomUserDetails，然后获取用户相关的一些信息。

例如下面的示例：

```java
CustomUserDetails self = DetailsHelper.getUserDetails();
Long userId = self.getUserId(); // 用户ID
String username = self.getUsername(); // 用户名
Long tenantId = self.getTenantId(); // 当前租户ID
Long roleId = self.getRoleId(); // 当前角色ID
String lang = self.getLanguage(); // 当前语言
```

**原理说明：**所有请求经过网关鉴权后，会将用户信息转成 Jwt_Token，然后请求服务，服务中有一个 JwtTokenFilter 会将 Jwt_Token 转成 CustomUserDetails。

需要注意的是，JwtTokenFilter 默认拦截 `/v1/*` 的请求，如果需要拦截其它API，可通过 `hzero.resource.pattern` 配置。

```yaml
hzero:
  resource:
    pattern: /v1/*,/v2/*
```


### 2、获取匿名用户

一些特殊场景下，可能并没有登录用户，这时可以使用匿名用户替换，这个匿名用户的用户名固定为 *ANONYMOUS*，用户ID固定为 `0`。

```java
CustomUserDetails details = DetailsHelper.getAnonymousDetails();
```

### 3、重置当前登录用户

某些特殊场景下，我们可能想要重置当前登录用户，这时可以调用 setCustomUserDetails 方法来重置。

```java
CustomUserDetails custom = new CustomUserDetails("custom", "custom");
custom.setUserId(1l);
custom.setLanguage("zh_CN");
DetailsHelper.setCustomUserDetails(custom);
```

### 4、使用注意事项

由于用户上下文信息是通过请求经过网关转换的JWT信息，属于请求线程里面的变量，如果后端另外开启子线程等异步线程，默认通过DetailsHelper获取不到用户信息。如果需要在异步线程中使用，可以通过参数传入等方式进行处理。

## 语言 LanguageHelper

可以通过 LanguageHelper 快捷得获取语言，默认返回当前登录用户 CustomUserDetails 的语言，没有则返回默认的 *zh_CN*。

```java
String lang = LanguageHelper.language();

Locale locale = LanguageHelper.locale();
```

**场景1**：最常用的就是在 Mapper.xml 中获取当前语言

```xml
<select id="selectTenant" resultType="org.hzero.iam.domain.entity.Tenant">
	<bind name="lang" value="@io.choerodon.mybatis.helper.LanguageHelper@language()"/>
	SELECT ht.id, ht.tenant_num, httl.tenant_name
	FROM hpfm_tenant ht
	JOIN hpfm_tenant_tl httl ON (ht.tenant_id = httl.tenant_id AND httl.lang = #{lang})
	WHERE ht.tenant_id = #{tenantId}
</select>
```

## 上下文 ApplicationContextHelper

### 1、获取 ApplicationContext

如果想获取 ApplicationContext 对象，可以实现 **ApplicationContextAware** 接口来注入。

例如：

```java
public class DemoService implements ApplicationContextAware {

    // 注入 ApplicationContext
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        Environment env = applicationContext.getBean(Environment.class);
    }
}
```

使用 ApplicationContextHelper 则可以通过静态方法来获取，而无需实现 ApplicationContextAware 接口。

```java
ApplicationContext applicationContext = ApplicationContextHelper.getContext();
Environment env = applicationContext.getBean(Environment.class);
```

需要注意的是，ApplicationContextHelper 也是实现 ApplicationContextAware 接口来设置 ApplicationContext 的，所以一般不建议在程序启动期间使用 ApplicationContextHelper 来获取 ApplicationContext 对象。

### 2、设置静态变量

某些场景下，我们可能会开发一些工具类，工具类并不会注册成bean对象，所以就无法实现 ApplicationContextAware 接口来注入 ApplicationContext，而 ApplicationContextHelper 在程序启动时可能还无法正常使用，这时可以使用静态注入方式。

ApplicationContextHelper 提供了如下几个静态注入方法，这些方法会每隔1秒循环获取对象来设置到静态变量上，直到成功为止（默认有一个最大尝试次数）。

```
/**
 * 异步从 ApplicationContextHelper 获取 bean 对象并设置到目标对象中，在某些启动期间需要初始化的bean可采用此方法。
 *
 * 适用于实例方法注入。
 *
 * @param type         bean type
 * @param target       目标类对象
 * @param setterMethod setter 方法，target 中需包含此方法名，且类型与 type 一致
 * @param <T>          type
 */
public static <T> void asyncInstanceSetter(Class<T> type, Object target, String setterMethod);

/**
 * 异步从 ApplicationContextHelper 获取 bean 对象并设置到目标对象中，在某些启动期间需要初始化的bean可采用此方法。
 * <br>
 * 一般可用于向静态类注入实例对象。
 *
 * @param type         bean type
 * @param target       目标类
 * @param targetField  目标字段
 */
public static void asyncStaticSetter(Class<?> type, Class<?> target, String targetField);
```

场景1：静态注入字段到实例对象中

```java
public class RedisMessageSource extends AbstractMessageSource implements IMessageSource {

    private RedisHelper redisHelper;
    private int redisDb = 1;

    public RedisMessageSource() {
        ApplicationContextHelper.asyncInstanceSetter(RedisHelper.class, this, "setRedisHelper");
        ApplicationContextHelper.asyncInstanceSetter(Environment.class, this, "setEnvironment");
    }

    private void setRedisHelper(RedisHelper redisHelper) {
        this.redisHelper = redisHelper;
    }

    private void setEnvironment(Environment environment) {
        this.redisDb = Integer.parseInt(environment.getProperty("hzero.service.platform.redis-db", "1"));
    }
}
```


















---

# 后端多语言消息

## 概述

在开发过程中，通常会有一些业务异常或提示消息要返回给前端用户，这时一般会先在消息文件中定义消息编码和消息内容，然后用相关工具获取多语言消息。

## 定义消息

### 1、创建消息文件

在开发的服务中，一般需要在 **resources** 目录下创建一个 **messages** 的子目录，然后在 messages 目录下创建多语言消息文件。

消息文件的格式为：`messages_服务简码_语言.properties`

例如：

* 英文消息：messages_hiam_en_US.properties
* 中文消息：messages_hiam_zh_CN.properties

<img src="https://file.open.hand-china.com/hsop-doc/doc_classify/0/13d6cb2c7bbb4d29a3438eff75e87032@image.png" alt="" width="auto" height="auto" />

<p/>

### 2、定义消息

接着在消息文件中定义对应语言的消息，消息编码格式为：`服务简码.消息级别.功能模块.消息描述编码`。

其中，消息级别可以定义如下值：

* info：操作成功，提示类消息
* warn：业务失败，提醒警告类消息
* error：错误较严重，阻塞流程或服务的异常类消息

如果消息要动态传入参数，可以通过 {n} 占位符来表示，n 从 0 开始，例如：`密码长度必须在 {0} - {1} 位之间`。

**场景举例：**

中文：

```properties
hiam.info.user.registerSuccess=您的账号注册成功
hiam.warn.user.passLength=密码长度必须在 {0} - {1} 位之间
hiam.error.user.accountExists=您输入的账号[{0}]已注册
```

英文：

```properties
hiam.info.user.registerSuccess=Account registration successful.
hiam.warn.user.passLength=Password length must be between {0} - {1} bits
hiam.error.user.accountExists=The account [{0}] already exists.
```

### 3、消息注册

消息文件创建好之后，还需将消息文件注册到 MessageAccessor 环境中才能使用。

可以创建一个服务配置类，接着创建一个 SmartInitializingSingleton 对象，然后将消息文件的路径添加到 MessageAccessor 中。

例如下面的代码：

```java
@Configuration
public class IamConfiguration {

    @Bean
    public SmartInitializingSingleton iamSmartInitializingSingleton() {
        return () -> {
            // 加入消息文件
            MessageAccessor.addBasenames("classpath:messages/messages_hiam");
        };
    }
}
```

## MessageAccessor


MessageAccessor 是用于获取 classpath 下资源多语言的工具，提供了多个重载的获取消息的方法。方法返回的 `Message` 对象非空，可放心使用。

主要提供了如下两类方法：

* getMessage：先从缓存获取多语言消息，没有则从本地消息文件获取多语言消息。参数说明如下：
    
    * code：消息编码
    * args：消息参数，当消息中包含参数时，可传入参数
    * defaultMessage：默认消息，当通过编码找不到消息时，则使用默认消息
    * locale：消息语言，不指定时默认为当前登录用户的语言
    
* getMessageLocal：从本地消息文件获取多语言消息，不走 Redis。
    
```java
/**
 * 添加资源文件位置
 *
 * @param names 如 <code>classpath:messages/messages_core</code>
 */
public static void addBasenames(String... names)

public static Message getMessage(String code, String defaultMessage)

public static Message getMessage(String code, String defaultMessage, Locale locale)

public static Message getMessage(String code, Object[] args, String defaultMessage)

public static Message getMessage(String code, Object[] args, String defaultMessage, Locale locale)

public static Message getMessage(String code)

public static Message getMessage(String code, Locale locale)

public static Message getMessage(String code, Object[] args)

public static Message getMessage(String code, Object[] args, Locale locale)

public static Message getMessageLocal(String code)

public static Message getMessageLocal(String code, Locale locale)

public static Message getMessageLocal(String code, Object[] args)

public static Message getMessageLocal(String code, Object[] args, Locale locale)

```

**场景举例：**

```java
public void getMessage() {
    // 获取消息：hiam.info.user.registerSuccess=您的账号注册成功
    String msg1 = MessageAccessor.getMessage("hiam.info.user.registerSuccess").getDesc();

    // 获取带参数的消息：hiam.warn.user.passLength=密码长度必须在 {0} - {1} 位之间
    String msg2 = MessageAccessor.getMessage("hiam.warn.user.passLength", new Object[]{6, 30}).getDesc();

    // 指定默认消息，没有则使用默认消息
    String msg3 = MessageAccessor.getMessage("hiam.warn.user.emailNotBlank", "Email is blank.").getDesc();

    // 指定消息的语言
    String msg4 = MessageAccessor.getMessage("hiam.warn.user.emailNotBlank", LanguageHelper.locale()).getDesc();
}
```

## 异常消息

程序开发中，大部分情况下可能不会使用 MessageAccessor 直接获取多语言消息，而是直接抛出业务异常，然后由[异常拦截处理器](local://hsop-document?doc_id=41082&doc_code=41082#%E5%BC%82%E5%B8%B8%E6%8B%A6%E6%88%AA%E5%A4%84%E7%90%86%E5%99%A8)来自动获取多语言消息，封装后返回给前端用户，这时只需在异常中指定消息编码即可。

**场景举例**：

```java
public User register(User user) {
    User exists = userRepository.selectByLoginName(user.getLoginName());

    if (exists != null) {
        // 您输入的账号[{0}]已注册
        throw new CommonException("hiam.error.user.accountExists", user.getLoginName());
    }

    if (user.getPassword().length() < 6 || user.getPassword().length() > 30) {
        // 密码长度必须在 {0} - {1} 位之间
        throw new CommonException("hiam.warn.user.passLength", 6, 30);
    }
    
    // ...
    return user;
}
```

## 返回消息管理

前面提到过，MessageAccessor 默认先从 Redis 缓存中获取多语言消息，获取不到再从本地文件中获取。

在 [开发管理 > 多语言管理 > 返回消息管理] 功能下，就可以维护多语言消息，这些消息会自动缓存到 Redis 中，这样就可以在程序运行期间更改多语言消息了。

> 注意：缓存消息在内存中有一个二级缓存，更新数据后，默认最多需要300秒后才会生效。


<img src="https://file.open.hand-china.com/hsop-doc/doc_classify/0/48ac8ad8d8af4967bc43fc26cffb2dec@image.png" alt="" width="auto" height="auto" />




---

# 通用线程池


## 概述

在开发中，如果想要创建一个 ThreadPoolExecutor 线程池，可能需要配置一大堆参数，或者想要监控线程池的状态、批量提交任务到线程池，那么怎么办呢？

`org.hzero.core.util.CommonExecutor` 就增强了默认的线程池，支持构建线程优先的线程池，支持查看线程池状态，支持批量提交任务到线程池等等。

## 构建线程优先的线程池

### 1、线程池提交任务

我们可能会采用如下的方式来构建一个线程池：

```java
ThreadPoolExecutor executor = new ThreadPoolExecutor(
        4, // 核心线程数
        8, // 最大线程数
        60, TimeUnit.SECONDS, // 最大线程存活时间
        new LinkedBlockingQueue<>(), // 任务队列
        new ThreadFactoryBuilder().setNameFormat("pool-%d").build(), // 指定线程池名称
        new ThreadPoolExecutor.AbortPolicy() // 指定拒绝策略
);
```

默认创建的 ThreadPoolExecutor 在提交任务时，当任务数量超过核心线程数后，任务会进入队列等待，只有当队列满了之后，才继续创建工作线程，直到达到最大线程数量。

下图展示了 ThreadPoolExecutor 提交任务的流程：


<img src="https://file.open.hand-china.com/hsop-doc/doc_classify/0/2a18dcb9044344439a9313e61782b313@image.png" alt="" width="auto" height="auto" />

### 2、构建线程优先线程池

一般你可能创建的是无界队列 *new LinkedBlockingQueue<>()* ，当工作线程达到核心线程数后，实际上，并不会再创建工作线程直到最大线程数，因为任务会一直进入队列中。就算你设置了队列的容量，任务也会先进入队列中，直到队列满了，才会继续创建工作线程。

如果想要继续创建工作线程直到最大线程数，而不是一开始就进入队列中，可以使用 CommonExecutor 来构建线程优先的线程池。线程优先的线程池 可以在达到核心线程数后，继续创建线程直到达到最大线程数，最后才进入到等待队列。

使用方式如下：

* 自定义参数构建

```java
ThreadPoolExecutor executor = CommonExecutor.buildThreadFirstExecutor(
	4, // 核心线程数
	8, // 最大线程数
	60, TimeUnit.SECONDS, // 线程空闲时间
	1024, // 任务队列容量
	"PoolName" // 线程池名称
);

// 使用默认参数构建：核心线程数=CPU核数，最大线程数=8*核心线程数，空闲时间=5min，队列大小=65536
ThreadPoolExecutor executor = CommonExecutor.buildThreadFirstExecutor("PoolName"); // 只需指定一个线程池名称
```

* 使用默认参数构建

默认参数：核心线程数=CPU核数，最大线程数=8*核心线程数，空闲时间=5min，队列大小=65536

```java
// 只需指定一个线程池名称
ThreadPoolExecutor executor = CommonExecutor.buildThreadFirstExecutor("PoolName"); 
```


## 批量提交异步任务

CommonExecutor 的 *batchExecuteAsync* 方法可以提交一批任务执行，并同步等待返回结果，批量任务的执行时间将取决于执行时间最长的一个任务。

使用方式：首先需要构建批量任务，将每个任务的执行封装到 AsyncTask 中，然后再用 batchExecuteAsync 方法提交批量任务。

**批量提交异步任务**：

```java
/**
 * 批量提交异步任务，使用默认的线程池
 *
 * @param tasks 将任务转化为 AsyncTask 批量提交
 */
public static <T> List<T> batchExecuteAsync(List<AsyncTask<T>> tasks, @NonNull String taskName);

/**
 * 批量提交异步任务，执行失败可抛出异常或返回异常编码即可 <br>
 * <p>
 * 需注意提交的异步任务无法控制事务，一般需容忍产生一些垃圾数据的情况下才能使用异步任务，异步任务执行失败将抛出异常，主线程可回滚事务.
 * <p>
 * 异步任务失败后，将取消剩余的任务执行.
 *
 * @param tasks    将任务转化为 AsyncTask 批量提交
 * @param executor 线程池，需自行根据业务场景创建相应的线程池
 * @return 返回执行结果
 */
public static <T> List<T> batchExecuteAsync(@NonNull List<AsyncTask<T>> tasks, @NonNull ThreadPoolExecutor executor, @NonNull String taskName);
```


**场景举例**：

```java
private final ThreadPoolExecutor executor = 
        CommonExecutor.buildThreadFirstExecutor(4, 16, 5, TimeUnit.MINUTES, 1024, "SecGrpExe");

@Autowired
private List<SecGrpAuthorityService<?>> serviceList;

public void batchExecuteTest(SecGrp secGrp) {
    List<AsyncTask<Boolean>> tasks = serviceList.stream().map(service -> new AsyncTask<Boolean>() {
        // 返回任务的名称
        @Override
        public String taskName() {
            return service.getClass().getSimpleName();
        }

        // 执行异步任务
        @Override
        public Boolean doExecute() {
            service.initSecGrpAuthority(secGrp, secGrp.getRoleId());
            return Boolean.TRUE;
        }
    }).collect(Collectors.toList());

    // 提交批量任务
    CommonExecutor.batchExecuteAsync(
            tasks, // 异步任务集合 
            executor, // 线程池
            "initSecGrpAuthority" // 批量任务名称
    );
}
```

> 注意：异步批量任务执行无法保证任务的顺序性，并且其中一个任务失败后就会结束执行，并抛出异常，不保证事务性的一致性，对事务有强一致要求的不建议使用。


## 查看线程池状态

线程池是程序运行中的重要资源，一般也需要对其进行监控，例如线程池的任务数、工作线程数，观察是否满负载工作，便于调整线程池配置。

CommonExecutor 的 *displayThreadPoolStatus*  方法可以按一定频率输出线程池状态到日志中。

```java
/**
 * 根据一定周期输出线程池的状态
 *
 * @param threadPool     线程池
 * @param threadPoolName 线程池名称
 */
public static void displayThreadPoolStatus(ThreadPoolExecutor threadPool, String threadPoolName);

/**
 * 根据一定周期输出线程池的状态
 *
 * @param threadPool     线程池
 * @param threadPoolName 线程池名称
 * @param period         周期
 * @param unit           时间单位
 */
public static void displayThreadPoolStatus(ThreadPoolExecutor threadPool, String threadPoolName, long period, TimeUnit unit);
```

例如：

```java
INFO 1 --- [pool-4-thread-1] org.hzero.core.util.CommonExecutor       : [>>ExecutorStatus<<] ThreadPool Name: [IamExecutor], Pool Status: [shutdown=false, Terminated=false], Pool Thread Size: 58, Largest Pool Size: 64, Active Thread Count: 0, Task Count: 7780, Tasks Completed: 7780, Tasks in Queue: 0

INFO 1 --- [pool-5-thread-1] org.hzero.core.util.CommonExecutor       : [>>ExecutorStatus<<] ThreadPool Name: [LdapExecutor], Pool Status: [shutdown=false, Terminated=false], Pool Thread Size: 0, Largest Pool Size: 0, Active Thread Count: 0, Task Count: 0, Tasks Completed: 0, Tasks in Queue: 0
```

## 线程池管理器

前面在查看线程池状态时需要查看日志，我们还支持在 info 监控端点来查看查看线程池状态，前提是自定义的线程池需要注册到线程池管理器中（CommonExecutor.ExecutorManager）。通过 CommonExecutor 的 buildThreadFirstExecutor 方法创建的线程池会自动注册到线程池管理器中。

例如：注册自定义线程池

```java
String poolName = "PoolName";
// 创建线程池
ThreadPoolExecutor executor = new ThreadPoolExecutor(4, 8, 60, TimeUnit.SECONDS, new LinkedBlockingQueue<>(1024));
// 注册到线程池管理器中
CommonExecutor.ExecutorManager.registerAndMonitorThreadPoolExecutor(poolName, executor);
```

如果想通过监控端点查看线程池状态，需要先配置 `hzero.actuator.show-executor-info=true` 开启展示线程池状态信息。

然后访问服务的 info 监控端点查看线程池状态：

<img src="https://file.open.hand-china.com/hsop-doc/doc_classify/0/248bd8ab23044687b02f49e66041c5ca@image.png" alt="" width="auto" height="auto" />

## 服务下线时关闭线程池

线程池是会占用服务器资源的，一般来说，在服务下线时，最好要同时关闭线程池，释放占用的资源。

CommonExecutor 提供了一个 *hookShutdownThreadPool* 方法在JVM关闭时关闭线程池。

场景举例：

```java
String poolName = "PoolName";
// 创建线程池
ThreadPoolExecutor executor = new ThreadPoolExecutor(4, 8, 60, TimeUnit.SECONDS, new LinkedBlockingQueue<>(1024));

// 在JVM关闭时关闭线程池
CommonExecutor.hookShutdownThreadPool(poolName, executor);

// 注册到线程池管理器中，注册的线程池会自动调用 hookShutdownThreadPool
// CommonExecutor.ExecutorManager.registerAndMonitorThreadPoolExecutor(poolName, executor);
```

> 注意：通过 CommonExecutor 创建的线程池或注册到管理器中的线程池无需再调用 hookShutdownThreadPool。




---

# 动态职责链

## 概述

也许你对职责链模式不太陌生，职责链模式通常用于一系列的操作封装，大致的原理就是构造一个拦截器链来执行一个接一个的任务，如果想要再添加一个任务可以再新增一个拦截器。

但标准的职责链模式可能无法满足一些特殊的场景，例如想要在某个主业务逻辑的 前、后 各有一个职责链，要支持调整职责链中拦截器的顺序，支持异步执行等等。

为了实现上诉的一些功能，我们实现了动态职责链模式，目前支持如下特性：

* 支持配置前置拦截器、后置拦截器
* 支持在拦截器链中增加、替换、删除拦截器
* 支持拦截器链转为异步执行或同步执行
* 支持配置多条职责链
* 支持项目上进行定制化业务逻辑

## 动态职责链模式

### 1、API说明

在 `org.hzero.core.interceptor` 包下提供了动态职责链模式的封装。

* HandlerInterceptor：业务处理拦截器，业务逻辑执行的封装，需要实现
* ChainId：职责链的ID，需要实现
* InterceptorChain：拦截器链的封装
* InterceptorChainBuilder：拦截器链构造器，构造 InterceptorChain 的核心类
* InterceptorChainConfigurer：拦截器链配置器，可配置统一业务模型的多条拦截器链
* AbstractInterceptorChainManager：拦截器链管理器，需要实现

<img src="https://file.open.hand-china.com/hsop-doc/doc_classify/0/a3ce0ad4a18842189f15c12e908ebfc5@image.png" alt="" width="auto" height="auto" />

<p>

### 2、使用示例说明

下面以用户创建和更新为例说明动态职责链模式的应用。

1、首先增加业务处理类，封装到 HandlerInterceptor 中，可以将链式的业务封装到多个 HandlerInterceptor 实现类中。

```java
@Component
public class CommonMemberRoleInterceptor implements HandlerInterceptor<User> {
    @Override
    public void interceptor(User user) {
        // 业务逻辑....
    }
}
```

2、实现 ChainId，声明拦截器链的ID

```java
public enum  UserOperation implements ChainId {
    // 创建用户
    CREATE_USER("CREATE_USER"),

    // 注册用户
    REGISTER_USER("REGISTER_USER"),

    // 其它....
    ;

    private final String id;
    UserOperation(String id) {
        this.id = id;
    }

    @Override
    @Nonnull
    public String id() {
        return id;
    }
}
```

3、在配置器中配置多条拦截器链

```java
@Order(0) // 配置生效的顺序
@Component
public class UserInterceptorChainConfigurer implements InterceptorChainConfigurer<User, InterceptorChainBuilder<User>> {

    @Override
    public void configure(InterceptorChainBuilder<User> builder) {
        // 配置第一个拦截器链
		builder
                .selectChain(UserOperation.CREATE_USER) // 指定连接器链的名称
                .pre() // 开始配置前置拦截器
                .addInterceptor(ValidationInterceptor.class) // 添加前置拦截器
                .post() // 开始配置后置拦截器
                .async() // 异步化执行(默认是同步)
                .addInterceptor(CommonMemberRoleInterceptor.class) // 添加后置拦截器
                .addInterceptor(UserConfigInterceptor.class) // 添加后置拦截器
                .addInterceptor(SendMessageInterceptor.class) // 添加后置拦截器
                .addInterceptor(LastHandlerInterceptor.class); // 添加后置拦截器

		// 配置第二个拦截器链
        builder
                .selectChain(UserOperation.REGISTER_USER)
                .post()
                .async()
                .addInterceptor(RegisterMemberRoleInterceptor.class)
                .addInterceptor(UserConfigInterceptor.class)
                .addInterceptor(LastHandlerInterceptor.class);
		//....			
    }
}
```

4、定制化已存在的拦截器链

如果项目上想定制化已经存在的拦截器链，可以再实现一个 InterceptorChainConfigurer 来定制化。

```java
@Order(10) // 指定配置顺序，定制化的要大于已存在的配置器顺序
@Component
public class CustomUserInterceptorChainConfigurer implements InterceptorChainConfigurer<User, InterceptorChainBuilder<User>> {

    @Override
    public void configure(InterceptorChainBuilder<User> builder) {
        builder
                .selectChain(UserOperation.CREATE_USER) // 选择已存在的拦截器链
                .pre() // 配置前置拦截器
				// 将 InternalMemberRoleInterceptor 添加到 ValidationInterceptor 之前
                .addInterceptorBefore(InternalMemberRoleInterceptor.class, ValidationInterceptor.class) 
                .post() // 配置后置拦截器
                .sync() // 将原本的异步转为同步
				// 移除拦截器
                .removeInterceptor(CommonMemberRoleInterceptor.class) 
				// 将 CommonMemberRoleInterceptor 添加到 SendMessageInterceptor 之后
                .addInterceptorAfter(CommonMemberRoleInterceptor.class, SendMessageInterceptor.class); 
    }
}
```

5、还需要继承 AbstractInterceptorChainManager 配置业务模型的拦截器链管理器

```java
@Component
public class UserInterceptorChainManager extends AbstractInterceptorChainManager<User> {

    public UserInterceptorChainManager(List<HandlerInterceptor<User>> handlerInterceptors,
                                       List<InterceptorChainConfigurer<User, InterceptorChainBuilder<User>>> interceptorChainConfigurers) {
        super(handlerInterceptors, interceptorChainConfigurers);
    }
}
```

6、使用拦截器链

首先要注入管理器 InterceptorChainManager，然后调用 doInterceptor 方法执行业务逻辑：
* 第一个参数指定拦截器链的名称
* 第二个参数为业务参数
* 第三个参数传入要在拦截器链路中间执行的核心业务逻辑。

```java
@Autowired
private UserInterceptorChainManager userInterceptorChainManager;

@Override
@Transactional(rollbackFor = Exception.class)
public User createUser(User user) {
    userInterceptorChainManager.doInterceptor(UserOperation.CREATE_USER, user, (u) -> {
        // 创建用户
        userCreateService.createUser(u);
    });

    return user;
}

@Override
@Transactional(rollbackFor = Exception.class)
public User register(User user) {
    userInterceptorChainManager.doInterceptor(UserOperation.REGISTER_USER, user, (u) -> {
        // 注册用户
        userRegisterService.registerUser(u);
    });

    return user;
}
```

---

# 观察者事件总线

## 概述

观察者模式相信你也不会陌生，观察者模式一般用于模块、系统间的解耦，在完成某些操作后，直接发送一个事件，观察者就会自动执行相关业务逻辑。

观察者事件总线则在观察者模式的基础上进行了一些增强：

* 支持同步、异步观察者执行
* 支持指定观察者的执行顺序
* 支持执行失败后回滚操作

## 观察者事件总线

在 `org.hzero.core.observer` 包下提供了观察者事件总线的封装。

### 1、API说明

* Observer：观察者接口类
* EventBus：观察者事件总线
* AsyncEventBus：支持异步执行的事件总线（注意异步执行 顺序无法保证）

<img src="https://file.open.hand-china.com/hsop-doc/doc_classify/0/84aa0cf6869c4493b8a85b5d838553f8@image.png" alt="" width="auto" height="auto" />


### 2、使用示例说明

1、开发观察者，观察者需要实现 Observer 接口。

方法说明：

* order：指定观察者的顺序
* update：业务更新逻辑，发出事件通知，将调用 update 执行观察者逻辑
* cancel：异步执行业务失败时回滚业务，一般就是 update 的反向操作。

```java
@Component
public class RoleLabelObserver implements Observer<Role> {
    @Autowired
    protected LabelRelRepository labelRelRepository;

    // 观察者顺序
    @Override
    public int order() {
        return 10;
    }

    // 业务执行
    @Override
    public void update(@Nonnull Role role, Object... args) {
        // 执行业务逻辑，例如插入数据
        LabelRel label = new LabelRel();
        label.setDataType("ROLE");
        label.setDataId(role.getId());
        labelRelRepository.insertSelective(label);
    }

    // 业务取消
    @Override
    public void cancel(Role role, Object... args) {
        // 反向执行业务逻辑，例如删除 update 中插入的数据
        LabelRel label = new LabelRel();
        label.setDataType("ROLE");
        label.setDataId(role.getId());
        labelRelRepository.delete(label);
    }
}
```

2、构建消息总线

消息总线有同步和异步的消息总线，分别对应 EventBus 和 AsyncEventBus。

异步消息总线会自动将所有的观察者转为异步执行，如果执行失败，会取消所有观察者的执行，并调用 cancel 方法回滚。

同步消息总线：

```java
public class RoleCreateService {
    protected EventBus<Role> eventBus;

    // 注入观察者 构建消息总线
    @Autowired
    private void setEventBus(List<Observer<Role>> roleObservers) {
        this.eventBus = new EventBus<>("CreateRole", roleObservers);
    }
}
```

异步消息总线：异步消息总线需要传入一个线程池。

```java
public class RoleCreateService {

    protected EventBus<Role> eventBus;

    // 注入观察者 构建消息总线
    @Autowired
    private void setEventBus(List<Observer<Role>> roleObservers) {
        ThreadPoolExecutor executor = CommonExecutor.getCommonExecutor();
        this.eventBus = new AsyncEventBus("CreateRole", roleObservers, executor);
    }
}
```

3、通过消息总线发布观察者事件

```java
public final Role createRole(Role role) {
    // 创建角色
    roleRepository.insertSelective(role);
    // 通知观察者创建了角色
    eventBus.notifyObservers(role);        
}
```




---

# 服务信息配置

## 概述

开发的每个服务一般都会有一个路由到服务名的映射，这样网关才能根据路由进行转发。因此，我们的服务中一般都会配置服务路由相关的一些信息。当然，还会有一些其它的信息配置，如 redisDb，端口等等。

## 服务路由配置

### 1、服务路由API

服务路由信息通过 `io.choerodon.core.swagger.ChoerodonRouteData` 来封装，主要的一些配置如下。

最长设置的参数：

* name：表示路由唯一ID，一般用服务简码表示  
* path：表示路由前缀，也可用服务简码表示，前端调用服务API时需加上路由前缀  
* serviceId：服务名称，网关通过路由前缀匹配到服务后，将基于负载均衡器请求该服务
* packages: 指定扫描API的包，多个可用逗号分隔，可为空
* stripSegment：去除路由的段数，例如 /iam/v1/user，在网关去除第一段后就变成了 /v1/users

```java
public class ChoerodonRouteData {
	// 路由ID
    private Long id;
	// 服务简码，如 hiam
    private String name;
	// 服务路径，如 /iam/**
    private String path;
	// 服务ID，如 hzero-iam
    private String serviceId;
	// 跳转的绝对地址，一般不需要配置
    private String url;
    // 去除前缀段数
    private int stripSegment = 1;
	// 敏感头过滤
    private String sensitiveHeaders;
	// 自定义敏感头
    private Boolean customSensitiveHeaders = true;
    // 必须指定，表示属于该服务的包路径
    private String packages;
}
```

然后需要通过 `io.choerodon.swagger.swagger.extra.ExtraDataManager` 来注册管理服务路由。

```java
public interface ExtraDataManager {
    ExtraData extraData = new ExtraData();
    
	// 返回服务描述信息
    ExtraData getData();
}
```

### 2、配置服务路由

一般会在服务配置包下（例如 org.hzero.iam.config）创建一个 `ExtraDataManager` 的子类来描述该服务的路由信息。

项目上开发时，建议每个服务自定义一个 ExtraDataManager 的实现类，使用配置的形式注入服务配置，便于开发人员本地开发。

 ```java
// 类上加上 @ChoerodonExtraData 
@ChoerodonExtraData
public class HiamExtraDataManager implements ExtraDataManager {
    @Autowired
    private Environment environment;

    @Override
    public ExtraData getData() {
        ChoerodonRouteData routeData = new ChoerodonRouteData();
        // 服务简码
        routeData.setName(environment.getProperty("hzero.service.current.name", "hiam"));
        // 服务路径
        routeData.setPath(environment.getProperty("hzero.service.current.path", "/iam/**"));
        // 服务ID
        routeData.setServiceId(environment.getProperty("hzero.service.current.service-name", "hzero-iam"));
        // 服务所在包路径
        routeData.setPackages("org.hzero.iam");
        // 去除第一段路由
        routeData.setStripSegment(1);
        // 重要：将 routeData 放入 extraData 中
        extraData.put(ExtraData.ZUUL_ROUTE_DATA, routeData);
        return extraData;
    }
}
```   

本地开发时，可以在环境变量或本地配置文件中配置带后缀的服务名

```yaml
hzero:
  service:
	current:
	  name: hiam-16007
	  path: /iam-16007/**
	  service-name: hzero-iam-16007
```
   
服务注册成功后，hzero-admin 会自动刷新路由到数据库，并通知网关服务拉取最新的路由。

> 个人本地开发建议服务名加上后缀做区分，如 hzero-iam-16007，同时需要修改 ExtraDataManager 中的路由配置，分别加上后缀，保持配置文件中的服务名和 ExtraDataManager 中配置的服务名一致。

### 3、服务管理

除了代码中配置服务路由，还可以通过 [平台治理 > 服务管理] 功能在页面上维护服务路由。具体请参考 [服务管理](https://open.hand-china.com/document-center/doc/product/10002/10270?doc_id=135584231116)


<img src="https://file.open.hand-china.com/hsop-doc/doc_classify/0/c3d51c0567674cf89c3e8c5eb787c92a@image.png" alt="" width="auto" height="auto" />


## 全局服务设置

### 1、HZeroService

`org.hzero.common.HZeroService` 中维护了HZERO所有服务的服务名、使用的 RedisDB、端口等配置，这些配置在很多场景下可能都会用到。

例如下面展示的 IAM、OAuth 服务：

```java
public final class HZeroService {
    /**
     * HZero 认证服务
     */
    @Component
    public static class Oauth {
        public static final String NAME = "${hzero.service.oauth.name:hzero-oauth}";
        public static final String CODE = "hoth";
        public static final String PATH = "/oauth/**";
        public static Integer PORT = 8020;
        public static Integer REDIS_DB = 3;
        public static String BUCKET_NAME = "hoth";

        @Value("${hzero.service.oauth.port:8020}")
        public void setPort(Integer port) {
            PORT = port;
        }

        @Value("${hzero.service.oauth.redis-db:3}")
        public void setRedisDb(Integer redisDb) {
            REDIS_DB = redisDb;
        }

        @Value("${hzero.service.bucket-name:hoth}")
        public void setBucketName(String bucketName) {
            BUCKET_NAME = bucketName;
        }
    }

    /**
     * HZero 用户身份服务
     */
    @Component
    public static class Iam {
        public static final String NAME = "${hzero.service.iam.name:hzero-iam}";
        public static final String CODE = "hiam";
        public static final String PATH = "/iam/**";
        public static Integer PORT = 8030;
        public static Integer REDIS_DB = 1;
        public static String BUCKET_NAME = "hiam";

        @Value("${hzero.service.iam.port:8030}")
        public void setPort(Integer port) {
            PORT = port;
        }

        @Value("${hzero.service.iam.redis-db:1}")
        public void setRedisDb(Integer redisDb) {
            REDIS_DB = redisDb;
        }

        @Value("${hzero.service.bucket-name:hiam}")
        public void setBucketName(String bucketName) {
            BUCKET_NAME = bucketName;
        }
    }
}
```

**场景1**：Feign调用服务名

我们在开发一些 Feign 客户端调用或 RestTemplate 的负载均衡调用时，往往需要调用服务的服务名，这时就可以使用 HZeroService 中定义的服务名。

```java
@FeignClient(value = HZeroService.Oauth.NAME, fallback = UserDetailsClientImpl.class, path = "/oauth")
public interface UserDetailsClient {

    @PostMapping("/user/role-id")
    ResponseEntity storeUserRole(@RequestParam("roleId") Long roleId);
}
```

**场景2**：切换 RedisDB

我们提供的 RedisHelper 组件支持 Redis DB 切换，比如当前服务默认的 redis-db=1，我们可以切到 db2 下去读取数据，这时可以使用 HZeroService 中配置的服务 RedisDB，切到对应服务下的 RedisDB 去读取缓存数据。

例如在 hzero-iam 服务中保存 PasswordPolicy 时，需要将其缓存到 hzero-oauth 服务的 RedisDB 下：

```java
public void cachePasswordPolicy(PasswordPolicy passwordPolicy) {
    String key = HZeroService.Oauth.CODE + ":password_policy:";
    SafeRedisHelper.execute(HZeroService.Oauth.REDIS_DB, (helper -> {
        helper.hshPut(key, passwordPolicy.getOrganizationId().toString(), helper.toJson(passwordPolicy));
    }));
}
```

### 2、修改服务配置

默认配置下，各个服务使用的 RedisDB 可能是不同的，项目上可能有需求要修改服务名、RedisDB 等配置。

因为我们标准服务的代码中都是通过引用 HZeroService 中定义的服务名或 RedisDB，所以，如果确定要修改服务名或 RedisDB，需要在每个服务中增加如下配置。

```yaml
hzero:
  service:
    # 认证服务  
    oauth:
      name: hzero-oauth
      redis-db: 3
      port: 8020
    # IAM 身份服务  
    iam:
      name: hzero-iam
      redis-db: 1
      port: 8030
    # 平台服务  
    platform:
      name: hzero-platfor
      port: 8100
      redis-db: 1
    # 文件服务  
    file:
      name: hzero-file
      port: 8110
      redis-db: 1
    # 消息服务  
    message:
      name: hzero-message
      port: 8120
      redis-db: 1
    # 调度服务  
    scheduler:
      name: hzero-scheduler
      port: 8130
      redis-db: 1
    # 导入服务  
    import:
      name: hzero-import
      port: 8140
      redis-db: 1
    # 接口服务  
    interface:
      name: hzero-interface
      port: 8150
      redis-db: 1
    # 报表服务  
    report:
      name: hzero-report
      port: 8210
      redis-db: 1
    # 工作流（Plus）  
    workflow-plus:
      name: hzero-workflow-plus
      port: 8220
      redis-db: 1
    # 自然语言处理  
    nlp:
      name: hzero-nlp
      port: 8230
      redis-db: 1
    # 监控服务  
    monitor:
      name: hzero-monitor
      port: 8260
      redis-db: 1
    # 支付服务  
    pay:
      name: hzero-pay
      port: 8250
      redis-db: 1
```




---

# 常用工具类

## 概述

hzero-starter-core 辅助了提供了一些常用工具类，建议熟悉下，避免重复造轮子。


## util 工具包

大部分的工具包都在 `org.hzero.core.util` 包下，提供了如下的一些工具类。

* **AssertUtils**：Assert 扩展
* **CheckStrength**：检测密码强度工具
* **FieldNameUtils**： 驼峰-下划线互转
* **EncoderUtils**： 对文件名称进行编码，处理一些特殊字符。
* **EncryptionUtils**：加密解密工具类
* **Reflections**： 反射工具类
* **Regexs**： 正则表达式工具类，提供了常用的正则表达式常量及校验方法
* **UUIDUtils**：生成UUID
* **ValidUtils**： 数据校验工具
* **Sequence**：分布式高效有序ID生产工具
* **SensitiveUtils**：敏感信息工具类
* **PinyinUtils**：拼音处理工具类
* **DomainUtils**：域名相关工具
* **SystemClock**：系统时钟
* **TrimUtils**：过滤空格的工具
* **JsonUtils**：JSON转换工具
* ....


## UUIDUtils

UUIDUtils 用于生成去掉连接符号的 uuid 字符串。

**举例说明**：

```java
public void test() {
    // 未去掉连接符
    String uuid1 = UUID.randomUUID().toString();
    System.out.println(uuid1); // 35a7b8d7-c2e2-4c0b-88fd-352bb58c6b9b

    // 去掉了连接符
    String uuid2 = UUIDUtils.generateUUID();
    System.out.println(uuid2); // 3a57f2a8485f456e8e7909fc72755329
}
```


## 校验 ValidUtils

前面在 [基础类BaseController](local://hsop-document?doc_id=40882&doc_code=40882#API%E6%8E%A5%E5%8F%A3%20BaseController) 一节中说明了 BaseController 提供了一些校验对象的方法，实际上那些校验方法就是基于 ValidUtils 来实现的。如果在其它地方想要调用 ValidUtils 来校验参数，可以直接使用 ValidUtils 工具类。

**举例说明**：

```java
@Autowired
private javax.validation.Validator validator;

public void validTest() {
    Client client = new Client();
    ValidUtils.valid(validator, client);
    // 分组校验
    ValidUtils.valid(validator, client, Client.Group1.class);
    ValidUtils.valid(validator, client, Client.Group2.class);

    // 校验集合
    List<Client> clients = new ArrayList<>();
    ValidUtils.valid(validator, clients);
}
```

### 更改通用校验返回消息
如果默认的校验信息不满足需求。 可通过维护字段注解中message属性来更改返回消息，其中message属性支持多语言维护。如
```
@NotNull(message = "error.tenantI_is_not_null")
private Long tenantId;
```
返回信息结果：
1.如果message属性为多语言编码，会返回对应多语言信息。
2.如果message属性不是多语言编码，直接返回message属性维护的消息。
3.message属性未维护，返回默认校验消息

## 拼音 PinyinUtils

PinyinUtils 可以将中文转成拼音，提取拼音首字母等。

**举例说明**：

```java
public void test()
    String text1 = "菜单管理"; // 有多音字
    String text2 = "用户管理"; // 无多音字

    String pinyin1 = PinyinUtils.getPinyin(text1);
    String capital1 = PinyinUtils.getPinyinCapital(text1);
    // 获取中文的拼音，结果：|caidanguanli|caishanguanli|caichanguanli|
    System.out.println("pinyin1 >> " + pinyin1);
    // 获取拼音首字母，结果：|ccgl|csgl|cdgl|
    System.out.println("capital1 >> " + capital1);


    String pinyin2 = PinyinUtils.getPinyin(text2);
    String capital2 = PinyinUtils.getPinyinCapital(text2);
    // 结果：|yonghuguanli|
    System.out.println("pinyin2 >> " + pinyin2);
    // 结果：|yhgl|
    System.out.println("capital2 >> " + capital2);
}
```

## 正则 Regexs

Regexs 下提供了一些常用的正则表达式以及正则校验方法，例如 数字类、字母类、邮箱、URL、中文、IP地址、图片、日期 等等，更多的可自行查看。

**举例说明**：

```java
public void test() {
    // 正则表达式
    String integer = Regexs.INTEGER; // 整数
    String email = Regexs.EMAIL; // 邮件
    String url = Regexs.URL; // URL
    String chinese = Regexs.CHINESE; // 中文
    // .......

    // 正则匹配方法
    // 判断是否是手机号
    boolean isPhone = Regexs.is("185666444", Regexs.MOBILE);
    // 判断是否是中文
    boolean isChinese = Regexs.is("是否中文", Regexs.CHINESE);
    // 判断是否是数字
    boolean isNumber = Regexs.isNumber("12345");
    // 判断是否是邮箱
    boolean isEmail = Regexs.isEmail("hzero@hand-china.com");
    // 判断是否是IP地址
    boolean isIp = Regexs.isIP("192.168.12.101");
    // 判断是否是URL地址
    boolean isUrl = Regexs.isUrl("www.baidu.com");
    // 判断是否是移动手机号
    boolean isMobile = Regexs.isMobile("18523226666", "+86");
}
```

## 反射 Reflections

Reflections 提供了如下一些反射方法。

```java
public class Reflections {
    /**
     * 通过反射, 获得Class定义中声明的泛型参数的类型, 注意泛型必须定义在父类处. 如无法找到, 返回Object.class.
     *
     * @param clazz class类
     * @return 返回第一个声明的泛型类型. 如果没有,则返回Object.class
     */
    public static Class getClassGenericType(final Class clazz);
    /**
     * 通过反射, 获得Class定义中声明的父类的泛型参数的类型. 如无法找到, 返回Object.class.
     *
     * @param clazz class类
     * @param index 获取第几个泛型参数的类型,默认从0开始,即第一个
     * @return 返回第index个泛型参数类型.
     */
    public static Class getClassGenericType(final Class clazz, final int index);
    /**
     * 获取实体的字段
     *
     * @param entityClass 实体类型
     * @param fieldName   字段名称
     * @return 该字段名称对应的字段, 如果没有则返回null.
     */
    public static Field getField(Class entityClass, String fieldName);
    /**
     * 改变private/protected的成员变量为public.
     *
     * @param field Field
     */
    public static void makeAccessible(Field field);
    /**
     * 获取Class的所有字段，以及父类字段
     */
    public static Field[] getAllField(Class<?> clazz);
    /**
     * 获取字段的值
     */
    public static Object getFieldValue(Object target, String fieldName);
    /**
     * 反射获取字段值
     */
    public static <T> T getFieldValue(Object target, String fieldName, Class<T> returnType);
    /**
     * 获取 Java 基本类型默认值
     *
     * @param clazz 基本类型 Class
     * @param <T>   基本类型
     * @return 默认值
     */
    public static <T> T getDefaultValue(Class<T> clazz);
}
```

## 域名 DomainUtils

DomainUtils 提供了与域名相关的一些方法，可以方便的从URL从提取一些信息。

```java
public class DomainUtils {
    public static final String HTTP = "http://";
    public static final String HTTPS = "https://";

    /**
     * 检查URL是否是有效的域名，会判断域名是否可访问
     */
    public static boolean isValidDomain(String url);
    /**
     * 从URL中提取参数
     */
    public static Map<String, String> getQueryMap(String url);
    /**
     * 返回URI的参数部分
     */
    public static String getQuery(String uri);
    /**
     * 返回URI的path部分
     */
    public static String getPath(String uri);
    /**
     * 返回URI的path和参数部分
     */
    public static String getPathQuery(String uri);
    /**
     * 获取地址的主机地址
     */
    public static String getHost(String uri);
    /**
     * 获取地址的域名信息
     */
    public static String getDomain(String uri);
}
```

**举例说明**：

```java
public static void main(String[] args) {
    String url = "http://hzeronb.saas.hand-china.com/iam/v1/users?tenantId=0&loginName=admin";

    System.out.println(DomainUtils.isValidDomain(url)); // true 
    System.out.println(DomainUtils.getDomain(url)); // http://hzeronb.saas.hand-china.com
    System.out.println(DomainUtils.getHost(url)); // hzeronb.saas.hand-china.com
    System.out.println(DomainUtils.getPath(url)); // /iam/v1/users
    System.out.println(DomainUtils.getQuery(url)); // tenantId=0&loginName=admin
    System.out.println(DomainUtils.getPathQuery(url)); // /iam/v1/users?tenantId=0&loginName=admin
    System.out.println(DomainUtils.getFromPath(url)); // /iam/v1/users?tenantId=0&loginName=admin
    System.out.println(DomainUtils.getQueryMap(url)); // {loginName=admin, tenantId=0}
}
```

## JsonUtils

JsonUtils 提供了几个转换 Json 字符串和对象的工具方法。

注意：JsonUtils 转换的 json 对象中如果带有日期类型，转换的结果会带有时区。

```java
public static void main(String[] args) {
    User user = new User();
    // 对象转为json字符串
    String json = JsonUtils.toJson(user);
    // json字符串转为对象
    user = JsonUtils.fromJson(json, User.class);

    List<User> list = new ArrayList<>();
    // 集合转json字符串
    String jsonList = JsonUtils.toJson(list);
    // json字符串转集合
    list = JsonUtils.fromJsonList(jsonList, User.class);
}
```



---

# 数据结构工具类

## 概述

`org.hzero.core.algorithm` 包下提供了一些数据结构，便于实现一些特殊的功能。

## 递归构建树

一些业务中，数据可能是一种树形结构，这时可以利用 tree 下的工具类将具有层级结构的数据使用递归关联起来，父对象中会有一个children列表对象来存放子子对象。

### 1、API说明

- Bean对象继承 `org.hzero.core.algorithm.tree.Child` 类，泛型为子对象的类型，也就是当前Bean的类。

- 接着就可以调用 `org.hzero.core.algorithm.tree.TreeBuilder.buildTree(...)` 静态方法来构建树了。

TreeBuilder.buildTree() 参数说明：

+ `List<T> objList` ： 该参数为原数据列表

+ `Node<P, T> nodeOperation` : 该参数为接口，继承了Key和ParentKey接口，需要自行实现，包含两个方法，getKey()方法用户获取当前节点的Key，getParentKey()用于获取父节点的Key，当前节点的key等于子节点的parentKey()时建立关联关系

+ `P rootKey` : 该参数为根节点的key，**这个参数非必须，但请尽可能提供这个参数，如果不提供，会多遍历一次列表取构建根节点**

+ `Key<P, T> key` : 获取当前节点的key

+ `ParentKey<P, T> parentKey` : 获取父节点的Key
        
### 2、使用示例

```java
class Entity extentds org.hzero.core.algorithm.tree.Child<Entity> {
    Integer id;
    Integer parentId;
    // other field ...
    // getter and setter ...
}

// method
List<Entity> objList = // from anywhere...
List<Entity> result = org.hzero.core.algorithm.tree.TreeBuilder
		.buildTree(objList, null, Entity::getId, Entity::getParentId)
```

```json
[                                                   [
    {                                                   {
        "id":1,                                             "id":1,
        "parentId":null,                                    "parentId":null,
        ...                                                 "children":[{
    },{                                                         "id":2,
        "id":2,                                                 "parentId":1,
        "parentId":1,               ==》》                          ...
        ...                                                 },{
    },{                                                         "id":3,
        "id":3,                                                 "parentId":1,
        "parentId":1,                                           ...
        ...                                                 }]
    }                                                       ...
    ...                                                 }
]                                                   ]
```

## 有序队列 LinkedQueue

> org.hzero.core.algorithm.structure.LinkedQueue

LinkedQueue 基于 LinkedList 实现，主要是支持有序队列，每次 add/append 元素之后的数据都是按照顺序排列的，每次插入时使用二分法查找插入的位置。

**举例说明**： 

在构造LinkedQueue 时至少需要提供一个 Comparator 比较器，用于数据排序。

```java
public static void main(String[] args) {
    // 无序
    LinkedList<Integer> unsort = new LinkedList<>();
    unsort.add(30);
    unsort.add(20);
    unsort.add(40);
    unsort.add(10);
    System.out.println(unsort); // [30, 20, 40, 10]

    // 有序
    LinkedQueue<Integer> sorted = new LinkedQueue<>(Comparator.comparingInt(o -> o));
    sorted.add(30);
    sorted.add(20);
    sorted.add(40);
    sorted.add(10);
    System.out.println(sorted); // [10, 20, 30, 40]
}
```

## 可设置有效期的Map：TTLMap

我们通常会使用 HashMap 来存储键值对，如果期望 Map 中的 key 在一定时间后自动过期，就可以使用 `org.hzero.core.algorithm.structure.TTLMap`。

当一个key放到 TTLMap 超过一段时间(ttl x timeUnit)后，再次get的时候会返回 null 值。

**举例说明**：

```java
public static void main(String[] args) throws InterruptedException {
    // 构建 TTLMap
    TTLMap<String, String> ttlMap = new TTLMap.Builder<String, String>()
            .ttl(5) // 测试设置5秒后过期
            .timeUnit(TimeUnit.SECONDS)
            .build();

    ttlMap.put("key", "ttl_map");

    System.out.println(ttlMap.get("key")); // ttl_map

    Thread.sleep(5000);

    System.out.println(ttlMap.get("key")); // null
}
```

















---

# JSON 相关

## 概述

`org.hzero.core.jackson` 包下提供了一些 JSON 增强组件，封装了一些jackson序列化和反序列化的操作，支持Java8时间格式的序列化，使用前需要在Spring Boot启动类上添加`org.hzero.core.jackson.annotation.EnableObjectMapper`注解。

## 忽略时区转换

默认情况下，所有 `java.util.Date` 在序列化和反序列化时会按照当前用户设置的时区对时间进行转换(固定格式`yyyy-MM-dd HH:mm:ss`)，如果时间不需要做转换，可以使用该功能忽略时区转换。

**使用说明：**

- 序列化和反序列化时忽略时区转换：在字段上添加 `@org.hzero.core.jackson.annotation.IgnoreTimeZone` 注解

- 序列化时忽略时区转换：在字段上添加 `@com.fasterxml.jackson.databind.annotation.JsonSerialize(using = org.hzero.core.jackson.serializer.IgnoreTimeZoneDateSerializer)` 注解

- 反序列化时忽略时区转换：在字段上添加 `@com.fasterxml.jackson.databind.annotation.JsonDeserialize(using = org.hzero.core.jackson.serializer.IgnoreTimeZoneDateDeserializer)` 注解

## 字符串两端空格过滤

前端往后端传输数据时，有些字段需要过滤两端空格，例如编码等，添加注解可以在反序列化时过滤掉空格。

**使用说明：**

在字段上添加注解 `@org.hzero.core.jackson.annotation.Trim` 注解，可以指定不同的过滤策略，默认两端过滤，也可以设置左或者右一端。

## 敏感信息加密

后端往前端传输数据时，有些数据可能需要按照某种规则屏蔽掉敏感信息，例如电话号码中间四位显示 “*”。

**使用说明：**

- **在数据返回前端之前，调用 `org.hzero.core.jackson.sensitive.SensitiveHelper.open()`** 方法，开启敏感信息屏蔽，当前线程内有效，一次序列化之后自动关闭。

- 在返回Bean的字段上添加`@org.hzero.core.jackson.annotation.Sensitive`注解，注解中可以指定多种规则取屏蔽敏感信息。
	+ left : 从左边开始前 n 位替换
	+ right : 从右边开始后 n 位替换
	+ cipher : 复杂密文规则，下标从1开始，屏蔽某一位字符直接指定下标即可；屏蔽某一个范围可用 `下标1-下标2`，或者 `下标1-` 表示屏蔽某一位及之后所有；多个规则之间可以用`,`分割，或者用数组形式传递。例如：`4-7,9,11-` 表示第 4，5，6，7，9，11 以及11之后的位使用加密字符替换
	+ clear : 明文规则，使用方式同 cipher，结果相反
	+ symbol : 加密字符，默认 `*`
	+ reverse : 规则反转，指定 cipher/clear 时无效。

---

# 加密解密工具

## 概述

在项目中，我们可能经常会用到加密相关的一些需求，`org.hzero.core.util.EncryptionUtils` 工具提供了多种加密方式。

EncryptionUtils 提供了 MD5 非对称加密、AES 对称加密、RSA 对称加密、RSA2 对称加密、XOR 异或加密 等几种加密方式。

## 密码学基础

我们可能经常会听到对称加密、非对称加密等，包括加密算法，这些都属于密码学的范畴，下面对这些概念做一些简单的介绍，便于理解我们提供的几种加密方式。

### 1、对称加密算法

对称加密，加密和解密使用的是同一个密钥。


<img src="https://file.open.hand-china.com/hsop-doc/doc_classify/0/7c3ef47faa6b40c1b7ff4caba492bca2@image.png" alt="" width="auto" height="auto" />

<p/>

常见的经典对称加密算法有 DES、IDEA、AES、国密 SM1 和 SM4。其中，AES（高级加密标准，Advanced  Encryption  Standard）：提供了 128 位、192 位和 256 位三种密钥长度。通常情况下，我们会使用 128 位的密钥，来获得足够的加密强度，同时保证性能不受影响。目前，AES 是国际上最认可的密码学算法。

### 2、非对称加密算法

非对称加密，加密和解密使用不同的密钥。在非对称加密算法中，公钥是公开信息，不需要保密，我们可以简单地将一个公钥分发给全部的通信方。非对称密钥其实主要解决了密钥分发的难题。


<img src="https://file.open.hand-china.com/hsop-doc/doc_classify/0/b79b0296be8948f9bab2f26b51dc1e21@image.png" alt="" width="auto" height="auto" />


<p/>

所有的非对称加密算法，都是基于各种数学难题来设计的，这些数学难题的特点是：正向计算很容易，反向推倒则无解。经典的非对称加密算法包括：RSA、ECC 和国密 SM2。

### 3、散列算法

散列算法应该是最常见到的密码学算法了。大量的应用都在使用 MD5 或者 SHA 算法计算一个唯一的 id。很多场景下，我们使用散列算法并不是为了满足什么加密需求，而是利用它可以对任意长度的输入，计算出一个定长的 id。作为密码学的算法，散列算法除了提供唯一的 id，其更大的利用价值还在于它的不可逆性。

经典的散列算法包括 MD5、SHA、国密 SM3。SHA 相比 MD5 安全度更高，但性能更低。


## EncryptionUtils

### 1、MD5 散列加密

EncryptionUtils.MD5 提供了如下的三个加密方法，返回加密后的字符串。

```java
public class EncryptionUtils {

    public static class MD5 {
        /**
         * MD5 加密
         *
         * @param content 加密内容
         * @return 加密结果
         */
        public static String encrypt(String content);

        /**
         * MD5 加密
         *
         * @param content 加密内容
         * @return 加密结果
         */
        public static String encrypt(byte[] content);

        /**
         * MD5 加密
         *
         * @param contentStream 加密内容
         * @return 加密结果
         */
        public static String encrypt(InputStream contentStream);
    }
}
```

**场景1**：使用MD5加密

MD5对同样的内容多次加密返回的字符串是一样的。

```java
public static void main(String[] args) {
    String content = "http://dev.hzero.com";
    String str1 = EncryptionUtils.MD5.encrypt(content);
    String str2 = EncryptionUtils.MD5.encrypt(content);
    System.out.println(str1); // a7b2d4aacdae7c897d4a95f775b5a96c
    System.out.println(str1.equals(str2)); // true
}
```

### 2、AES 对称加密

AES 加密首先要生成一个密钥，这个密钥用于加密、解密。

```java
public class EncryptionUtils {

    public static class AES {
        /**
         * 生成秘钥
         */
        public static String generaterKey();
        /**
         * 生成秘钥
         */
        public static String generaterKey(int keySize);
        /**
         * 生成密钥
         */
        private static SecretKeySpec getSecretKeySpec(String secretKeyStr);
        /**
         * 加密
         */
        public static String encrypt(String content, String secretKey);

        public static String encryptWithUrlEncoder(String content, String secretKey);

        /**
         * 解密
         */
        public static String decrypt(String content, String secretKey);
        /**
         * 解密
         */
        public static String decryptWithUrlDecoder(String content, String secretKey);
    }
}
```

**场景1**：生成密钥，加密、解密

```java
public static void main(String[] args) {
    // 密钥
    String secretKey = EncryptionUtils.AES.generaterKey();

    String content = "http://dev.hzero.com";
    // 加密
    String encrypt = EncryptionUtils.AES.encrypt(content, secretKey);
    // 解密
    String decrypt = EncryptionUtils.AES.decrypt(encrypt, secretKey);

    System.out.println(decrypt.equals(content)); // true
}
```
> AES 模式使用的 ECB 模式，项目可通过修改如下配置调整加密模式，以 CBC 模式为例
```
hzero:
  encryption:
    aes:
      encryptionAlgorithm: AES/CBC/PKCS5Padding
```
**需要注意的是，Mybatis增强组件的数据加密存储功能也使用了该加密方法，所以不建议在有历史的情况下修改加密模式。若确实需要修改加密模式请注意调整历史数据**

### 3、RSA 非对称加密


EncryptionUtils.RSA 用于生成一个密钥对，支持利用公钥、私钥加密解密，支持加盐加密解密。

```
public class EncryptionUtils {

   public static class RSA {

        /**
         * 生成秘钥对
         *
         * @return first : 私钥 / second : 公钥
         */
        public static Pair<String, String> generateKeyPair();
        /**
         * 生成秘钥对
         *
         * @param keySize 密钥大小
         * @return first : 私钥/second : 公钥
         */
        public static Pair<String, String> generateKeyPair(int keySize);
        /**
         * 获取公钥
		 *
         * @param publicKey 公钥
         * @return 公钥
         */
        public static RSAPublicKey getPublicKey(String publicKey);
        /**
         * 获取私钥
         *
         * @param privateKey 私钥
         * @return 私钥
         */
        public static RSAPrivateKey getPrivateKey(String privateKey);
        /**
         * 私钥签名内容
         *
         * @param content    内容
         * @param privateKey 私钥
         * @return 私钥签名
         */
        public static String sign(String content, String privateKey);
        /**
         * 公钥校验签名
         *
         * @param content   内容
         * @param sign      签名
         * @param publicKey 公钥
         * @return 是否匹配
         */
        public static boolean verify(String content, String sign, String publicKey);
        /**
         * 使用公钥或者私钥加密
         *
         * @param content 内容
         * @param key     公钥或者私钥
         * @return 密文
         */
        public static String encrypt(String content, Key key);
        /**
         * 使用公钥或者私钥解密
         *
         * @param content 内容
         * @param key     公钥或者私钥
         * @return 明文
         */
        public static String decrypt(String content, Key key);
        /**
         * 使用公钥加密
         *
         * @param content   明文
         * @param publicKey 公钥
         * @return 密文
         */
        public static String encrypt(String content, String publicKey);
        /**
         * 使用公钥加盐加密
         *
         * @param content   明文
         * @param key       盐
         * @param publicKey 公钥
         * @return 密文
         */
        public static String encrypt(String content, String key, String publicKey);
        /**
         * 使用公钥加盐加密
         *
         * @param content   明文
         * @param key       盐
         * @param publicKey 公钥
         * @param withSalt  明文加盐
         * @return 密文
         */
        public static String encrypt(String content, String key, String publicKey, WithSalt withSalt);
        /**
         * 使用私钥解密
         *
         * @param content    密文
         * @param privateKey 私钥
         * @return 明文
         */
        public static String decrypt(String content, String privateKey);
        /**
         * 使用私钥解密去盐
         *
         * @param content    密文
         * @param key        盐
         * @param privateKey 私钥
         * @return 明文
         */
        public static String decrypt(String content, String key, String privateKey);
        /**
         * 使用私钥解密去盐
         *
         * @param content     密文
         * @param key         盐
         * @param privateKey  私钥
         * @param withoutSalt 解密内容去盐
         * @return 明文
         */
        public static String decrypt(String content, String key, String privateKey, WithoutSalt withoutSalt);
    }
}
```

**场景1**：生成密钥对

```
public static void main(String[] args) {
    // 生成密钥对
    Pair<String, String> pair =  EncryptionUtils.RSA.generateKeyPair();
    // 私钥
    String privateKey = pair.getFirst();
    // 公钥
    String publicKey = pair.getSecond();

    System.out.println("privateKey=" + privateKey);
    System.out.println("publicKey=" + publicKey);
}
```

**场景2**：公钥加密，私钥解密

```java
public static void main(String[] args) {
    // 私钥
    String privateKey = "...";
    // 公钥
    String publicKey = "....";

    String content = "http://dev.hzero.com";
    // 公钥加密
    String encrypt = EncryptionUtils.RSA.encrypt(content, publicKey);
    // 私钥解密
    String decrypt = EncryptionUtils.RSA.decrypt(encrypt, privateKey);

    System.out.println(decrypt.equals(content)); // true
}
```

**场景3**：加盐加密、解密

```java
public static void main(String[] args) {
    // 私钥
    String privateKey = "...";
    // 公钥
    String publicKey = "...";
    // 盐
    String salt = "hzero";

    String content = "http://dev.hzero.com";
    // 公钥加盐加密
    String encrypt = EncryptionUtils.RSA.encrypt(content, salt, publicKey);
    // 私钥去盐解密
    String decrypt = EncryptionUtils.RSA.decrypt(encrypt, salt, privateKey);

    System.out.println(decrypt.equals(content)); // true
}
```


### 4、RSA2 非对称加密

RSA2 相比 RSA 的主要区别在于签名算法的差异，RSA 使用 `SHA1WithRSA`，RSA2 使用 `SHA256WithRSA`。

```java
public class EncryptionUtils {

    public static class RSA2 {
        /**
         * 生成秘钥对
         *
         * @return first : 私钥/second : 公钥
         */
        public static Pair<String, String> generateKeyPair();
        /**
         * 获取公钥
         *
         * @param publicKey 公钥
         * @return 公钥
         */
        private static RSAPublicKey getPublicKey(String publicKey);
        /**
         * 获取私钥
         *
         * @param privateKey 私钥
         * @return 私钥
         */
        private static RSAPrivateKey getPrivateKey(String privateKey);
        /**
         * 私钥签名内容
         *
         * @param content    内容
         * @param privateKey 私钥
         * @return 私钥签名
         */
        public static String sign(String content, String privateKey);
        /**
         * 公钥校验签名
         *
         * @param content   内容
         * @param sign      签名
         * @param publicKey 公钥
         * @return 是否匹配
         */
        public static boolean verify(String content, String sign, String publicKey);
        /**
         * 使用公钥或者私钥加密
         *
         * @param content 内容
         * @param key     公钥或者私钥
         * @return 密文
         */
        public static String encrypt(String content, Key key);
        /**
         * 使用公钥或者私钥解密
         *
         * @param content 内容
         * @param key     公钥或者私钥
         * @return 明文
         */
        public static String decrypt(String content, Key key);
    }
}
```

**场景1**：签名、验证签名

注意：sign 签名时是使用私钥签名，verify 验证使用公钥验证.

```java
public static void main(String[] args) {
    Pair<String, String> pair =  EncryptionUtils.RSA2.generateKeyPair();
    // 私钥
    String privateKey = pair.getFirst();
    // 公钥
    String publicKey = pair.getSecond();

    String content = "http://dev.hzero.com";
    // 私钥签名
    String sign = EncryptionUtils.RSA2.sign(content, privateKey);
    // 公钥验证
    boolean verify = EncryptionUtils.RSA2.verify(content, sign, publicKey);

    System.out.println(verify); // true
}
```

### 5、XOR 异或加密

XOR 用于对数字异或加密，首先需要生成一个 long 类型的密钥，然后将 long 类型的数据加密成字符串，之后解密成 long。

```java
public class EncryptionUtils {

    public static class XOR {
		// 生成密钥
        public static long generateKey();
		// 加密
        public static String encrypt(long key, long content);
		// 解密
        public static long decrypt(long key, String encrypt);
    }
}
```

**场景1**：对数字异或加密

```java
public static void main(String[] args) {
    // 密钥
    long secretKey = EncryptionUtils.XOR.generateKey();

    long content = 666666666L;
    // 加密
    String encrypt = EncryptionUtils.XOR.encrypt(secretKey, content);
    // 解密
    long decrypt = EncryptionUtils.XOR.decrypt(secretKey, encrypt);

    System.out.println(encrypt); // 30475246049
    System.out.println(decrypt == content); // true
}
```




---

# 常用注解

## 概述

提供一些功能性注解支持。

## 属性判断注解

某些情况下可能有需要判断 application.yml 配置文件中是否存在某个配置的情况，这时可以用如下两个注解。

* `@ConditionalOnExistingProperty`：用于判断是否存在某个属性配置：

* `@ConditionalOnMissingProperty`：用于判断是否不存在某个属性配置：

**场景举例：**

例如在创建 bean 对象时判断某些属性存在或不存在时才生效。

```
@Bean
@ConditionalOnMissingProperty("spring.redis.cluster.nodes")
public RedisHelper redisHelper() {
    return new DynamicRedisHelper();
}

@Bean
@ConditionalOnExistingProperty("spring.redis.cluster.nodes")
public RedisHelper redisHelper() {
    return new RedisHelper();
}
```






---

