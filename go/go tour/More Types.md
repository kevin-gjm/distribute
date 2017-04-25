#### Pointers

​    Go has pointers.    A pointer holds the memory address of a value.  

​    The type `*T` is a pointer to a `T` value. Its zero value is `nil`.  

```go
var p *int
```

​    The `&` operator generates a pointer to its operand.  

```go
i := 42
p = &i
```

​    The `*` operator denotes the pointer's underlying value.  

```go
fmt.Println(*p) // read i through the pointer p
*p = 21         // set i through the pointer p
```

​    This is known as "dereferencing" or "indirecting".  

​    Unlike C, ***Go has no pointer arithmetic.***   没有指针的算术运算

## Structs

​    A `struct` is a collection of fields.  

​    (And a `type` declaration does what you'd expect.)  

```go
type Vertex struct {
	X int
	Y int
}
```

Struct fields are accessed using a dot.

```go
	v := Vertex{1, 2}
	v.X = 4
```

#### Pointers to structs

​    Struct fields can be accessed through a struct pointer.  

​    To access the field `X` of a struct when we have the struct pointer `p` we could    write `(*p).X`.    However, that notation is cumbersome, so the language permits us instead  to    write just `p.X`, without the explicit dereference.  

```go
	v := Vertex{1, 2}
	p := &v
	(*p).Y=10
	p.X = 1e9
```

#### Struct Literals

​    A struct literal denotes a newly allocated struct value by listing the values of its fields.  

​    You can list just a subset of fields by using the `Name:` syntax. (And the order of named fields is irrelevant.)  

​    The special prefix `&` returns a pointer to the struct value.  

```go
var (
	v1 = Vertex{1, 2}  // has type Vertex
	v2 = Vertex{X: 1}  // Y:0 is implicit
	v3 = Vertex{}      // X:0 and Y:0
	p  = &Vertex{1, 2} // has type *Vertex
)
```

#### Arrays

​    The type `[n]T` is an array of `n` values of type `T`.  

​    The expression  

```go
var a [10]int
```

​    declares a variable `a` as an array of ten integers.  

​    An array's length is part of its type, so arrays cannot be resized.    This seems limiting, but don't worry;    Go provides a convenient way of working with arrays.  

```go
var a [2]string
primes := [6]int{2, 3, 5, 7, 11, 13}
```

#### Slices

​    An array has a fixed size.    A slice, on the other hand, is a dynamically-sized,    flexible view into the elements of an array.    In practice, slices are much more common than arrays.  

​    The type `[]T` is a slice with elements of type `T`.  

​    This expression creates a slice of the first five elements of the array `a`:  

```
a[0:5]
```

```go
primes := [6]int{2, 3, 5, 7, 11, 13}

var s []int = primes[1:4]  //前闭后开
```

#### Slices are like references to arrays

​    A slice does not store any data,    it just describes a section of an underlying array.  

​    Changing the elements of a slice ***modifies the    corresponding elements of its underlying array.***  

​    Other slices that share the same underlying array will see those changes.  

```go
var t []string
t[0] ="hello" //这样使用时错误的：runtime error: index out of range
```

#### Slice literals

​    A slice literal is like an array literal(常量) without the length.  

​    This is an array literal:  

```
[3]bool{true, true, false}
```

​    And this creates the same array as above,    then builds a slice that references it:  

```
[]bool{true, true, false}
```

```go
var s []int=[]int{1,2,3,4}
```



### Slice defaults

​    When slicing, you may ***omit the high or low bounds to use their defaults instead***.  

​    The default is zero for the low bound and the length of the slice for the high bound.  

​    For the array  

```
var a [10]int
```

​    these slice expressions are equivalent:  

```
a[0:10]
a[:10]
a[0:]
a[:]
```

#### Slice length and capacity

​    A slice has both a *length* and a *capacity*.  

​    The length of a slice is the number of elements it contains.  

​    The capacity of a slice ***is the number of elements in the underlying array,    counting from the first element in the slice.***  

​    The length and capacity of a slice `s` can be obtained using the expressions    `len(s)` and `cap(s)`.  

​    You can extend a slice's length by re-slicing it,    provided it has sufficient capacity.    Try changing one of the slice operations in the example program to extend it    beyond its capacity and see what happens.  

```go
import "fmt"

func main() {
	s := []int{2, 3, 5, 7, 11, 13}
	printSlice(s)

	// Slice the slice to give it zero length.
	s = s[:0]
	printSlice(s)

	// Extend its length.
	s = s[0:4]
	printSlice(s)
	
	//s = s[:0]
	//printSlice(s)
	
	// Drop its first two values.
	s = s[2:]
	printSlice(s)  //  cap = 4 counting from the first element in the slice
}

func printSlice(s []int) {
	fmt.Printf("len=%d cap=%d %v\n", len(s), cap(s), s)
}
```

#### Nil slices

​    The zero value of a slice is `nil`.  

​    A nil slice has a length and capacity of 0    and has no underlying array.  

#### Creating a slice with make

​    Slices can be created with the built-in `make` function;    this is how you create dynamically-sized arrays.  

​    The `make` function allocates a zeroed array    and returns a slice that refers to that array:  

```
a := make([]int, 5)  // len(a)=5
```

​    To specify a capacity, pass a third argument to `make`:  

```go
b := make([]int, 0, 5) // len(b)=0, cap(b)=5

b = b[:cap(b)] // len(b)=5, cap(b)=5
b = b[1:]      // len(b)=4, cap(b)=4   //why cap ????
```

#### Slices of slices

​    Slices can contain any type, including other slices.  

#### Appending to a slice

​    It is common to append new elements to a slice, and so Go provides a built-in    `append` function. The [documentation](https://golang.org/pkg/builtin/#append)    of the built-in package describes `append`.  

```
func append(s []T, vs ...T) []T
```

​    The first parameter `s` of `append` is a slice of type `T`, and the rest are    `T` values to append to the slice.  

​    The resulting value of `append` is a slice containing all the elements of the    original slice plus the provided values.  

​    If the backing array of `s` is ***too small to fit all the given values a bigger    array will be allocated***. The returned slice will point to the newly allocated    array.  

#### Range

​    The `range` form of the `for` loop iterates over a slice or map.  

​    When ranging over a slice, two values are returned for each iteration.    ***The first is the index, and the second is a copy of the element at that index.***  

```go
for i, v := range pow {
		fmt.Printf("2**%d = %d\n", i, v)
	}
```

​    You can skip the index or value by assigning to `_`.  

​    If you only want the index, drop the ", value" entirely.  

```go
pow := make([]int, 10)
	for i := range pow {
		pow[i] = 1 << uint(i) // == 2**i
	}
	for _, value := range pow {
		fmt.Printf("%d\n", value)
	}
```

#### Maps

​    A map maps keys to values.  

​    The zero value of a map is `nil`.    A `nil` map has no keys, nor can keys be added.  

​    The `make` function returns a map of the given type,    initialized and ready for use.  

```
map[key]type  ---> var m map[string]int
```

```go
type Vertex struct {
	Lat, Long float64
}

var m map[string]Vertex

func main() {
	m = make(map[string]Vertex) // if not have this m will be nil
	m["Bell Labs"] = Vertex{
		40.68433, -74.39967,
	}
	fmt.Println(m["Bell Labs"])
}
```

#### Map literals

​    Map literals are like struct literals, but the keys are required.  

```go
type Vertex struct {
	Lat, Long float64
}

var m = map[string]Vertex{
	"Bell Labs": Vertex{
		40.68433, -74.39967,
	},
	"Google": Vertex{
		37.42202, -122.08408,
	},
}
```

If the top-level type is just a type name, you can omit it from the elements of the literal.

```go
var m = map[string]Vertex{
	"Bell Labs": {40.68433, -74.39967},
	"Google":    {37.42202, -122.08408},
}
```

#### Mutating Maps

​    Insert or update an element in map `m`:  

```
m[key] = elem
```

​    Retrieve an element:  

```
elem = m[key]
```

​    Delete an element:  

```
delete(m, key)
```

​    Test that a key is present with a two-value assignment:  

```
elem, ok = m[key]
```

​    If `key` is in `m`, `ok` is `true`. If not, `ok` is `false`.  

​    If `key` is not in the map, then `elem` is the zero value for the map's element type.  

​    *Note*: if `elem` or `ok` have not yet been declared you could ***use a short declaration form***:  

```
elem, ok := m[key]
```

#### Function values

​    Functions are values too. They can be passed around just like other values.  

​    Function values may be used as function arguments and return values.  

```go
package main

import (
	"fmt"
	"math"
)

func compute(fn func(float64, float64) float64) float64 {
	return fn(3, 4)
}

func main() {
	hypot := func(x, y float64) float64 {
		return math.Sqrt(x*x + y*y)
	}
	fmt.Println(hypot(5, 12))

	fmt.Println(compute(hypot))
	fmt.Println(compute(math.Pow))
}
```

#### Function closures

​    Go functions may be closures. A closure is a function value that references variables from outside its body. The function may access and assign to the referenced variables; in this sense the function is "bound" to the variables.  

​    For example, the `adder` function returns a closure. Each closure is bound to its own `sum` variable.  

​    Go 函数可以是一个闭包。闭包是一个函数值，它引用了函数体之外的变量。    这个函数可以对这个引用的变量进行访问和赋值；换句话说这个函数被“绑定”在这个变量上。  

​    例如，函数 `adder` 返回一个闭包。每个返回的闭包都被绑定到其各自的 `sum` 变量上。  

```go
package main

import "fmt"

func adder() func(int) int { //每一次返回sum都是返回的一部分
	sum := 0
	return func(x int) int {
		sum += x
		return sum
	}
}

func main() {
	pos, neg := adder(), adder()
	for i := 0; i < 10; i++ {
		fmt.Println(
			pos(i),
			neg(-2*i),
		)
	}
}
```

