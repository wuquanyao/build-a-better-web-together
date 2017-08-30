# Hosts

## Listen and Serve

You can start the server(s) listening to any type of `net.Listener` or even `http.Server` instance.
The method for initialization of the server should be passed at the end, via `Run` function.

The most common method that Go developers are use to serve their servers are
by passing a network address with form of "hostname:ip". With Iris
we use the `iris.Addr` which is an `iris.Runner` type

```go
// Listening on tcp with network address 0.0.0.0:8080
app.Run(iris.Addr(":8080"))
```

Sometimes you have created a standard net/http server somewhere else in your app and want to use that to serve the Iris web app

```go
// Same as before but using a custom http.Server which may being used somewhere else too
app.Run(iris.Server(&http.Server{Addr:":8080"}))
```

The most advanced usage is to create a custom or a standard `net.Listener` and pass that to `app.Run`

```go
// Using a custom net.Listener
l, err := net.Listen("tcp4", ":8080")
if err != nil {
    panic(err)
}
app.Run(iris.Listener(l))
```

```go
// UNIX socket
l, err := netutil.UNIX("/tmpl/srv.sock", 0666)
if err != nil {
panic(err)
}

app.Run(iris.Listener(l))
```

### Secure

If you have signed file keys you can use the `iris.TLS` to serve `https` based on those certification keys

```go
// TLS using files
app.Run(iris.TLS("127.0.0.1:443", "mycert.cert", "mykey.key"))
```

The method you should use when your app is ready for **production** is the `iris.AutoTLS` which starts a secure server with automated certifications provided by https://letsencrypt.org for **free**

```go
// Automatic TLS
app.Run(iris.AutoTLS(":443", "example.com", "admin@example.com"))
```

### Any `iris.Runner`

There may be times that you want something very special to listen on, which is not a type of `net.Listener`. You are able to do that by `iris.Raw`, but you're responsible of that method

```go
// Using any func() error,
// the responsibility of starting up a listener is up to you with this way,
// for the sake of simplicity we will use the
// ListenAndServe function of the `net/http` package.
app.Run(iris.Raw(&http.Server{Addr:":8080"}).ListenAndServe)
```


## Host configurators

All the above forms of listening are accepting a last, variadic argument of `func(*iris.Supervisor)`. This is used to add configurators for that specific host you passed via those functions.

For example let's say that we want to add a callback which is fired when
the server is shutdown

```go
app.Run(iris.Addr(":8080", func(h *iris.Supervisor) {
    h.RegisterOnShutdown(func() {
        println("server terminated")
    })
}))
```

You can even do that before `app.Run` method, but the difference is that
these host configurators will be executed to all hosts that you may use to serve your web app (via `app.NewHost` we'll see that in a minute)

```go
app := iris.New()
app.ConfigureHost(func(h *iris.Supervisor) {
    h.RegisterOnShutdown(func() {
        println("server terminated")
    })
})
app.Run(iris.Addr(":8080"))
```

> You can get the running hosts via the `app.Hosts` field note that **this is filled after the app.Run method**.

## Multi hosts

You can serve your Iris web app using more than one server, the `iris.Router` is compatible with the `net/http/Handler` function therefore, as you can understand, it can be used to be adapted at any `net/http` server, however there is an easier way, by using the `app.NewHost` which is also copying all the host configurators and it closes all the hosts attached to the particular web app on `app.Shutdown`.

```go
app := iris.New()
app.Get("/", indexHandler)

// run in different goroutine in order to not block the main "goroutine".
go app.Run(iris.Addr(":8080"))
// start a second server which is listening on tcp 0.0.0.0:9090,
// without "go" keyword because we want to block at the last server-run.
app.NewHost(&http.Server{Addr:":9090"}).ListenAndServe()
```