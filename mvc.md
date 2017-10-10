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

    Supported types for method functions receivers: int, int64, bool and string.

Response via output arguments, optionally, i.e

```go
func(c *ExampleController) Get() string |
                                (string, string) |
                                (string, int) |
                                int |
                                (int, string |
                                (string, error) |
                                error |
                                (int, error) |
                                (customStruct, error) |
                                customStruct |
                                (customStruct, int) |
                                (customStruct, string) |
                                mvc.Result or (mvc.Result, error)
```

where [mvc.Result](https://github.com/kataras/iris/blob/master/mvc/method_result.go) is an interface which contains only that function: `Dispatch(ctx iris.Context)`.

## Using Iris MVC for code reuse

By creating components that are independent of one another, developers are able to reuse components quickly and easily in other applications. The same (or similar) view for one application can be refactored for another application with different data because the view is simply handling how the data is being displayed to the user.

If you're new to back-end web development read about the MVC architectural pattern first, a good start is that [wikipedia article](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller).

## Quick MVC Tutorial Part 1

```go
package main

import (
    "github.com/kataras/iris"
    "github.com/kataras/iris/mvc"
)

func main() {
    app := iris.New()

    app.Controller("/helloworld", new(HelloWorldController))

    app.Run(iris.Addr("localhost:8080"))
}

type HelloWorldController struct {
    mvc.Controller

    // [ Your fields here ]
    // Request lifecycle data
    // Models
    // Database
    // Global properties
}

//
// GET: /helloworld

func (c *HelloWorldController) Get() string {
    return "This is my default action..."
}

//
// GET: /helloworld/{name:string}

func (c *HelloWorldController) GetBy(name string) string {
    return "Hello " + name
}

//
// GET: /helloworld/welcome

func (c *HelloWorldController) GetWelcome() (string, int) {
    return "This is the GetWelcome action func...", iris.StatusOK
}

//
// GET: /helloworld/welcome/{name:string}/{numTimes:int}

func (c *HelloWorldController) GetWelcomeBy(name string, numTimes int) {
    // Access to the low-level Context,
    // output arguments are optional of course so we don't have to use them here.
    c.Ctx.Writef("Hello %s, NumTimes is: %d", name, numTimes)
}

/*
func (c *HelloWorldController) Post() {} handles HTTP POST method requests
func (c *HelloWorldController) Put() {} handles HTTP PUT method requests
func (c *HelloWorldController) Delete() {} handles HTTP DELETE method requests
func (c *HelloWorldController) Connect() {} handles HTTP CONNECT method requests
func (c *HelloWorldController) Head() {} handles HTTP HEAD method requests
func (c *HelloWorldController) Patch() {} handles HTTP PATCH method requests
func (c *HelloWorldController) Options() {} handles HTTP OPTIONS method requests
func (c *HelloWorldController) Trace() {} handles HTTP TRACE method requests
*/

/*
func (c *HelloWorldController) All() {} handles All method requests
//        OR
func (c *HelloWorldController) Any() {} handles All method requests
*/
```

> The [_examples/mvc](https://github.com/kataras/iris/tree/_examples/mvc) and [mvc/controller_test.go](https://github.com/kataras/iris/blob/master/mvc/controller_test.go) files explain each feature with simple paradigms, they show how you can take advandage of the Iris MVC Binder, Iris MVC Models and many more...

Every `exported` func prefixed with an HTTP Method(`Get`, `Post`, `Put`, `Delete`...) in a controller is callable as an HTTP endpoint. In the sample above, all funcs writes a string to the response. Note the comments preceding each method.

An HTTP endpoint is a targetable URL in the web application, such as `http://localhost:8080/helloworld`, and combines the protocol used: HTTP, the network location of the web server (including the TCP port): `localhost:8080` and the target URI `/helloworld`.

The first comment states this is an [HTTP GET](https://www.w3schools.com/tags/ref_httpmethods.asp) method that is invoked by appending "/helloworld" to the base URL. The third comment specifies an [HTTP GET](https://www.w3schools.com/tags/ref_httpmethods.asp) method that is invoked by appending "/helloworld/welcome" to the URL.

Controller knows how to handle the "name" on `GetBy` or the "name" and "numTimes" at `GetWelcomeBy`, because of the `By` keyword, and builds the dynamic route without boilerplate; the third comment specifies an [HTTP GET](https://www.w3schools.com/tags/ref_httpmethods.asp) dynamic method that is invoked by any URL that starts with "/helloworld/welcome" and followed by two more path parts, the first one can accept any value and the second can accept only numbers, i,e: "http://localhost:8080/helloworld/welcome/golang/32719", otherwise a [404 Not Found HTTP Error](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10.4.5) will be sent to the client instead.

----

Click [here](mvc_2.md) to navigate to the mvc tutorial part 2.

----