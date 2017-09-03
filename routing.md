# Routing

## Handlers

A Handler, as the name implies, handle requests.

```go
// A Handler responds to an HTTP request.
// It writes reply headers and data to the
// Context.ResponseWriter() and then return.
// Returning signals that the request is finished;
// it is not valid to use the Context after or
// concurrently with the completion of the Handler call.
//
// Depending on the HTTP client software, HTTP protocol version,
// and any intermediaries between the client and the iris server,
// it may not be possible to read from the
// Context.Request().Body after writing to the context.ResponseWriter().
// Cautious handlers should read the Context.Request().Body first, and then reply.
//
// Except for reading the body, handlers should not modify the provided Context.
//
// If Handler panics, the server (the caller of Handler) assumes that
// the effect of the panic was isolated to the active request.
// It recovers the panic, logs a stack trace to the server error log
// and hangs up the connection.
type Handler func(Context)
```

Once the handler is registered, we can use the returned [`Route`](https://godoc.org/github.com/kataras/iris/core/router#Route) instance to give a name to the handler registration for easier lookup in code or in templates.

For more information, checkout the [Routing and reverse lookups](routing_reverse.md) section.

## API

All HTTP methods are supported, developers can also register handlers for same paths for different methods.

The first parameter is the HTTP Method,
second parameter is the request path of the route,
third variadic parameter should contains one or more iris.Handler executed
by the registered order when a user requests for that specific resouce path from the server.

Example code:

```go
app := iris.New()

app.Handle("GET", "/contact", func(ctx iris.Context) {
    ctx.HTML("<h1> Hello from /contact </h1>")
})
```

In order to make things easier for the end-developer, iris provides functions for all HTTP Methods.
The first parameter is the request path of the route,
second variadic parameter should contains one or more iris.Handler executed
by the registered order when a user requests for that specific resouce path from the server.

Example code:

```go
app := iris.New()

// Method: "GET"
app.Get("/", handler)

// Method: "POST"
app.Post("/", handler)

// Method: "PUT"
app.Put("/", handler)

// Method: "DELETE"
app.Delete("/", handler)

// Method: "OPTIONS"
app.Options("/", handler)

// Method: "TRACE"
app.Trace("/", handler)

// Method: "CONNECT"
app.Connect("/", handler)

// Method: "HEAD"
app.Head("/", handler)

// Method: "PATCH"
app.Patch("/", handler)

// register the route for all HTTP Methods
app.Any("/", handler)

func handler(ctx iris.Context){
    ctx.Writef("Hello from method: %s and path: %s", ctx.Method(), ctx.Path())
}
```

## Grouping Routes

A set of routes that are being groupped by path prefix can (optionally) share the same middleware handlers and template layout.
A group can have a nested group too.

`.Party` is being used to group routes, developers can declare an unlimited number of (nested) groups.

Example code:

```go
app := iris.New()

users := app.Party("/users", myAuthMiddlewareHandler)

// http://localhost:8080/users/42/profile
users.Get("/{id:int}/profile", userProfileHandler)
// http://localhost:8080/users/messages/1
users.Get("/inbox/{id:int}", userMessageHandler)
```

The same could be also written using a function which accepts the child router(the Party).

```go
app := iris.New()

app.PartyFunc("/users", func(users iris.Party) {
    users.Use(myAuthMiddlewareHandler)

    // http://localhost:8080/users/42/profile
    users.Get("/{id:int}/profile", userProfileHandler)
    // http://localhost:8080/users/messages/1
    users.Get("/inbox/{id:int}", userMessageHandler)
})
```

> `id:int` is a (typed) dynamic path parameter, learn more at the [parameterized paths](routing_parameters.md) section.
