---
title: Golang统一处理HTTP请求中的异常捕获
date: 2017-09-21 15:28:00
categories: golang
tags: [golang,httprouter]
---

*对所有注册的路由handle进行异常捕获*

golang使用panic()产生异常，然后可以recover()来捕获到异常，否则主程序直接宕掉，这是我们不希望看到的。
或者全程检查error，不主动抛出异常。即便这样，可能异常依然不能避免。

现在即要recover()，又不用在每个handle里面都去recover()一遍。

<!--more-->

*从普通的路由注册代码开始*

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

通过Wrap方法把处理方法包裹起来。wrap方法需要有一个handle类型的参数，同时返回值要能被httprouter接收
```golang
func RegRouters(r *httprouter.Router) {
	r.GET("/", WrapHandle(Home))
	r.GET("/contact", WrapHandle(Contact))
}

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

再把handle简化一下，增加一个context包，将handle的参数包裹起来，当然context还可以进一步处理。

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

简单的请求上下文的原型：
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

```
