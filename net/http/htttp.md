#http包

import "net/http"

----------

##简介

##概览
http包提供了HTTP的客户端和服务端实现。
Get, Head, Post, and PostForm构造了HTTP或者HTTPS的请求。

```go
resp, err := http.Get("http://example.com/")
...
resp, err := http.Post("http://example.com/upload", "image/jpeg", &buf)
...
resp, err := http.PostForm("http://example.com/form",
	url.Values{"key": {"Value"}, "id": {"123"}})
```

当完成响应body后，客户端必须关闭它。
```go
resp, err := http.Get("http://example.com/")
if err != nil {
	// handle error
}
defer resp.Body.Close()
body, err := ioutil.ReadAll(resp.Body)
// ...
```

为了控制HTTP客户端头、重定向策略和其他设置，创建一个Client：
```go
client := &http.Client{
	CheckRedirect: redirectPolicyFunc,
}

resp, err := client.Get("http://example.com")
// ...

req, err := http.NewRequest("GET", "http://example.com", nil)
// ...
req.Header.Add("If-None-Match", `W/"wyzzy"`)
resp, err := client.Do(req)
// ...
```
为了控制代理、TLS配置、keep-alives、压缩和其他设置，创建一个Transport：
```go
tr := &http.Transport{
	TLSClientConfig:    &tls.Config{RootCAs: pool},
	DisableCompression: true,
}
client := &http.Client{Transport: tr}
resp, err := client.Get("https://example.com")
```
对于多个goroutines的并发，Clients和Transport是安全的。为了获得高效率，应该被创建一次然后重用。

ListenAndServe用给定的地址和handler开启一个HTTP服务器。handler经常是空的，那意味着使用了DefaultServeMux。Handle和HandleFunc将handlers加入了DefaultServeMux：
```go
http.Handle("/foo", fooHandler)
http.HandleFunc("/bar", func(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello, %q", html.EscapeString(r.URL.Path))
})
log.Fatal(http.ListenAndServe(":8080", nil))
```
创建一个自定义的Server可以更好的控制服务器的行为：
```go
s := &http.Server{
	Addr:           ":8080",
	Handler:        myHandler,
	ReadTimeout:    10 * time.Second,
	WriteTimeout:   10 * time.Second,
	MaxHeaderBytes: 1 << 20,
}
log.Fatal(s.ListenAndServe())
```

##变量

```go
var (
    ErrHeaderTooLong        = &ProtocolError{"header too long"}
    ErrShortBody            = &ProtocolError{"entity body too short"}
    ErrNotSupported         = &ProtocolError{"feature not supported"}
    ErrUnexpectedTrailer    = &ProtocolError{"trailer header without chunked transfer encoding"}
    ErrMissingContentLength = &ProtocolError{"missing ContentLength in HEAD response"}
    ErrNotMultipart         = &ProtocolError{"request Content-Type isn't multipart/form-data"}
    ErrMissingBoundary      = &ProtocolError{"no multipart boundary param in Content-Type"}
)
```

```go
var (
    ErrWriteAfterFlush = errors.New("Conn.Write called after Flush")
    ErrBodyNotAllowed  = errors.New("http: request method or response status code does not allow body")
    ErrHijacked        = errors.New("Conn has been hijacked")
    ErrContentLength   = errors.New("Conn.Write wrote more than the declared Content-Length")
)
```
以上是由HTTP server 引发的错误。

```go
var DefaultClient = &Client{}
```
DefaultClient是默认的Client，可以发起Get、Head和Post请求。

```go
var DefaultServeMux = NewServeMux()
```
DefaultServeMux是Server所用的默认的ServerMux。

```go
var ErrBodyReadAfterClose = errors.New("http: invalid Read on closed Body")
```
body被关闭之后读取Request和Response，返回ErrBodyReadAfterClose 。HTTP Handler在它的ResponseWriter上调用WriterHeader或者Write 方法之后，如果此时读取body，将会引起这种错误，这是一种很典型的情形。

```go
var ErrHandlerTimeout = errors.New("http: Handler timeout")
```
当ResponseWriter的Write在已经超时的handlers中调用，返回ErrHandlerTimeout。

```go
var ErrLineTooLong = errors.New("header line too long")
```
```go
var ErrMissingFile = errors.New("http: no such file")
```
当提供的文件的field name不知request中或者不是文件field，FromFile会返回ErrMissingFile。

```go
var ErrNoCookie = errors.New("http: named cookie not present")
```
```go
var ErrNoLocation = errors.New("http: no Location header in response")
```

###func CanonicalHeaderKey
```go
func CanonicalHeaderKey(s string) string
```
返回header key `s`的正规格式。它将首字母和hyphen后面的任意字母转为大写形式，其余的转为小写。比如`"accept-encoding"`的canonical key是` "Accept-Encoding"`。

###func DetectContentType
```go
func DetectContentType(data []byte) string
```
实现了http://mimesniff.spec.whatwg.org描述的算法，用来确定给定数据的内容类型。它考虑了最多512字节的数据。DetectContentType常常返回一个有效的MIME类型：如果它无法确定一个具体的类型，它返回` "application/octet-stream"`。

###func Error
```go
func Error(w ResponseWriter, error string, code int)
```
Error向请求回答带有具体错误信息的request和HTTP code。错误信息应该是纯文本格式。

###func Handle
```go
func Handle(pattern string, handler Handler)
```
Handle在DefaultServeMux注册了给定形式（pattern）的handler。ServeMux 的相关文档解释了形式（patterns ）如何被解析。

###func HandleFunc
```go
func HandleFunc(pattern string, handler func(ResponseWriter, *Request))
```
HandleFunc在DefaultServeMux注册了给定形式（pattern）的handler 函数。ServeMux 的相关文档解释了形式（patterns ）如何被解析。

###func ListenAndServe
```go
func ListenAndServe(addr string, handler Handler) error
```
ListenAndServe在TCP网络地址上监听，然后带着handler来处理来到的链接并调用Serve。Handler典型值为nil，在这种情况下使用DefaultServerMux。
以下是一个小小的server：
```go
package main

import (
	"io"
	"net/http"
	"log"
)

// hello world, the web server
func HelloServer(w http.ResponseWriter, req *http.Request) {
	io.WriteString(w, "hello, world!\n")
}

func main() {
	http.HandleFunc("/hello", HelloServer)
	err := http.ListenAndServe(":12345", nil)
	if err != nil {
		log.Fatal("ListenAndServe: ", err)
	}
}
```

###func ListenAndServeTLS
```go
func ListenAndServeTLS(addr string, certFile string, keyFile string, handler Handler) error
```
ListenAndServeTLS的行为与ListenAndServe完全相同，只不过它等待的是HTTPS连接。而且，带有证书和匹配的private key的文件必须提供给server。如果证书被证书颁发机构签署，在server的证书后面是CA的证书，然后应是`certFile`。
```go
import (
	"log"
	"net/http"
)

func handler(w http.ResponseWriter, req *http.Request) {
	w.Header().Set("Content-Type", "text/plain")
	w.Write([]byte("This is an example server.\n"))
}

func main() {
	http.HandleFunc("/", handler)
	log.Printf("About to listen on 10443. Go to https://127.0.0.1:10443/")
	err := http.ListenAndServeTLS(":10443", "cert.pem", "key.pem", nil)
	if err != nil {
		log.Fatal(err)
	}
}
```
可以使用 crypto/tls包中的generate_cert.go来生成cert.pem 和key.pem。

###func MaxBytesReader
```go
func MaxBytesReader(w ResponseWriter, r io.ReadCloser, n int64) io.ReadCloser
```
MaxBytesReader类似于LimitReader ，但是它的目的是限制到来的request bodies。与io.LimitReader相比，MaxBytesReader的结果是一个io.ReadCloser，它对于Read超过了limit返回non-EOF错误，并且当它的Close方法被调用的时候会关闭底层的reader。

###func NotFound
```go
func NotFound(w ResponseWriter, r *Request)
```
NotFound向请求回答HTTP 404 not found错误。

###func ParseHTTPVersion
```go
func ParseHTTPVersion(vers string) (major, minor int, ok bool)
```
解析HTTP版的字符串。"HTTP/1.0" 返回 (1, 0, true).

###func ParseTime
```go
func ParseTime(text string) (t time.Time, err error)
```
解析time header (比如 the Date: header)，它会尝试每一种HTTP/1.1允许的格式: TimeFormat, time.RFC850, and time.ANSIC。

###func ProxyFromEnvironment
```go
func ProxyFromEnvironment(req *Request) (*url.URL, error)
```
ProxyFromEnvironment返回给定request的代理url. 一般该URL由用户的环境变量 $HTTP_PROXY and $NO_PROXY （or $http_proxy and $no_proxy）指定。如果用户的全局代理环境无效则返回一个错误。 如果全局环境变量没有定义或者，则会返回一个nil的URL和一个nil的错误。

一种特殊的情形，如果req.URL.Host是"localhost"（带有或者不带有端口号），会返回一个nil的URL和一个nil的错误。

###func ProxyURL
```go
func ProxyURL(fixedURL *url.URL) func(*Request) (*url.URL, error)
```
返回一个代理函数（在Transport中使用），它常常返回相同的URL。

###func Redirect
```go
func Redirect(w ResponseWriter, r *Request, urlStr string, code int)
```
Redirect向请求回答一个url重定向，它可能是一个相对于请求路径的路径。

###func Serve
```go
func Serve(l net.Listener, handler Handler) error
```
Serve在Listener`l`上接收HTTP连接，为每一个创建一个新的service goroutine。service goroutine读取请求然后调用handler来回答它们。Handler典型值为nil，在这种情况下使用DefaultServerMux。

###func ServeContent
```go
func ServeContent(w ResponseWriter, req *Request, name string, modtime time.Time, content io.ReadSeeker)
```
ServeContent使用提供的ReadSeeker中的content来回答request。ServeContent比io.Copy更好的地方主要是它恰当地处理Range request、设置MIME类型和处理 If-Modified-Since请求。

如果响应的内容类型头没有设置,该函数首先会尝试从文件的文件扩展名推断文件类型。如果推断不出来，则会读取文件的第一个块并传送给DetectContentType来检测类型。 文件名称也可以不使用。 如果文字名称为空，则服务器不会传送给响应。

如果修改时间不为0，ServeContent会把它放在服务器响应的Last-Modified头里面。如果客户端请求中包含了If-Modified-Since头，ServeContent会使用modtime来判断是否把内容传给客户端。

content的Seek方法必须能够工作。 ServeContent通过定位到文件结尾来确定文件大小。 

如果调用函数已经设置w的ETag 头，ServeContent使用它来处理使用 If-Range 和 If-None-Match的请求。

*os.File中实现了io.ReadSeeker接口。

###func ServeFile
```go
func ServeFile(w ResponseWriter, r *Request, name string)
```
ServeFile向请求输出带有名字的文件或者目录。

###func SetCookie
```go
func SetCookie(w ResponseWriter, cookie *Cookie)
```
SetCookie向给定的 ResponseWriter的headers增加Set-Cookie头。

###func StatusText
```go
func StatusText(code int) string
```
StatusText返回对应于HTTP 状态码对应的文字。如果code未知，则返回空字符串。

###type Client struct
```go
type Client struct {
    // Transport specifies the mechanism by which individual
    // HTTP requests are made.
    // If nil, DefaultTransport is used.
    Transport RoundTripper

    // CheckRedirect specifies the policy for handling redirects.
    // If CheckRedirect is not nil, the client calls it before
    // following an HTTP redirect. The arguments req and via are
    // the upcoming request and the requests made already, oldest
    // first. If CheckRedirect returns an error, the Client's Get
    // method returns both the previous Response and
    // CheckRedirect's error (wrapped in a url.Error) instead of
    // issuing the Request req.
    //
    // If CheckRedirect is nil, the Client uses its default policy,
    // which is to stop after 10 consecutive requests.
    CheckRedirect func(req *Request, via []*Request) error

    // Jar specifies the cookie jar.
    // If Jar is nil, cookies are not sent in requests and ignored
    // in responses.
    Jar CookieJar

    // Timeout specifies a time limit for requests made by this
    // Client. The timeout includes connection time, any
    // redirects, and reading the response body. The timer remains
    // running after Get, Head, Post, or Do return and will
    // interrupt reading of the Response.Body.
    //
    // A Timeout of zero means no timeout.
    //
    // The Client's Transport must support the CancelRequest
    // method or Client will return errors when attempting to make
    // a request with Get, Head, Post, or Do. Client's default
    // Transport (DefaultTransport) supports CancelRequest.
    Timeout time.Duration
}
```

###func (*Client) Do
```go
func (c *Client) Do(req *Request) (resp *Response, err error)
```

###func (*Client) Get
```go
func (c *Client) Get(url string) (resp *Response, err error)
```

###func (*Client) Head
```go
func (c *Client) Head(url string) (resp *Response, err error)
```

###func (*Client) Post
```go
func (c *Client) Post(url string, bodyType string, body io.Reader) (resp *Response, err error)
```

###func (*Client) PostForm
```go
func (c *Client) PostForm(url string, data url.Values) (resp *Response, err error)
```

###type CloseNotifier interface
```go
type CloseNotifier interface {
    // CloseNotify returns a channel that receives a single value
    // when the client connection has gone away.
    CloseNotify() <-chan bool
}
```

###func (ConnState) String
```go
func (c ConnState) String() string
```

###type Cookie struct
```go
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

###func (*Cookie) String
```go
func (c *Cookie) String() string
```

###type CookieJar interface
```go
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

###func (Dir) Open
```go
func (d Dir) Open(name string) (File, error)
```

###type File interface
```go
type File interface {
    io.Closer
    io.Reader
    Readdir(count int) ([]os.FileInfo, error)
    Seek(offset int64, whence int) (int64, error)
    Stat() (os.FileInfo, error)
}
```

###type FileSystem interface
```go
type FileSystem interface {
    Open(name string) (File, error)
}
```

###type Flusher interface
```go
type Flusher interface {
    // Flush sends any buffered data to the client.
    Flush()
}
```

###type Handler interface
```go
type Handler interface {
    ServeHTTP(ResponseWriter, *Request)
}
```

###func FileServer
```go
func FileServer(root FileSystem) Handler
```

###func NotFoundHandler
```go
func NotFoundHandler() Handler
```

###func RedirectHandler
```go
func RedirectHandler(url string, code int) Handler
```

###func StripPrefix
```go
func StripPrefix(prefix string, h Handler) Handler
```

###func TimeoutHandler
```go
func TimeoutHandler(h Handler, dt time.Duration, msg string) Handler
```

###func (HandlerFunc) ServeHTTP
```go
func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request)
```

###func (Header) Add
```go
func (h Header) Add(key, value string)
```

###func (Header) Del
```go
func (h Header) Del(key string)
```

###func (Header) Get
```go
func (h Header) Get(key string) string
```

###func (Header) Set
```go
func (h Header) Set(key, value string)
```

###func (Header) Write
```go
func (h Header) Write(w io.Writer) error
```

###func (Header) WriteSubset
```go
func (h Header) WriteSubset(w io.Writer, exclude map[string]bool) error
```

###type Hijacker interface
```go
type Hijacker interface {
    // Hijack lets the caller take over the connection.
    // After a call to Hijack(), the HTTP server library
    // will not do anything else with the connection.
    // It becomes the caller's responsibility to manage
    // and close the connection.
    Hijack() (net.Conn, *bufio.ReadWriter, error)
}
```

###type ProtocolError struct
```go
type ProtocolError struct {
    ErrorString string
}
```

###func (*ProtocolError) Error
```go
func (err *ProtocolError) Error() string
```

###type Request struct
```go
type Request struct {
    // Method specifies the HTTP method (GET, POST, PUT, etc.).
    // For client requests an empty string means GET.
    Method string

    // URL specifies either the URI being requested (for server
    // requests) or the URL to access (for client requests).
    //
    // For server requests the URL is parsed from the URI
    // supplied on the Request-Line as stored in RequestURI.  For
    // most requests, fields other than Path and RawQuery will be
    // empty. (See RFC 2616, Section 5.1.2)
    //
    // For client requests, the URL's Host specifies the server to
    // connect to, while the Request's Host field optionally
    // specifies the Host header value to send in the HTTP
    // request.
    URL *url.URL

    // The protocol version for incoming requests.
    // Client requests always use HTTP/1.1.
    Proto      string // "HTTP/1.0"
    ProtoMajor int    // 1
    ProtoMinor int    // 0

    // A header maps request lines to their values.
    // If the header says
    //
    //	accept-encoding: gzip, deflate
    //	Accept-Language: en-us
    //	Connection: keep-alive
    //
    // then
    //
    //	Header = map[string][]string{
    //		"Accept-Encoding": {"gzip, deflate"},
    //		"Accept-Language": {"en-us"},
    //		"Connection": {"keep-alive"},
    //	}
    //
    // HTTP defines that header names are case-insensitive.
    // The request parser implements this by canonicalizing the
    // name, making the first character and any characters
    // following a hyphen uppercase and the rest lowercase.
    //
    // For client requests certain headers are automatically
    // added and may override values in Header.
    //
    // See the documentation for the Request.Write method.
    Header Header

    // Body is the request's body.
    //
    // For client requests a nil body means the request has no
    // body, such as a GET request. The HTTP Client's Transport
    // is responsible for calling the Close method.
    //
    // For server requests the Request Body is always non-nil
    // but will return EOF immediately when no body is present.
    // The Server will close the request body. The ServeHTTP
    // Handler does not need to.
    Body io.ReadCloser

    // ContentLength records the length of the associated content.
    // The value -1 indicates that the length is unknown.
    // Values >= 0 indicate that the given number of bytes may
    // be read from Body.
    // For client requests, a value of 0 means unknown if Body is not nil.
    ContentLength int64

    // TransferEncoding lists the transfer encodings from outermost to
    // innermost. An empty list denotes the "identity" encoding.
    // TransferEncoding can usually be ignored; chunked encoding is
    // automatically added and removed as necessary when sending and
    // receiving requests.
    TransferEncoding []string

    // Close indicates whether to close the connection after
    // replying to this request (for servers) or after sending
    // the request (for clients).
    Close bool

    // For server requests Host specifies the host on which the
    // URL is sought. Per RFC 2616, this is either the value of
    // the "Host" header or the host name given in the URL itself.
    // It may be of the form "host:port".
    //
    // For client requests Host optionally overrides the Host
    // header to send. If empty, the Request.Write method uses
    // the value of URL.Host.
    Host string

    // Form contains the parsed form data, including both the URL
    // field's query parameters and the POST or PUT form data.
    // This field is only available after ParseForm is called.
    // The HTTP client ignores Form and uses Body instead.
    Form url.Values

    // PostForm contains the parsed form data from POST or PUT
    // body parameters.
    // This field is only available after ParseForm is called.
    // The HTTP client ignores PostForm and uses Body instead.
    PostForm url.Values

    // MultipartForm is the parsed multipart form, including file uploads.
    // This field is only available after ParseMultipartForm is called.
    // The HTTP client ignores MultipartForm and uses Body instead.
    MultipartForm *multipart.Form

    // Trailer specifies additional headers that are sent after the request
    // body.
    //
    // For server requests the Trailer map initially contains only the
    // trailer keys, with nil values. (The client declares which trailers it
    // will later send.)  While the handler is reading from Body, it must
    // not reference Trailer. After reading from Body returns EOF, Trailer
    // can be read again and will contain non-nil values, if they were sent
    // by the client.
    //
    // For client requests Trailer must be initialized to a map containing
    // the trailer keys to later send. The values may be nil or their final
    // values. The ContentLength must be 0 or -1, to send a chunked request.
    // After the HTTP request is sent the map values can be updated while
    // the request body is read. Once the body returns EOF, the caller must
    // not mutate Trailer.
    //
    // Few HTTP clients, servers, or proxies support HTTP trailers.
    Trailer Header

    // RemoteAddr allows HTTP servers and other software to record
    // the network address that sent the request, usually for
    // logging. This field is not filled in by ReadRequest and
    // has no defined format. The HTTP server in this package
    // sets RemoteAddr to an "IP:port" address before invoking a
    // handler.
    // This field is ignored by the HTTP client.
    RemoteAddr string

    // RequestURI is the unmodified Request-URI of the
    // Request-Line (RFC 2616, Section 5.1) as sent by the client
    // to a server. Usually the URL field should be used instead.
    // It is an error to set this field in an HTTP client request.
    RequestURI string

    // TLS allows HTTP servers and other software to record
    // information about the TLS connection on which the request
    // was received. This field is not filled in by ReadRequest.
    // The HTTP server in this package sets the field for
    // TLS-enabled connections before invoking a handler;
    // otherwise it leaves the field nil.
    // This field is ignored by the HTTP client.
    TLS *tls.ConnectionState
}
```

###func NewRequest
```go
func NewRequest(method, urlStr string, body io.Reader) (*Request, error)
```

###func ReadRequest
```go
func ReadRequest(b *bufio.Reader) (req *Request, err error)
```

###func (*Request) AddCookie
```go
func (r *Request) AddCookie(c *Cookie)
```

###func (*Request) Cookie
```go
func (r *Request) Cookie(name string) (*Cookie, error)
```

###func (*Request) Cookies
```go
func (r *Request) Cookies() []*Cookie
```

###func (*Request) FormFile
```go
func (r *Request) FormFile(key string) (multipart.File, *multipart.FileHeader, error)
```

###func (*Request) FormValue
```go
func (r *Request) FormValue(key string) string
```

###func (*Request) MultipartReader
```go
func (r *Request) MultipartReader() (*multipart.Reader, error)
```

###func (*Request) ParseForm
```go
func (r *Request) ParseForm() error
```

###func (*Request) ParseMultipartForm
```go
func (r *Request) ParseMultipartForm(maxMemory int64) error
```

###func (*Request) PostFormValue
```go
func (r *Request) PostFormValue(key string) string
```

###func (*Request) ProtoAtLeast
```go
func (r *Request) ProtoAtLeast(major, minor int) bool
```

###func (*Request) Referer
```go
func (r *Request) Referer() string
```

###func (*Request) SetBasicAuth
```go
func (r *Request) SetBasicAuth(username, password string)
```

###func (*Request) UserAgent
```go
func (r *Request) UserAgent() string
```

###func (*Request) Write
```go
func (r *Request) Write(w io.Writer) error
```

###func (*Request) WriteProxy
```go
func (r *Request) WriteProxy(w io.Writer) error
```

###type Response struct
```go
type Response struct {
    Status     string // e.g. "200 OK"
    StatusCode int    // e.g. 200
    Proto      string // e.g. "HTTP/1.0"
    ProtoMajor int    // e.g. 1
    ProtoMinor int    // e.g. 0

    // Header maps header keys to values.  If the response had multiple
    // headers with the same key, they may be concatenated, with comma
    // delimiters.  (Section 4.2 of RFC 2616 requires that multiple headers
    // be semantically equivalent to a comma-delimited sequence.) Values
    // duplicated by other fields in this struct (e.g., ContentLength) are
    // omitted from Header.
    //
    // Keys in the map are canonicalized (see CanonicalHeaderKey).
    Header Header

    // Body represents the response body.
    //
    // The http Client and Transport guarantee that Body is always
    // non-nil, even on responses without a body or responses with
    // a zero-length body. It is the caller's responsibility to
    // close Body.
    //
    // The Body is automatically dechunked if the server replied
    // with a "chunked" Transfer-Encoding.
    Body io.ReadCloser

    // ContentLength records the length of the associated content.  The
    // value -1 indicates that the length is unknown.  Unless Request.Method
    // is "HEAD", values >= 0 indicate that the given number of bytes may
    // be read from Body.
    ContentLength int64

    // Contains transfer encodings from outer-most to inner-most. Value is
    // nil, means that "identity" encoding is used.
    TransferEncoding []string

    // Close records whether the header directed that the connection be
    // closed after reading Body.  The value is advice for clients: neither
    // ReadResponse nor Response.Write ever closes a connection.
    Close bool

    // Trailer maps trailer keys to values, in the same
    // format as the header.
    Trailer Header

    // The Request that was sent to obtain this Response.
    // Request's Body is nil (having already been consumed).
    // This is only populated for Client requests.
    Request *Request

    // TLS contains information about the TLS connection on which the
    // response was received. It is nil for unencrypted responses.
    // The pointer is shared between responses and should not be
    // modified.
    TLS *tls.ConnectionState
}
```

###func Get
```go
func Get(url string) (resp *Response, err error)
```

###func Head
```go
func Head(url string) (resp *Response, err error)
```

###func Post
```go
func Post(url string, bodyType string, body io.Reader) (resp *Response, err error)
```

###func PostForm
```go
func PostForm(url string, data url.Values) (resp *Response, err error)
```

###func ReadResponse
```go
func ReadResponse(r *bufio.Reader, req *Request) (*Response, error)
```

###func (*Response) Cookies
```go
func (r *Response) Cookies() []*Cookie
```

###func (*Response) Location
```go
func (r *Response) Location() (*url.URL, error)
```

###func (*Response) ProtoAtLeast
```go
func (r *Response) ProtoAtLeast(major, minor int) bool
```

###func (*Response) Write
```go
func (r *Response) Write(w io.Writer) error
```

###type ResponseWriter interface
```go
type ResponseWriter interface {
    // Header returns the header map that will be sent by WriteHeader.
    // Changing the header after a call to WriteHeader (or Write) has
    // no effect.
    Header() Header

    // Write writes the data to the connection as part of an HTTP reply.
    // If WriteHeader has not yet been called, Write calls WriteHeader(http.StatusOK)
    // before writing the data.  If the Header does not contain a
    // Content-Type line, Write adds a Content-Type set to the result of passing
    // the initial 512 bytes of written data to DetectContentType.
    Write([]byte) (int, error)

    // WriteHeader sends an HTTP response header with status code.
    // If WriteHeader is not called explicitly, the first call to Write
    // will trigger an implicit WriteHeader(http.StatusOK).
    // Thus explicit calls to WriteHeader are mainly used to
    // send error codes.
    WriteHeader(int)
}
```

###type RoundTripper interface
```go
type RoundTripper interface {
    // RoundTrip executes a single HTTP transaction, returning
    // the Response for the request req.  RoundTrip should not
    // attempt to interpret the response.  In particular,
    // RoundTrip must return err == nil if it obtained a response,
    // regardless of the response's HTTP status code.  A non-nil
    // err should be reserved for failure to obtain a response.
    // Similarly, RoundTrip should not attempt to handle
    // higher-level protocol details such as redirects,
    // authentication, or cookies.
    //
    // RoundTrip should not modify the request, except for
    // consuming and closing the Body, including on errors. The
    // request's URL and Header fields are guaranteed to be
    // initialized.
    RoundTrip(*Request) (*Response, error)
}
```

###func NewFileTransport
```go
func NewFileTransport(fs FileSystem) RoundTripper
```

###type ServeMux struct
```go
type ServeMux struct {
    // contains filtered or unexported fields
}
```

###func NewServeMux
```go
func NewServeMux() *ServeMux
```

###func (*ServeMux) Handle
```go
func (mux *ServeMux) Handle(pattern string, handler Handler)
```

###func (*ServeMux) HandleFunc
```go
func (mux *ServeMux) HandleFunc(pattern string, handler func(ResponseWriter, *Request))
```

###func (*ServeMux) Handler
```go
func (mux *ServeMux) Handler(r *Request) (h Handler, pattern string)
```

###func (*ServeMux) ServeHTTP
```go
func (mux *ServeMux) ServeHTTP(w ResponseWriter, r *Request)
```

###type Server struct
```go
type Server struct {
    Addr           string        // TCP address to listen on, ":http" if empty
    Handler        Handler       // handler to invoke, http.DefaultServeMux if nil
    ReadTimeout    time.Duration // maximum duration before timing out read of the request
    WriteTimeout   time.Duration // maximum duration before timing out write of the response
    MaxHeaderBytes int           // maximum size of request headers, DefaultMaxHeaderBytes if 0
    TLSConfig      *tls.Config   // optional TLS config, used by ListenAndServeTLS

    // TLSNextProto optionally specifies a function to take over
    // ownership of the provided TLS connection when an NPN
    // protocol upgrade has occurred.  The map key is the protocol
    // name negotiated. The Handler argument should be used to
    // handle HTTP requests and will initialize the Request's TLS
    // and RemoteAddr if not already set.  The connection is
    // automatically closed when the function returns.
    TLSNextProto map[string]func(*Server, *tls.Conn, Handler)

    // ConnState specifies an optional callback function that is
    // called when a client connection changes state. See the
    // ConnState type and associated constants for details.
    ConnState func(net.Conn, ConnState)

    // ErrorLog specifies an optional logger for errors accepting
    // connections and unexpected behavior from handlers.
    // If nil, logging goes to os.Stderr via the log package's
    // standard logger.
    ErrorLog *log.Logger
    // contains filtered or unexported fields
}
```

###func (*Server) ListenAndServe
```go
func (srv *Server) ListenAndServe() error
```

###func (*Server) ListenAndServeTLS
```go
func (srv *Server) ListenAndServeTLS(certFile, keyFile string) error
```

###func (*Server) Serve
```go
func (srv *Server) Serve(l net.Listener) error
```

###func (*Server) SetKeepAlivesEnabled
```go
func (s *Server) SetKeepAlivesEnabled(v bool)
```

###type Transport struct
```go
type Transport struct {

    // Proxy specifies a function to return a proxy for a given
    // Request. If the function returns a non-nil error, the
    // request is aborted with the provided error.
    // If Proxy is nil or returns a nil *URL, no proxy is used.
    Proxy func(*Request) (*url.URL, error)

    // Dial specifies the dial function for creating TCP
    // connections.
    // If Dial is nil, net.Dial is used.
    Dial func(network, addr string) (net.Conn, error)

    // TLSClientConfig specifies the TLS configuration to use with
    // tls.Client. If nil, the default configuration is used.
    TLSClientConfig *tls.Config

    // TLSHandshakeTimeout specifies the maximum amount of time waiting to
    // wait for a TLS handshake. Zero means no timeout.
    TLSHandshakeTimeout time.Duration

    // DisableKeepAlives, if true, prevents re-use of TCP connections
    // between different HTTP requests.
    DisableKeepAlives bool

    // DisableCompression, if true, prevents the Transport from
    // requesting compression with an "Accept-Encoding: gzip"
    // request header when the Request contains no existing
    // Accept-Encoding value. If the Transport requests gzip on
    // its own and gets a gzipped response, it's transparently
    // decoded in the Response.Body. However, if the user
    // explicitly requested gzip it is not automatically
    // uncompressed.
    DisableCompression bool

    // MaxIdleConnsPerHost, if non-zero, controls the maximum idle
    // (keep-alive) to keep per-host.  If zero,
    // DefaultMaxIdleConnsPerHost is used.
    MaxIdleConnsPerHost int

    // ResponseHeaderTimeout, if non-zero, specifies the amount of
    // time to wait for a server's response headers after fully
    // writing the request (including its body, if any). This
    // time does not include the time to read the response body.
    ResponseHeaderTimeout time.Duration
    // contains filtered or unexported fields
}
```

###func (*Transport) CancelRequest
```go
func (t *Transport) CancelRequest(req *Request)
```

###func (*Transport) CloseIdleConnections
```go
func (t *Transport) CloseIdleConnections()
```

###func (*Transport) RegisterProtocol
```go
func (t *Transport) RegisterProtocol(scheme string, rt RoundTripper)
```

###func (*Transport) RoundTrip
```go
func (t *Transport) RoundTrip(req *Request) (resp *Response, err error)
```

