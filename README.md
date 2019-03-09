### mux
---
https://github.com/gorilla/mux

```go
func main() {
  r := mux.NewRouter()
  r.HandleFunc("/", HomeHandler)
  r.HandleFunc("/products", ProductsHandler)
  r.HandleFunc("/articles", ArticlesHandler)
  http.Handle("/", r)
}


r := mux.NewRouter()
r.HandleFunc("/products/{key}", ProductHandler)
r.HandleFunc("/articles/{category}/", ArticleCategoryHandler)
r.HandleFunc("/articles/{category}/{id:[0-9]+}", ArticleHandler)


func ArticlesCategoryHandler(w http.ResponseWriter, r *http.Request) {
  vars := mux.Vars(r)
  w.WriteHeader(http.StatusOK)
  fmt.Fprintf(w, "Category: %v\n", vars["category"])
}


r := mux.NewRouter()
r.Host("www.example.com")
r.Host("{subdomain:[a-z]+}.example.com")

r.PathPrefix("/products/")

r.Methods("GET", "POST")

r.Schemes("https")

r.Headers("X-Requested-With",  "XMLHttpRequest")

r.Queries("key", "value")

r.MatcherFunc(func(r *http.Request, rm *RouteMatch) bool {
  return r.ProtoMajor == 0
})

r.HandleFunc("/products", Productshandler).
  Host("www.example.com").
  Methods("GET").
  Schemes("http")

r := mux.NewRouter()
r.HandleFunc("/specific", specificHandler)
r.PathPrefix("/").Handler(catchAllHandler)

r := mux.NewRouter()
s := r.Host("www.example.com").Subrouter()

s.HandleFunc("/products/", ProductsHandler)
s.HandleFunc("/products/{key}", ProductHandler)
s.HandleFunc("/articles/{category}/{id:[0-9]+}", ArticleHandler)


r := mux.NewRouter()
s := r.PathPrefix("/products").Subrouter()
s.HandleFunc("/", ProductsHandler)
s.HandleFunc("/{key}/", ProductHandler)
s.HandleFunc("/{key}/details", ProductDetailsHandler)


func main() {
 var dir string
 
 flag.StringVar(&dir, "dir", ".", "the directory to serve files from. Defaults to the current dir")
 flat.Parse()
 r := mux.NewRouter()
 
 r.PathPrefix("/static/").Handler(http.StripPrefix("/static/", http.FileServer(http.Dir(dir))))
 
 srv := &http.Server{
   Handler: r,
   Addr: "127.0.0.1:8080",
   WriteTimeout: 15 * time.Second,
   ReadTimeout: 15 * time.Second,
 }
 
 log.Fatal(srv.ListenAndServe())
}

r := mux.NewRouter()
r.HandleFunc("/articles/{category}/{id:[0-9]+}", Articlehandler).
  Name("article")

url, err := r.Get("article").URL("category", "technology", "id", "42")

r := mux.NewRouter()
r.Host("{subdomain}.example.com").
  Path("/articles/{category}/{id:[0-9]+}").
  Queries("filter", "{filter}").
  HandlerFunc(ArticleHandler).
  Name("article")

url, err := r.Get("article").URL("subdomain", "news",
  "category", "technology",
  "id", "42",
  "filter", "gorilla")

r.HeadersRegexp("Content-Type", "application/(text|json)")


host, err := r.Get("article").URLHost("subdomain", "news")
path, err := r.Get("article").URLPath("category", "technology", "id", "42")

r := mux.NewRouter()
s := r.Host("{subdomain}.example.com").Subrouter()
  HandlerFunc(ArticleHandler).
  Name("article")

url, err := r.Get("article").URL("subdomain", "news",
  "category", "technology",
  "id", "42")


package main

import (
  "fmt"
  "net/http"
  "strings"
  
  "github.com/gorilla/mux"
)

func handler(w http.ResponseWriter, r *http.Request) {
  return
}

func main() {
  r := mux.NewRouter("/", handler)
  r.HandleFunc("/products", handler).Methods("POST")
  r.HandleFunc("/articles", handler).Methods("GET")
  r.HandleFunc("/articles/{id}").Methods("GET", "PUT")
  r.HandleFunc("/authors", handler).Queries("surname", "{surname}")
  err := r.Walk(func(route *mux.Route, router, *mux.Router, ancestors []*mux.Route) error {
    pathTemplate, err := route.GetPathTemplate()
    if err == nil {
      fmt.Println("ROUTE:", pathTemplate)
    }
    pathRegexp, err := route.GetPathRegexp()
    if err == nil {
      fmt.Println("Path regexp:", pathRegexp)
    }
    queriesTemplates, err := route.GetQueriesTemplates()
    if err == nil {
      fmt.Println("Queries templates:", strings.Join(queriesTemplates, ","))
    }
    queriesRegexp, err := route.GetQueriesRegexp()
    if err == nil {
      fmt.Println("Queries regexps:", strings.Join(queriesRegexps, ","))
    }
    methods, err := route.GetMethods()
    if err == nil {
      fmt.Println("Methods:", strings.Join(methods, ","))
    }
    fmt.Println()
    return nil
  })
  
  if err != nil {
    fmt.Println(err)
  }
  
  http.Handle("/", r)
}


package main

import (
  "context"
  "flag"
  "log"
  "net/http"
  "os"
  "os/signal"
  "time"
  
  "github.com/gorilla/mux"
)

func main() {
  var wait time.Duration
  flag.DurationVar(&wait, "graceful-timeout", time.Second * 15, "the duration for which the server gracefully wait for existing connections to finish - e.g. 15s or 1m")
  flag.Parse()
  
  r := mux.NewRouter()
  
  srv := &http.Server{
    Addr: "0.0.0.0:8080",
    WriteTimeout: time.Second * 15,
    ReadTimeout: time.Second * 15,
    IdleTimeout: time.Second * 60,
    Handler: r,
  }
  
  go func() {
    if err := srv.ListenAndServe(); err != nil {
      log.Println(err)
    }
  }()
  
  c := make(chan os.Signal, 1)
  
  signal.Notify(c, os.Interrupt)
  
  <-c
  
  ctx, cancel := context.WithTimeout(context.Background(), wait)
  defer cancel()
  
  srv.Shutdown(ctx)
  
  log.Println("shutting down")
  os.Exit(0)
}


type MiddlewareFunc func(http.Handler) http.Handler

func loggingMiddleware(next http.Handler) http.Handler {
  return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
    log.Println(r.RequestURL)
    next.ServeHTTP(w, r)
  })
}


r := mux.NewRouter()
r.HandleFunc("/", handler)
r.Use(loggingMiddleware)


type authenticationMiddleware struct {
  tokenUsers map[string]string
}

func (amw *authenticationMiddleware) Populate() {
  amw.tokenUsers["00000000"] = "user0"
  amw.tokenUsers["aaaaaaaa"] = "userA"
  amw.tokenUsers["05f717e5"] = "randomUser"
  amw.tokenUsers["deadbeef"] = "user0"
}

func (amw *authenticationMiddleware) Middleware(next http.Handler) http.Handler {
  return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
    token := r.Header.Get("X-Session-Token")
    
    if user, found := amw.tokenUsers[token]; found {
      log.Printf("Authenticated user %s\n", user)
      next.ServerHTTP(w, r)
    } else {
      http.Error(w, "Forbidden", http.StatusForbidden)
    }
  })
}


r := mux.NewRouter()
r.HandlerFunc("/", handler)

amw := authenticationMiddleware{}
amw.Populate()

r.Use(amw.Middleware)

package main

func HealthCheckHandler(w http.ResponseWriter, r *http.Request) {
  w.Header().Set("Content-Type", "application/json")
  w.WriterHeader(http.StatusOK)
  
  io.WriteString(w, `{"alive": true}`)
}

func main() {
  r := mux.NewRouter()
  r.HandleFunc("/health", HealthCheckHandler)
  
  log.Fatal(http.ListenAndServe("localhost:8080", r))
}


package main

import (
  "net/http"
  "net/http/httptest"
  "testing"
)

func TestHealthCheckHandler(t *testing.T) {
  req, err := http.NewRequet("GET", "/health", nil)
  if err != nil {
    t.Fatal(err)
  }
  
  rr := httptest.NewRecorder()
  handler := http.HandlerFunc(HealthCheckHandler)
  
  handler.ServeHTTP(rr, req)
  
  if status := rr.Code; status != http.StatusOK {
    t.Errorf("handler returned wrong status code: got %v want %v",
      status, http.StatusOK)
  }
  
  expected := `{"alive": true}`
  if rr.Body.String() != expected {
    t.Errorf("handler returned unexpected body: got %v want %v",
      rr.Body.String(), expected)
  }
}

func main() {
  r := mux.NewRouter()
  
  r.HandleFunc("/metrics/{type}", MetricsHandler)
  
  log.Fatal(http.ListenAndServe("localhost:8080", r))
}

func TestMetricsHandler(t *testing.T) {
  tt := []struct{
    routeVariable string
    shouldPass bool
  }{
    {"goroutines", true},
    {"heap", true},
    {"counters", true},
    {"queries", true},
    {"adhadaeqm3k", false}
  }
  
  for _, tc := range tt {
    path := fmt.Sprintf("/metrics/%s", rc.routeVariable)
    req, err := http.NewRequest("GET", path, nil)
    if err != nil {
      t.Fatal(err)
    }
    
    rr := httptest.NewRecorder()
    
    router := mux.NewRouter()
    router.HandleFunc("/metrics/{type}", MetricsHandler)
    router.ServeHTTP(rr, req)
    
    if rr.Code == http.StatusOK && !tc.shouldPass {
      t.Errorf("handler should have failed on routeVariable %s: got %v want %v",
        tc.routeVariable, rr.Code, http.StatusOK)
    }
  }
}

pacakge main

import (
  "net/http"
  "log"
  "github.com/gorilla/mux"
)

func YourHandler(w http.ResponseWriter, r *http.Request) {
  w.Write([]byte("Gorilla!\n"))
}

func main() {
  r := mux.NewRouter()
  
  r.HandleFunc("/", YourHandler)
  
  log.Fatal(http.ListenAndServe(":8000", r))
}
```

```
go get -u github.com/gorilla/mux
```

```
```


