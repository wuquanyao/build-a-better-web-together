# Wrapping the Router

**Very rare**, you may never need that but it's here in any case you need it.

There are times you need to override or decide whether the Router will be executed on an incoming request. If you've any previous experience with the `net/http` and other web frameworks this function will be familiar with you (it has the form of a net/http middleware, but instead of accepting the next handler it accepts the Router as a function to be executed or not).

```go
// WrapperFunc is used as an expected input parameter signature
// for the WrapRouter. It's a "low-level" signature which is compatible
// with the net/http.
// It's being used to run or no run the router based on a custom logic.
type WrapperFunc func(w http.ResponseWriter, r *http.Request, firstNextIsTheRouter http.HandlerFunc)

// WrapRouter adds a wrapper on the top of the main router.
// Usually it's useful for third-party middleware
// when need to wrap the entire application with a middleware like CORS.
//
// Developers can add more than one wrappers,
// those wrappers' execution comes from last to first.
// That means that the second wrapper will wrap the first, and so on.
//
// Before build.
func WrapRouter(wrapperFunc WrapperFunc)
```

Iris' router searches for its routes based on the `HTTP Method` a Router Wrapper can override that behavior and execute custom code.

Example Code:

```go
package main

import (
    "net/http"
    "strings"

    "github.com/kataras/iris"
)

// In this example you'll just see one use case of .WrapRouter.
// You can use the .WrapRouter to add custom logic when or when not the router should
// be executed in order to execute the registered routes' handlers.
//
// To see how you can serve files on root "/" without a custom wrapper
// just navigate to the "file-server/single-page-application" example.
//
// This is just for the proof of concept, you can skip this tutorial if it's too much for you.


func main() {
    app := iris.New()

    app.OnErrorCode(iris.StatusNotFound, func(ctx iris.Context) {
        ctx.HTML("<b>Resource Not found</b>")
    })

    app.Get("/", func(ctx iris.Context) {
        ctx.ServeFile("./public/index.html", false)
    })

    app.Get("/profile/{username}", func(ctx iris.Context) {
        ctx.Writef("Hello %s", ctx.Params().Get("username"))
    })

    // serve files from the root "/", if we used .StaticWeb it could override
    // all the routes because of the underline need of wildcard.
    // Here we will see how you can by-pass this behavior
    // by creating a new file server handler and
    // setting up a wrapper for the router(like a "low-level" middleware)
    // in order to manually check if we want to process with the router as normally
    // or execute the file server handler instead.

    // use of the .StaticHandler
    // which is the same as StaticWeb but it doesn't
    // registers the route, it just returns the handler.
    fileServer := app.StaticHandler("./public", false, false)

    // wrap the router with a native net/http handler.
    // if url does not contain any "." (i.e: .css, .js...)
    // (depends on the app , you may need to add more file-server exceptions),
    // then the handler will execute the router that is responsible for the
    // registered routes (look "/" and "/profile/{username}")
    // if not then it will serve the files based on the root "/" path.
    app.WrapRouter(func(w http.ResponseWriter, r *http.Request, router http.HandlerFunc) {
        path := r.URL.Path
        // Note that if path has suffix of "index.html" it will auto-permant redirect to the "/",
        // so our first handler will be executed instead.

        if !strings.Contains(path, ".") { 
            // if it's not a resource then continue to the router as normally. <-- IMPORTANT
            router(w, r)
            return
        }
        // acquire and release a context in order to use it to execute
        // our file server
        // remember: we use net/http.Handler because here we are in the "low-level", before the router itself.
        ctx := app.ContextPool.Acquire(w, r)
        fileServer(ctx)
        app.ContextPool.Release(ctx)
    })

    // http://localhost:8080
    // http://localhost:8080/index.html
    // http://localhost:8080/app.js
    // http://localhost:8080/css/main.css
    // http://localhost:8080/profile/anyusername
    app.Run(iris.Addr(":8080"))

    // Note: In this example we just saw one use case,
    // you may want to .WrapRouter or .Downgrade in order to bypass the iris' default router, i.e:
    // you can use that method to setup custom proxies too.
    //
    // If you just want to serve static files on other path than root
    // you can just use the StaticWeb, i.e:
    // 					     .StaticWeb("/static", "./public")
    // ________________________________requestPath, systemPath
}
```

There is not much to say here, it's just a function wrapper which accepts the native response writer and request and the next handler which is the Iris' Router itself, it's being or not executed whether is called or not, **it's a middleware for the whole Router**.