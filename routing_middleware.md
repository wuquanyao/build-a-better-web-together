# Middleware

When we talk about Middleware in Iris we're talking about running code before and/or after our main handler code in a HTTP request lifecycle. For example, logging middleware might write the incoming request details to a log, then call the handler code, before writing details about the response to the log. One of the cool things about middleware is that these units are extremely flexible and reusable.

A middleware is just a **Handler** form of `func(ctx iris.Context)`, the middleware is being executed when the previous middleware calls the `ctx.Next()`, this can be used for authentication, i.e: if logged in then `ctx.Next()` otherwise fire an error response. 

## Writing a middleware

```go
package main

import "github.com/kataras/iris"

func main() {
    app := iris.New()
    app.Get("/", before, mainHandler, after)
    app.Run(iris.Addr(":8080"))
}

func before(ctx iris.Context) {
    shareInformation := "this is a sharable information between handlers"

    requestPath := ctx.Path()
    println("Before the mainHandler: " + requestPath)

    ctx.Values().Set("info", shareInformation)
    ctx.Next() // execute the next handler, in this case the main one.
}

func after(ctx iris.Context) {
    println("After the mainHandler")
}

func mainHandler(ctx iris.Context) {
    println("Inside mainHandler")

    // take the info from the "before" handler.
    info := ctx.Values().GetString("info")

    // write something to the client as a response.
    ctx.HTML("<h1>Response</h1>")
    ctx.HTML("<br/> Info: " + info)

    ctx.Next() // execute the "after".
}
```

```bash
$ go run main.go # and navigate to the http://localhost:8080
Now listening on: http://localhost:8080
Application started. Press CTRL+C to shut down.
Before the mainHandler: /
Inside mainHandler
After the mainHandler
```

### Globally

```go
package main

import "github.com/kataras/iris"

func main() {
    app := iris.New()
    // register the "before" handler as the first handler which will be executed
    // on all domain's routes.
    // or use the `UseGlobal` to register a middleware which will fire across subdomains.
    app.Use(before)
    // register the "after" handler as the last handler which will be executed
    // after all domain's routes' handler(s).
    app.Done(after)

    // register our routes.
    app.Get("/", indexHandler)
    app.Get("/contact", contactHandler)

    app.Run(iris.Addr(":8080"))
}

func before(ctx iris.Context) {
     // [...]
}

func after(ctx iris.Context) {
    // [...]
}

func indexHandler(ctx iris.Context) {
    // write something to the client as a response.
    ctx.HTML("<h1>Index</h1>")

    ctx.Next() // execute the "after" handler registered via `Done`.
}

func contactHandler(ctx iris.Context) {
    // write something to the client as a response.
    ctx.HTML("<h1>Contact</h1>")

    ctx.Next() // execute the "after" handler registered via `Done`.
}
```