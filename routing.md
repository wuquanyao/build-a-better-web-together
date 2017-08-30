# Routing

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

// http://myhost.com/users/42/profile
users.Get("/{id:int}/profile", userProfileHandler)
// http://myhost.com/users/messages/1
users.Get("/inbox/{id:int}", userMessageHandler)
```

The same could be also written using a function which accepts the child router(the Party).

```go
app := iris.New()

app.PartyFunc("/users", func(users iris.Party) {
    users.Use(myAuthMiddlewareHandler)

    // http://myhost.com/users/42/profile
    users.Get("/{id:int}/profile", userProfileHandler)
    // http://myhost.com/users/messages/1
    users.Get("/inbox/{id:int}", userMessageHandler)
})
```