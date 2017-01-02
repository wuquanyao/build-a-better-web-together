# Flash messages

**A flash message is used in order to keep a message in session through one or several requests of the same user**.   
By default, it is removed from the session after it has been displayed to the user.   
Flash messages are usually used in combination with HTTP redirections, because in this case there is no view, so messages can only be displayed in the request that follows redirection.

**A flash message has a name and a content (AKA key and value). It is an entry of a map**.   
The name is a string: often "notice", "success", or "error", but it can be anything. The content is usually a string. You can put HTML tags in your message if you display it raw. You can also set the message value to a number or an array: it will be serialized and kept in session like a string.

----


```go
HasFlash() bool

GetFlash(string) interface{}

GetFlashString(string) string

GetFlashes() map[string]interface{}

SetFlash(string, interface{})

DeleteFlash(string)

ClearFlashes()

```

Example

```go

package main

import (
	"github.com/kataras/iris"
)

func main() {

	iris.Get("/set", func(c *iris.Context) {
		c.Session().SetFlash("name", "iris")
		c.Session().Writef("Message set, is available for the next request")
	})

	iris.Get("/get", func(c *iris.Context) {
		name := c.Session().GetFlashString("name")
		if name =="" {
			c.Writef(err.Error())
			return
		}
		c.Writef("Hello %s", name)
	})

	iris.Get("/test", func(c *iris.Context) {

		name := c.Session().GetFlashString("name")
		if name =="" {
			c.Writef("name not found")
			return
		}

		c.Writef("Ok you are comming from /set, the value of the name is %s", name)
		c.Writef(", and again from the same context: %s", name)

	})

	iris.Listen(":8080")
}


```
