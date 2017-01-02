# Using native http.Handler via iris.ToHandler()

```go
iris.Get("/letsget", iris.ToHandler(nativehandler{}))
iris.Post("/letspost", iris.ToHandler(nativehandler{}))
iris.Put("/letsput", iris.ToHandler(nativehandler{}))
iris.Delete("/letsdelete", iris.ToHandler(nativehandler{}))

```
