# Static files

Serve files or directories, use the correct for your case, if you don't know which one, just use the `StaticWeb(reqPath string, systemPath string)`.

```go
// Favicon serves static favicon
// accepts 2 parameters, second is optional
// favPath (string), declare the system directory path of the __.ico
// requestPath (string), it's the route's path, by default this is the "/favicon.ico" because some browsers tries to get this by default first,
// you can declare your own path if you have more than one favicon (desktop, mobile and so on)
//
// this func will add a route for you which will static serve the /yuorpath/yourfile.ico to the /yourfile.ico (nothing special that you can't handle by yourself)
// Note that you have to call it on every favicon you have to serve automatically (dekstop, mobile and so on)
//
// panics on error
Favicon(favPath string, requestPath ...string) RouteNameFunc

// StaticHandler returns a new Handler which serves static files
StaticHandler(reqPath string, systemPath string, showList bool, enableGzip bool) HandlerFunc

// StaticWeb same as Static but if index.html e
// xists and request uri is '/' then display the index.html's contents
// accepts three parameters
// first parameter is the request url path (string)
// second parameter is the system directory (string)
StaticWeb(reqPath string, systemPath string) RouteNameFunc

// StaticEmbedded  used when files are distrubuted inside the app executable, using go-bindata mostly
// First parameter is the request path, the path which the files in the vdir will be served to, for example "/static"
// Second parameter is the (virtual) directory path, for example "./assets"
// Third parameter is the Asset function
// Forth parameter is the AssetNames function
//
// For more take a look at the
// example: https://github.com/iris-contrib/examples/tree/master/static_files_embedded
StaticEmbedded(requestPath string, vdir string, assetFn func(name string) ([]byte, error), namesFn func() []string) RouteNameFunc

// StaticContent serves bytes, memory cached, on the reqPath
// a good example of this is how the websocket server uses that to auto-register the /iris-ws.js
StaticContent(reqPath string, cType string, content []byte) RouteNameFunc

// StaticServe serves a directory as web resource
// it's the simpliest form of the Static* functions
// Almost same usage as StaticWeb
// accepts only one required parameter which is the systemPath
// (the same path will be used to register the GET&HEAD routes)
// if the second parameter is empty, otherwise the requestPath is the second parameter
// it uses gzip compression (compression on each request, no file cache)
StaticServe(systemPath string, requestPath ...string)

```

```go
iris.Static("/public", "./static/assets/", 1)
//-> /public/assets/favicon.ico
```

```go
iris.StaticFS("/ftp", "./myfiles/public", 1)
```

```go
iris.StaticWeb("/","./my_static_html_website", 1)
```

```go
StaticServe(systemPath string, requestPath ...string)
```

### Context static file serving

```go
// ServeFile serves a view file, to send a file
// to the client you should use the SendFile(serverfilename,clientfilename)
// receives two parameters
// filename/path (string)
// gzipCompression (bool)
//
// You can define your own "Content-Type" header also, after this function call
ServeFile(filename string, gzip bool) error
```

Serve static individual file

```go

iris.Get("/txt", func(ctx *iris.Context) {
    ctx.ServeFile("./myfolder/staticfile.txt", false)
}
```

For example if you want manual serve static individual files dynamically you can do something like that:

```go
package main

import (
    "strings"
    "github.com/kataras/iris"
    "github.com/kataras/iris/utils"
)

func main() {

    iris.Get("/*file", func(ctx *iris.Context) {
            requestpath := ctx.Param("file")

            path := strings.Replace(requestpath, "/", utils.PathSeperator, -1)

            if !utils.DirectoryExists(path) {
                ctx.NotFound()
                return
            }

            ctx.ServeFile(path, false) // make this true to use gzip compression
    })

    iris.Listen(":8080")
}
```

The previous example is almost identical with:

```go
StaticWeb(requestPath string, systemPath string) RouteNameFunc
```

```go
func main() {
  iris.StaticWeb("/public", "./mywebpage")
  // Serves all files inside this directory to the GET&HEAD route: 0.0.0.0:8080/public
  // use iris.StaticHandler for more options like gzip.
  iris.Listen(":8080")
}
```

```go
func main() {
  iris.StaticServe("./static/mywebpage","/webpage")
  // Serves all files inside filesystem path ./static/mywebpage to the GET&HEAD route: 0.0.0.0:8080/webpage
  iris.Listen(":8080")
}
```

### Disabling caching


Caching can be disabled by setting `github.com/kataras/iris/config`'s `StaticCacheDuration` to `time.Duration(1)` **before calling any of the named functions**. Setting `StaticCacheDuration` to `time.Duration(0)` will reset the cache time to 10 seconds.

## Favicon

Imagine that we have a folder named `static` which has subfolder `favicons` and this folder contains a favicon, for example `iris_favicon_32_32.ico`.

```go
// ./main.go
package main

import "github.com/kataras/iris"

func main() {
    iris.Favicon("./static/favicons/iris_favicon_32_32.ico")

    iris.Get("/", func(ctx *iris.Context) {
        ctx.HTML(iris.StatusOK, "You should see the favicon now at the side of your browser.")
    })

    iris.Listen(":8080")
}


```

# Examples:
- [favicon](https://github.com/iris-contrib/examples/tree/master/favicon)
- [static files](https://github.com/iris-contrib/examples/tree/master/static_files)
- [static files embedded](https://github.com/iris-contrib/examples/tree/master/static_files_embedded)
- [static serve](https://github.com/iris-contrib/examples/tree/master/static_staticserve)
- [static web](https://github.com/iris-contrib/examples/tree/master/static_web), this is the most common.
