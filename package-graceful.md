# Graceful

[This is a package](https://github.com/iris-contrib/graceful).


Enables graceful shutdown.

```go

package main

import (
	"time"
	"github.com/kataras/iris"
	"github.com/iris-contrib/graceful"
)

func main() {
	api := iris.New()
	api.Get("/", func(ctx *iris.Context) {
		ctx.Writef("Welcome to the home page!")
	})

	graceful.Run(":3001", time.Duration(10)*time.Second, api)
}


```
