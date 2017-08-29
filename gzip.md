# Gzip

Gzip compression is easy.


Activate **auto-gzip** for all responses and template engines,
just set `app.Use(iris.Gzip)`. You can also enable gzipping for specific  responses from context:


```go
// ClientSupportsGzip retruns true if the client supports gzip compression.
ClientSupportsGzip() bool

// WriteGzip accepts bytes, which are compressed to gzip format and sent to the client.
// returns the number of bytes written and an error ( if the client doesn' supports gzip compression)
//
// This function writes temporary gzip contents, the ResponseWriter is untouched.
WriteGzip(b []byte) (int, error)

// TryWriteGzip accepts bytes, which are compressed to gzip format and sent to the client.
// If client does not supprots gzip then the contents are written as they are, uncompressed.
//
// This function writes temporary gzip contents, the ResponseWriter is untouched.
TryWriteGzip(b []byte) (int, error)

// GzipResponseWriter converts the current response writer into a response writer
// which when its .Write called it compress the data to gzip and writes them to the client.
//
// Can be also disabled with its .Disable and .ResetBody to rollback to the usual response writer.
GzipResponseWriter() *GzipResponseWriter

// Gzip enables or disables (if enabled before) the gzip response writer,if the client
// supports gzip compression, so the following response data will
// be sent as compressed gzip data to the client.
Gzip(enable bool)
```

How to use:

```go
app.Get("/", func(ctx iris.Context){
    ctx.WriteGzip([]byte("my gziped compressed content here"))
})

```