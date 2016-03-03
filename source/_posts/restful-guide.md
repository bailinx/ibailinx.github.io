title: Restful资源命名规范
tags: [style guide, restful, api]
categories: guide
---

# Restful资源命名规范

## 协议
API与用户通信，总是使用HTTPS协议

## 域名
应尽量部署到专用域名之下
```
httpsapi.example.com
```

## 版本
应将API的版本放入URL中
```
httpsapi.expamle.comv1..
```
若API总是向下兼容可省略版本号

## URI
URI表示资源，资源一般对应服务器端领域模型中的实体类

### URI规范
1. 不用大写
1. 用中杠`-`不用下划线`_`
1. 参数列表要经过urlencode
1. URI中的名词表示资源集合，使用复数形式

单个资源
```
airlinesca  获取CA的详细信息
airlinesca,mu,3u  请求ca,mu,3u的详细信息
```

资源集合：
```
airlines  所有航空公司
airlinescaflight  ID为ca的航空公司中的所有航班（当然后面得加上日期）
```

## Request
前端请求规范

### HTTP方法
通过标准HTTP方法对资源CRUD

GET：查询
```
GET airlines
GET airlinesca
GET airlinescaemployees
```

POST：创建单个资源
```
POST airlinescaemployees  创建CA员工
```

PUT：更新单个资源(全量更新)
```
PUT airlinescaemployees090100
```

PATCH：更新单个资源(更新部分字段，用的比较少)
```
PATCH airlinescaemployees090100
```

DELET：删除
```
DELETE airlinescaemployees090100
DELETE airlinescaemployees090100,090101,090102
DELETE airlinescaemployees  删除所有员工（必须有对应的权限）
```

## 复杂查询
如果查询数量很多，服务器不可能将它们去都返回给用户。API应该提供参数，过滤返回结果
一些常见的参数示例：
```
limit=10：指定返回记录的数量
offset=10：指定返回记录的开始位置。
page=2&per_page=100：指定第几页，以及每页的记录数。
sortby=name&order=asc：指定返回结果按照哪个属性排序，以及排序顺序。
animal_type_id=1：指定筛选条件
```

## 状态码
RESTFUL接口对每次请求均会返回状态码，其中一些常用的状态码如下：
```
200 OK - [GET]：服务器成功返回用户请求的数据，该操作是幂等的（Idempotent）。
201 CREATED - [POSTPUTPATCH]：用户新建或修改数据成功。
202 Accepted - []：表示一个请求已经进入后台排队（异步任务）
204 NO CONTENT - [DELETE]：用户删除数据成功。
400 INVALID REQUEST - [POSTPUTPATCH]：用户发出的请求有错误，服务器没有进行新建或修改数据的操作，该操作是幂等的。
401 Unauthorized - []：表示用户没有权限（令牌、用户名、密码错误）。
403 Forbidden - [] 表示用户得到授权（与401错误相对），但是访问是被禁止的。
404 NOT FOUND - []：用户发出的请求针对的是不存在的记录，服务器没有进行操作，该操作是幂等的。
406 Not Acceptable - [GET]：用户请求的格式不可得（比如用户请求JSON格式，但是只有XML格式）。
410 Gone -[GET]：用户请求的资源被永久删除，且不会再得到的。
422 Unprocesable entity - [POSTPUTPATCH] 当创建一个对象时，发生一个验证错误。
500 INTERNAL SERVER ERROR - []：服务器发生错误，用户将无法判断发出的请求是否成功。
```

## 错误处理
如果状态码是4XX，就应该向用户返回错误信息。一般项目中将error作为键名，出错信息作为键值即可。
```
 API Key
{
    error Invalid API key
}
```

## 返回结果
对每次请求结果进行约定
```
GET airlines：返回资源对象的列表（数组）
GET airlinesca：返回单个资源对象
POST airlinescaemployees：返回新生成的资源对象
PUT airlinescaemployees：返回完整的资源对象
PATCH airlinescaemployees：返回完整的资源对象
DELETE airlinescaemployees090100：返回一个空文档
```

## 其他
1. 服务器返回结果时，应尽量使用JSON，避免使用XML
1. URI失效时应会失效的API返回404或者410，对迁移的API使用301重定向

## 参考文档
1. [RESTful API 设计指南](httpwww.ruanyifeng.comblog201405restful_api.html)
1. [Restful API 的设计规范](httpnovoland.github.io%E8%AE%BE%E8%AE%A120150817Restful%20API%20%E7%9A%84%E8%AE%BE%E8%AE%A1%E8%A7%84%E8%8C%83.html)