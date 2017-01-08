# Logger
Logs every incoming http requests.


We have two loggers, same job & how-to-use code but different style of output:

- [This is the logger middleware using the log.Logger](https://github.com/iris-contrib/middleware/tree/master/logger)

```sh
$ go get github.com/iris-contrib/middleware/logger
```

- [This is the logger middleware using the zap structured logger](https://github.com/iris-contrib/middleware/tree/master/loggerzap)

```sh
$ go get github.com/iris-contrib/middleware/loggerzap
```

```go
 New(config ...Config) iris.HandlerFunc
```

How to use

```go
package main

import (
	"github.com/kataras/iris"
	"github.com/iris-contrib/middleware/logger"
	// or logger "github.com/iris-contrib/middleware/loggerzap"
	// the rest code should be identical.
)

/*
With configs:

errorLogger := logger.New(logger.Config{
	// Status displays status code
	Status: true,
	// IP displays request's remote address
	IP: true,
	// Method displays the http method
	Method: true,
	// Path displays the request path
	Path: true,
})

iris.Use(errorLogger)

With default configs:

iris.Use(logger.New())
*/
func main() {

	iris.Use(logger.New())

	iris.Get("/", func(ctx *iris.Context) {
		ctx.Writef("hello")
	})

	iris.Get("/1", func(ctx *iris.Context) {
		ctx.Writef("hello")
	})

	iris.Get("/2", func(ctx *iris.Context) {
		ctx.Writef("hello")
	})

	// log http errors
	errorLogger := logger.New()


	iris.OnError(iris.StatusNotFound, func(ctx *iris.Context) {
		errorLogger.Serve(ctx)
		ctx.Writef("My Custom 404 error page ")
	})
	//

	iris.Listen(":8080")

}
```
