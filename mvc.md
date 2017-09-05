# MVC

![](https://github.com/kataras/iris/raw/master/_examples/mvc/web_mvc_diagram.png)

Iris has **first-class support for the MVC pattern**, you'll not find
these stuff anywhere else in the Go world.

Iris supports Request data, Models, Persistence Data and Binding
with the fastest possible execution.

## Characteristics

All HTTP Methods are supported, for example if want to serve `GET`
then the controller should have a function named `Get()`,
you can define more than one method function to serve in the same Controller struct.

Persistence data inside your Controller struct (share data between requests)
via `iris:"persistence"` tag right to the field or Bind using `app.Controller("/" , new(myController), theBindValue)`.

Models inside your Controller struct (set-ed at the Method function and rendered by the View)
via `iris:"model"` tag right to the field, i.e ```User UserModel `iris:"model" name:"user"` ``` view will recognise it as `{{.user}}`.
If `name` tag is missing then it takes the field's name, in this case the `"User"`.

Access to the request path and its parameters via the `Path and Params` fields.

Access to the template file that should be rendered via the `Tmpl` field.

Access to the template data that should be rendered inside
the template file via `Data` field.

Access to the template layout via the `Layout` field.

Access to the low-level `iris.Context` via the `Ctx` field.

Get the relative request path by using the controller's name via `RelPath()`.

Get the relative template path directory by using the controller's name via `RelTmpl()`.

Flow as you used to, `Controllers` can be registered to any `Party`,
including Subdomains, the Party's begin and done handlers work as expected.

Optional `BeginRequest(ctx)` function to perform any initialization before the method execution,
useful to call middlewares or when many methods use the same collection of data.

Optional `EndRequest(ctx)` function to perform any finalization after any method executed.

Inheritance, recursively, see for example our `mvc.SessionController`, it has the `iris.Controller` as an embedded field
and it adds its logic to its `BeginRequest`, [here](https://github.com/kataras/iris/blob/master/mvc/session_controller.go). 

Read access to the current route  via the `Route` field.

Register one or more relative paths and able to get path parameters, i.e

If `app.Controller("/user", new(user.Controller))`

- `func(*Controller) Get()` - `GET:/user` , as usual.
- `func(*Controller) Post()` - `POST:/user`, as usual.
- `func(*Controller) GetLogin()` - `GET:/user/login`
- `func(*Controller) PostLogin()` - `POST:/user/login`
- `func(*Controller) GetProfileFollowers()` - `GET:/user/profile/followers`
- `func(*Controller) PostProfileFollowers()` - `POST:/user/profile/followers`
- `func(*Controller) GetBy(id int64)` - `GET:/user/{param:long}`
- `func(*Controller) PostBy(id int64)` - `POST:/user/{param:long}`

If `app.Controller("/profile", new(profile.Controller))`

- `func(*Controller) GetBy(username string)` - `GET:/profile/{param:string}`

If `app.Controller("/assets", new(file.Controller))`

- `func(*Controller) GetByWildard(path string)` - `GET:/assets/{param:path}`

## Using Iris MVC for code reuse

By creating components that are independent of one another, developers are able to reuse components quickly and easily in other applications. The same (or similar) view for one application can be refactored for another application with different data because the view is simply handling how the data is being displayed to the user.

If you're new to back-end web development read about the MVC architectural pattern first, a good start is that [wikipedia article](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller).



Example Code:

```go
package main

import (
    "github.com/kataras/iris"

    "github.com/kataras/iris/middleware/logger"
    "github.com/kataras/iris/middleware/recover"
)

// This example is equivalent to the
// https://github.com/kataras/iris/blob/master/_examples/hello-world/main.go

func main() {
    app := iris.New()
    // Optionally, add two built'n handlers
    // that can recover from any http-relative panics
    // and log the requests to the terminal.
    app.Use(recover.New())
    app.Use(logger.New())

    app.Controller("/", new(ExampleController))

    // http://localhost:8080
    // http://localhost:8080/ping
    // http://localhost:8080/hello
    app.Run(iris.Addr(":8080"))
}

// ExampleController serves the "/", "/ping" and "/hello".
type ExampleController struct {
    iris.Controller
}

// Get serves
// Method:   GET
// Resource: http://localhost:8080
func (c *ExampleController) Get() {
    c.ContentType = "text/html"
    c.Text = "<h1>Welcome!</h1>"
}

// GetPing serves
// Method:   GET
// Resource: http://localhost:8080/ping
func (c *ExampleController) GetPing() {
    c.Text = "pong"
}

// GetHello serves
// Method:   GET
// Resource: http://localhost:8080/hello
func (c *ExampleController) GetHello() {
    c.Ctx.JSON(iris.Map{"message": "Hello Iris!"})
}

/* Can use more than one, the factory will make sure
that the correct http methods are being registered for each route
for this controller, uncomment these if you want:

func (c *ExampleController) Post() {}
func (c *ExampleController) Put() {}
func (c *ExampleController) Delete() {}
func (c *ExampleController) Connect() {}
func (c *ExampleController) Head() {}
func (c *ExampleController) Patch() {}
func (c *ExampleController) Options() {}
func (c *ExampleController) Trace() {}
*/

/*
func (c *ExampleController) All() {}
//        OR
func (c *ExampleController) Any() {}
*/
```

Follow the examples below

- [Session Controller](https://github.com/kataras/iris/blob/master/_examples/mvc/session-controller/main.go)
- [A simple but featured Controller with model and views](https://github.com/kataras/iris/tree/master/_examples/mvc/controller-with-model-and-view).
- [Login showcase](https://github.com/kataras/iris/tree/master/mvc/login) **NEW**