# Handlers

Handlers, as the name implies, handle requests.   
Each of the handler registration methods described in the following subchapters returns a [`RouteNameFunc`](https://godoc.org/github.com/kataras/iris#RouteNameFunc) type.

Handlers must implement the Handler interface:

```go
type Handler interface {
	Serve(*Context)
}

// HandlerFunc implements the Handler, it's like the http.Handler and http.HandlerFunc you used to use.
type HandlerFunc func(*Context)

// convert helpers:

// Converts an http.Handler/HandlerFunc/func(w http.ResponseWriter, r *http.Request) to an iris Handler, this way you can use third-party handlers.
ToHandler(maybeHttpHandler interface{}) iris.HandlerFunc

// Converts an iris Handler to a native http.Handler, this way you can use your iris' handler to integrate with other http servers you may need somewhere.
ToNativeHandler(h iris.Handler) http.Handler
```

Once the handler is registered, we can use the returned [`RouteNameFunc`](https://godoc.org/github.com/kataras/iris#RouteNameFunc) type (which is actually just a `func` type) to give a name to the handler registration for easier lookup in code or in templates.
For more information, checkout the [Routing and reverse lookups](routing.md) section.
