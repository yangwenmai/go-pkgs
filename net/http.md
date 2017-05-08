# http

## 辅助方法

### CanonicalHeaderKey

与`textproto.CanonicalMIMEHeaderKey`效果相同。

## Handler 和 HandlerFunc

`Handler`是一个`interface`：

```go
type Handler interface {
    ServeHTTP(ResponseWriter, *Request)
}
```

`HandlerFunc`是一类函数，它的参数列表与`Handler`的`ServeHTTP`方法的参数列表相同：

```go
type HandlerFunc func(ResponseWriter, *Request)
```

并且它实现了`ServeHTTP`方法：

```go
func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
    f(w, r)
}
```

因此，`HandlerFunc`的主要作用就是将一个普通函数包装成一个`Handler`：

```go
var f = func(w http.ResponseWriter, r *http.Request) {}
var g interface{} = http.HandlerFunc(f)
_, ok := g.(http.Handler)
fmt.Println(ok) // true
```

## Cookie

```go
// This implementation is done according to RFC 6265:
//
//    http://tools.ietf.org/html/rfc6265

// A Cookie represents an HTTP cookie as sent in the Set-Cookie header of an
// HTTP response or the Cookie header of an HTTP request.
type Cookie struct {
    Name       string
    Value      string
    Path       string
    Domain     string
    Expires    time.Time
    RawExpires string

    // MaxAge=0 means no 'Max-Age' attribute specified.
    // MaxAge<0 means delete cookie now, equivalently 'Max-Age: 0'
    // MaxAge>0 means Max-Age attribute present and given in seconds
    MaxAge   int
    Secure   bool
    HttpOnly bool
    Raw      string
    Unparsed []string // Raw text of unparsed attribute-value pairs
}
```

例如：

```go
c := http.Cookie{
    Name:   "token",
    Value:  "abcd1234",
    Path:   "/",
    MaxAge: 3600,
}
fmt.Println(c.String()) // "token=abcd1234; Path=/; Max-Age=3600"
```

## CookieJar

`CookieJar`是一个接口：

```go
// A CookieJar manages storage and use of cookies in HTTP requests.
//
// Implementations of CookieJar must be safe for concurrent use by multiple
// goroutines.
//
// The net/http/cookiejar package provides a CookieJar implementation.
type CookieJar interface {
    // SetCookies handles the receipt of the cookies in a reply for the
    // given URL.  It may or may not choose to save the cookies, depending
    // on the jar's policy and implementation.
    SetCookies(u *url.URL, cookies []*Cookie)

    // Cookies returns the cookies to send in a request for the given URL.
    // It is up to the implementation to honor the standard cookie use
    // restrictions such as in RFC 6265.
    Cookies(u *url.URL) []*Cookie
}
```

`net/http/cookiejar`提供了一个具体的实现。

## Header

与`textproto.MIMEHeader`类似，本质上也是一个`map[string][]string`。同样拥有`Add`，`Set`，`Get`，`Del`方法，此外还有`Write`和`WriteSubset`方法。

```go
h := http.Header{}
h.Set("Content-Type", "application/json")
h.Add("Accept-Encoding", "gzip")
h.Add("Accept-Encoding", "deflate")
h.Set("Cache-Control", "no-cache")

h.Write(os.Stdout)
// Accept-Encoding: gzip
// Accept-Encoding: deflate
// Cache-Control: no-cache
// Content-Type: application/json

h.WriteSubset(os.Stdout, map[string]bool{
    "Accept-Encoding": true,
})
// Cache-Control: no-cache
// Content-Type: application/json
```