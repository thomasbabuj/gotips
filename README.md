![header](https://raw.githubusercontent.com/beyondns/gotips/master/gophers.jpg)

> **Golang short tips & trics**

This list of short golang code tips & trics will help keep collected knowledge in one place. Do not hesitate to pull request new ones, just add new tip on top of list with title, date, description and code, please see tips as a reference.


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

