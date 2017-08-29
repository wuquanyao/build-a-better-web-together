# 🔥 Hot Features

- Focus on high performance
- Easy Fluent API
- Highly customizable
- Robust routing and middleware ecosystem
    * Build RESTful APIs with iris unique expressionist path interpreter
	* Dynamic path parameterized or wildcard routes are not conflict with static routes 
	* Remove trailing slash from the URL with option to redirect
	* Virtual hosts and subdomains made easy
	* Group API's and static or even dynamic subdomains
	* `net/http` and `negroni-like` handlers are compatible via `iris.FromStd` 
	* Register custom handlers for any HTTP error
	* Transactions and rollback when you need it
	* Cache the response when you need it
	* A single function to serve your embedded assets, always compatible with `go-bindata`
	* HTTP to HTTPS
 	* HTTP to HTTPS WWW
	* [learn the reasons that differ from what you've seen so far](https://github.com/kataras/iris/tree/master/_examples/#routing-grouping-dynamic-path-parameters-macros-and-custom-context)
	* MVC **NEW**
- Context
	* Highly scalable rich content render (Markdown, JSON, JSONP, XML...)
	* Body binders and handy functions to send HTTP responses
	* Limit request body
	* Serve static resources or embedded assets
	* Localization i18N
	* Compression (Gzip is built'n)
- Authentication
	* Basic Authentication
	* OAuth, OAuth2 supporting 27+ popular websites
	* JWT
- Server
	* Automatically install and serve certificates from https://letsencrypt.org when serving via TLS
	* Gracefully shutdown by-default
	* Register on shutdown, error or interrupt events
	* Attach more than one server, fully compatible with `net/http#Server`
- View system: supporting 5 template engines. Fully compatible with `html/template`
- HTTP Sessions library [you can still use your favorite if you want to]
- Websocket library, its API similar to socket.io [you can still use your favorite if you want to]
- Hot Reload on source code changes[*](https://github.com/kataras/rizla)
- Typescript integration + Web IDE

Iris is one of the most featured web frameworks out there, not all features are here and don't expect from me to write down all of their usages in this gitbook, if you see that I'm missing something please make a PR to the [book repository](https://github.com/kataras/build-a-better-web-together).