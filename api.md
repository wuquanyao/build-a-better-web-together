# API

**Use of GET, POST, PUT, DELETE, HEAD, PATCH, OPTIONS, CONNECT & TRACE**

```go
package main

import "github.com/kataras/iris"


func main() {
    app := iris.New()
    // declare the routes
    app.Get("/home", testGet)
    app.Post("/login", testPost)
    app.Put("/add", testPut)
    app.Delete("/remove", testDelete)
    iris.Head("/testHead", testHead)
    iris.Patch("/testPatch", testPatch)
    iris.Options("/testOptions", testOptions)
    iris.Connect("/testConnect", testConnect)
    iris.Trace("/testTrace", testTrace)

    // start the server
    app.Run(iris.Addr(":8080"))
}

func testGet(ctx iris.Context) {
    // [...]
}
func testPost(ctx iris.Context) {
    // [...]
}

// and so on....
```