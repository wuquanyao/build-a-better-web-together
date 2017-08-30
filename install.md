# Installation

The only requirement is the [Go Programming Language](https://golang.org/dl/), at least version 1.8 but **1.9** is highly recommended.

```sh
$ go get -u github.com/kataras/iris
```

> _iris_ takes advantage of the [vendor directory](https://docs.google.com/document/d/1Bz5-UB7g2uPBdOx-rw5t9MxJwkfpx90cqG9AFL0JAYo) feature. You get truly reproducible builds, as this method guards against upstream renames and deletes.



Q: What's the difference between Go 1.8 and Go 1.9 Iris?

A: Go 1.9 has support for **type alias** and Iris was ready for that from Go 1.8.3, it had a file, the `kataras/iris/context.go` with restrict build access to Go 1.9. 

**Before** Go 1.9 you had to import the "github.com/kataras/iris/context" to create a Handler:

```go
package main

import (
    "github.com/kataras/iris"
    "github.com/kataras/iris/context"
)

func main() {
    app := iris.New()
    app.Get("/", func(ctx context.Context){})
    app.Run(iris.Addr(":8080"))
}

```

**From** Go 1.9 and **After** you don't have to import that, you can **OPTIONALLY** do that instead:

```go
package main

import "github.com/kataras/iris"


func main() {
    app := iris.New()
    app.Get("/", func(ctx iris.Context){})
    app.Run(iris.Addr(":8080"))
}

```

The same for the `kataras/iris/core/router/APIBuilder#PartyFunc`

```go
app.PartyFunc("/cpanel", func(child iris.Party) { // instead of importing the router package and use router.Party
    child.Get("/", func(ctx iris.Context){})
}
// OR 
cpanel := app.Party("/cpanel")
cpanel.Get("/", func(ctx iris.Context){}) 
```



