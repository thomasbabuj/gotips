![header](https://raw.githubusercontent.com/beyondns/gotips/master/gophers.jpg)

> **Golang short tips & trics**

This list of short golang code tips & trics will help keep collected knowledge in one place. Do not hesitate to pull request new ones, just add new tip on top of list with title, date, description and code, please see tip #0 as a reference.

## #0 - Slices
> 2016-24-01

```golang
// Append
a = append(a, b...)

// Clone
b = make([]T, len(a))
copy(b,a)

// Remove element, keep order
a = a[:i+copy(a[i:], a[i+1:])]

// Remove element, change order
a[i] = a[len(a)-1] 
a = a[:len(a)-1]

```


### License
[![CC0](http://i.creativecommons.org/p/zero/1.0/88x31.png)](http://creativecommons.org/publicdomain/zero/1.0/)

