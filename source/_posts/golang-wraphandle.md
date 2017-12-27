---
title: Golang统一处理HTTP请求中的异常捕获
date: 2016-10-21 15:11:00
categories: golang
tags: [golang,httprouter,exception]
---

最近写GOLANG项目，不使用框架，路由选择httprouter
现在想实现一个需求：***在不修改httprouter源码的前提下，对所有注册的路由handle进行异常捕获。***

大家都知道golang使用panic()产生异常，然后可以recover()来捕获到异常，否则主程序直接宕掉，这是我们不希望看到的。
或者全程检查error，不主动抛出异常。即便这样，可能异常依然不能避免。

既然要recover()，但又不想在每个handle里面都去recover()一遍，如果你也有这样的需求，下面讲到的可能对你有用。
```golang
func RegRouters(r *httprouter.Router) {
	r.GET("/", Home)
	r.GET("/contact", Contact)
}

func Home(w http.ResponseWriter, r *http.Request, params httprouter.Params) {
	defer func() {
			if pr := recover(); pr != nil {
				fmt.Printf("panic recover: %v\r\n", pr)
				debug.PrintStack()
			}
		}()
	//something
}

func Contact(w http.ResponseWriter, r *http.Request, params httprouter.Params) {
	defer func() {
			if pr := recover(); pr != nil {
				fmt.Printf("panic recover: %v\r\n", pr)
				debug.PrintStack()
			}
		}()
	//something
}
```
正常的路由表长这样。在C#中相当于对每个请求加try...catch，将业务上代码包裹起来。
可能你已经想到了，对，包裹起来，像这样：
```golang
func RegRouters(r *httprouter.Router) {
	r.GET("/", WrapHandle(Home))
	r.GET("/contact", WrapHandle(Contact))
}
```
那这个wrap方法长什么样呢？他需要有一个handle类型的参数，同时返回值要能被httprouter接收，直接上代码
```golang
func WrapHandle(handle httprouter.Handle) httprouter.Handle {
	return func(w http.ResponseWriter, r *http.Request, p httprouter.Params) {
		defer func() {
			if pr := recover(); pr != nil {
				fmt.Printf("panic recover: %v\r\n", pr)
				debug.PrintStack()
			}
		}()
		handle(w, r, p)
	}
}
```
是不是很简单~ 我们再改造一下，加一个上下文，把handle简化一下
```golang
func WrapHandle(handle func(ctx *context.Context)) httprouter.Handle {
	return func(w http.ResponseWriter, r *http.Request, p httprouter.Params) {
		defer func() {
			if pr := recover(); pr != nil {
				fmt.Println("panic recover: %v", pr)
				debug.PrintStack()
			}
		}()
		ctx := context.NewContext(w, r, p)
		handle(ctx)
	}
}
func Home(ctx *context.Context) {
	defer func() {
			if pr := recover(); pr != nil {
				fmt.Printf("panic recover: %v\r\n", pr)
				debug.PrintStack()
			}
		}()
	//something
}

func Contact(ctx *context.Context) {
	defer func() {
			if pr := recover(); pr != nil {
				fmt.Printf("panic recover: %v\r\n", pr)
				debug.PrintStack()
			}
		}()
	//something
	id = ctx.FormInt64("id") //通过context从FORM取值，并转换为int64
}
```
看出有什么不同了吗？对，多了一个context包，handle的参数也减少到一个，主要是context上可以做更多的事情。

一个简单的请求上下文的原型：

```golang
type Context struct {
	responseWriter http.ResponseWriter
	request        *http.Request
	params         httprouter.Params
	Data           map[string]interface{}
}

func NewContext(w http.ResponseWriter, r *http.Request, params httprouter.Params) *Context {
	ctx := new(Context)
	ctx.responseWriter = w
	ctx.request = r
	ctx.params = params
	ctx.Data = make(map[string]interface{})
	return ctx
}

func (ctx *Context) FormValue(name string) string {
	return ctx.request.FormValue(name)
}

func (ctx *Context) FormInt64(name string) int64 {
	value, _ := strconv.ParseInt(ctx.FormValue(name), 10, 64)
	return value
}

//更多扩展....
```
好了，先到这里，希望对你有帮助。