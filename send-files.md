# Send files

Send a file, force-download to the client

**context's methods**
```go
// ServeContent serves content, headers are autoset
// receives three parameters, it's low-level function, instead you can use .ServeFile(string,bool)/SendFile(string,string)
//
// You can define your own "Content-Type" header also, after this function call
// Doesn't implements resuming (by range), use ctx.SendFile instead
ServeContent(content io.ReadSeeker, filename string, modtime time.Time, enableGzip bool) error

// ServeFile serves a view file, to send a file ( zip for example) to the client you should use the SendFile(serverfilename,clientfilename)
// receives two parameters
// filename/path (string)
// gzipCompression (bool)
//
// You can define your own "Content-Type" header also, after this function call
// This function doesn't implement resuming (by range), use ctx.SendFile instead
//
// Use it when you want to serve css/js/... files to the client, for bigger files and 'force-download' use the SendFile
ServeFile(filename string, enableGzip bool) error 	
// You can define your own "Content-Type" header also, after this function call
// for example: ctx.Response.Header.Set("Content-Type","thecontent/type")
SendFile(filename string, destinationName string)
```

```go
package main

import "github.com/kataras/iris"

func main() {

    iris.Get("/servezip", func(c *iris.Context) {
        file := "./files/first.zip"
        c.SendFile(file, "saveAsName.zip")
    })

    iris.Listen(":8080")
}
```



You can also send bytes manually, which will be downloaded by the user:

```go
package main

import "github.com/kataras/iris"

func main() {

    iris.Get("/servezip", func(c *iris.Context) {
        // read your file or anything
        var binary data[]
        c.Data(iris.StatusOK, data)

        // or c.Write(data)
    })

    iris.Listen(":8080")
}

```
