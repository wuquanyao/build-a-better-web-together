# Context

The `iris.Context` source code can be found [here](https://github.com/kataras/iris/blob/master/context/context.go). Keep note using an IDE/Editors with `auto-complete` feature will help you a lot.

```go
// Context is the midle-man server's "object" for the clients.
//
// A New context is being acquired from a sync.Pool on each connection.
// The Context is the most important thing on the iris's http flow.
//
// Developers send responses to the client's request through a Context.
// Developers get request information from the client's request a Context.
//
// This context is an implementation of the context.Context sub-package.
// context.Context is very extensible and developers can override
// its methods if that is actually needed.
type Context interface {
    // ResponseWriter returns an http.ResponseWriter compatible response writer, as expected.
    ResponseWriter() ResponseWriter
    // ResetResponseWriter should change or upgrade the Context's ResponseWriter.
    ResetResponseWriter(ResponseWriter)

    // Request returns the original *http.Request, as expected.
    Request() *http.Request

    // SetCurrentRouteName sets the route's name internally,
    // in order to be able to find the correct current "read-only" Route when
    // end-developer calls the `GetCurrentRoute()` function.
    // It's being initialized by the Router, if you change that name
    // manually nothing really happens except that you'll get other
    // route via `GetCurrentRoute()`.
    // Instead, to execute a different path
    // from this context you should use the `Exec` function
    // or change the handlers via `SetHandlers/AddHandler` functions.
    SetCurrentRouteName(currentRouteName string)
    // GetCurrentRoute returns the current registered "read-only" route that
    // was being registered to this request's path.
    GetCurrentRoute() RouteReadOnly

    // AddHandler can add handler(s)
    // to the current request in serve-time,
    // these handlers are not persistenced to the router.
    //
    // Router is calling this function to add the route's handler.
    // If AddHandler called then the handlers will be inserted
    // to the end of the already-defined route's handler.
    //
    AddHandler(...Handler)
    // SetHandlers replaces all handlers with the new.
    SetHandlers(Handlers)
    // Handlers keeps tracking of the current handlers.
    Handlers() Handlers

    // HandlerIndex sets the current index of the
    // current context's handlers chain.
    // If -1 passed then it just returns the
    // current handler index without change the current index.rns that index, useless return value.
    //
    // Look Handlers(), Next() and StopExecution() too.
    HandlerIndex(n int) (currentIndex int)
    // HandlerName returns the current handler's name, helpful for debugging.
    HandlerName() string
    // Next calls all the next handler from the handlers chain,
    // it should be used inside a middleware.
    //
    // Note: Custom context should override this method in order to be able to pass its own context.Context implementation.
    Next()
    // NextHandler returns(but it is NOT executes) the next handler from the handlers chain.
    //
    // Use .Skip() to skip this handler if needed to execute the next of this returning handler.
    NextHandler() Handler
    // Skip skips/ignores the next handler from the handlers chain,
    // it should be used inside a middleware.
    Skip()
    // StopExecution if called then the following .Next calls are ignored.
    StopExecution()
    // IsStopped checks and returns true if the current position of the Context is 255,
    // means that the StopExecution() was called.
    IsStopped() bool

    //  +------------------------------------------------------------+
    //  | Current "user/request" storage                             |
    //  | and share information between the handlers - Values().     |
    //  | Save and get named path parameters - Params()              |
    //  +------------------------------------------------------------+

    // Params returns the current url's named parameters key-value storage.
    // Named path parameters are being saved here.
    // This storage, as the whole Context, is per-request lifetime.
    Params() *RequestParams

    // Values returns the current "user" storage.
    // Named path parameters and any optional data can be saved here.
    // This storage, as the whole Context, is per-request lifetime.
    //
    // You can use this function to Set and Get local values
    // that can be used to share information between handlers and middleware.
    Values() *memstore.Store
    // Translate is the i18n (localization) middleware's function,
    // it calls the Get("translate") to return the translated value.
    //
    // Example: https://github.com/kataras/iris/tree/master/_examples/miscellaneous/i18n
    Translate(format string, args ...interface{}) string

    //  +------------------------------------------------------------+
    //  | Path, Host, Subdomain, IP, Headers etc...                  |
    //  +------------------------------------------------------------+

    // Method returns the request.Method, the client's http method to the server.
    Method() string
    // Path returns the full request path,
    // escaped if EnablePathEscape config field is true.
    Path() string
    // RequestPath returns the full request path,
    // based on the 'escape'.
    RequestPath(escape bool) string

    // Host returns the host part of the current url.
    Host() string
    // Subdomain returns the subdomain of this request, if any.
    // Note that this is a fast method which does not cover all cases.
    Subdomain() (subdomain string)
    // RemoteAddr tries to parse and return the real client's request IP.
    //
    // Based on allowed headers names that can be modified from Configuration.RemoteAddrHeaders.
    //
    // If parse based on these headers fail then it will return the Request's `RemoteAddr` field
    // which is filled by the server before the HTTP handler.
    //
    // Look `Configuration.RemoteAddrHeaders`,
    //      `Configuration.WithRemoteAddrHeader(...)`,
    //      `Configuration.WithoutRemoteAddrHeader(...)` for more.
    RemoteAddr() string
    // GetHeader returns the request header's value based on its name.
    GetHeader(name string) string
    // IsAjax returns true if this request is an 'ajax request'( XMLHttpRequest)
    //
    // There is no a 100% way of knowing that a request was made via Ajax.
    // You should never trust data coming from the client, they can be easily overcome by spoofing.
    //
    // Note that "X-Requested-With" Header can be modified by any client(because of "X-"),
    // so don't rely on IsAjax for really serious stuff,
    // try to find another way of detecting the type(i.e, content type),
    // there are many blogs that describe these problems and provide different kind of solutions,
    // it's always depending on the application you're building,
    // this is the reason why this `IsAjax`` is simple enough for general purpose use.
    //
    // Read more at: https://developer.mozilla.org/en-US/docs/AJAX
    // and https://xhr.spec.whatwg.org/
    IsAjax() bool

    //  +------------------------------------------------------------+
    //  | Response Headers helpers                                   |
    //  +------------------------------------------------------------+

    // Header adds a header to the response writer.
    Header(name string, value string)

    // ContentType sets the response writer's header key "Content-Type" to the 'cType'.
    ContentType(cType string)
    // GetContentType returns the response writer's header value of "Content-Type"
    // which may, setted before with the 'ContentType'.
    GetContentType() string

    // StatusCode sets the status code header to the response.
    // Look .GetStatusCode too.
    StatusCode(statusCode int)
    // GetStatusCode returns the current status code of the response.
    // Look StatusCode too.
    GetStatusCode() int

    // Redirect redirect sends a redirect response the client
    // accepts 2 parameters string and an optional int
    // first parameter is the url to redirect
    // second parameter is the http status should send, default is 302 (StatusFound),
    // you can set it to 301 (Permant redirect), if that's nessecery
    Redirect(urlToRedirect string, statusHeader ...int)

    //  +------------------------------------------------------------+
    //  | Various Request and Post Data                              |
    //  +------------------------------------------------------------+

    // URLParam returns the get parameter from a request , if any.
    URLParam(name string) string
    // URLParamInt returns the url query parameter as int value from a request,
    // returns an error if parse failed.
    URLParamInt(name string) (int, error)
    // URLParamInt64 returns the url query parameter as int64 value from a request,
    // returns an error if parse failed.
    URLParamInt64(name string) (int64, error)
    // URLParams returns a map of GET query parameters separated by comma if more than one
    // it returns an empty map if nothing found.
    URLParams() map[string]string

    // FormValue returns a single form value by its name/key
    FormValue(name string) string
    // FormValues returns all post data values with their keys
    // form data, get, post & put query arguments
    //
    // NOTE: A check for nil is necessary.
    FormValues() map[string][]string
    // PostValue returns a form's only-post value by its name,
    // same as Request.PostFormValue.
    PostValue(name string) string
    // FormFile returns the first file for the provided form key.
    // FormFile calls ctx.Request.ParseMultipartForm and ParseForm if necessary.
    //
    // same as Request.FormFile.
    FormFile(key string) (multipart.File, *multipart.FileHeader, error)

    //  +------------------------------------------------------------+
    //  | Custom HTTP Errors                                         |
    //  +------------------------------------------------------------+

    // NotFound emits an error 404 to the client, using the specific custom error error handler.
    // Note that you may need to call ctx.StopExecution() if you don't want the next handlers
    // to be executed. Next handlers are being executed on iris because you can alt the
    // error code and change it to a more specific one, i.e
    // users := app.Party("/users")
    // users.Done(func(ctx context.Context){ if ctx.StatusCode() == 400 { /*  custom error code for /users */ }})
    NotFound()

    //  +------------------------------------------------------------+
    //  | Body Readers                                               |
    //  +------------------------------------------------------------+

    // SetMaxRequestBodySize sets a limit to the request body size
    // should be called before reading the request body from the client.
    SetMaxRequestBodySize(limitOverBytes int64)

    // UnmarshalBody reads the request's body and binds it to a value or pointer of any type
    // Examples of usage: context.ReadJSON, context.ReadXML.
    UnmarshalBody(v interface{}, unmarshaler Unmarshaler) error
    // ReadJSON reads JSON from request's body and binds it to a value of any json-valid type.
    ReadJSON(jsonObject interface{}) error
    // ReadXML reads XML from request's body and binds it to a value of any xml-valid type.
    ReadXML(xmlObject interface{}) error
    // ReadForm binds the formObject  with the form data
    // it supports any kind of struct.
    ReadForm(formObject interface{}) error

    //  +------------------------------------------------------------+
    //  | Body (raw) Writers                                         |
    //  +------------------------------------------------------------+

    // Write writes the data to the connection as part of an HTTP reply.
    //
    // If WriteHeader has not yet been called, Write calls
    // WriteHeader(http.StatusOK) before writing the data. If the Header
    // does not contain a Content-Type line, Write adds a Content-Type set
    // to the result of passing the initial 512 bytes of written data to
    // DetectContentType.
    //
    // Depending on the HTTP protocol version and the client, calling
    // Write or WriteHeader may prevent future reads on the
    // Request.Body. For HTTP/1.x requests, handlers should read any
    // needed request body data before writing the response. Once the
    // headers have been flushed (due to either an explicit Flusher.Flush
    // call or writing enough data to trigger a flush), the request body
    // may be unavailable. For HTTP/2 requests, the Go HTTP server permits
    // handlers to continue to read the request body while concurrently
    // writing the response. However, such behavior may not be supported
    // by all HTTP/2 clients. Handlers should read before writing if
    // possible to maximize compatibility.
    Write(body []byte) (int, error)
    // Writef formats according to a format specifier and writes to the response.
    //
    // Returns the number of bytes written and any write error encountered.
    Writef(format string, args ...interface{}) (int, error)
    // WriteString writes a simple string to the response.
    //
    // Returns the number of bytes written and any write error encountered.
    WriteString(body string) (int, error)
    // WriteWithExpiration like Write but it sends with an expiration datetime
    // which is refreshed every package-level `StaticCacheDuration` field.
    WriteWithExpiration(body []byte, modtime time.Time) (int, error)
    // StreamWriter registers the given stream writer for populating
    // response body.
    //
    // Access to context's and/or its' members is forbidden from writer.
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

    //  +------------------------------------------------------------+
    //  | Body Writers with compression                              |
    //  +------------------------------------------------------------+
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

    //  +------------------------------------------------------------+
    //  | Rich Body Content Writers/Renderers                        |
    //  +------------------------------------------------------------+

    // ViewLayout sets the "layout" option if and when .View
    // is being called afterwards, in the same request.
    // Useful when need to set or/and change a layout based on the previous handlers in the chain.
    //
    // Note that the 'layoutTmplFile' argument can be setted to iris.NoLayout || view.NoLayout
    // to disable the layout for a specific view render action,
    // it disables the engine's configuration's layout property.
    //
    // Look .ViewData and .View too.
    //
    // Example: https://github.com/kataras/iris/tree/master/_examples/view/context-view-data/
    ViewLayout(layoutTmplFile string)

    // ViewData saves one or more key-value pair in order to be passed if and when .View
    // is being called afterwards, in the same request.
    // Useful when need to set or/and change template data from previous hanadlers in the chain.
    //
    // If .View's "binding" argument is not nil and it's not a type of map
    // then these data are being ignored, binding has the priority, so the main route's handler can still decide.
    // If binding is a map or context.Map then these data are being added to the view data
    // and passed to the template.
    //
    // After .View, the data are not destroyed, in order to be re-used if needed (again, in the same request as everything else),
    // to clear the view data, developers can call:
    // ctx.Set(ctx.Application().ConfigurationReadOnly().GetViewDataContextKey(), nil)
    //
    // If 'key' is empty then the value is added as it's (struct or map) and developer is unable to add other value.
    //
    // Look .ViewLayout and .View too.
    //
    // Example: https://github.com/kataras/iris/tree/master/_examples/view/context-view-data/
    ViewData(key string, value interface{})

    // GetViewData returns the values registered by `context#ViewData`.
    // The return value is `map[string]interface{}`, this means that
    // if a custom struct registered to ViewData then this function
    // will try to parse it to map, if failed then the return value is nil
    // A check for nil is always a good practise if different
    // kind of values or no data are registered via `ViewData`.
    //
    // Similarly to `viewData := ctx.Values().Get("iris.viewData")` or
    // `viewData := ctx.Values().Get(ctx.Application().ConfigurationReadOnly().GetViewDataContextKey())`.
    GetViewData() map[string]interface{}

    // View renders templates based on the adapted view engines.
    // First argument accepts the filename, relative to the view engine's Directory,
    // i.e: if directory is "./templates" and want to render the "./templates/users/index.html"
    // then you pass the "users/index.html" as the filename argument.
    //
    // Look: .ViewData and .ViewLayout too.
    //
    // Examples: https://github.com/kataras/iris/tree/master/_examples/view/
    View(filename string) error

    // Binary writes out the raw bytes as binary data.
    Binary(data []byte) (int, error)
    // Text writes out a string as plain text.
    Text(text string) (int, error)
    // HTML writes out a string as text/html.
    HTML(htmlContents string) (int, error)
    // JSON marshals the given interface object and writes the JSON response.
    JSON(v interface{}, options ...JSON) (int, error)
    // JSONP marshals the given interface object and writes the JSON response.
    JSONP(v interface{}, options ...JSONP) (int, error)
    // XML marshals the given interface object and writes the XML response.
    XML(v interface{}, options ...XML) (int, error)
    // Markdown parses the markdown to html and renders to client.
    Markdown(markdownB []byte, options ...Markdown) (int, error)

    //  +------------------------------------------------------------+
    //  | Serve files                                                |
    //  +------------------------------------------------------------+

    // ServeContent serves content, headers are autoset
    // receives three parameters, it's low-level function, instead you can use .ServeFile(string,bool)/SendFile(string,string)
    //
    // You can define your own "Content-Type" header also, after this function call
    // Doesn't implements resuming (by range), use ctx.SendFile instead
    ServeContent(content io.ReadSeeker, filename string, modtime time.Time, gzipCompression bool) error
    // ServeFile serves a view file, to send a file ( zip for example) to the client you should use the SendFile(serverfilename,clientfilename)
    // receives two parameters
    // filename/path (string)
    // gzipCompression (bool)
    //
    // You can define your own "Content-Type" header also, after this function call
    // This function doesn't implement resuming (by range), use ctx.SendFile instead
    //
    // Use it when you want to serve css/js/... files to the client, for bigger files and 'force-download' use the SendFile.
    ServeFile(filename string, gzipCompression bool) error
    // SendFile sends file for force-download to the client
    //
    // Use this instead of ServeFile to 'force-download' bigger files to the client.
    SendFile(filename string, destinationName string) error

    //  +------------------------------------------------------------+
    //  | Cookies                                                    |
    //  +------------------------------------------------------------+

    // SetCookie adds a cookie
    SetCookie(cookie *http.Cookie)
    // SetCookieKV adds a cookie, receives just a name(string) and a value(string)
    //
    // If you use this method, it expires at 2 hours
    // use ctx.SetCookie or http.SetCookie if you want to change more fields.
    SetCookieKV(name, value string)
    // GetCookie returns cookie's value by it's name
    // returns empty string if nothing was found.
    GetCookie(name string) string
    // RemoveCookie deletes a cookie by it's name.
    RemoveCookie(name string)
    // VisitAllCookies takes a visitor which loops
    // on each (request's) cookies' name and value.
    VisitAllCookies(visitor func(name string, value string))

    // MaxAge returns the "cache-control" request header's value
    // seconds as int64
    // if header not found or parse failed then it returns -1.
    MaxAge() int64

    //  +------------------------------------------------------------+
    //  | Advanced: Response Recorder and Transactions               |
    //  +------------------------------------------------------------+

    // Record transforms the context's basic and direct responseWriter to a ResponseRecorder
    // which can be used to reset the body, reset headers, get the body,
    // get & set the status code at any time and more.
    Record()
    // Recorder returns the context's ResponseRecorder
    // if not recording then it starts recording and returns the new context's ResponseRecorder
    Recorder() *ResponseRecorder
    // IsRecording returns the response recorder and a true value
    // when the response writer is recording the status code, body, headers and so on,
    // else returns nil and false.
    IsRecording() (*ResponseRecorder, bool)

    // BeginTransaction starts a scoped transaction.
    //
    // You can search third-party articles or books on how Business Transaction works (it's quite simple, especially here).
    //
    // Note that this is unique and new
    // (=I haver never seen any other examples or code in Golang on this subject, so far, as with the most of iris features...)
    // it's not covers all paths,
    // such as databases, this should be managed by the libraries you use to make your database connection,
    // this transaction scope is only for context's response.
    // Transactions have their own middleware ecosystem also, look iris.go:UseTransaction.
    //
    // See https://github.com/kataras/iris/tree/master/_examples/ for more
    BeginTransaction(pipe func(t *Transaction))
    // SkipTransactions if called then skip the rest of the transactions
    // or all of them if called before the first transaction
    SkipTransactions()
    // TransactionsSkipped returns true if the transactions skipped or canceled at all.
    TransactionsSkipped() bool

    // Exec calls the framewrok's ServeCtx
    // based on this context but with a changed method and path
    // like it was requested by the user, but it is not.
    //
    // Offline means that the route is registered to the iris and have all features that a normal route has
    // BUT it isn't available by browsing, its handlers executed only when other handler's context call them
    // it can validate paths, has sessions, path parameters and all.
    //
    // You can find the Route by app.GetRoute("theRouteName")
    // you can set a route name as: myRoute := app.Get("/mypath", handler)("theRouteName")
    // that will set a name to the route and returns its RouteInfo instance for further usage.
    //
    // It doesn't changes the global state, if a route was "offline" it remains offline.
    //
    // app.None(...) and app.GetRoutes().Offline(route)/.Online(route, method)
    //
    // Example: https://github.com/kataras/iris/tree/master/_examples/routing/route-state
    //
    // User can get the response by simple using rec := ctx.Recorder(); rec.Body()/rec.StatusCode()/rec.Header().
    //
    // Context's Values and the Session are kept in order to be able to communicate via the result route.
    //
    // It's for extreme use cases, 99% of the times will never be useful for you.
    Exec(method string, path string)

    // Application returns the iris app instance which belongs to this context.
    // Worth to notice that this function returns an interface
    // of the Application, which contains methods that are safe
    // to be executed at serve-time. The full app's fields
    // and methods are not available here for the developer's safety.
    Application() Application
}
```

The [examples](https://github.com/kataras/iris/tree/master/_examples) will give you the direction.
