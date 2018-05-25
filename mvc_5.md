# A Singleton Controller

```go
package main

import (
	"fmt"
	"sync/atomic"

	"github.com/kataras/iris"
	"github.com/kataras/iris/mvc"
)

func main() {
	app := iris.New()
	mvc.New(app.Party("/")).Handle(&globalVisitorsController{visits: 0})

	// http://localhost:8080
	app.Run(iris.Addr(":8080"))
}

type globalVisitorsController struct {
	// When a singleton controller is used then concurent safe access is up to the developers, because
	// all clients share the same controller instance instead.
	// Note that any controller's methods
	// are per-client, but the struct's field can be shared across multiple clients if the structure
	// does not have any dynamic struct field dependencies that depend on the iris.Context
	// and ALL field's values are NOT zero, at this case we use uint64 which it's no zero (even if we didn't set it
	// manually ease-of-understand reasons) because it's a value of &{0}.
	// All the above declares a Singleton, note that you don't have to write a single line of code to do this, Iris is smart enough.
	//
	// see `Get`.
	visits uint64
}

func (c *globalVisitorsController) Get() string {
	count := atomic.AddUint64(&c.visits, 1)
	return fmt.Sprintf("Total visitors: %d", count)
}

```

----

More folder structure guidelines can be found at the [https://github.com/kataras/iris/tree/master/_examples/#structuring](https://github.com/kataras/iris/tree/master/_examples/#structuring) section.

Follow the examples below

- [Hello world](https://github.com/kataras/iris/blob/master/_examples/mvc/hello-world/main.go) **UPDATED**
- [Session Controller](https://github.com/kataras/iris/blob/master/_examples/mvc/session-controller/main.go) **UPDATED**
- [Overview - Plus Repository and Service layers](https://github.com/kataras/iris/tree/master/_examples/mvc/overview) **UPDATED**
- [Login showcase - Plus Repository and Service layers](https://github.com/kataras/iris/tree/master/_examples/mvc/login) **UPDATED**
- [Singleton](https://github.com/kataras/iris/tree/master/_examples/mvc/singleton) **NEW**
- [Websocket Controller](https://github.com/kataras/iris/tree/master/_examples/mvc/websocket) **NEW**
- [Register Middleware](https://github.com/kataras/iris/tree/master/_examples/mvc/middleware) **NEW**
- [Vue.js Todo MVC](https://github.com/kataras/iris/tree/master/_examples/tutorial/vuejs-todo-mvc) **NEW**