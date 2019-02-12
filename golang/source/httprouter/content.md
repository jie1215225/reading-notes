# httprouter源码笔记

> golang

首先来看看`httprouter`的结构

一共有四个模块
分别是`参数`,`目录`,`路由`,`树`

首先从`path`开始看起，这一共就两个文件，一个`path.go`,一个`path_test.go`

`path.go`有一个`CleanPath`结构，用来情况 `..` 和 `.`的情况，然后使开头为`\`
简单来说，就是`相对路径`改`绝对路径`

大概的实现过程就是
如果存在空的，就替换为`\`，类似`\abc\..\bc`替换为`\bc`

> 扩展，关于go的条件编译

```go
// +build go1.7
```
表示在go1.7版本上编译

详情[go build 条件编译](https://blog.csdn.net/varding/article/details/12675971)


在`params`模块上，有两个文件`params_go17.go`和`params_legacy.go`

前者使用`context`来实现，所以需要go1.7版本以上

在`router.go`文件中实现了`http.Handler`

这边用到了一个静态检查的写法
```go
var _ http.Handler = New()

func New() *Router {
    return &Router{
		RedirectTrailingSlash:  true,
		RedirectFixedPath:      true,
		HandleMethodNotAllowed: true,
		HandleOPTIONS:          true,
	}
}
```
这样可以保证`Router`实现了`http.Handler`的方法

这边说说`Router`的结构


```go
func (r *Router) GET(path string, handle Handle) {
	r.Handle("GET", path, handle)
}
```
这样交给`Handle`方法来处理

为了匹配到`http.Handler`
`params`就用到了

这里用到`适配器`
将`Handler(method, path string, handler http.Handler)` => 变成`Hande(method, path, handle Handle)`

最后通过`tree.go` 进行解析，然后把需要的参数放入`Params`中