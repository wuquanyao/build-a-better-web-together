# Configuration

All configurations have default values, things will work as you expected with `iris.New()`

but if you want to customize iris then pass `iris.New(iris.Configuration{})` or use the options-func, example: `iris.New(iris.OptionCharset("UTF-8"))`

`.New` **by configuration**
```go
import "github.com/kataras/iris"
//...

myConfig := iris.Configuration{Charset: "UTF-8", IsDevelopment:true, Sessions: iris.SessionsConfiguration{Cookie:"mycookie"}, Websocket: iris.WebsocketConfiguration{Endpoint: "/my_endpoint"}}

app := iris.New(myConfig)
```

`.New` **by options**

```go
import "github.com/kataras/iris"
//...

iris.New(iris.OptionCharset("UTF-8"), iris.OptionIsDevelopment(true), iris.OptionSessionsCookie("mycookie"), iris.OptionWebsocketEndpoint("/my_endpoint"))

// if you want to set configuration after the .New use the .Set:
iris.Set(iris.OptionDisableBanner(true))
```


### Some of the available settings are:

```go
// OptionDisablePathCorrection corrects and redirects the requested path to the registed path
// for example, if /home/ path is requested but no handler for this Route found,
// then the Router checks if /home handler exists, if yes,
// (permant)redirects the client to the correct path /home
//
// Default is false
OptionDisablePathCorrection(val bool)


// OptionDisableBanner outputs the iris banner at startup
//
// Default is false
OptionDisableBanner(val bool)

// OptionLoggerOut is the destination for output
//
// Default is os.Stdout
OptionLoggerOut(val io.Writer)

// OptionLoggerPreffix is the logger's prefix to write at beginning of each line
//
// Default is [IRIS]
OptionLoggerPreffix(val string)

// OptionProfilePath a the route path, set it to enable http pprof tool
// Default is empty, if you set it to a $path, these routes will handled:
OptionProfilePath(val string)

// OptionDisableTemplateEngines set to true to disable loading the default template engine (html/template) and disallow the use of iris.UseEngine
// Default is false
OptionDisableTemplateEngines(val bool)

// OptionIsDevelopment iris will act like a developer, for example
// If true then re-builds the templates on each request
// Default is false
OptionIsDevelopment(val bool)

// OptionTimeFormat time format for any kind of datetime parsing
OptionTimeFormat(val string)

// OptionCharset character encoding for various rendering
// used for templates and the rest of the responses(via serializer)
// Default is "UTF-8"
OptionCharset(val string)

// OptionGzip enables gzip compression on your Render actions, this includes any type of render, templates and pure/raw content
// If you don't want to enable it globaly, you could just use the third parameter on context.Render("myfileOrResponse", structBinding{}, iris.RenderOptions{"gzip": true})
// Default is false
OptionGzip(val bool)

// OptionOther are the custom, dynamic options, can be empty
// this fill used only by you to set any app's options you want
// for each of an Iris instance
OptionOther(val ...options.Options) //map[string]interface{}, options is github.com/kataras/go-options

// OptionSessionsCookie string, the session's client cookie name, for example: "qsessionid"
OptionSessionsCookie(val string)

// OptionSessionsDecodeCookie set it to true to decode the cookie key with base64 URLEncoding
// Defaults to false
OptionSessionsDecodeCookie(val bool)

// OptionSessionsExpires the duration of which the cookie must expires (created_time.Add(Expires)).
// If you want to delete the cookie when the browser closes, set it to -1 but in this case, the server side's session duration is up to GcDuration
//
// Default infinitive/unlimited life duration(0)
OptionSessionsExpires(val time.Duration)

// OptionSessionsCookieLength the length of the sessionid's cookie's value, let it to 0 if you don't want to change it
// Defaults to 32
OptionSessionsCookieLength(val int)

// OptionSessionsGcDuration every how much duration(GcDuration) the memory should be clear for unused cookies (GcDuration)
// for example: time.Duration(2)*time.Hour. it will check every 2 hours if cookie hasn't be used for 2 hours,
// deletes it from backend memory until the user comes back, then the session continue to work as it was
//
// Default 2 hours
OptionSessionsGcDuration(val time.Duration)

// OptionSessionsDisableSubdomainPersistence set it to true in order dissallow your q subdomains to have access to the session cookie
// defaults to false
OptionSessionsDisableSubdomainPersistence(val bool)

// OptionWebsocketWriteTimeout time allowed to write a message to the connection.
// Default value is 15 * time.Second
OptionWebsocketWriteTimeout(val time.Duration)

// OptionWebsocketPongTimeout allowed to read the next pong message from the connection
// Default value is 60 * time.Second
OptionWebsocketPongTimeout(val time.Duration)

// OptionWebsocketPingPeriod send ping messages to the connection with this period. Must be less than PongTimeout
// Default value is (PongTimeout * 9) / 10
OptionWebsocketPingPeriod(val time.Duration)

// OptionWebsocketMaxMessageSize max message size allowed from connection
// Default value is 1024
OptionWebsocketMaxMessageSize(val int64)

// OptionWebsocketBinaryMessages set it to true in order to denotes binary data messages instead of utf-8 text
// see https://github.com/kataras/iris/issues/387#issuecomment-243006022 for more
// defaults to false
OptionWebsocketBinaryMessages(val bool)

// OptionWebsocketEndpoint is the path which the websocket server will listen for clients/connections
// Default value is empty string, if you don't set it the Websocket server is disabled.
OptionWebsocketEndpoint(val string)

// OptionWebsocketReadBufferSize is the buffer size for the underline reader
OptionWebsocketReadBufferSize(val int)

// OptionWebsocketWriteBufferSize is the buffer size for the underline writer
OptionWebsocketWriteBufferSize(val int)
```go


### ServerConfiguration

Note: One iris instance can have and listening to many iris' Servers, one iris' Server can be used/passed to many iris instance, with that in mind, the server configuration has its own options to set.


```go
app := iris.New()

// your routes and other configuration here...
app.Get("/", indexHandler)

// start the main server in goroutine
go app.Listen(":80")

// create and start a new native server whith the iris' router
fsrv := &http.Server{Handler: app.Router, Addr: ":8080"}
fsrv.ListenAndServe()
```

```go
app := iris.New()

// your routes and other configuration here...
app.Get("/", indexHandler)

// build your iris in order make the Router
app.Build()

// create and start a new native server whith the iris' router
fsrv := &http.Server{Handler: app.Router, Addr: ":8080"}
go fsrv.ListenAndServe()

// or ignore the main server, you can use only the native one if you liked so.
app.Listen(":80")
```

**List** of all iris' Server's config:

```go
// VHost is the addr or the domain that server listens to, which it's optional
// When to set VHost manually:
// 1. it's automatically setted when you're calling
//     $instance.Listen/ListenUNIX/ListenTLS/ListenLETSENCRYPT functions or
//     ln,_ := iris.TCP4/UNIX/TLS/LETSENCRYPT; $instance.Serve(ln)
// 2. If you using a balancer, or something like nginx
//    then set it in order to have the correct url
//    when calling the template helper '{{url }}'
//    *keep note that you can use {{urlpath }}) instead*
//
// Note: this is the main's server Host, you can setup unlimited number of net/http servers
// listening to the $instance.Handler after the manually-called $instance.Build
//
// Default comes from iris.Listen/.Serve with iris' listeners (iris.TCP4/UNIX/TLS/LETSENCRYPT)
VHost string

// VScheme is the scheme (http:// or https://) putted at the template function '{{url }}'
// It's an optional field,
// When to set VScheme manually:
// 1. You didn't start the main server using $instance.Listen/ListenTLS/ListenLETSENCRYPT or $instance.Serve($instance.TCP4()/.TLS...)
// 2. if you're using something like nginx and have iris listening with addr only(http://) but the nginx mapper is listening to https://
//
// Default comes from iris.Listen/.Serve with iris' listeners (TCP4,UNIX,TLS,LETSENCRYPT)
VScheme string

ReadTimeout  time.Duration // maximum duration before timing out read of the request
WriteTimeout time.Duration // maximum duration before timing out write of the response

// MaxHeaderBytes controls the maximum number of bytes the
// server will read parsing the request header's keys and
// values, including the request line. It does not limit the
// size of the request body.
// If zero, DefaultMaxHeaderBytes is used.
MaxHeaderBytes int

// TLSNextProto optionally specifies a function to take over
// ownership of the provided TLS connection when an NPN/ALPN
// protocol upgrade has occurred. The map key is the protocol
// name negotiated. The Handler argument should be used to
// handle HTTP requests and will initialize the Request's TLS
// and RemoteAddr if not already set. The connection is
// automatically closed when the function returns.
// If TLSNextProto is nil, HTTP/2 support is enabled automatically.
TLSNextProto map[string]func(*http.Server, *tls.Conn, http.Handler)

// ConnState specifies an optional callback function that is
// called when a client connection changes state. See the
// ConnState type and associated constants for details.
ConnState func(net.Conn, http.ConnState)

```    

```go
package main

import (
	"github.com/kataras/iris"
)

func main(){
	app := iris.New(iris.Configuration{VHost: "mydomain.com", VScheme: "https://"})

	app.Listen(":8080")
}
```


View all configuration fields and options by navigating to the [kataras/iris/configuration.go source file](https://github.com/kataras/iris/blob/master/configuration.go), to view the testing configuration move [here](https://github.com/kataras/iris/blob/master/httptest/httptest.go)
