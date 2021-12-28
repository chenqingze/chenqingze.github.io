# web api设计规约  

##  前言：

参考资料：

《Web API的设计与开发》水野贵明 著，盛荣 译；

[Hypertext Transfer Protocol -- HTTP/1.1](https://tools.ietf.org/html/rfc2616)

[REST API v3](https://developer.github.com/v3/)


## 一、概要设计方法

### 1、协议

API与Client的通信协议，总是使用HTTPS协议。全站https。

PS：使用HTTPS协议和RESTful API本身没有多大关系，但是对于增加网站的安全是非常重要的，特别是如果提供的是公开的API，那么HTTPS久更显得重要了。

### 2、域名

应该尽量将API部署在专用的域名下面，比如：

 https://api.github.com 

如果API变化较大，可以把API设计为子域名，比如：

 https://example.com/api/v1 

### 3、版本（Versioning）

一般而言应该将API放入URL中，比如：

 https://example.com/api/v1 

还可以将版本号放入HTTP信息头中，但这样不如放入URL方便和直观。

### 4、路径（Endpoint）

在协议中，每个网址代表一种资源的存放地址，所以网址终不能有动词，只能有名词，而且名词一般都应该与数据库的表字段对应，且API中的名词应该使用复数。例如：
/user/:id	查询id为:id的用户（只返回一条记录）
/users/:username 查询用户名为:username的用户（返回多条记录）
/users	查询所有用户

PS：根据RFC3986定义，URL是大小写敏感的，所以应该尽量使用小写字母来命名！


## 二、 在保证api原子性的同时，尽量可以一次返回所需要数据,减少请求次数
例如：返回好友列表。
尽量一次性把好友的信息都返回回来，不要第一次请求所有好友id，第二次再给句好友id请求api获取好友信息。

```不推荐下面这样：
{
"friends"：[
        123456,
        124578,
        124598,
        124578
	]
}
```
```推荐下面这样
{
"friends"：[
			{
				"id":123456,
				"name":"张三",
				"profileIcon":"http://image.example.com/profile/123456"
			}，
			{
				"id":124578,
				"name":"李四",
				"profileIcon":"http://image.example.com/profile/124578"
			},
			{
				"id":124598,
				"name":"王五",
				"profileIcon":"http://image.example.com/profile/124598"
			},
			{
				"id":124578,
				"name":"小六",
				"profileIcon":"http://image.example.com/profile/124578"
			}
		]
}
````



## 三、让前端选择响应内容

为了减少不必要的返回，需要通过api调用时通过查询参数来获取需要的内容
>示例：http://api.example.com/v1/users/123456?fields=name,age,birth	

上面的api的功能是查询用户信息，查询参数fields 指定后端返回的用户信息所含有的具体字段，注意：如果省略fields查询参数，则默认返回所有的用户信息字段。

如果字段信息太多，也可以后端预先定义好一些字段的组合，响应群（Response Group），然后请求时直接传递响应群组的名称即可。
>示例：http://api.example.com/v1/users/123456?rgroup=large

|响应群名称|内容|
|--------|----|
|small|username,phone,birth|
|medium|username,phone,birth,profileIcon|
|large|username,phone,birth,profileIcon,gender,age,address|



## 四、 返回json数据必须封装成对象，不可以直接返回序列。避免JavaScript被json注入

>示例：
```
推荐
{
"friends"：[
        123456,
        124578,
        124598,
        124578
	]
}
```
```
不推荐
[
    123456,
    124578,
    124598,
    124578
]
```



## 五、 查询列表，在返回列表序列的同时还要，返回是否有后续数据的信息

例如：分页查询，一般需要指定从哪里开始获取，如果每页展示20条数据时，可以试着获取21条数据，如果能成功获取21条，就说明至少存在1条后续数据，这时就可以返回前20条数据的同时一并还有后续数据的信息。（再详细一些，还可以返回下一页所需要的参数或url信息）
>分页查询示例：
```
/users?limit=10：指定返回记录的数量
/users?offset=10：指定返回记录的开始位置
/users?page=2&per_page=100：指定第几页，以及每页的记录数
/users?sortby=name&order=asc：指定返回结果按照哪个属性排序，以及排序顺序
/users?state=close：指定筛选条件
```


>分页返回示例：
```
{
	"items":[
		.
		.
		.
		.
	],
	"hasNext":true,
	"url":"http://www.example/v1/page/2" 【说明：可选】
	"nextPageStartRow":21			 【说明：可选】
}
```



## 六、api和 json 数据项的名称统一使用驼峰命名法



## 七、返回数据为序列时，api使用复数形式 例如：/users,除此之外使用单数形式



## 八、日期格式统一使用 RFC 3339 日期格式：年/月/日 



## 九、用js 处理大数值的情况时，用字符串表示，或者增加额外的字符串字段表示大数值

>例如：
```
{
	"id":26642342343243242343242342342,
	"id_str":"26642342343243242343242342342"
}
或
{
	"id":"26642342343243242343242342342"
}
```



## 十、通过状态码来表示出错信息

### 1. 状态码分类及说明
|状态码|含义|
|-----|----|
|1字头|消息|
|2字头|成功|
|3字头|重定向|
|4字头|客户端原因引起的错误|
|5字头|服务器端引起的错误|

|状态码|名称|说明|
|-----|----|---|
|200|OK|请求成功|
|201|Created|请求成功，新的资源已创建|
|202|Accepted|请求成功|
|204|No Content|没有内容|
|300|Multiple Choices|存在多个资源|
|301|Moved Permanently|资源被永久转移|
|302|Found|请求的资源暂时被转移|
|303|See Other|引用他处|
|304|Not Modif|自上一次访问后没有发生更新|
|307|Temporary Redirect|请求的资源暂时被转移|
|400|Bad Request|请求不正确|
|401|Unauthorized|需要认证|
|403|Forbidden|禁止访问|
|404|Not Found|没找到指定的资源|
|405|Method Not Allowed|无法使用指定的方法|
|406|Not Acceptable|同Accept相关联的首部里含有无法处理的内容|
|408|Request timeout|请求在规定的时间内没有处理结束，请求超时|
|409|Conflict|资源存在冲突|
|410|Gone|指定的资源已经不存在|
|413|Request Entity Too Large|请求消息体太大|
|414|Request-URI Too Long|请求URI太长|
|415|UNsupported Media Type|不支持所指定的媒体类型|
|429|Too Many Requests|请求次数过多|
|500|Internal Server Error|服务器内部发生错误|
|503|Service Unavailable|服务器暂时停止运行|
### 2.状态码使用详解
###### GET 用于获取资源；安全且幂等

- 200（OK） - 表示已在响应中发出
- 204（无内容） - 资源有空表示
- 301（Moved Permanently） - 资源的URI已被更新
- 303（See Other） - 其他（如，负载均衡）
- 304（not modified）- 资源未更改（缓存）
- 400 （bad request）- 指代坏请求（如，参数错误）
- 404 （not found）- 资源不存在
- 406 （not acceptable）- 服务端不支持所需表示
- 500 （internal server error）- 通用错误响应
- 503 （Service Unavailable）- 服务端当前无法处理请求

######  POST 用于创建资源、创建子资源、部分更新资源；不安全且不幂等

- 200（OK）- 如果现有资源已被更改
- 201（created）- 如果新资源被创建
- 202（accepted）- 已接受处理请求但尚未完成（异步处理）
- 301（Moved Permanently）- 资源的URI被更新
- 303（See Other）- 其他（如，负载均衡）
- 400（bad request）- 指代坏请求
- 404 （not found）- 资源不存在
- 406 （not acceptable）- 服务端不支持所需表示
- 409 （conflict）- 通用冲突
- 412 （Precondition Failed）- 前置条件失败（如执行条件更新时的冲突）
- 415 （unsupported media type）- 接受到的表示不受支持
- 500 （internal server error）- 通用错误响应
- 503 （Service Unavailable）- 服务当前无法处理请求

###### PUT 用于更新资源，必须更新资源的所有属性（id除外）；不安全但幂等

- 200 （OK）- 如果已存在资源被更改
- 201 （created）- 如果新资源被创建
- 301（Moved Permanently）- 资源的URI已更改
- 303 （See Other）- 其他（如，负载均衡）
- 400 （bad request）- 指代坏请求
- 404 （not found）- 资源不存在
- 406 （not acceptable）- 服务端不支持所需表示
- 409 （conflict）- 通用冲突
- 412 （Precondition Failed）- 前置条件失败（如执行条件更新时的冲突）
- 415 （unsupported media type）- 接受到的表示不受支持
- 500 （internal server error）- 通用错误响应
- 503 （Service Unavailable）- 服务当前无法处理请求

###### DELETE 用于删除资源；不安全但幂等

- 200 （OK）- 资源已被删除
- 301 （Moved Permanently）- 资源的URI已更改
- 303 （See Other）- 其他，如负载均衡
- 400 （bad request）- 指代坏请求
- 404 （not found）- 资源不存在
- 409 （conflict）- 通用冲突
- 500 （internal server error）- 通用错误响应
- 503 （Service Unavailable）- 服务端当前无法处理请求

#### 2.1 状态码示例

>*注意：结合2.2~2.5重点理解下面的请求及返回状态码
```
服务端可用信息：Users: John, Peter
METHOD  URL                    STATUS  RESPONSE
GET     /users                 200     [John, Peter]
GET     /user/john             200     John
GET     /user/kyle             404     Not found
GET     /users?name=kyle       200     []	//这个要重点注意，返回200，不是404
DELETE  /user/john             204     No Content
```

#### 2.2 状态码2**
200：一般的请求成功时使用；
201：POST请求服务器创数据成功;
202：当异步处理客户请求时，服务器已接受来自客户端的请求，但处理尚未结束
204：当响应消息为空时返回，例如使用DELETE请求删除数据成功时，服务器返回204；
>注意：如果是PUT/PATCH操作成功时（即修改数据成功时），应返回200为状态码，同时应一并返回该方法操作的数据

#### 2.3 状态码3**
3**  暂时略
#### 2.4 状态码4**
400：其他错误（无法描述的错误）
401：认证类型错误（服务器不知道你是谁）
403：授权类型错误（服务器知道你是谁，但你没有执行该操作的权限）
404：访问的目标数据(Request-URI)不存在（指Request-URI不存在）
405：请求方法不被允许


#### 2.5 状态码5**
5** 暂时略



##  十一、出现错误时要求返回错误代码,统一使用4位数错误代码（与http状态码区别开来）【待整理】

|错误代码|含义|
|-----|----|
|1字头|通用错误|
|2字头|用户信息错误|


```


        200 OK - [GET]：服务器成功返回用户请求的数据，该操作是幂等的（Idempotent）。
        201 CREATED - [POST/PUT/PATCH]：用户新建或修改数据成功。
        202 Accepted - [*]：表示一个请求已经进入后台排队（异步任务）
        204 NO CONTENT - [DELETE]：用户删除数据成功。
        400 INVALID REQUEST - [POST/PUT/PATCH]：用户发出的请求有错误，服务器没有进行新建或修改数据的操作，该操作是幂等的。
        401 Unauthorized - [*]：表示用户没有权限（令牌、用户名、密码错误）。
        403 Forbidden - [*] 表示用户得到授权（与401错误相对），但是访问是被禁止的。
        404 NOT FOUND - [*]：用户发出的请求针对的是不存在的记录，服务器没有进行操作，该操作是幂等的。
        406 Not Acceptable - [GET]：用户请求的格式不可得（比如用户请求JSON格式，但是只有XML格式）。
        410 Gone -[GET]：用户请求的资源被永久删除，且不会再得到的。
        422 Unprocesable entity - [POST/PUT/PATCH] 当创建一个对象时，发生一个验证错误。
        500 INTERNAL SERVER ERROR - [*]：服务器发生错误，用户将无法判断发出的请求是否成功。


```
