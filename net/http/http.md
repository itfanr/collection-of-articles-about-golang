#htp包

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


###func InterfaceAddrs
```go
func InterfaceAddrs() ([]Addr, error)
```

###func Interfaces
```go
func Interfaces() ([]Interface, error)
```

###func JoinHostPort
```go
func JoinHostPort(host, port string) string
```

###func LookupAddr
```go
func LookupAddr(addr string) (name []string, err error)
```

###func LookupCNAME
```go
func LookupCNAME(name string) (cname string, err error)
```

###func LookupHost
```go
func LookupHost(host string) (addrs []string, err error)
```

###func LookupIP
```go
func LookupIP(host string) (addrs []IP, err error)
```

###func LookupMX
```go
func LookupMX(name string) (mx []*MX, err error)
```

###func LookupNS
```go
func LookupNS(name string) (ns []*NS, err error)
```

###func LookupPort
```go
func LookupPort(network, service string) (port int, err error)
```

###func LookupSRV
```go
func LookupSRV(service, proto, name string) (cname string, addrs []*SRV, err error)
```

###func LookupTXT
```go
func LookupTXT(name string) (txt []string, err error)
```

###func SplitHostPort
```go
func SplitHostPort(hostport string) (host, port string, err error)
```

###type Addr interface
```go
type Addr interface {
    Network() string // name of the network
    String() string  // string form of address
}
```

###type AddrError struct
```go
type AddrError struct {
    Err  string
    Addr string
}
```

###func (*AddrError) Error
```go
func (e *AddrError) Error() string
```

###func (*AddrError) Temporary
```go
func (e *AddrError) Temporary() bool
```

###func (*AddrError) Timeout
```go
func (e *AddrError) Timeout() bool
```

###type Conn interface
```go
type Conn interface {
    // Read reads data from the connection.
    // Read can be made to time out and return a Error with Timeout() == true
    // after a fixed time limit; see SetDeadline and SetReadDeadline.
    Read(b []byte) (n int, err error)

    // Write writes data to the connection.
    // Write can be made to time out and return a Error with Timeout() == true
    // after a fixed time limit; see SetDeadline and SetWriteDeadline.
    Write(b []byte) (n int, err error)

    // Close closes the connection.
    // Any blocked Read or Write operations will be unblocked and return errors.
    Close() error

    // LocalAddr returns the local network address.
    LocalAddr() Addr

    // RemoteAddr returns the remote network address.
    RemoteAddr() Addr

    // SetDeadline sets the read and write deadlines associated
    // with the connection. It is equivalent to calling both
    // SetReadDeadline and SetWriteDeadline.
    //
    // A deadline is an absolute time after which I/O operations
    // fail with a timeout (see type Error) instead of
    // blocking. The deadline applies to all future I/O, not just
    // the immediately following call to Read or Write.
    //
    // An idle timeout can be implemented by repeatedly extending
    // the deadline after successful Read or Write calls.
    //
    // A zero value for t means I/O operations will not time out.
    SetDeadline(t time.Time) error

    // SetReadDeadline sets the deadline for future Read calls.
    // A zero value for t means Read will not time out.
    SetReadDeadline(t time.Time) error

    // SetWriteDeadline sets the deadline for future Write calls.
    // Even if write times out, it may return n > 0, indicating that
    // some of the data was successfully written.
    // A zero value for t means Write will not time out.
    SetWriteDeadline(t time.Time) error
}
```

###func Dial
```go
func Dial(network, address string) (Conn, error)
```

###func DialTimeout
```go
func DialTimeout(network, address string, timeout time.Duration) (Conn, error)
```

###func FileConn
```go
func FileConn(f *os.File) (c Conn, err error)
```

###func Pipe
```go
func Pipe() (Conn, Conn)
```

###type DNSConfigError struct
```go
type DNSConfigError struct {
    Err error
}
```

###func (*DNSConfigError) Error
```go
func (e *DNSConfigError) Error() string
```

###func (*DNSConfigError) Temporary
```go
func (e *DNSConfigError) Temporary() bool
```

###func (*DNSConfigError) Timeout
```go
func (e *DNSConfigError) Timeout() bool
```

###type DNSError struct
```go
type DNSError struct {
    Err       string // description of the error
    Name      string // name looked for
    Server    string // server used
    IsTimeout bool
}
```

###func (*DNSError) Error
```go
func (e *DNSError) Error() string
```

###func (*DNSError) Temporary
```go
func (e *DNSError) Temporary() bool
```

###func (*DNSError) Timeout
```go
func (e *DNSError) Timeout() bool
```

###type Dialer struct
```go
type Dialer struct {
    // Timeout is the maximum amount of time a dial will wait for
    // a connect to complete. If Deadline is also set, it may fail
    // earlier.
    //
    // The default is no timeout.
    //
    // With or without a timeout, the operating system may impose
    // its own earlier timeout. For instance, TCP timeouts are
    // often around 3 minutes.
    Timeout time.Duration

    // Deadline is the absolute point in time after which dials
    // will fail. If Timeout is set, it may fail earlier.
    // Zero means no deadline, or dependent on the operating system
    // as with the Timeout option.
    Deadline time.Time

    // LocalAddr is the local address to use when dialing an
    // address. The address must be of a compatible type for the
    // network being dialed.
    // If nil, a local address is automatically chosen.
    LocalAddr Addr

    // DualStack allows a single dial to attempt to establish
    // multiple IPv4 and IPv6 connections and to return the first
    // established connection when the network is "tcp" and the
    // destination is a host name that has multiple address family
    // DNS records.
    DualStack bool

    // KeepAlive specifies the keep-alive period for an active
    // network connection.
    // If zero, keep-alives are not enabled. Network protocols
    // that do not support keep-alives ignore this field.
    KeepAlive time.Duration
}
```

###func (*Dialer) Dial
```go
func (d *Dialer) Dial(network, address string) (Conn, error)
```

###type Error interface
```go
type Error interface {
    error
    Timeout() bool   // Is the error a timeout?
    Temporary() bool // Is the error temporary?
}
```

###func (Flags) String
```go
func (f Flags) String() string
```

###func ParseMAC
```go
func ParseMAC(s string) (hw HardwareAddr, err error)
```

###func (HardwareAddr) String
```go
func (a HardwareAddr) String() string
```

###func IPv4
```go
func IPv4(a, b, c, d byte) IP
```

###func ParseCIDR
```go
func ParseCIDR(s string) (IP, *IPNet, error)
```

###func ParseIP
```go
func ParseIP(s string) IP
```

###func (IP) DefaultMask
```go
func (ip IP) DefaultMask() IPMask
```

###func (IP) Equal
```go
func (ip IP) Equal(x IP) bool
```

###func (IP) IsGlobalUnicast
```go
func (ip IP) IsGlobalUnicast() bool
```

###func (IP) IsInterfaceLocalMulticast
```go
func (ip IP) IsInterfaceLocalMulticast() bool
```

###func (IP) IsLinkLocalMulticast
```go
func (ip IP) IsLinkLocalMulticast() bool
```

###func (IP) IsLinkLocalUnicast
```go
func (ip IP) IsLinkLocalUnicast() bool
```

###func (IP) IsLoopback
```go
func (ip IP) IsLoopback() bool
```

###func (IP) IsMulticast
```go
func (ip IP) IsMulticast() bool
```

###func (IP) IsUnspecified
```go
func (ip IP) IsUnspecified() bool
```

###func (IP) MarshalText
```go
func (ip IP) MarshalText() ([]byte, error)
```

###func (IP) Mask
```go
func (ip IP) Mask(mask IPMask) IP
```

###func (IP) String
```go
func (ip IP) String() string
```

###func (IP) To16
```go
func (ip IP) To16() IP
```

###func (IP) To4
```go
func (ip IP) To4() IP
```

###func (*IP) UnmarshalText
```go
func (ip *IP) UnmarshalText(text []byte) error
```

###type IPAddr struct
```go
type IPAddr struct {
    IP   IP
    Zone string // IPv6 scoped addressing zone
}
```

###func ResolveIPAddr
```go
func ResolveIPAddr(net, addr string) (*IPAddr, error)
```

###func (*IPAddr) Network
```go
func (a *IPAddr) Network() string
```

###func (*IPAddr) String
```go
func (a *IPAddr) String() string
```

###type IPConn struct
```go
type IPConn struct {
    // contains filtered or unexported fields
}
```

###func DialIP
```go
func DialIP(netProto string, laddr, raddr *IPAddr) (*IPConn, error)
```

###func ListenIP
```go
func ListenIP(netProto string, laddr *IPAddr) (*IPConn, error)
```

###func (*IPConn) Close
```go
func (c *IPConn) Close() error
```

###func (*IPConn) File
```go
func (c *IPConn) File() (f *os.File, err error)
```

###func (*IPConn) LocalAddr
```go
func (c *IPConn) LocalAddr() Addr
```

###func (*IPConn) Read
```go
func (c *IPConn) Read(b []byte) (int, error)
```

###func (*IPConn) ReadFrom
```go
func (c *IPConn) ReadFrom(b []byte) (int, Addr, error)
```

###func (*IPConn) ReadFromIP
```go
func (c *IPConn) ReadFromIP(b []byte) (int, *IPAddr, error)
```

###func (*IPConn) ReadMsgIP
```go
func (c *IPConn) ReadMsgIP(b, oob []byte) (n, oobn, flags int, addr *IPAddr, err error)
```

###func (*IPConn) RemoteAddr
```go
func (c *IPConn) RemoteAddr() Addr
```

###func (*IPConn) SetDeadline
```go
func (c *IPConn) SetDeadline(t time.Time) error
```

###func (*IPConn) SetReadBuffer
```go
func (c *IPConn) SetReadBuffer(bytes int) error
```

###func (*IPConn) SetReadDeadline
```go
func (c *IPConn) SetReadDeadline(t time.Time) error
```

###func (*IPConn) SetWriteBuffer
```go
func (c *IPConn) SetWriteBuffer(bytes int) error
```

###func (*IPConn) SetWriteDeadline
```go
func (c *IPConn) SetWriteDeadline(t time.Time) error
```

###func (*IPConn) Write
```go
func (c *IPConn) Write(b []byte) (int, error)
```

###func (*IPConn) WriteMsgIP
```go
func (c *IPConn) WriteMsgIP(b, oob []byte, addr *IPAddr) (n, oobn int, err error)
```

###func (*IPConn) WriteTo
```go
func (c *IPConn) WriteTo(b []byte, addr Addr) (int, error)
```

###func (*IPConn) WriteToIP
```go
func (c *IPConn) WriteToIP(b []byte, addr *IPAddr) (int, error)
```

###func CIDRMask
```go
func CIDRMask(ones, bits int) IPMask
```

###func IPv4Mask
```go
func IPv4Mask(a, b, c, d byte) IPMask
```

###func (IPMask) Size
```go
func (m IPMask) Size() (ones, bits int)
```

###func (IPMask) String
```go
func (m IPMask) String() string
```

###type IPNet struct
```go
type IPNet struct {
    IP   IP     // network number
    Mask IPMask // network mask
}
```

###func (*IPNet) Contains
```go
func (n *IPNet) Contains(ip IP) bool
```

###func (*IPNet) Network
```go
func (n *IPNet) Network() string
```

###func (*IPNet) String
```go
func (n *IPNet) String() string
```

###type Interface struct
```go
type Interface struct {
    Index        int          // positive integer that starts at one, zero is never used
    MTU          int          // maximum transmission unit
    Name         string       // e.g., "en0", "lo0", "eth0.100"
    HardwareAddr HardwareAddr // IEEE MAC-48, EUI-48 and EUI-64 form
    Flags        Flags        // e.g., FlagUp, FlagLoopback, FlagMulticast
}
```

###func InterfaceByIndex
```go
func InterfaceByIndex(index int) (*Interface, error)
```

###func InterfaceByName
```go
func InterfaceByName(name string) (*Interface, error)
```

###func (*Interface) Addrs
```go
func (ifi *Interface) Addrs() ([]Addr, error)
```

###func (*Interface) MulticastAddrs
```go
func (ifi *Interface) MulticastAddrs() ([]Addr, error)
```

###func (InvalidAddrError) Error
```go
func (e InvalidAddrError) Error() string
```

###func (InvalidAddrError) Temporary
```go
func (e InvalidAddrError) Temporary() bool
```

###func (InvalidAddrError) Timeout
```go
func (e InvalidAddrError) Timeout() bool
```

###type Listener interface
```go
type Listener interface {
    // Accept waits for and returns the next connection to the listener.
    Accept() (c Conn, err error)

    // Close closes the listener.
    // Any blocked Accept operations will be unblocked and return errors.
    Close() error

    // Addr returns the listener's network address.
    Addr() Addr
}
```

###func FileListener
```go
func FileListener(f *os.File) (l Listener, err error)
```

###func Listen
```go
func Listen(net, laddr string) (Listener, error)
```

###type MX struct
```go
type MX struct {
    Host string
    Pref uint16
}
```

###type NS struct
```go
type NS struct {
    Host string
}
```

###type OpError struct
```go
type OpError struct {
    // Op is the operation which caused the error, such as
    // "read" or "write".
    Op string

    // Net is the network type on which this error occurred,
    // such as "tcp" or "udp6".
    Net string

    // Addr is the network address on which this error occurred.
    Addr Addr

    // Err is the error that occurred during the operation.
    Err error
}
```

###func (*OpError) Error
```go
func (e *OpError) Error() string
```

###func (*OpError) Temporary
```go
func (e *OpError) Temporary() bool
```

###func (*OpError) Timeout
```go
func (e *OpError) Timeout() bool
```

###type PacketConn interface
```go
type PacketConn interface {
    // ReadFrom reads a packet from the connection,
    // copying the payload into b.  It returns the number of
    // bytes copied into b and the return address that
    // was on the packet.
    // ReadFrom can be made to time out and return
    // an error with Timeout() == true after a fixed time limit;
    // see SetDeadline and SetReadDeadline.
    ReadFrom(b []byte) (n int, addr Addr, err error)

    // WriteTo writes a packet with payload b to addr.
    // WriteTo can be made to time out and return
    // an error with Timeout() == true after a fixed time limit;
    // see SetDeadline and SetWriteDeadline.
    // On packet-oriented connections, write timeouts are rare.
    WriteTo(b []byte, addr Addr) (n int, err error)

    // Close closes the connection.
    // Any blocked ReadFrom or WriteTo operations will be unblocked and return errors.
    Close() error

    // LocalAddr returns the local network address.
    LocalAddr() Addr

    // SetDeadline sets the read and write deadlines associated
    // with the connection.
    SetDeadline(t time.Time) error

    // SetReadDeadline sets the deadline for future Read calls.
    // If the deadline is reached, Read will fail with a timeout
    // (see type Error) instead of blocking.
    // A zero value for t means Read will not time out.
    SetReadDeadline(t time.Time) error

    // SetWriteDeadline sets the deadline for future Write calls.
    // If the deadline is reached, Write will fail with a timeout
    // (see type Error) instead of blocking.
    // A zero value for t means Write will not time out.
    // Even if write times out, it may return n > 0, indicating that
    // some of the data was successfully written.
    SetWriteDeadline(t time.Time) error
}
```

###func FilePacketConn
```go
func FilePacketConn(f *os.File) (c PacketConn, err error)
```

###func ListenPacket
```go
func ListenPacket(net, laddr string) (PacketConn, error)
```

###type ParseError struct
```go
type ParseError struct {
    Type string
    Text string
}
```

###func (*ParseError) Error
```go
func (e *ParseError) Error() string
```

###type SRV struct
```go
type SRV struct {
    Target   string
    Port     uint16
    Priority uint16
    Weight   uint16
}
```

###type TCPAddr struct
```go
type TCPAddr struct {
    IP   IP
    Port int
    Zone string // IPv6 scoped addressing zone
}
```

###func ResolveTCPAddr
```go
func ResolveTCPAddr(net, addr string) (*TCPAddr, error)
```

###func (*TCPAddr) Network
```go
func (a *TCPAddr) Network() string
```

###func (*TCPAddr) String
```go
func (a *TCPAddr) String() string
```

###type TCPConn struct
```go
type TCPConn struct {
    // contains filtered or unexported fields
}
```

###func DialTCP
```go
func DialTCP(net string, laddr, raddr *TCPAddr) (*TCPConn, error)
```

###func (*TCPConn) Close
```go
func (c *TCPConn) Close() error
```

###func (*TCPConn) CloseRead
```go
func (c *TCPConn) CloseRead() error
```

###func (*TCPConn) CloseWrite
```go
func (c *TCPConn) CloseWrite() error
```

###func (*TCPConn) File
```go
func (c *TCPConn) File() (f *os.File, err error)
```

###func (*TCPConn) LocalAddr
```go
func (c *TCPConn) LocalAddr() Addr
```

###func (*TCPConn) Read
```go
func (c *TCPConn) Read(b []byte) (int, error)
```

###func (*TCPConn) ReadFrom
```go
func (c *TCPConn) ReadFrom(r io.Reader) (int64, error)
```

###func (*TCPConn) RemoteAddr
```go
func (c *TCPConn) RemoteAddr() Addr
```

###func (*TCPConn) SetDeadline
```go
func (c *TCPConn) SetDeadline(t time.Time) error
```

###func (*TCPConn) SetKeepAlive
```go
func (c *TCPConn) SetKeepAlive(keepalive bool) error
```

###func (*TCPConn) SetKeepAlivePeriod
```go
func (c *TCPConn) SetKeepAlivePeriod(d time.Duration) error
```

###func (*TCPConn) SetLinger
```go
func (c *TCPConn) SetLinger(sec int) error
```

###func (*TCPConn) SetNoDelay
```go
func (c *TCPConn) SetNoDelay(noDelay bool) error
```

###func (*TCPConn) SetReadBuffer
```go
func (c *TCPConn) SetReadBuffer(bytes int) error
```

###func (*TCPConn) SetReadDeadline
```go
func (c *TCPConn) SetReadDeadline(t time.Time) error
```

###func (*TCPConn) SetWriteBuffer
```go
func (c *TCPConn) SetWriteBuffer(bytes int) error
```

###func (*TCPConn) SetWriteDeadline
```go
func (c *TCPConn) SetWriteDeadline(t time.Time) error
```

###func (*TCPConn) Write
```go
func (c *TCPConn) Write(b []byte) (int, error)
```

###type TCPListener struct
```go
type TCPListener struct {
    // contains filtered or unexported fields
}
```

###func ListenTCP
```go
func ListenTCP(net string, laddr *TCPAddr) (*TCPListener, error)
```

###func (*TCPListener) Accept
```go
func (l *TCPListener) Accept() (Conn, error)
```

###func (*TCPListener) AcceptTCP
```go
func (l *TCPListener) AcceptTCP() (*TCPConn, error)
```

###func (*TCPListener) Addr
```go
func (l *TCPListener) Addr() Addr
```

###func (*TCPListener) Close
```go
func (l *TCPListener) Close() error
```

###func (*TCPListener) File
```go
func (l *TCPListener) File() (f *os.File, err error)
```

###func (*TCPListener) SetDeadline
```go
func (l *TCPListener) SetDeadline(t time.Time) error
```

###type UDPAddr struct
```go
type UDPAddr struct {
    IP   IP
    Port int
    Zone string // IPv6 scoped addressing zone
}
```

###func ResolveUDPAddr
```go
func ResolveUDPAddr(net, addr string) (*UDPAddr, error)
```

###func (*UDPAddr) Network
```go
func (a *UDPAddr) Network() string
```

###func (*UDPAddr) String
```go
func (a *UDPAddr) String() string
```

###type UDPConn struct
```go
type UDPConn struct {
    // contains filtered or unexported fields
}
```

###func DialUDP
```go
func DialUDP(net string, laddr, raddr *UDPAddr) (*UDPConn, error)
```

###func ListenMulticastUDP
```go
func ListenMulticastUDP(net string, ifi *Interface, gaddr *UDPAddr) (*UDPConn, error)
```

###func ListenUDP
```go
func ListenUDP(net string, laddr *UDPAddr) (*UDPConn, error)
```

###func (*UDPConn) Close
```go
func (c *UDPConn) Close() error
```

###func (*UDPConn) File
```go
func (c *UDPConn) File() (f *os.File, err error)
```

###func (*UDPConn) LocalAddr
```go
func (c *UDPConn) LocalAddr() Addr
```

###func (*UDPConn) Read
```go
func (c *UDPConn) Read(b []byte) (int, error)
```

###func (*UDPConn) ReadFrom
```go
func (c *UDPConn) ReadFrom(b []byte) (int, Addr, error)
```

###func (*UDPConn) ReadFromUDP
```go
func (c *UDPConn) ReadFromUDP(b []byte) (n int, addr *UDPAddr, err error)
```

###func (*UDPConn) ReadMsgUDP
```go
func (c *UDPConn) ReadMsgUDP(b, oob []byte) (n, oobn, flags int, addr *UDPAddr, err error)
```

###func (*UDPConn) RemoteAddr
```go
func (c *UDPConn) RemoteAddr() Addr
```

###func (*UDPConn) SetDeadline
```go
func (c *UDPConn) SetDeadline(t time.Time) error
```

###func (*UDPConn) SetReadBuffer
```go
func (c *UDPConn) SetReadBuffer(bytes int) error
```

###func (*UDPConn) SetReadDeadline
```go
func (c *UDPConn) SetReadDeadline(t time.Time) error
```

###func (*UDPConn) SetWriteBuffer
```go
func (c *UDPConn) SetWriteBuffer(bytes int) error
```

###func (*UDPConn) SetWriteDeadline
```go
func (c *UDPConn) SetWriteDeadline(t time.Time) error
```

###func (*UDPConn) Write
```go
func (c *UDPConn) Write(b []byte) (int, error)
```

###func (*UDPConn) WriteMsgUDP
```go
func (c *UDPConn) WriteMsgUDP(b, oob []byte, addr *UDPAddr) (n, oobn int, err error)
```

###func (*UDPConn) WriteTo
```go
func (c *UDPConn) WriteTo(b []byte, addr Addr) (int, error)
```

###func (*UDPConn) WriteToUDP
```go
func (c *UDPConn) WriteToUDP(b []byte, addr *UDPAddr) (int, error)
```

###type UnixAddr struct
```go
type UnixAddr struct {
    Name string
    Net  string
}
```

###func ResolveUnixAddr
```go
func ResolveUnixAddr(net, addr string) (*UnixAddr, error)
```

###func (*UnixAddr) Network
```go
func (a *UnixAddr) Network() string
```

###func (*UnixAddr) String
```go
func (a *UnixAddr) String() string
```

###type UnixConn struct
```go
type UnixConn struct {
    // contains filtered or unexported fields
}
```

###func DialUnix
```go
func DialUnix(net string, laddr, raddr *UnixAddr) (*UnixConn, error)
```

###func ListenUnixgram
```go
func ListenUnixgram(net string, laddr *UnixAddr) (*UnixConn, error)
```

###func (*UnixConn) Close
```go
func (c *UnixConn) Close() error
```

###func (*UnixConn) CloseRead
```go
func (c *UnixConn) CloseRead() error
```

###func (*UnixConn) CloseWrite
```go
func (c *UnixConn) CloseWrite() error
```

###func (*UnixConn) File
```go
func (c *UnixConn) File() (f *os.File, err error)
```

###func (*UnixConn) LocalAddr
```go
func (c *UnixConn) LocalAddr() Addr
```

###func (*UnixConn) Read
```go
func (c *UnixConn) Read(b []byte) (int, error)
```

###func (*UnixConn) ReadFrom
```go
func (c *UnixConn) ReadFrom(b []byte) (int, Addr, error)
```

###func (*UnixConn) ReadFromUnix
```go
func (c *UnixConn) ReadFromUnix(b []byte) (n int, addr *UnixAddr, err error)
```

###func (*UnixConn) ReadMsgUnix
```go
func (c *UnixConn) ReadMsgUnix(b, oob []byte) (n, oobn, flags int, addr *UnixAddr, err error)
```

###func (*UnixConn) RemoteAddr
```go
func (c *UnixConn) RemoteAddr() Addr
```

###func (*UnixConn) SetDeadline
```go
func (c *UnixConn) SetDeadline(t time.Time) error
```

###func (*UnixConn) SetReadBuffer
```go
func (c *UnixConn) SetReadBuffer(bytes int) error
```

###func (*UnixConn) SetReadDeadline
```go
func (c *UnixConn) SetReadDeadline(t time.Time) error
```

###func (*UnixConn) SetWriteBuffer
```go
func (c *UnixConn) SetWriteBuffer(bytes int) error
```

###func (*UnixConn) SetWriteDeadline
```go
func (c *UnixConn) SetWriteDeadline(t time.Time) error
```

###func (*UnixConn) Write
```go
func (c *UnixConn) Write(b []byte) (int, error)
```

###func (*UnixConn) WriteMsgUnix
```go
func (c *UnixConn) WriteMsgUnix(b, oob []byte, addr *UnixAddr) (n, oobn int, err error)
```

###func (*UnixConn) WriteTo
```go
func (c *UnixConn) WriteTo(b []byte, addr Addr) (n int, err error)
```

###func (*UnixConn) WriteToUnix
```go
func (c *UnixConn) WriteToUnix(b []byte, addr *UnixAddr) (n int, err error)
```

###type UnixListener struct
```go
type UnixListener struct {
    // contains filtered or unexported fields
}
```

###func ListenUnix
```go
func ListenUnix(net string, laddr *UnixAddr) (*UnixListener, error)
```

###func (*UnixListener) Accept
```go
func (l *UnixListener) Accept() (c Conn, err error)
```

###func (*UnixListener) AcceptUnix
```go
func (l *UnixListener) AcceptUnix() (*UnixConn, error)
```

###func (*UnixListener) Addr
```go
func (l *UnixListener) Addr() Addr
```

###func (*UnixListener) Close
```go
func (l *UnixListener) Close() error
```

###func (*UnixListener) File
```go
func (l *UnixListener) File() (f *os.File, err error)
```

###func (*UnixListener) SetDeadline
```go
func (l *UnixListener) SetDeadline(t time.Time) (err error)
```

###func (UnknownNetworkError) Error
```go
func (e UnknownNetworkError) Error() string
```

###func (UnknownNetworkError) Temporary
```go
func (e UnknownNetworkError) Temporary() bool
```

###func (UnknownNetworkError) Timeout
```go
func (e UnknownNetworkError) Timeout() bool
```

