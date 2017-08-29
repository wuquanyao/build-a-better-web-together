# Cookies

Cookie management, even your little brother can do this!

```go
// SetCookie adds a cookie.
SetCookie(cookie *http.Cookie)

// SetCookieKV adds a cookie, receives just a key(string) and a value(string).
SetCookieKV(key, value string)

// GetCookie returns the cookie's value by it's name
// returns empty string if nothing was found.
GetCookie(name string) string

// RemoveCookie removes a cookie by it's name/key.
RemoveCookie(name string)


// VisitAllCookies takes a visitor which loops on each (request's) cookie key and value.
VisitAllCookies(visitor func(key string, value string))

```
How to use:
```go

app.Get("/set", func(ctx iris.Context){
    ctx.SetCookieKV("name","iris")
    ctx.Writef("Cookie has been setted.")
})

app.Get("/get", func(ctx iris.Context){
    name := ctx.GetCookie("name")
    ctx.Writef("Cookie's value: %s", name)
})

app.Get("/remove", func(ctx iris.Context){
    if name := ctx.GetCookie("name"); name != "" {
       ctx.RemoveCookie("name")  
    }
    ctx.Writef("Cookie has been removed.")
})

```