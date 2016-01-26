![header](https://raw.githubusercontent.com/beyondns/gotips/master/gophers.jpg)

> **Golang short tips & trics**

This list of short golang code tips & trics will help keep collected knowledge in one place. Do not hesitate to pull request new ones, just add new tip on top of list with title, date, description and code, please see tip #0 as a reference.

## #2 - Import packages
> 2016-26-01

```golang
import "fmt"    // fmt.Print()
import ft "fmt" // ft.Print()
import . "fmt"  // Print()
import _ "fmt"  // not use, but run init

```

## #1 - Map
> 2016-26-01

Map is a key-value storage
```golang
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

* [SliceTricks](https://github.com/golang/go/wiki/SliceTricks)

### Links
* [50 Shades of Go: Traps, Gotchas, and Common Mistakes for New Golang Devs](http://devs.cloudimmunity.com/gotchas-and-common-mistakes-in-go-golang/)

### Inspired by


### License
[![CC0](http://i.creativecommons.org/p/zero/1.0/88x31.png)](http://creativecommons.org/publicdomain/zero/1.0/)

