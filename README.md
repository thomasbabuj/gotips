![header](https://raw.githubusercontent.com/beyondns/gotips/master/gophers.jpg)

> **Golang short tips & trics**

This list of short golang code tips & trics will help keep collected knowledge in one place. 

> Can you help make it better?

Please feel free to send a PR with your own Golang tip to be published here. Any improvements or suggestions are welcome!  

> Contributing

Make PR add new tip on top of list with title, date, description, code and links, please see tips as a reference.

> Don't forget to Star the repo, as this will help to promote the project!

# Tips list

- 17 - [Use context API](https://github.com/beyondns/gotips#17---use-context-api)
- 16 - [Go routines syncronization](https://github.com/beyondns/gotips#16---go-routines-syncronization)
- 15 - [Time interval measurement](https://github.com/beyondns/gotips#15---time-interval-measurement)
- 14 - [Benchmark switch vs else if](https://github.com/beyondns/gotips#14---benchmark-switch-vs-else-if)
- 13 - [Use ASM in Go Code](https://github.com/beyondns/gotips#13---use-asm-in-go-code)
- 12 - [JSON with unknown structure](https://github.com/beyondns/gotips#12---json-with-unknown-structure)
- 11 - [Websocket over HTTP2](https://github.com/beyondns/gotips#11---websocket-over-http2)
- 10 - [HTTP2](https://github.com/beyondns/gotips#10---http2)
-  9 - [Error handling](https://github.com/beyondns/gotips#9---error-handling)
-  8 - [Memory management with pools of objects](https://github.com/beyondns/gotips#8---memory-management-with-pools-of-objects)
-  7 - [Sort slice of time periods](https://github.com/beyondns/gotips#7---sort-slice-of-time-periods)
-  6 - [Fast http server](https://github.com/beyondns/gotips#6---fast-http-server)
-  5 - [Close channel to notify many](https://github.com/beyondns/gotips#5---close-channel-to-notify-many)
-  4 - [Is channel closed?](https://github.com/beyondns/gotips#4---is-channel-closed)
-  3 - [Http request/response with close notify and timeout](https://github.com/beyondns/gotips#3---http-requestresponse-with-close-notify-and-timeout)
-  2 - [Import packages](https://github.com/beyondns/gotips#2---import-packages)
-  1 - [Map](https://github.com/beyondns/gotips#1---map)
-  0 - [Slices](https://github.com/beyondns/gotips#0---slices)

## #17 - Use context API
> 2016-16-02 by [@beyondns](https://github.com/beyondns)

Some APIs are designed with context interface, google search is an example. Use context to send cancel signal.

```go
	done := make(chan error,1)
	query := "golang context"
	ctx, cancel := context.WithCancel(context.Background())
	// or use ctx, cancel = context.WithTimeout(context.Background(), queryTimeLimit)
	go func() {
		start := time.Now()
		_, err := google.Search(ctx, query)
		elapsed := time.Since(start)
		fmt.Printf("search time %v", elapsed)
		done <- err
	}()
	select {
	case <-time.After(queryTimeLimit):
		cancel()
		<-done // wait
		fmt.Printf("time out")
	case err :=<- done:
		if err != nil {
			panic(err)
		}
	}	
	fmt.Printf("Done")
```

* [blog.golang.org/context](https://blog.golang.org/context)
* [blog.golang.org/context/server/server.go](https://blog.golang.org/context/server/server.go)

## #16 - Go routines syncronization
> 2016-16-02 by [@beyondns](https://github.com/beyondns)

* Wait groups
```go
	const N = 3
	var wg sync.WaitGroup
	wg.Add(N)
	for i := 0; i < N; i++ {
		go func(i int) {
			defer wg.Done()
			fmt.Println(i)
		}(i)
	}
	wg.Wait()
	fmt.Println("Done")
```
[run](https://play.golang.org/p/0LLtAQk6Lm)


* Channels
```go
	const N = 3
	done := make(chan struct{})
	for i := 0; i < N; i++ {
		go func(i int) {
			defer func() { done <- struct{}{} }()
			fmt.Println(i)
		}(i)
	}
	var counter = 0
	for {
		select {
		case <-time.After(time.Second * 1):
			panic("time out")
		case <-done:
			counter++
			if counter == N {
				goto DONE
			}
		}
	}
DONE:
	fmt.Println("Done")
```
[run](https://play.golang.org/p/88J0jUm95v)

* [how-to-wait-for-all-goroutines-to-finish-executing-before-continuing](http://nathanleclaire.com/blog/2014/02/15/how-to-wait-for-all-goroutines-to-finish-executing-before-continuing/)


## #15 - Time interval measurement
> 2016-14-02 by [@beyondns](https://github.com/beyondns)

```go
  start := time.Now()
  DoHardWork() // or time.Sleep(1 * time.Second)
  finish := time.Since(start)
  fmt.Printf("Hard work finished in %d ms.\n", finish.Nanoseconds()/10e6)
``` 

## #14 - Benchmark switch vs else if
> 2016-14-02 by [@beyondns](https://github.com/beyondns)

```go
package boom

import (
	"fmt"
	"math/rand"
	"testing"
)

var words = []string{
	"alpha", "beta", "gamma", "delta", "epsilon", "zeta", "eta", "theta",
	"iota", "kappa", "lambda", "mu", "nu", "xi", "omicron", "pi", "rho",
	"sigma", "tau", "upsilon", "phi", "chi", "psi", "omega",
}
var wlen = len(words)

func BenchmarkSwitch(b *testing.B) {
	m := make(map[string]int)
	rand.Seed(376234242)
	for i := 0; i < b.N; i++ {
		j := rand.Intn(wlen)
		s := words[j]
		switch s {
		case "alpha":
			m[s]++
		case "beta":
			m[s]++
		case "gamma":
			m[s]++
		case "delta":
			m[s]++
		case "epsilon":
			m[s]++
		case "zeta":
			m[s]++
		case "eta":
			m[s]++
		case "theta":
			m[s]++
		case "iota":
			m[s]++
		case "kappa":
			m[s]++
		case "lambda":
			m[s]++
		case "mu":
			m[s]++
		case "nu":
			m[s]++
		case "xi":
			m[s]++
		case "omicron":
			m[s]++
		case "pi":
			m[s]++
		case "rho":
			m[s]++
		case "sigma":
			m[s]++
		case "tau":
			m[s]++
		case "upsilon":
			m[s]++
		case "phi":
			m[s]++
		case "chi":
			m[s]++
		case "psi":
			m[s]++
		case "omega":
			m[s]++
		}
	}
	var c int
	for _,v:=range m{
		c=c+v
	}
	fmt.Printf("%d %d\n", len(m),c)
}

func BenchmarkIf(b *testing.B) {
	m := make(map[string]int)
	rand.Seed(376234242)
	for i := 0; i < b.N; i++ {
		j:= rand.Intn(wlen)
		w := words[j]
		if w == "alpha" {
			m[w]++
		} else if w == "beta" {
			m[w]++
		} else if w == "gamma" {
			m[w]++
		} else if w == "delta" {
			m[w]++
		} else if w == "epsilon" {
			m[w]++
		} else if w == "zeta" {
			m[w]++
		} else if w == "eta" {
			m[w]++
		} else if w == "theta" {
			m[w]++
		} else if w == "iota" {
			m[w]++
		} else if w == "kappa" {
			m[w]++
		} else if w == "lambda" {
			m[w]++
		} else if w == "mu" {
			m[w]++
		} else if w == "nu" {
			m[w]++
		} else if w == "xi" {
			m[w]++
		} else if w == "omicron" {
			m[w]++
		} else if w == "pi" {
			m[w]++
		} else if w == "rho" {
			m[w]++
		} else if w == "sigma" {
			m[w]++
		} else if w == "tau" {
			m[w]++
		} else if w == "upsilon" {
			m[w]++
		} else if w == "phi" {
			m[w]++
		} else if w == "chi" {
			m[w]++
		} else if w == "psi" {
			m[w]++
		} else if w == "omega" {
			m[w]++
		}
	}
	c:=0
	for _,v:=range m{
		c=c+v
	}
	fmt.Printf("%d %d\n", len(m),c)
}

```

```bash
go test -bench=.
```

* [golang.org/pkg/testing/](https://golang.org/pkg/testing/)
* [github.com/golang/go/issues/10000](https://github.com/golang/go/issues/10000)

## #13 - Use ASM in Go Code 
> 2016-10-02 by [@beyondns](https://github.com/beyondns)

Place add.go & add_asm.s in src/add sub folder, then build.

```assembly
// add_asm.s
TEXT add(SB),$0
        MOVL x+0(FP), BX
        MOVL y+4(FP), BP
        ADDL BP, BX
        MOVL BX, ret+8(FP)
        RET
```
```go
// main.go
func add(x, y int32) int32

func main() {
	r:=add(2,5)
	fmt.Println(r)
}
```

```bash
export GOPATH=$PWD 
go build add
```

To play with go/asm code compile
```bash
go tool compile -S code.go > code.s
```

* [golang.org/doc/asm](https://golang.org/doc/asm)
* [goroutines.com/asm](https://goroutines.com/asm)

## #12 - JSON with unknown structure
> 2016-08-02 by [@papercompute](https://github.com/papercompute)

Parse JSON with unknown structure, just pass map[string]interface{} to json.Unmarshal

```go
func ProcessJSON(s string) map[string]interface{} {
	var m map[string]interface{}

	if err := json.Unmarshal([]byte(s), &m); err != nil {
		panic(err)
	}

	if err := passM(m); err != nil {
		panic(err)
	}
	return m
}

func passKV(k string, v interface{}) error {
	var ok bool

	if _, ok = v.(map[string]interface{}); ok {
		err := passM(v.(map[string]interface{}))
		if err != nil {
			return err
		}
	}

	if _, ok = v.([]interface{}); ok {
		err := passA(v.([]interface{}))
		if err != nil {
			return err
		}
	}

	fmt.Printf("%s : %v\n", k, v)

	return nil
}

func passM(m map[string]interface{}) error {
	for k, v := range m {
		if err := passKV(k, v); err != nil {
			return err
		}
	}
	return nil
}

func passA(a []interface{}) error {
	for _, v := range a {
		switch v.(type) {
		case map[string]interface{}:
			if err := passM(v.(map[string]interface{})); err != nil {
				return err
			}
		case []interface{}:
			if err := passA(v.([]interface{})); err != nil {
				return err
			}
		default:
			return errors.New("wrong expression in array")
		}
	}
	return nil
}
```

## #11 - Websocket over HTTP2
> 2016-08-02 by [@beyondns](https://github.com/beyondns)

In go 1.6 HTTP2 is supported out of the box with http.ListenAndServeTLS and can be used with websockets. Just use wss connection. 

```go

func WSServer(ws *websocket.Conn) {
	var err error
	for {
		var reply string

		if err = websocket.Message.Receive(ws, &reply); err != nil {
			fmt.Println("Can't receive")
			break
		}

		fmt.Println("Received back from client: " + reply)

		msg := "Received:  " + reply
		fmt.Println("Sending to client: " + msg)

		if err = websocket.Message.Send(ws, msg); err != nil {
			fmt.Println("Can't send")
			break
		}
	}
}

func main() {
	http.Handle("/ws", websocket.Handler(WSServer))
	http.Handle("/", http.FileServer(http.Dir(".")))
	err := http.ListenAndServeTLS(":8000", "srv.cert", "srv.key", nil)
	if err != nil {
		log.Fatal(err)
	}
}
```

JS Client

```js
<script type="text/javascript">
    function WebSocketTest() {
        if ("WebSocket" in window) {
            console.log("WebSocket is supported by your Browser!");

            var ws = new WebSocket("wss://localhost:8000/ws");

            ws.onopen = function() {
                ws.send("Message from client");
                console.log("Message is sent...");
            };

            ws.onmessage = function(evt) {
                console.log("Message is received...", evt.data);
            };

            ws.onclose = function() {
                console.log("Connection is closed...");
            };
        } else {
            console.log("WebSocket NOT supported by your Browser!");
        }
    }
</script>
```

* [websocket](golang.org/x/net/websocket)


## #10 - HTTP2
> 2016-07-02 by [@beyondns](https://github.com/beyondns)

[Go 1.6's net/http package supports HTTP/2 for both the client and server out of the box.](https://github.com/golang/go/wiki/Go-1.6-release-party)  

In go 1.6+ HTTP2 is supported transparently, just use http.ListenAndServeTLS

Generate key & cert files with openssl
```bash
openssl req -x509 -newkey rsa:2048 -nodes -keyout srv.key -out srv.cert -days 365
```

HTTP2 server
```go
import (
	"fmt"
	"log"
	"flag"
	"net/http"	
	"golang.org/x/net/http2" // optional in go 1.6+
)

func Handler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello http2")
}

func main() {
	var server http.Server

	host := flag.String("h", ":8000", "h host:port")
	flag.BoolVar(&http2.VerboseLogs, "verbose", false, "Verbose HTTP/2 debugging.")
	flag.Parse()

	server.Addr = *host
	http2.ConfigureServer(&server, nil)

	http.HandleFunc("/", Handler)

	log.Println("ListenAndServe on ", *host)

	log.Fatal(server.ListenAndServeTLS("srv.cert", "srv.key"))
}

```

CURL
```bash
curl --http2 --insecure https://localhost:8080
```

HTTP2 client
```go
	host := flag.String("h", "https://127.0.0.1:8000", "h host:port")

	flag.Parse()
	tr := &http.Transport{TLSClientConfig: tlsConfigInsecure}
	defer tr.CloseIdleConnections()

	req, err := http.NewRequest("GET", *host, nil)
	if err != nil {
		panic(err)
	}
	res, err := tr.RoundTrip(req)
	if err != nil {
		panic(err)
	}
	defer res.Body.Close()

	log.Printf("Got res: %+v\n", res)
	if g, w := res.StatusCode, 200; g != w {
		panic(fmt.Sprintf("StatusCode = %v; want %v", g, w))
	}

	data, err := ioutil.ReadAll(res.Body)
	if err != nil {
		panic(fmt.Sprintf("Body read error: %v", err))
	}
	fmt.Printf("%s\n", string(data))

```
* [h2demo](https://github.com/golang/net/blob/master/http2/h2demo)
* [tools-for-debugging-testing-and-using-http-2](https://blog.cloudflare.com/tools-for-debugging-testing-and-using-http-2/)
* [curl-with-http2-support](https://serversforhackers.com/video/curl-with-http2-support)

## #9 - Error handling
> 2016-06-02 by [@beyondns](https://github.com/beyondns)

Use last return variable in function as an error  
Use panic / defer / recover to handle more complex errors

```go

func MyFunc1(v interface{}) (interface{}, error) {
	var ok bool

	if !ok {
		return nil, errors.New("not ok error")
	}
	return v, nil
}

func MyFunc2() {
	defer func() {
		if err := recover(); err != nil {
			// recover from panic
			fmt.Println("recover: ", err)
		}
	}()
	v := struct{}{}
	if _, err := MyFunc1(v); err != nil {
		panic(err) // panic
	}

	fmt.Println("never happen")
}

func main() {
	MyFunc2()
	fmt.Println("main finish")
}

```

## #8 - Memory management with pools of objects
> 2016-04-02 by [@beyondns](https://github.com/beyondns)

Use thread safe pools to collect objects for reuse.

```go
var p sync.Pool
var o *T 
if v := p.Get(); v != nil {
	o = v.(*T)
} else {
	o = new(T)
}

// use o

p.Put(o) // return to reuse

```
* [pool](https://golang.org/src/sync/pool.go)
* [manual memory management](https://github.com/teh-cmc/mmm)

## #7 - Sort slice of time periods
> 2016-02-02 by [@beyondns](https://github.com/beyondns)

For custom data structures it's necessary to use custom compare function to sort elements in slice.

```go

type xTime struct {
	time.Time
}

func (t *xTime) UnmarshalJSON(buf []byte) error {
	tm, err := time.Parse("2006-01-02", strings.Trim(string(buf), `"`))
	if err != nil {
		return err
	}
	t.Time = tm
	return nil
}

type Period struct {
	Start xTime `json:"start"`
	End   xTime `json:"end"`
}

type Data struct {
	Ps []Period `json:"periods"`
}

type Periods []Period

func (a Periods) Len() int           { return len(a) }
func (a Periods) Swap(i, j int)      { a[i], a[j] = a[j], a[i] }
func (a Periods) Less(i, j int) bool { return compDate(a[i].Start, a[j].Start) < 0 }

func compDate(a, b xTime) int {
	ay, am, ad := a.Time.Date()
	by, bm, bd := b.Time.Date()
	if ay > by {
		return 1
	}
	if ay < by {
		return -1
	}

	if am > bm {
		return 1
	}
	if am < bm {
		return -1
	}

	if ad > bd {
		return 1
	}
	if ad < bd {
		return -1
	}

	return 0
}

func main() {

	data := `
{"periods": [
    {"start": "2015-11-01", "end": "2015-11-01"},
    {"start": "2015-12-02", "end": "2015-12-03"},
    {"start": "2015-12-07", "end": "2015-12-13"},
    {"start": "2015-10-18", "end": "2016-10-30"}
  ]}
`
	var d Data
	if err := json.Unmarshal([]byte(data), &d); err != nil {
		panic(err)
	}
	sort.Sort(Periods(d.Ps))
	fmt.Printf("%v", d)

}

```

## #6 - Fast http server
> 2016-01-02 by [@beyondns](https://github.com/beyondns)

If you don't need net/http package functionality just use net and "tcp"

```go

func main() {

	host := flag.String("h", "127.0.0.1:8000", "h host:port")

	flag.Parse()

	l, err := net.Listen("tcp", *host)
	if err != nil {
		fmt.Println("Error listening:", err.Error())
		os.Exit(1)
	}

	defer l.Close()

	fmt.Printf("Listening on %s\n", *host)
	var i = 0
	for {
		conn, err := l.Accept()
		if err != nil {
			continue
		}

		go handleRequest(&conn)
	}

}

func handleRequest(c *net.Conn) {
	defer (*c).Close()
	buf := make([]byte, 2048) // tip: use preallocated
	n, err := (*c).Read(buf)
	if err != nil || n <= 0 {
		return
	}

	// parse http if necessary 

	(*c).Write([]byte("HTTP/1.1 200 Ok\r\nConnection: close\r\nContent-Length: 5\r\n\r\nhello"))
}

```
Just test it

```bash
./wrk -c 1024 -d 20s http://127.0.0.1:8000
```

* [fasthttp](https://github.com/valyala/fasthttp)
* [handling-1-million-requests-per-minute-with-golang](http://marcio.io/2015/07/handling-1-million-requests-per-minute-with-golang/)


## #5 - Close channel to notify many
> 2016-28-01 by [@beyondns](https://github.com/beyondns)

```go
	c:=make(chan int)

	for i:=0;i<5;i++{
		go func(i int){ 
			_, ok :=<-c;
			fmt.Printf("closed %d, %t\n",i,ok) // random order
		}(i)
	}


	// c<-1
	close(c)
	time.Sleep(time.Second)	

```

## #4 - Is channel closed?
> 2016-28-01

```go
	c:=make(chan int)
	go func(){
		if _, ok :=<-c;ok{
			fmt.Println("not closed",<-c)
		} else {
			fmt.Println("closed",<-c)
		}
			
	}()
	// c<-1
	close(c)
	time.Sleep(time.Second)	

```

## #3 - Http request/response with close notify and timeout
> 2016-27-01 by [@beyondns](https://github.com/beyondns)


```go
const RequestMaxWaitTimeInterval = time.Second * 15

func Handler(w http.ResponseWriter, r *http.Request) {
	u:=r.URL.Query().Get("url")
	if u==""{
		http.Error(w, "url is not specified", http.StatusBadRequest)
		return
	}
	var err error
	u,err=url.QueryUnescape(u)
	if err!=nil{
		http.Error(w, fmt.Sprintf("url unescape error %v",err), http.StatusBadRequest)
		return
	}
	closeNotify := w.(http.CloseNotifier).CloseNotify()
	//flusher := w.(http.Flusher)

	tr := &http.Transport{}
	client := &http.Client{Transport: tr}
	c := make(chan error, 1)

	req, err := http.NewRequest("GET", u, nil)
	if err != nil {
		http.Error(w, fmt.Sprintf("error %v", err), http.StatusBadRequest)
		return
	}

	go func() {
		resp, err := client.Do(req)

		defer func() { c <- err }()
		defer func() {
			if resp != nil && resp.Body != nil {
				resp.Body.Close()
			}
		}()

		if err != nil {
			return
		}

		if resp.StatusCode != 200 {
			http.Error(w, fmt.Sprintf("resp.StatusCode %s", resp.Status), resp.StatusCode)
			return
		}

		data, err := ioutil.ReadAll(resp.Body)
		if err != nil {
			http.Error(w, fmt.Sprintf("ioutil.ReadAll, error %v", err), http.StatusBadRequest)
			return
		}

		w.Header().Set("My-custom-header", "my-header-data")
		w.WriteHeader(http.StatusOK)
		w.Write(data)

	}()
	select {
	case <-time.After(RequestMaxWaitTimeInterval):
		tr.CancelRequest(req)
		log.Printf("Request timeout")
		<-c // Wait for goroutine to return.
		return
	case <-closeNotify:
		tr.CancelRequest(req)
		log.Printf("Request canceled")
		<-c // Wait for goroutine to return.
		return
	case err := <-c:
		if err!=nil{
			log.Printf("Error in request goroutine %v",err)
		}
		return
	}
}
```


## #2 - Import packages
> 2016-26-01 by [@beyondns](https://github.com/beyondns)

```go
import "fmt"    // fmt.Print()
import ft "fmt" // ft.Print()
import . "fmt"  // Print()
import _ "fmt"  // not use, but run init
```

## #1 - Map
> 2016-26-01

Map is a hash table
```go
// Set/Get/Test 
m[k]=v
v=m[k]
if _,ok:=m[k];ok{
	// m[k] exists
} 

// Iterate over map, random order
for k,v:=range m{
	// k - key
	// v - value
}

// Function argument
type M map[TK]TV
func f(m M) // m - passed by value, but memory the same 
func f(m *M)// m - passed by reference, but memory the same 

// Double, triple,... map
m := make(map[TK1]map[TK2]TV)
m[K1]=make(map[TK2]TV)
m[K1][K2]=V

// Implemet set with empty struct 
s := make(map[TK]struct{})
s[K]=struct{}{} // set K
delete(s,K)     // unset K
```
* [go-maps-in-action](https://blog.golang.org/go-maps-in-action)

## #0 - Slices
> 2016-24-01 by [@beyondns](https://github.com/beyondns)

Slice is a dynamic array
```golang

// Iterate over slice
for i,v:=range s{ // use range, inc order
	// i - index
	// v - value
} 
for i:=0;i<len(s);i++{ // use index, inc order
	// i - index
	// s[i] - value
} 
for i:=len(s)-1;i>=0;i--{ // use index, reverse order
	// i - index
	// s[i] - value
} 


// Function argument
func f(s []T) // s - passed by value, but memory the same 
func f(s *[]T) // s - passed by refernce, but memory the same 

// Append
a = append(a, b...)

// Clone
b = make([]T, len(a))
copy(b,a)

// Remove element, keep order
a = a[:i+copy(a[i:], a[i+1:])]
// or
a = append(a[:i], a[i+1:]...)

// Remove element, change order
a[i] = a[len(a)-1] 
a = a[:len(a)-1]

```
* [go-slices-usage-and-internals](https://blog.golang.org/go-slices-usage-and-internals)
* [SliceTricks](https://github.com/golang/go/wiki/SliceTricks)

### General Golang links
* [50 Shades of Go: Traps, Gotchas, and Common Mistakes for New Golang Devs](http://devs.cloudimmunity.com/gotchas-and-common-mistakes-in-go-golang/)

### Inspired by
* [jstips](https://github.com/loverajoel/jstips)

### License
[![CC0](http://i.creativecommons.org/p/zero/1.0/88x31.png)](http://creativecommons.org/publicdomain/zero/1.0/)

