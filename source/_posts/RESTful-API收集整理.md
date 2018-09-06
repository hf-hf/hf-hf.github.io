---
title: RESTful API收集整理
date: 2018-09-06 15:48:34
tags:
    - RESTful
---
![homePage](/upload/homePage/20180906170330.jpg)
<!--more-->
# RESTful API 
# 路径定义
## 版本号
在 RESTful API 中，API 接口应该尽量兼容之前的版本。但是，在实际业务开发场景中，可能随着业务需求的不断迭代，现有的 API 接口无法支持旧版本的适配，此时如果强制升级服务端的 API 接口将导致客户端旧有功能出现故障。实际上，Web 端是部署在服务器，因此它可以很容易为了适配服务端的新的 API 接口进行版本升级，然而像 Android 端、IOS 端、PC 端等其他客户端是运行在用户的机器上，因此当前产品很难做到适配新的服务端的 API 接口，从而出现功能故障，这种情况下，用户必须升级产品到最新的版本才能正常使用。

为了解决这个版本不兼容问题，在设计 RESTful API 的一种实用的做法是使用版本号。一般情况下，我们会在 url 中保留版本号，并同时兼容多个版本。

```
【GET】  /v1/users/{user_id}  // 版本 v1 的查询用户列表的 API 接口
【GET】  /v2/users/{user_id}  // 版本 v2 的查询用户列表的 API 接口
```

现在，我们可以不改变版本 v1 的查询用户列表的 API 接口的情况下，新增版本 v2 的查询用户列表的 API 接口以满足新的业务需求，此时，客户端的产品的新功能将请求新的服务端的 API 接口地址。虽然服务端会同时兼容多个版本，但是同时维护太多版本对于服务端而言是个不小的负担，因为服务端要维护多套代码。这种情况下，常见的做法不是维护所有的兼容版本，而是只维护最新的几个兼容版本，例如维护最新的三个兼容版本。在一段时间后，当绝大多数用户升级到较新的版本后，废弃一些使用量较少的服务端的老版本API 接口版本，并要求使用产品的非常旧的版本的用户强制升级。

注意的是，“不改变版本 v1 的查询用户列表的 API 接口”主要指的是对于客户端的调用者而言它看起来是没有改变。而实际上，如果业务变化太大，服务端的开发人员需要对旧版本的 API 接口使用适配器模式将请求适配到新的API 接口上。

## 资源路径
RESTful API 的设计以资源为核心，每一个 URI 代表一种资源。因此，URI 不能包含动词，只能是名词。注意的是，形容词也是可以使用的，但是尽量少用。一般来说，不论资源是单个还是多个，API 的名词要以复数进行命名。此外，命名名词的时候，要使用小写、数字及下划线来区分多个单词。这样的设计是为了与 json 对象及属性的命名方案保持一致。例如，一个查询系统标签的接口可以进行如下设计。

```
【GET】  /v1/tags/{tag_id} 
```

同时，资源的路径应该从根到子依次如下。

```
/{resources}/{resource_id}/{sub_resources}/{sub_resource_id}/{sub_resource_property}
```

我们来看一个“添加用户的角色”的设计，其中“用户”是主资源，“角色”是子资源。

```
【POST】  /v1/users/{user_id}/roles/{role_id} // 添加用户的角色
```

有的时候，当一个资源变化难以使用标准的 RESTful API 来命名，可以考虑使用一些特殊的 actions 命名。

```
/{resources}/{resource_id}/actions/{action}
```

举个例子，“密码修改”这个接口的命名很难完全使用名词来构建路径，此时可以引入 action 命名。

```
【PUT】  /v1/users/{user_id}/password/actions/modify // 密码修改
```

当路径中出现多个单词时，请使用中划线 `-` 来连接。

```
【GET】  /v1/order/my-orders // 获取我的订单信息
```

## 请求方式
可以通过 GET、 POST、 PUT、 PATCH、 DELETE 等方式对服务端的资源进行操作。其中，GET 用于查询资源，POST 用于创建资源，PUT 用于更新服务端的资源的全部信息，PATCH 用于更新服务端的资源的部分信息，DELETE 用于删除服务端的资源。

这里，笔者使用“用户”的案例进行回顾通过 GET、 POST、 PUT、 PATCH、 DELETE 等方式对服务端的资源进行操作。

```
【GET】          /users                # 查询用户信息列表
【GET】          /users/1001           # 查看某个用户信息
【POST】         /users                # 新建用户信息
【PUT】          /users/1001           # 更新用户信息(全部字段)
【PATCH】        /users/1001           # 更新用户信息(部分字段)
【DELETE】       /users/1001           # 删除用户信息
```

## 查询参数
RESTful API 接口应该提供参数，过滤返回结果。其中，offset 指定返回记录的开始位置。一般情况下，它会结合 limit 来做分页的查询，这里 limit 指定返回记录的数量。

```
【GET】  /{version}/{resources}/{resource_id}?offset=0&limit=20
```

同时，orderby 可以用来排序，但仅支持单个字符的排序，如果存在多个字段排序，需要业务中扩展其他参数进行支持。

```
【GET】  /{version}/{resources}/{resource_id}?orderby={field} [asc|desc]
```

为了更好地选择是否支持查询总数，我们可以使用 count 字段，count 表示返回数据是否包含总条数，它的默认值为 false。

```
【GET】  /{version}/{resources}/{resource_id}?count=[true|false]
```

上面介绍的 offset、 limit、 orderby 是一些公共参数。此外，业务场景中还存在许多个性化的参数。我们来看一个例子。

```
【GET】  /v1/categorys/{category_id}/apps/{app_id}?enable=[1|0]&os_type={field}&device_ids={field,field,…}
```

注意的是，不要过度设计，只返回用户需要的查询参数。此外，需要考虑是否对查询参数创建数据库索引以提高查询性能。

## 状态码
使用适合的状态码很重要，而不应该全部都返回状态码 200，或者随便乱使用。这里，列举笔者在实际开发过程中常用的一些状态码，以供参考。

|状态码|描述|
|----|---:|
|200|请求成功|
|201|创建成功|
|400|错误的请求|
|401|未验证|
|403|被拒绝|
|404|无法找到|
|409|资源冲突|
|500|服务器内部错误|

## 异常响应

当 RESTful API 接口出现非 2xx 的 HTTP 错误码响应时，采用全局的异常结构响应信息。

```
HTTP/1.1 400 Bad Request
Content-Type: application/json
{
    "code": "INVALID_ARGUMENT",
    "message": "{error message}",
    "cause": "{cause message}",
    "request_id": "01234567-89ab-cdef-0123-456789abcdef",
    "host_id": "{server identity}",
    "server_time": "2014-01-01T12:00:00Z"
}
```

## 请求参数
在设计服务端的 RESTful API 的时候，我们还需要对请求参数进行限制说明。例如一个支持批量查询的接口，我们要考虑最大支持查询的数量。

```
【GET】     /v1/users/batch?user_ids=1001,1002      // 批量查询用户信息
参数说明
- user_ids: 用户ID串，最多允许 20 个。
```

此外，在设计新增或修改接口时，我们还需要在文档中明确告诉调用者哪些参数是必填项，哪些是选填项，以及它们的边界值的限制。

```
【POST】     /v1/users                             // 创建用户信息
请求内容
{
    "username": "lgz",                 // 必填, 用户名称, max 10
    "realname": "梁桂钊",               // 必填, 用户名称, max 10
    "password": "123456",              // 必填, 用户密码, max 32
    "email": "lianggzone@163.com",     // 选填, 电子邮箱, max 32
    "weixin": "LiangGzone",            // 选填，微信账号, max 32
    "sex": 1                           // 必填, 用户性别[1-男 2-女 99-未知]
}
```

## 响应参数
针对不同操作，服务端向用户返回的结果应该符合以下规范。

```
【GET】     /{version}/{resources}/{resource_id}      // 返回单个资源对象
【GET】     /{version}/{resources}                    // 返回资源对象的列表
【POST】    /{version}/{resources}                    // 返回新生成的资源对象
【PUT】     /{version}/{resources}/{resource_id}      // 返回完整的资源对象
【PATCH】   /{version}/{resources}/{resource_id}      // 返回完整的资源对象
【DELETE】  /{version}/{resources}/{resource_id}      // 状态码 200，返回完整的资源对象。
                                                      // 状态码 204，返回一个空文档
```

如果是单条数据，则返回一个对象的 JSON 字符串。

```
HTTP/1.1 200 OK
{
    "id" : "01234567-89ab-cdef-0123-456789abcdef",
    "name" : "example",
    "created_time": 1496676420000,
    "updated_time": 1496676420000,
    ...
}
```

如果是列表数据，则返回一个封装的结构体。

```
HTTP/1.1 200 OK
{
    "count":100,
    "items":[
        {
            "id" : "01234567-89ab-cdef-0123-456789abcdef",
            "name" : "example",
            "created_time": 1496676420000,
            "updated_time": 1496676420000,
            ...
        },
        ...
    ]
}
```

## 一个完整的案例
最后，我们使用一个完整的案例将前面介绍的知识整合起来。这里，使用“获取用户列表”的案例。

```
【GET】     /v1/users?[&keyword=xxx][&enable=1][&offset=0][&limit=20] 获取用户列表
功能说明：获取用户列表
请求方式：GET
参数说明
- keyword: 模糊查找的关键字。[选填]
- enable: 启用状态[1-启用 2-禁用]。[选填]
- offset: 获取位置偏移，从 0 开始。[选填]
- limit: 每次获取返回的条数，缺省为 20 条，最大不超过 100。 [选填]
响应内容
HTTP/1.1 200 OK
{
    "count":100,
    "items":[
        {
            "id" : "01234567-89ab-cdef-0123-456789abcdef",
            "name" : "example",
            "created_time": 1496676420000,
            "updated_time": 1496676420000,
            ...
        },
        ...
    ]
}
失败响应
HTTP/1.1 403 UC/AUTH_DENIED
Content-Type: application/json
{
    "code": "INVALID_ARGUMENT",
    "message": "{error message}",
    "cause": "{cause message}",
    "request_id": "01234567-89ab-cdef-0123-456789abcdef",
    "host_id": "{server identity}",
    "server_time": "2014-01-01T12:00:00Z"
}
错误代码
- 403 UC/AUTH_DENIED    授权受限
```

# 参数接收

本篇文章主要说下接口的数据参数到底该如何接收，我们知道一个http请求最重要的意义就是将数据在服务器上进行传入与传出，本章主要讲的也就是传入。一次请求传递参数的方式主要有 URL路径中、请求头中、请求体中还有通过cookie等，下面我们分别对几种方式进行讲解。

## MediaType的选择
MediaType即是Internet Media Type，互联网媒体类型；也叫做MIME类型，在Http协议消息头中，使用Content-Type来表示具体请求中的媒体类型信息。

对于POST、PUT、PATCH这种HTTP方法，统一使用 application/json，将参数放在请求体中以JSON格式传递至服务器
对于GET、DELETE的HTTP方法，使用默认类型（application/x-www-form-urlencoded）
★备注 
特殊情况特殊考虑，例如进行文件上传时，使用 multipart/form-data类型等

## 路径参数
对应spring mvc框架中@PathVariable注解

接收参数方式为：
```
public ResponseMessage get(@PathVariable("customerId")int customerId){
	//...
}
```

## 请求头参数
对应spring mvc框架中@RequestHeader注解

对于提供给APP（android、ios、pc） 的接口我们可能需要关注一些调用信息，例如 用户登录信息、调用来源、app版本号、api的版本号、安全验证信息 等等，我们将这些信息放入头信息（HTTP HEAD中），下面给出在参数命名的例子：

Access-Token 用户的登录token（用于置换用户登录信息）
Api-Version api的版本号
App-Version app版本号
Call-Source 调用来源(IOS、ANDROID、PC、WECHAT、WEB)
Authorization 安全校验参数（后面会有文章详细介绍该如何做安全校验）
这时你可能会思考一下几个问题：

为什么需要收集 api版本号、app版本号、调用来源这些信息呢？ 
这里解释下，主要有几个原因： 
①方便线上环境定位问题，这也是一个重要的原因（我们后面会讲通过切面全局打印非GET请求的接口调用日志）。
②我们可以通过这些参数信息处理我们的业务逻辑，而没有必要在用到的时候我们才想起来让调用者将信息传递过来，导致同一功能性的参数，参数名和参数值不统一的情况发生。
是每个接口都要这些参数么？ 
是的，建议将所有的接口都传递上述参数信息。
怎么做这些参数的校验呢？ 
你可以写个拦截器，统一校验你的接口中全局的header参数
★备注 
- Header参数大小写不敏感，所以参数Access-Token和ACCESS-TOKEN是一个参数

## 请求体参数
参数传递分为了大体两种 URL请求查询参数、请求体参数，对于请求体参数我们选择以JSON格式传递过来，URL请求查询参数、请求体参数这两种方式分别对应了spring mvc框架中的 @RequestParam、@RequestBody两个注解进行修饰。 
我们下面以添加一个用户举例，用POSTMAN截图如下：

![创建用户](http://upload-images.jianshu.io/upload_images/7477749-c6fdcdc835d413f7?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 开发实例
```
import javax.validation.Valid;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.PutMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseStatus;
import org.springframework.web.bind.annotation.RestController;

import com.zhuma.demo.comm.model.po.User;
import com.zhuma.demo.user.service.user.UserService;
import com.github.pagehelper.PageInfo;

/**
 * 用户管理控制器
 */
@RestController
@RequestMapping("/users")
public class UserController {

    private final UserService userService;

    @Autowired
    public UserController(UserService userService) {
        this.userService = userService;
    }

    @GetMapping
    public PageInfo<User> getUserList(@RequestParam(name="pageNum", defaultValue="1") Integer pageNum, 
                                      @RequestParam(name="pageSize", defaultValue="10") Integer pageSize,
                                        @RequestParam(name="queryUser") User queryUser) {
        return userService.pageList(queryUser, pageNum, pageSize);
    }

    @GetMapping("/{userId}")
    User getUser(@PathVariable("userId") Long userId) {
        return userService.getUserById(userId);
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public User addUser(@Valid @RequestBody User user) {
        return userService.register(user);
    }

    @PutMapping
    public User updateUser(@RequestBody User user) {
        return userService.updateDbAndCache(user);
    }

    @DeleteMapping("/{userId}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    void deleteUser(@PathVariable("userId") Long userId) {
        userService.deleteUserById(userId);
    }

```

上面我们截取一个管理用户的功能控制器，其中可能有一些你不了解的注解，例如@Valid、@ResponseStatus，@Valid一般配合参数校验使用，@ResponseStatus直接修饰目标方法，不管是否触发异常，都会返回指定的异常和错误码，这里我们主要看@GetMapping、@PostMapping、 @PutMapping、@DeleteMapping 分别处理查询、修改、删除功能，@RequestParam、@RequestBody 分别表示 查询参数、json body请求体参数。

# 返回值

## 格式选择
返回格式目前主流的应该只有XML、JSON两种吧，这里我们不做对比，我们使用JSON作为接口的返回格式。

## 数据返回格式
数据的返回格式其实是个比较纠结的问题，在restful风格中很多文章都讲解使用的是http状态码控制请求的结果状态，例如：http状态码为200~300的时候，为正常状态，response响应体即为所需要返回的数据，404时代表没有查询到数据，响应体即为空，500为系统错误，响应体也为空等等，但是这种方式也是存在很大问题的，一是http状态码是有限的，而且每个状态码都已经被赋予特殊的含义，在企业开发中当接口遇到错误的时候，我们可能更希望将结果状态码标记的更为详细，更利于前端开发者使用，毕竟写接口的目的也是方便前端使用，这样也可以降低前后端开发人员沟通成本。 
下面给出我们在实际开发中使用的返回值统一格式：
```
public class ResponseMessage implements Serializable {
    private static final long serialVersionUID = 8992436576262574064L;
    /**
     * 是否成功
     */
    private boolean success;

    /**
     * 反馈数据
     */
    private Object data;

    /**
     * 反馈信息
     */
    private String message;

    /**
     * 响应码
     */
    private int code;

    /**
     * 过滤字段：指定需要序列化的字段
     */
    private transient Map<Class<?>, Set<String>> includes;

    /**
     * 过滤字段：指定不需要序列化的字段
     */
    private transient Map<Class<?>, Set<String>> excludes;

    private transient boolean onlyData;

    private transient String callback;

    public Map<String, Object> toMap() {
        Map<String, Object> map = new HashMap<>();
        map.put("success", this.success);
        if (data != null)
            map.put("data", this.getData());
        if (message != null)
            map.put("message", this.getMessage());
        map.put("code", this.getCode());
        return map;
    }

    protected ResponseMessage() {}
    protected ResponseMessage(String message) {
        this.code = 500;
        this.message = message;
        this.success = false;
    }

    protected ResponseMessage(boolean success, Object data) {
        this.code = success ? 200 : 500;
        this.data = data;
        this.success = success;
    }

    protected ResponseMessage(boolean success, Object data ,Object message) {
        this.code = success ? 200 : 500;
        this.data = data;
        this.success = success;
        this.message = (String)message;
    }

    protected ResponseMessage(boolean success, Object data, int code) {
        this(success, data);
        this.code = code;
    }



    public ResponseMessage include(Class<?> type, String... fields) {
        return include(type, Arrays.asList(fields));
    }

    public ResponseMessage include(Class<?> type, Collection<String> fields) {
        if (includes == null)
            includes = new HashMap<>();
        if (fields == null || fields.isEmpty()) return this;
        fields.forEach(field -> {
            if (field.contains(".")) {
                String tmp[] = field.split("[.]", 2);
                try {
                    Field field1 = type.getDeclaredField(tmp[0]);
                    if (field1 != null) {
                        include(field1.getType(), tmp[1]);
                    }
                } catch (Throwable e) {
                }
            } else {
                getStringListFormMap(includes, type).add(field);
            }
        });
        return this;
    }

    public ResponseMessage exclude(Class<?> type, Collection<String> fields) {
        if (excludes == null)
            excludes = new HashMap<>();
        if (fields == null || fields.isEmpty()) return this;
        fields.forEach(field -> {
            if (field.contains(".")) {
                String tmp[] = field.split("[.]", 2);
                try {
                    Field field1 = type.getDeclaredField(tmp[0]);
                    if (field1 != null) {
                        exclude(field1.getType(), tmp[1]);
                    }
                } catch (Throwable e) {
                }
            } else {
                getStringListFormMap(excludes, type).add(field);
            }
        });
        return this;
    }

    public ResponseMessage exclude(Collection<String> fields) {
        if (excludes == null)
            excludes = new HashMap<>();
        if (fields == null || fields.isEmpty()) return this;
        Class<?> type;
        if (data != null) type = data.getClass();
        else return this;
        exclude(type, fields);
        return this;
    }

    public ResponseMessage include(Collection<String> fields) {
        if (includes == null)
            includes = new HashMap<>();
        if (fields == null || fields.isEmpty()) return this;
        Class<?> type;
        if (data != null) type = data.getClass();
        else return this;
        include(type, fields);
        return this;
    }

    public ResponseMessage exclude(Class<?> type, String... fields) {
        return exclude(type, Arrays.asList(fields));
    }


    public ResponseMessage exclude(String... fields) {
        return exclude(Arrays.asList(fields));
    }

    public ResponseMessage include(String... fields) {
        return include(Arrays.asList(fields));
    }

    protected Set<String> getStringListFormMap(Map<Class<?>, Set<String>> map, Class<?> type) {
        Set<String> list = map.get(type);
        if (list == null) {
            list = new HashSet<>();
            map.put(type, list);
        }
        return list;
    }

    public boolean isSuccess() {
        return success;
    }

    public void setSuccess(boolean success) {
        this.success = success;
    }

    public Object getData() {
        return data;
    }

    public ResponseMessage setData(Object data) {
        this.data = data;
        return this;
    }

    @Override
    public String toString() {
        return JSON.toJSONStringWithDateFormat(this, DateTimeUtils.YEAR_MONTH_DAY_HOUR_MINUTE_SECOND);
    }

    public int getCode() {
        return code;
    }

    public ResponseMessage setCode(int code) {
        this.code = code;
        return this;
    }

    public static ResponseMessage fromJson(String json) {
        return JSON.parseObject(json, ResponseMessage.class);
    }

    public Map<Class<?>, Set<String>> getExcludes() {
        return excludes;
    }

    public Map<Class<?>, Set<String>> getIncludes() {
        return includes;
    }

    public ResponseMessage onlyData() {
        setOnlyData(true);
        return this;
    }

    public void setOnlyData(boolean onlyData) {
        this.onlyData = onlyData;
    }

    public boolean isOnlyData() {
        return onlyData;
    }

    public ResponseMessage callback(String callback) {
        this.callback = callback;
        return this;
    }

    public String getCallback() {
        return callback;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }

    public static ResponseMessage ok() {
        return ok(null);
    }

    public static ResponseMessage created(Object data) {
        return new ResponseMessage(true, data, 201);
    }

    public static ResponseMessage ok(Object data) {
        return new ResponseMessage(true, data);
    }

    public static ResponseMessage error(String message) {
        return new ResponseMessage(message);
    }

    public static ResponseMessage error(String message, int code) {
        return new ResponseMessage(message).setCode(code);
    }

}
```