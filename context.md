# Context

The context source code can be found [here](https://github.com/kataras/iris/blob/master/context.go). Keep note that many context's functions are not written here, use IDE/Editors with `auto-complete` feature.

```go

// implements the http.ResponseWriter, http.Flusher
// and has some nice features, such as get the status code given
// by previous handler, get the body written so far and much more
ResponseWriter *iris.ResponseWriter {
  http.ResponseWriter // the original response writer
  
  Header() http.Header
  ResetHeaders()

  Write(b []byte) (int, error)
  Body() []byte

  SetBodyString(s string)
  SetBody(b []byte)
  ResetBody()

  SetContentType(cType string)
  ContentType() string

  WriteHeader(statusCode int)
  StatusCode() int

  Hijack() (net.Conn, *bufio.ReadWriter, error)
  SetBeforeFlush(cb func())
  Flush()

  Reset()
}

// the original *http.Request
Request *http.Request

Next()
StopExecution()
IsStopped() bool
GetHandlerName() string

Get(key string) interface{}
GetString(key string) string
GetInt(key string) (int, error)
Set(key string, value interface{})
VisitValues(visitor func([]byte, interface{}))

Param(key string) string
ParamDecoded(key string) string
ParamInt(key string) (int, error)
ParamInt64(key string) (int64, error)

Method() string
Host() string
ServerHost() string
Subdomain() string
VirtualHostname() string
Path() string
RequestPath(escape bool)string
RemoteAddr() string
RequestHeader(k string)string
IsAjax() bool

URLParam(key string)string
URLParams() map[string]string
URLParamsAsMulti() map[string][]string
URLParamInt(key string) (int, error)
URLParamInt64(key string) (int64, error)

FormValues() map[string][]string
FormValue(name string)string
PostValue(name string) string
FormFile(key string) (mime/multipart.File, *mime/multipart.FileHeader, error)

UnmarshalBody(v interface{}, unmarshaler iris.Unmarshaler) error
ReadJSON(v interface{}) error
ReadXML(v interface{}) error
ReadForm(v interface{}) error

ResetBody()
SetContentType(s string)
SetHeader(k string, v string)
SetStatusCode(statusCode int)
Redirect(url string, statusCode int)
RedirectTo(routeName string, args ...interface{})

EmitError(statusCode int)
NotFound()
Panic()

Write(b []byte) (int, error)
Writef(format string, a...interface{}) (int, error)
WriteString(s string) (int, error)
SetBodyString(s string)
WriteGzip(b []byte) (int, error)
TryWriteGzip(b []byte) (int, error)


RenderWithStatus(status int, name string, binding interface{}, options ...map[string]interface{}) (err error)
Render(name string, binding interface{}, options ...map[string]interface{}) error
MustRender(name string, binding interface{}, options ...map[string]interface{})
RenderTemplateSource(status int, src string, binding interface{}, options ...map[string]interface{}) error
TemplateString(name string, binding interface{}, options ...map[string]interface{}) string

HTML(status int, htmlContents string)
JSONP(status int, callback string, v interface{}) error
Text(status int, v string) error
XML(status int, v interface{}) error
MarkdownString(markdownText string) string
Markdown(status int, markdown string)

ServeContent(content io.ReadSeeker, filename string, modtime time.Time, gzipCompression bool) error
ServeFile(filename string, gzipCompression bool) error
SendFile(filename string, destinationName string)


VisitAllCookies(visitor func(key string, value string))
GetCookie(name string) string
SetCookie(cookie *http.Cookie)
SetCookieKV(name, value string)
RemoveCookie(name string)
MaxAge() int64

Session() {
  ID() string
  Get(string) interface{}
  HasFlash() bool
  GetFlash(string) interface{}
  GetString(string) string
  GetFlashString(string) string
  GetInt(string) (int, error)
  GetInt64(string) (int64, error)
  GetFloat32(string) (float32, error)
  GetFloat64(string) (float64, error)
  GetBoolean(string) (bool, error)
  GetAll() map[string]interface{}
  GetFlashes() map[string]interface{}
  VisitAll(func(string,interface{}))
  Set(string, interface{})
  SetFlash(string, interface{})
  Delete(string)
  DeleteFlash(string)
  Clear()
  ClearFlashes()
}
SessionDestroy()

BeginTransaction(pipe func(transaction *Transaction))
SkipTransactions()
TransactionsSkipped() bool

Log(format string, a ...interface{})
Framework() *Framework

// golang/x/net/context
Deadline() (deadline time.Time, ok bool)
Done() <-chan struct{}
Err() error
Value(key interface{}) interface{}

```


The [examples](https://github.com/iris-contrib/examples) will give you the direction.
