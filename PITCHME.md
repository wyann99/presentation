# Go in Depth

Dong Bin

---?image=assets/golang-logo.png

---

# Standard Type
* Array
* Slice
* String
* Map
* Interface

---
# Array
Fixed size sequential collection of elements
```go
	
var buffer [256]byte
intSet := [6]int{1, 2, 3, 5}
days := [...]string{"Sat", "Sun"} //len(days) == 2
```
---
# Slice
Reference to a part of array|
```C
// $GOROOT/src/pkg/runtime/runtime.h
struct Slice
{    // must not move anything
    byte*    array;        // actual data
    uintgo    len;        // number of elements
    uintgo    cap;        // allocated number of elements
};
```