# Context's built'n gzip

Gzip compression is easy.


Activate **auto-gzip** for all responses and template engines,
just set `iris.Config.Gzip = true` or `iris.New(iris.OptionGzip(true))` or `iris.Set(OptionGzip(true))` . You can also enable gzipping for specific `Render()` calls:

```go
//...
context.Render("mytemplate.html", bindingStruct{}, iris.RenderOptions{"gzip": false})
context.Render("my-custom-response", iris.Map{"anything":"everything"} , iris.RenderOptions{"gzip": false})
```

```go
// WriteGzip accepts bytes, which are compressed to gzip format and sent to the client.
// returns the number of bytes written and an error ( if the client doesn' supports gzip compression)
WriteGzip(b []byte) (int, error)

// TryWriteGzip accepts bytes, which are compressed to gzip format and sent to the client.
// If client does not supprots gzip then the contents are written as they are, uncompressed.
TryWriteGzip(b []byte) (int, error)
```

How to use:
```go
iris.Get("/", func(ctx *iris.Context){
    ctx.WriteGzip([]byte("my gziped compressed content here"))
})

```

## Other

See [Static files](static-files.md) and learn how you can serve big files, assets or webpages with gzip compression.
