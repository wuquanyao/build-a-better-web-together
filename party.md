# Party

Let's party with Iris web framework!

```go
package main

import "github.com/kataras/iris"

func main() {
    admin := iris.Party("/admin",
	    func(ctx *iris.Context){
     	   ctx.Writef("Middleware for all party's routes!")
		   ctx.Next();
		})
    {
        // add a silly middleware
        admin.UseFunc(func(ctx *iris.Context) {
            //your authentication logic here...
            println("from ", ctx.Path())
            authorized := true
            if authorized {
                ctx.Next()
            } else {
                ctx.Text(401, ctx.Path()+" is not authorized for you")
            }

        })
        admin.Get("/", func(ctx *iris.Context) {
            ctx.Writef("from /admin/ or /admin if you pathcorrection on")
        })
        admin.Get("/dashboard", func(ctx *iris.Context) {
            ctx.Writef("/admin/dashboard")
        })
        admin.Delete("/delete/:userId", func(ctx *iris.Context) {
            ctx.Writef("admin/delete/%s", ctx.Param("userId"))
        })
    }


    beta := admin.Party("/beta")
    beta.Get("/hey", func(ctx *iris.Context) { ctx.Writef("hey from /admin/beta/hey") })

    iris.Listen(":8080")
}
```
