# Streaming

**context.StreamWriter**

```go
// StreamWriter registers the given stream writer for populating
// response body.
//
// Access to Context's and/or its' members is forbidden from writer.
//
// This function may be used in the following cases:
//
//     * if response body is too big (more than iris.LimitRequestBodySize(if setted)).
//     * if response body is streamed from slow external sources.
//     * if response body must be streamed to the client in chunks.
//     (aka `http server push`).
//
// receives a function which receives the response writer
// and returns false when it should stop writing, otherwise true in order to continue
StreamWriter(writer func(w io.Writer) bool)
```

**silly examples**

```go
package main

import (
	"fmt" // just an optional helper
	"io"
	"time" // showcase the delay

	"github.com/kataras/iris"
)

func main() {
	timeWaitForCloseStream := 4 * time.Second

	iris.Get("/", func(ctx *iris.Context) {
		i := 0
		// goroutine in order to no block and just wait,
		// goroutine is OPTIONAL and not a very good option but it depends on the needs
		// Look below for an alternative code style
		// Send the response in chunks and wait for a second between each chunk.
		go ctx.StreamWriter(func(w io.Writer) bool {
			i++
			fmt.Fprintf(w, "this is a message number %d\n", i) // write
			time.Sleep(time.Second)                            // imaginary delay

			return true // continue and flush
		})

		// when this handler finished the client should be see the stream writer's contents
		// simulate a job here...
		time.Sleep(timeWaitForCloseStream)
	})

	// or iris.ListenLETSENCRYPT to (logically) enable http/2 flush on stream writer
	iris.Listen(":8080")
}

```


```go

package main

import (
	"fmt" // just an optional helper
	"io"
	"time" // showcase the delay

	"github.com/kataras/iris"
)

func main() {

	iris.Get("/", func(ctx *iris.Context) {

		// Send the response in chunks and wait for a second between each chunk.
		ctx.StreamWriter(func(w io.Writer) bool {
			for i := 0; i <= 4; i++ {
				fmt.Fprintf(w, "this is a message number %d\n", i) // write
				time.Sleep(time.Second)
			}

			// when this handler finished the client should be see the stream writer's contents
			return false // stop and flush the contents
		})

	})

	// or iris.ListenLETSENCRYPT to (logically) enable http/2 flush on stream writer
	iris.Listen(":8080")
}


```