# Declaration

You might have asked yourself:

* Q: Other frameworks need more lines to start a server, why is Iris different?
* A: Iris gives you the freedom to choose between four ways to use Iris

  1. global **iris.**
  2. declare a new iris station with default config: **iris.New\(\)**
  3. declare a new iris station with custom config: ** api := iris.New\(iris.Configuration{...}\)**
  4. declare a new iris station with custom options: ** api := iris.New\(iris.OptionCharset\("UTF-8"\), iris.OptionSessionsCookie\("mycookie"\), ...\)**


Configuration is **OPTIONAL**, setted by with`$instance.Config.`, \/ `$instance.Set(Option...), $iris.New(config_or_options)`

```go

import "github.com/kataras/iris"

// 1.
func firstWay() {

    iris.Get("/home",func(ctx *iris.Context){})
    iris.Listen(":8080")
}
// 2.
func secondWay() {

    api := iris.New()
    api.Get("/home",func(ctx *iris.Context){})
    api.Listen(":8080")
}

// 3.
func thirdWay() {

   config := iris.Configuration{IsDevelopment: true}
   iris.New(config)
   iris.Get("/home", func(c*iris.Context){})
   iris.Listen(":8080")
}

// 4.
func forthWay() {

    api := iris.New()
    api.Set(iris.OptionCharset("UTF-8"))

    api.Get("/home",func(ctx *iris.Context){})
    api.Listen(":8080")
}

// after .New, at runtime, possible because Iris has default values. Configuration is TOTALLY OPTIONAL DSESIRE
func main() {
    iris.Config.Websocket.Endpoind = "/ws"

   //...

   iris.Listen(":8080")
}
```


> Note that with 2.,  3. & 4. you **can serve more than one Iris server** in the
> same app, when it's necessary.

`.New` **by configuration**

```go

import "github.com/kataras/iris"

//...



myConfig := iris.Configuration{Charset: "UTF-8", IsDevelopment:true, Sessions: iris.SessionsConfiguration{Cookie:"mycookie"}, Websocket: iris.WebsocketConfiguration{Endpoint: "/my_endpoint"}}

iris.New(myConfig)

```

`.New` **by options**

```go

import "github.com/kataras/iris"

//...



iris.New(iris.OptionCharset("UTF-8"), iris.OptionIsDevelopment(true),
iris.OptionSessionsCookie("mycookie"), iris.OptionWebsocketEndpoint("/my_endpoint"))



// if you want to set configuration after the .New use the .Set:

iris.Set(iris.OptionDisableBanner(true))

```
