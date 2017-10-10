# Quick MVC Tutorial #3

Nothing stops you from using your favorite **folder structure**. Iris is a low level web framework, it has got MVC first-class support but it doesn't limit your folder structure, this is your choice.

Structuring depends on your own needs. We can't tell you how to design your own application for sure but you're free to take a closer look to one typical example below;

[![folder structure example](https://github.com/kataras/iris/raw/master/_examples/mvc/using-method-result/folder_structure.png)](https://github.com/kataras/iris/raw/master/_examples/mvc/using-method-result/folder_structure.png)

Shhh, let's spread the code itself.

```go
// file: controllers/hello_controller.go

package controllers

import (
    "errors"

    "github.com/kataras/iris/mvc"
)

// HelloController is our sample controller
// it handles GET: /hello and GET: /hello/{name}
type HelloController struct {
    mvc.C
}

var helloView = mvc.View{
    Name: "hello/index.html",
    Data: map[string]interface{}{
        "Title":     "Hello Page",
        "MyMessage": "Welcome to my awesome website",
    },
}

// Get will return a predefined view with bind data.
//
// `mvc.Result` is just an interface with a `Dispatch` function.
// `mvc.Response` and `mvc.View` are the built'n result type dispatchers
// you can even create custom response dispatchers by
// implementing the `github.com/kataras/iris/mvc#Result` interface.
func (c *HelloController) Get() mvc.Result {
    return helloView
}

// you can define a standard error in order to be re-usable anywhere in your app.
var errBadName = errors.New("bad name")

// you can just return it as error or even better
// wrap this error with an mvc.Response to make it an mvc.Result compatible type.
var badName = mvc.Response{Err: errBadName, Code: 400}

// GetBy returns a "Hello {name}" response.
// Demos:
// curl -i http://localhost:8080/hello/iris
// curl -i http://localhost:8080/hello/anything
func (c *HelloController) GetBy(name string) mvc.Result {
    if name != "iris" {
        return badName
        // or
        // GetBy(name string) (mvc.Result, error) {
        //  return nil, errBadName
        // }
    }

    // return mvc.Response{Text: "Hello " + name} OR:
    return mvc.View{
        Name: "hello/name.html",
        Data: name,
    }
}
```

```html
<!-- file: views/hello/index.html -->
<html>

<head>
    <title>{{.Title}} - My App</title>
</head>

<body>
    <p>{{.MyMessage}}</p>
</body>

</html>
```

```html
<!-- file: views/hello/name.html -->
<html>

<head>
    <title>{{.}}' Portfolio - My App</title>
</head>

<body>
    <h1>Hello {{.}}</h1>
</body>

</html>
```

> Navigate to the [_examples/view](https://github.com/kataras/iris/tree/master/_examples/#view) for more examples like shared layouts, tmpl funcs, reverse routing and more!

```go
// file: models/movie.go

package models

import "github.com/kataras/iris/context"

// Movie is our sample data structure.
type Movie struct {
    ID     int64  `json:"id"`
    Name   string `json:"name"`
    Year   int    `json:"year"`
    Genre  string `json:"genre"`
    Poster string `json:"poster"`
}

// Dispatch completes the `kataras/iris/mvc#Result` interface.
// Sends a `Movie` as a controlled http response.
// If its ID is zero or less then it returns a 404 not found error
// else it returns its json representation,
// (just like the controller's functions do for custom types by default).
//
// Don't overdo it, the application's logic should not be here.
// It's just one more step of validation before the response,
// simple checks can be added here.
//
// It's just a showcase,
// imagine the potentials this feature gives when designing a bigger application.
//
// This is called where the return value from a controller's method functions
// is type of `Movie`.
// For example the `controllers/movie_controller.go#GetBy`.
func (m Movie) Dispatch(ctx context.Context) {
    if m.ID <= 0 {
        ctx.NotFound()
        return
    }
    ctx.JSON(m, context.JSON{Indent: " "})
}
```

> For those who wonder `iris.Context`(go 1.9 type alias feature) and `context.Context` is the same [exact thing](faq.md#type-aliases).

```go
// file: services/movie_service.go

package services

import (
    "errors"
    "sync"

    "github.com/kataras/iris/_examples/mvc/using-method-result/models"
)

// MovieService handles CRUID operations of a movie entity/model.
// It's here to decouple the data source from the higher level compoments.
// As a result a different service for a specific datasource (or repository)
// can be used from the main application without any additional changes.
type MovieService interface {
    GetSingle(query func(models.Movie) bool) (movie models.Movie, found bool)
    GetByID(id int64) (models.Movie, bool)

    InsertOrUpdate(movie models.Movie) (models.Movie, error)
    DeleteByID(id int64) bool

    GetMany(query func(models.Movie) bool, limit int) (result []models.Movie)
    GetAll() []models.Movie
}

// NewMovieServiceFromMemory returns a new memory-based movie service.
func NewMovieServiceFromMemory(source map[int64]models.Movie) MovieService {
    return &MovieMemoryService{
        source: source,
    }
}

/*
A Movie Service can have different data sources:
func NewMovieServiceFromDB(db datasource.MySQL) {
    return &MovieDatabaseService{
        db: db,
    }
}

Another pattern is to initialize the database connection
or any source here based on a "string" name or an "enum".
func NewMovieService(source string) MovieService {
    if source == "memory" {
        return NewMovieServiceFromMemory(datasource.Movies)
    }
    if source == "database" {
        db = datasource.NewDB("....")
        return NewMovieServiceFromDB(db)
    }
    [...]
    return nil
}
*/

// MovieMemoryService is a "MovieService"
// which manages the movies using the memory data source (map).
type MovieMemoryService struct {
    source map[int64]models.Movie
    mu     sync.RWMutex
}

// GetSingle receives a query function
// which is fired for every single movie model inside
// our imaginary data source.
// When that function returns true then it stops the iteration.
//
// It returns the query's return last known boolean value
// and the last known movie model
// to help callers to reduce the LOC.
//
// It's actually a simple but very clever prototype function
// I'm using everywhere since I firstly think of it,
// hope you'll find it very useful as well.
func (s *MovieMemoryService) GetSingle(query func(models.Movie) bool) (movie models.Movie, found bool) {
    s.mu.RLock()
    for _, movie = range s.source {
        found = query(movie)
        if found {
            break
        }
    }
    s.mu.RUnlock()

    // set an empty models.Movie if not found at all.
    if !found {
        movie = models.Movie{}
    }

    return
}

// GetByID returns a movie based on its id.
// Returns true if found, otherwise false, the bool should be always checked
// because the models.Movie may be filled with the latest element
// but not the correct one, although it can be used for debugging.
func (s *MovieMemoryService) GetByID(id int64) (models.Movie, bool) {
    return s.GetSingle(func(m models.Movie) bool {
        return m.ID == id
    })
}

// InsertOrUpdate adds or updates a movie to the (memory) storage.
//
// Returns the new movie and an error if any.
func (s *MovieMemoryService) InsertOrUpdate(movie models.Movie) (models.Movie, error) {
    id := movie.ID

    if id == 0 { // Create new action
        var lastID int64
        // find the biggest ID in order to not have duplications
        // in productions apps you can use a third-party
        // library to generate a UUID as string.
        s.mu.RLock()
        for _, item := range s.source {
            if item.ID > lastID {
                lastID = item.ID
            }
        }
        s.mu.RUnlock()

        id = lastID + 1
        movie.ID = id

        // map-specific thing
        s.mu.Lock()
        s.source[id] = movie
        s.mu.Unlock()

        return movie, nil
    }

    // Update action based on the movie.ID,
    // here we will allow updating the poster and genre if not empty.
    // Alternatively we could do pure replace instead:
    // s.source[id] = movie
    // and comment the code below;
    current, exists := s.GetByID(id)
    if !exists { // ID is not a real one, return an error.
        return models.Movie{}, errors.New("failed to update a nonexistent movie")
    }

    // or comment these and s.source[id] = m for pure replace
    if movie.Poster != "" {
        current.Poster = movie.Poster
    }

    if movie.Genre != "" {
        current.Genre = movie.Genre
    }

    // map-specific thing
    s.mu.Lock()
    s.source[id] = current
    s.mu.Unlock()

    return movie, nil
}

// DeleteByID deletes a movie by its id.
//
// Returns true if deleted otherwise false.
func (s *MovieMemoryService) DeleteByID(id int64) bool {
    if _, exists := s.GetByID(id); !exists {
        // we could do _, exists := s.source[id] instead
        // but we don't because you should learn
        // how you can use that service's functions
        // with any other source, i.e database.
        return false
    }

    // map-specific thing
    s.mu.Lock()
    delete(s.source, id)
    s.mu.Unlock()

    return true
}

// GetMany same as GetSingle but returns one or more models.Movie as a slice.
// If limit <=0 then it returns everything.
func (s *MovieMemoryService) GetMany(query func(models.Movie) bool, limit int) (result []models.Movie) {
    loops := 0

    s.mu.RLock()
    for _, movie := range s.source {
        loops++

        passed := query(movie)
        if passed {
            result = append(result, movie)
        }
        // we have to return at least one movie if "passed" was true.
        if limit >= loops {
            break
        }
    }
    s.mu.RUnlock()

    return
}

// GetAll returns all movies.
func (s *MovieMemoryService) GetAll() []models.Movie {
    movies := s.GetMany(func(m models.Movie) bool { return true }, -1)
    return movies
}
```

```go
// file: controllers/movie_controller.go

package controllers

import (
    "errors"

    "github.com/kataras/iris/_examples/mvc/using-method-result/models"
    "github.com/kataras/iris/_examples/mvc/using-method-result/services"

    "github.com/kataras/iris"
    "github.com/kataras/iris/mvc"
)

// MovieController is our /movies controller.
type MovieController struct {
    // mvc.C is just a lightweight lightweight alternative
    // to the "mvc.Controller" controller type,
    // use it when you don't need mvc.Controller's fields
    // (you don't need those fields when you return values from the method functions).
    mvc.C

    // Our MovieService, it's an interface which
    // is binded from the main application.
    Service services.MovieService
}

// Get returns list of the movies.
// Demo:
// curl -i http://localhost:8080/movies
func (c *MovieController) Get() []models.Movie {
    return c.Service.GetAll()
}

// GetBy returns a movie.
// Demo:
// curl -i http://localhost:8080/movies/1
func (c *MovieController) GetBy(id int64) models.Movie {
    m, _ := c.Service.GetByID(id)
    return m
}

// PutBy updates a movie.
// Demo:
// curl -i -X PUT -F "genre=Thriller" -F "poster=@/Users/kataras/Downloads/out.gif" http://localhost:8080/movies/1
func (c *MovieController) PutBy(id int64) (models.Movie, error) {
    // get the request data for poster and genre
    file, info, err := c.Ctx.FormFile("poster")
    if err != nil {
        return models.Movie{}, errors.New("failed due form file 'poster' missing")
    }
    // we don't need the file so close it now.
    file.Close()

    // imagine that is the url of the uploaded file...
    poster := info.Filename
    genre := c.Ctx.FormValue("genre")

    // update the movie and return it.
    return c.Service.InsertOrUpdate(models.Movie{
        ID:     id,
        Poster: poster,
        Genre:  genre,
    })
}

// DeleteBy deletes a movie.
// Demo:
// curl -i -X DELETE -u admin:password http://localhost:8080/movies/1
func (c *MovieController) DeleteBy(id int64) interface{} {
    wasDel := c.Service.DeleteByID(id)
    if wasDel {
        // and return the deleted movie's ID
        return iris.Map{"deleted": id}
    }
    // here we can see that a method function can return any of those two types(map or int),
    // we don't have to specify the return type to a specific type.
    return iris.StatusBadRequest
}
```

```go
// file: datasource/movies.go

package datasource

import "github.com/kataras/iris/_examples/mvc/using-method-result/models"

// Movies is our imaginary data source.
var Movies = map[int64]models.Movie{
    1: {
        ID:     1,
        Name:   "Casablanca",
        Year:   1942,
        Genre:  "Romance",
        Poster: "https://iris-go.com/images/examples/mvc-movies/1.jpg",
    },
    2: {
        ID:     2,
        Name:   "Gone with the Wind",
        Year:   1939,
        Genre:  "Romance",
        Poster: "https://iris-go.com/images/examples/mvc-movies/2.jpg",
    },
    3: {
        ID:     3,
        Name:   "Citizen Kane",
        Year:   1941,
        Genre:  "Mystery",
        Poster: "https://iris-go.com/images/examples/mvc-movies/3.jpg",
    },
    4: {
        ID:     4,
        Name:   "The Wizard of Oz",
        Year:   1939,
        Genre:  "Fantasy",
        Poster: "https://iris-go.com/images/examples/mvc-movies/4.jpg",
    },
    5: {
        ID:     5,
        Name:   "North by Northwest",
        Year:   1959,
        Genre:  "Thriller",
        Poster: "https://iris-go.com/images/examples/mvc-movies/5.jpg",
    },
}
```

```go
// file: middleware/basicauth.go

package middleware

import "github.com/kataras/iris/middleware/basicauth"

// BasicAuth middleware sample.
var BasicAuth = basicauth.New(basicauth.Config{
    Users: map[string]string{
        "admin": "password",
    },
})
```

```go
// file: main.go

package main

import (
    "github.com/kataras/iris/_examples/mvc/using-method-result/controllers"
    "github.com/kataras/iris/_examples/mvc/using-method-result/datasource"
    "github.com/kataras/iris/_examples/mvc/using-method-result/middleware"
    "github.com/kataras/iris/_examples/mvc/using-method-result/services"

    "github.com/kataras/iris"
)

func main() {
    app := iris.New()

    // Load the template files.
    app.RegisterView(iris.HTML("./views", ".html"))

    // Register our controllers.
    app.Controller("/hello", new(controllers.HelloController))

    // Create our movie service (memory), we will bind it to the movies controller.
    service := services.NewMovieServiceFromMemory(datasource.Movies)

    app.Controller("/movies", new(controllers.MovieController),
        // Bind the "service" to the MovieController's Service (interface) field.
        service,
        // Add the basic authentication(admin:password) middleware
        // for the /movies based requests.
        middleware.BasicAuth)

    // Start the web server at localhost:8080
    // http://localhost:8080/hello
    // http://localhost:8080/hello/iris
    // http://localhost:8080/movies/1
    app.Run(
        iris.Addr("localhost:8080"),
        iris.WithoutVersionChecker,
        iris.WithoutServerError(iris.ErrServerClosed),
        iris.WithOptimizations, // enables faster json serialization and more
    )
}
```

----

More folder structure guidelines can be found at the [https://github.com/kataras/iris/tree/master/_examples/#structuring](https://github.com/kataras/iris/tree/master/_examples/#structuring) section.

Follow the examples below

- [Session Controller](https://github.com/kataras/iris/blob/master/_examples/mvc/session-controller/main.go)
- [A simple but featured Controller with model and views](https://github.com/kataras/iris/tree/master/_examples/mvc/controller-with-model-and-view).
- [Login showcase](https://github.com/kataras/iris/tree/master/mvc/login) **NEW**