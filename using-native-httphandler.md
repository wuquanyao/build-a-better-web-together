# Using native http.Handler

```go

type nativehandler struct {}

func (_ nativehandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {

}

func main() {
    iris.Handle("", "/path", iris.ToHandler(nativehandler{}))
    //"" means ANY(GET,POST,PUT,DELETE and so on)
}


```
