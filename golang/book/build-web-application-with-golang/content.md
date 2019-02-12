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

### 预防跨站脚本
`跨站脚本攻击(Cross Site Scripting)`又称`XSS`

`html/template`已经做了转义

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
