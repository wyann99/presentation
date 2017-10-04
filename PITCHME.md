# Go in Depth

Dong Bin

![Logo](assets/golang-logo.png)

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
---?image=assets/slice.png

Note:
Slice include a pointer, length and capacity
---
# Slice
Reference to a part of array
```go
type SliceHeader struct {
        Data uintptr
        Len  int
        Cap  int
}
  var s1 []int
  var s2 = make([]int, 10)
  var s3 = make([]int, 10, 20)
  fmt.Printf("len:%d, cap:%d\n", len(s1), cap(s1))
  fmt.Printf("len:%d, cap:%d\n", len(s2), cap(s2))
  fmt.Printf("len:%d, cap:%d\n", len(s3), cap(s3))
```
@[1-5]
@[6-11]

---

# Slice Append
```go
	array := [3]int{10,20,30}
	s := array[:]
	fmt.Printf("slice value: %v, array value: %v \n", s, array)
	s2 := append(s, 40)
	fmt.Printf("slice value: %v, array value: %v \n", s2, array)

	s = array[:2]
	fmt.Printf("slice value: %v, array value: %v \n", s, array)
	s2 = append(s, 40)
	fmt.Printf("slice value: %v, array value: %v \n", s2, array)
```
@[1-5](Append will create a new array and append value to the end)
@[7-10](Append will modify the existing array)

---
# String
```go
type StringHeader struct {
        Data uintptr
        Len  int
}
s := "hello world"
b := []byte(s)
s = string(b)
```
@[1-4](String structure, Compare to slice, string has no cap)
@[5-7](Convertion between string and byte[])
Note:
Notice string invariant
