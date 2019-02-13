# Go Web 编程笔记 {ignore=true}

[TOC]

## 环境配置

安装方式主要有
1. 源码安装
2. 官方提供的安装包安装
3. 第三方工具安装

> 具体细节不提，这方面的资料非常多

这边着重说一下多版本的问题，用来处理需要一个系统中安装多个版本的Go
[GVM](https://github.com/moovweb/gvm)

> 从go 1.8开始，GOPATH环境变量现在有一个默认值，如果它没有被设置。 它在Unix上默认为$HOME/go,在Windows上默认为%USERPROFILE%/go。

## Go语言基础

Go是没有继承。只有struct的组合

## Web基础

首先理解在输入网址之后发生了什么

``` mermaid
graph TD;
    clent(客户端) --域名解析请求--> DNS(DNS服务器)
    DNS --IP地址--> clent
    clent --用IP地址向服务器发送请求 --> 服务器
    服务器 --返回数据--> clent
    clent -- 渲染网页--> clent
```

> 域名解析是一种迭代查询

HTTP请求包
```HTTP
GET /domains/example/ HTTP/1.1        //请求行: 请求方法 请求URI HTTP协议/协议版本
Host：www.iana.org                //服务端的主机名
User-Agent：Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.4 (KHTML, like Gecko) Chrome/22.0.1229.94 Safari/537.4            //浏览器信息
Accept：text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8    //客户端能接收的mine
Accept-Encoding：gzip,deflate,sdch        //是否支持流压缩
Accept-Charset：UTF-8,*;q=0.5        //客户端字符编码集
//空行,用于分割请求头和消息体
//消息体,请求资源参数,例如POST传递的参数
```
HTTP响应包
```HTTP
HTTP/1.1 200 OK                        //状态行
Server: nginx/1.0.8                    //服务器使用的WEB软件名及版本
Date:Date: Tue, 30 Oct 2012 04:14:25 GMT        //发送时间
Content-Type: text/html                //服务器发送信息的类型
Transfer-Encoding: chunked            //表示发送HTTP包是分段发的
Connection: keep-alive                //保持连接状态
Content-Length: 90                    //主体内容长度
//空行 用来分割消息头和主体
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"... //消息体
```



## 数据库
注册数据库驱动

```go
//https://github.com/mattn/go-sqlite3驱动
func init() {
    sql.Register("sqlite3", &SQLiteDriver{})
}

//https://github.com/mikespook/mymysql驱动
// Driver automatically registered in database/sql
var d = Driver{proto: "tcp", raddr: "127.0.0.1:3306"}
func init() {
    Register("SET NAMES utf8")
    sql.Register("mymysql", &d)
}
```

可以多使用`database/sql`接口的数据库驱动
[SQLite](https://github.com/mattn/go-sqlite3)
[MySQL](https://github.com/go-sql-driver/mysql)
[postgreSQL](https://github.com/lib/pq)

扩展学习`Redis`, `MongoDB`

[关于Go SQL的使用教学](http://go-database-sql.org/index.html)

`session`翻译过来就是会话，是一种抽象的东西
通常`session`客户端保存是利用不设置超时的`cookie`来实现的，如果遇到没有开启`cookie`则跟到`url`后面
服务端实现可以通过存储数据库或者内存来
[防止session劫持](http://www.cnblogs.com/phpstudy2015-6/p/6776919.html)

`CSRF`攻击
[防止CSRF攻击](https://www.jianshu.com/p/00fa457f6d3e)

`XSS`攻击
[防止XSS攻击](https://tech.meituan.com/2018/09/27/fe-security.html)

### 密码存储问题

- 弱智级 明文存储
- 初级 哈希处理后存储
- 中级 多次哈希后存储
- 高级 哈希加盐存储
- 专家级 使用`scrypt`方案

[scrypt维基介绍](https://zh.wikipedia.org/wiki/Scrypt)

[GO中对scrypt的支持](https://godoc.org/golang.org/x/crypto/scrypt)

关于go的`daemon`
[守护进程](https://www.zhihu.com/question/38609004/answer/529315259)
第三方解决方案`Supervisord`

## 备份与恢复
文件同步工具`rsync`
`MySQL`和`redis`备份
热备份和冷备份

衍生
[关于反爬虫](https://segmentfault.com/a/1190000005840672)


---
## 课后作业
> 实现一个简易的Web框架

目标:
- 实现对用户请求路径的分配
- HTTP的请求方法

首先维持一个`map`来保存路由信息，然后遍历检查是否对应的路由信息

还需要维护一个params来实现扩展
例如:
当要绑定`/login/:name`那么可以通过params的的name来获取这个路由的具体数据
比较笨重的实现方法是通过`regexp`来实现

借鉴`httproutes`来实现轻量级的
`trie`[前缀数](https://zh.wikipedia.org/wiki/Trie)

转发路由`ServeHTTP`还需要对请求方法进行分类

`beego`对于`controller`有点像java的
举例来说：
就是这个路由组合beegon.controller。然后在里面重写相应的请求
接着在路由匹配的地方通过反射来实现
