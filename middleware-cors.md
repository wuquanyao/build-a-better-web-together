# HTTP access control

Iris provides two ways for configuring your Cors, one is by [middleware](https://github.com/iris-contrib/middleware/tree/master/cors)
 and the other is by [plugin](https://github.com/iris-contrib/plugin/tree/master/cors).
 
The difference is that the 
- `cors plugin` is a wrapper which wraps the whole `iris.Router`. 
- `cors plugin` cannot be registered per-route, only per-application.
- `cors plugin` has all the options that the [rs/cors](https://github.com/rs/cors] middleware can accept.


- `cors middleware` is just an `iris.HandlerFunc` which can be registered per-route.
- `cors middleware` doesn't provides a lot of options, just the `AddOrigin` and `AllowCredentials()` but these are enough for the most use cases.

**Choose the cors middleware if you must use it per-route, otherwise cors plugin which offers more options.**

Let's take a quick overview, starting with the `cors plugin`.


## Cors Plugin 

### Install

```sh
$ go get -u github.com/iris-contrib/plugin/cors
```


```go
iris.Plugins.Add(cors.Default())
```

### Getting Started

```go
package main

import (
    "github.com/kataras/iris"
    "github.com/iris-contrib/plugin/cors"
)

func main() {
    app := iris.New()
    // cors.Default() setup the router with default options being
    // all origins accepted with simple methods (GET, POST). See
    // documentation below for more options.
    c := cors.Default()

    app.Plugins.Add(c)

    app.Get("/", func(ctx *iris.Context) {
        ctx.JSON(iris.StatusOK, iris.Map{"hello": "world"})
    })


    app.Listen(":8080")
}
```

### Parameters

Parameters are passed to the middleware thru the `cors.New` method as follow:

```go
c := cors.New(cors.Options{
    AllowedOrigins: []string{"http://foo.com"},
    AllowCredentials: true,
})

```

* **AllowedOrigins** `[]string`: A list of origins a cross-domain request can be executed from. If the special `*` value is present in the list, all origins will be allowed. An origin may contain a wildcard (`*`) to replace 0 or more characters (i.e.: `http://*.domain.com`). Usage of wildcards implies a small performance penality. Only one wildcard can be used per origin. The default value is `*`.
* **AllowOriginFunc** `func (origin string) bool`: A custom function to validate the origin. It take the origin as argument and returns true if allowed or false otherwise. If this option is set, the content of `AllowedOrigins` is ignored
* **AllowedMethods** `[]string`: A list of methods the client is allowed to use with cross-domain requests. Default value is simple methods (`GET` and `POST`).
* **AllowedHeaders** `[]string`: A list of non simple headers the client is allowed to use with cross-domain requests.
* **ExposedHeaders** `[]string`: Indicates which headers are safe to expose to the API of a CORS API specification
* **AllowCredentials** `bool`: Indicates whether the request can include user credentials like cookies, HTTP authentication or client side SSL certificates. The default is `false`.
* **MaxAge** `int`: Indicates how long (in seconds) the results of a preflight request can be cached. The default is `0` which stands for no max age.
* **OptionsPassthrough** `bool`: Instructs preflight to let other potential next handlers to process the `OPTIONS` method. Turn this on if your application handles `OPTIONS`.
* **Debug** `bool`: Debugging flag adds additional output to debug server side CORS issues.


## Cors Middleware


### Install 

```sh
$ go get -u github.com/iris-contrib/middleware/cors
```

```go

package main

import (
	"github.com/kataras/iris"
	"github.com/iris-contrib/middleware/cors"
)

func main() {
	iris.Use(cors.Default()) // enable all origins, disallow credentials

	iris.Get("/home", func(c *iris.Context) {
		c.Write("Hello from /home")
	})

	iris.Listen(":8080")

}

```

### Change origins and allowCredentials option

```go

package main

import (
	"github.com/kataras/iris"
	"github.com/iris-contrib/middleware/cors"
)

func main() {
  c := cors.New()
  // to enable credentials headers
  c.AllowCredentials()

  // allow origin per-domain
  c.AddOrigin("http://yourdomainhere.com")
  c.AddOrigin("http://otherdomainhere.com")

	iris.Use(c)

	iris.Get("/home", func(c *iris.Context) {
		c.Write("Hello from /home")
	})

	iris.Listen(":8080")

}

```



