![header](https://raw.githubusercontent.com/beyondns/gotips/master/gophers.jpg)

> **Golang short tips & trics**

This list of short golang code tips & trics will help keep collected knowledge in one place. Do not hesitate to pull request new ones, just add new tip on top of list with title, date, description and code, please see tips as a reference.

## #8 - Memory management with pools of objects
> 2016-04-02

Use pools to collect objects for reuse.

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

* [manual memory management](https://github.com/teh-cmc/mmm)

## #7 - Sort slice of time periods
> 2016-02-02

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
	}
	sort.Sort(Periods(d.Ps))
	fmt.Printf("%v", d)

}

```

## #6 - Fast http server
> 2016-01-02

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
> 2016-28-01

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
> 2016-27-01


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
> 2016-26-01

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
> 2016-24-01

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

### Links
* [50 Shades of Go: Traps, Gotchas, and Common Mistakes for New Golang Devs](http://devs.cloudimmunity.com/gotchas-and-common-mistakes-in-go-golang/)

### Inspired by
[jstips](https://github.com/loverajoel/jstips)

### License
[![CC0](http://i.creativecommons.org/p/zero/1.0/88x31.png)](http://creativecommons.org/publicdomain/zero/1.0/)

