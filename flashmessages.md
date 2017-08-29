# Flash messages

**A flash message is used in order to keep a message in session through one or several requests of the same user**.   
By default, it is removed from the session after it has been displayed to the user.   
Flash messages are usually used in combination with HTTP redirections, because in this case there is no view, so messages can only be displayed in the request that follows redirection.

**A flash message has a name and a content (AKA key and value). It is an entry of a map**.   
The name is a string: often "notice", "success", or "error", but it can be anything. The content is usually a string. You can put HTML tags in your message if you display it raw. You can also set the message value to a number or an array: it will be serialized and kept in session like a string.

----


```go
// SetFlash sets a flash message by its key.
//
// A flash message is used in order to keep a message in session through one or several requests of the same user.
// It is removed from session after it has been displayed to the user.
// Flash messages are usually used in combination with HTTP redirections,
// because in this case there is no view, so messages can only be displayed in the request that follows redirection.
//
// A flash message has a name and a content (AKA key and value).
// It is an entry of an associative array. The name is a string: often "notice", "success", or "error", but it can be anything.
// The content is usually a string. You can put HTML tags in your message if you display it raw.
// You can also set the message value to a number or an array: it will be serialized and kept in session like a string.
//
// Flash messages can be set using the SetFlash() Method
// For example, if you would like to inform the user that his changes were successfully saved,
// you could add the following line to your Handler:
//
// SetFlash("success", "Data saved!");
//
// In this example we used the key 'success'.
// If you want to define more than one flash messages, you will have to use different keys.
SetFlash(key string, value interface{})

// GetFlash returns a stored flash message based on its "key"
// which will be removed on the next request.
//
// To check for flash messages we use the HasFlash() Method
// and to obtain the flash message we use the GetFlash() Method.
// There is also a method GetFlashes() to fetch all the messages.
//
// Fetching a message deletes it from the session.
// This means that a message is meant to be displayed only on the first page served to the user.
GetFlash(key string) interface{}

// HasFlash returns true if this session has available flash messages.
HasFlash() bool

// PeekFlash returns a stored flash message based on its "key".
// Unlike GetFlash, this will keep the message valid for the next requests,
// until GetFlashes or GetFlash("key").
PeekFlash(key string) interface{}

// GetFlashString same as GetFlash but returns as string, if nil then returns an empty string.
GetFlashString(key string) string

// GetFlashes returns all flash messages as map[string](key) and interface{} value
// NOTE: this will cause at remove all current flash messages on the next request of the same user.
GetFlashes() map[string]interface{}


// DeleteFlash removes a flash message by its key.
DeleteFlash(key string)

// ClearFlashes removes all flash messages.
ClearFlashes()
```

Example Code:

```go
package main

import (
	"github.com/kataras/iris"
	"github.com/kataras/iris/sessions"
)

func main() {
	app := iris.New()
	sess := sessions.New(sessions.Config{Cookie: "myappsessionid"})

	app.Get("/set", func(ctx iris.Context) {
		s := sess.Start(ctx)
		s.SetFlash("name", "iris")
		ctx.Writef("Message setted, is available for the next request")
	})

	app.Get("/get", func(ctx iris.Context) {
		s := sess.Start(ctx)
		name := s.GetFlashString("name")
		if name == "" {
			ctx.Writef("Empty name!!")
			return
		}
		ctx.Writef("Hello %s", name)
	})

	app.Get("/test", func(ctx iris.Context) {
		s := sess.Start(ctx)
		name := s.GetFlashString("name")
		if name == "" {
			ctx.Writef("Empty name!!")
			return
		}

		ctx.Writef("Ok you are coming from /set ,the value of the name is %s", name)
		ctx.Writef(", and again from the same context: %s", name)
	})

	app.Run(iris.Addr(":8080"))
}
```