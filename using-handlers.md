# Using Handlers

```go

type myHandlerGet struct {
}

func (m myHandlerGet) Serve(ctx *iris.Context) {
    ctx.Writef("From %s", ctx.Path())
}

// and so on


iris.Handle("GET", "/get", myHandlerGet{})
iris.Handle("POST", "/post", post)
iris.Handle("PUT", "/put", put)
iris.Handle("DELETE", "/delete", del)
```
