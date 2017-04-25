#### Methods

​    Go does not have classes.    However, ***you can define methods on types.***  （like an method of class）

​    A method is a function with a special *receiver* argument.  

​    The ***receiver appears in its own argument list between the `func` keyword and    the method name.***  

​    In this example, the `Abs` method has a receiver of type `Vertex` named `v`.  

```go
type Vertex struct {
	X, Y float64
}

func (v Vertex) Abs() float64 {   //类似给类Vertex定义了一个方法
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}
```

#### Methods are functions

Remember: a method is just a function with a receiver argument.  

​    Here's `Abs` written as a regular function with no change in functionality.  

```go
func Abs(v Vertex) float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}
```

​    You can declare a method on non-struct types, too.  

​    In this example we see a numeric type `MyFloat` with an `Abs` method.  

​    You ***can only declare a method with a receiver whose type is defined in the same    package*** as the method.    You cannot declare a method with a receiver whose type is defined in another    package (which includes the built-in types such as `int`).  

```go
type MyFloat float64

func (f MyFloat) Abs() float64 {
	if f < 0 {
		return float64(-f)
	}
	return float64(f)
}

```

## Pointer receivers

​    You can declare methods with pointer receivers.  

​    This means the receiver type has the literal syntax `*T` for some type `T`.    (Also, `T` cannot itself be a pointer such as `*int`.)  

​    For example, the `Scale` method here is defined on `*Vertex`.  

​    Methods with pointer receivers can modify the value to which the receiver    points (as `Scale` does here).    ***Since methods often need to modify their receiver, pointer receivers are more    common than value receivers***.  (可以改变指针所指内容)

​    Try removing the `*` from the declaration of the `Scale` function on line 16    and observe how the program's behavior changes.  

​    ***With a value receiver, the `Scale` method operates on a copy of the original***    `Vertex` value.    (This is the same behavior as for any other function argument.)    The `Scale` method must have a pointer receiver to change the `Vertex` value    declared in the `main` function.  

## Methods and pointer indirection

​    Comparing the previous two programs, you might notice that    ***functions with a pointer argument must take a pointer***:  

```
var v Vertex
ScaleFunc(v)  // Compile error!
ScaleFunc(&v) // OK
```

​    ***while methods with pointer receivers take either a value or a pointer as the    receiver when they are called:***  

```
var v Vertex
v.Scale(5)  // OK
p := &v
p.Scale(10) // OK
```

​    For the statement `v.Scale(5)`, even though `v` is a value and not a pointer,    the method with the pointer receiver is called automatically.    That is, as a convenience, Go interprets the statement `v.Scale(5)` as    `(&v).Scale(5)` since the `Scale` method has a pointer receiver.  

​    The equivalent thing happens in the reverse direction.  

​    Functions that take a value argument must take a value of that specific type:  

```
var v Vertex
fmt.Println(AbsFunc(v))  // OK
fmt.Println(AbsFunc(&v)) // Compile error!
```

​    ***while methods with value receivers take either a value or a pointer as the    receiver when they are called:***  

```
var v Vertex
fmt.Println(v.Abs()) // OK
p := &v
fmt.Println(p.Abs()) // OK
```

​    In this case, the method call `p.Abs()` is interpreted as `(*p).Abs()`.  

#### Choosing a value or pointer receiver

​    There are two reasons to use a pointer receiver.  

​    ***The first is so that the method can modify the value that its receiver points to.***  

​    ***The second is to avoid copying the value on each method call.***    This can be more efficient if the receiver is a large struct, for example.  

​    In this example, both `Scale` and `Abs` are with receiver type `*Vertex`,    even though the `Abs` method needn't modify its receiver.  

​    In general, all methods on a given type ***should have either value or pointer    receivers, but not a mixture of both.***    (We'll see why over the next few pages.)  （需要有一个但是不能两个都有--->redeclared）

#### Interfaces

​    An *interface type* is defined as a set of method signatures.  

​    A value of interface type can hold any value that implements those methods.  

#### Interfaces are implemented implicitly

​    A type implements an interface by implementing its methods.    There is no explicit declaration of intent, no "implements" keyword.  

​    Implicit interfaces decouple the definition of an interface from its    implementation, which could then appear in any package without prearrangement.  

```go
package main

import "fmt"

type I interface {
	M()
}

type T struct {
	S string
}

// This method means type T implements the interface I,
// but we don't need to explicitly declare that it does so.
func (t T) M() {
	fmt.Println(t.S)
}

func main() {
	var i I = T{"hello"}
	i.M()
}
```

#### Interface values

​    Under the covers, interface values can be thought of as a tuple of a value and a    concrete type:  

```
(value, type)
```

​    An interface value holds a value of a specific underlying concrete type.  

​    Calling a method on an interface value executes the method of the same name on    its underlying type.

  ```go
type I interface {
	M()
}
var i I
fmt.Printf("(%v, %T)\n", i, i) //return (<nil>,<nil>) not init.
i.M()  //------>runtime error: invalid memory address or nil pointer dereference
  ```

#### Interface values with nil underlying values

​    If the concrete value inside the interface itself is nil,    the method will be called with a nil receiver.  

​    In some languages ***this would trigger a null pointer exception***,    but in Go it is common to write methods that gracefully handle being called    with a nil receiver (as with the method `M` in this example.)  

​    Note that an interface value that holds a nil concrete value is itself non-nil.  

```go
type I interface {
	M()
}

type T struct {
	S string
}

func (t *T) M() {
	if t == nil {
		fmt.Println("<nil>")
		return
	}
	fmt.Println(t.S)
}
func main() {
	var i I
	var t *T
	i = t
  	i.M() //nil
}
```

#### Nil interface values

​    A nil interface value holds neither value nor concrete type.  

​    Calling a method on a nil interface is a run-time error because there is no    type inside the interface tuple to indicate which *concrete* method to call.  (before runtime error)

#### The empty interface

​    The interface type that specifies zero methods is known as the *empty interface*:  

```
interface{}
```

​    An empty interface may hold values of any type.    (Every type implements at least zero methods.)  

​    ***Empty interfaces are used by code that handles values of unknown type***.    For example, `fmt.Print` takes any number of arguments of type `interface{}`.  (like c void* can hold all pointer type)

```go
func main() {
	var i interface{}
	describe(i)
	i = 42
	describe(i)
	i = "hello"
	describe(i)
}

func describe(i interface{}) {
	fmt.Printf("(%v, %T)\n", i, i)
}
	
```

#### Type assertions

​    A *type assertion* provides access to an interface value's underlying concrete value.  

```
t := i.(T)
```

​    This statement asserts that the interface value `i` holds the concrete type `T`    and assigns the underlying `T` value to the variable `t`.  

​    ***If `i` does not hold a `T`, the statement will trigger a panic.***  

​    To *test* whether an interface value holds a specific type,    a type assertion can return two values: the underlying value    and a boolean value that reports whether the assertion succeeded.  

```
t, ok := i.(T)
```

​    If `i` holds a `T`, then `t` will be the underlying value and `ok` will be true.  

​    If not, `ok` will be false and `t` will be the zero value of type `T`,    and no panic occurs.  

​    Note the similarity between this syntax and that of reading from a map.  

#### Type switches

​    A *type switch* is a construct that permits several type assertions in series.  

​    A type switch is like a regular switch statement, but the cases in a type    switch specify types (not values), and those values are compared against    the type of the value held by the given interface value.  

```
switch v := i.(type) {
case T:
    // here v has type T
case S:
    // here v has type S
default:
    // no match; here v has the same type as i
}
```

​    ***The declaration in a type switch has the same syntax as a type assertion `i.(T)`,    but the specific type `T` is replaced with the keyword `type`***.  (指定关键字type返回类型)

​    This switch statement tests whether the interface value `i`    holds a value of type `T` or `S`.    In each of the `T` and `S` cases, the variable `v` will be of type    `T` or `S` respectively and hold the value held by `i`.    In the default case (where there is no match), the variable `v` is    of the same interface type and value as `i`.  

#### Stringers

​    One of the most ubiquitous interfaces is [`Stringer`](https://golang.org/pkg/fmt/#Stringer) defined by the [`fmt`](https://golang.org/pkg/fmt/) package.  

```go
type Stringer interface {
    String() string
}
```

​    A `Stringer` is a type that can describe itself as a string. The `fmt` package    (and many others) look for this interface to print values.  

```go
package main

import "fmt"

type Person struct {
	Name string
	Age  int
}

func (p Person) String() string {
	return fmt.Sprintf("%v (%v years)", p.Name, p.Age) //use Sprintf here !!!!
}

func main() {
	a := Person{"Arthur Dent", 42}
	z := Person{"Zaphod Beeblebrox", 9001}
	fmt.Println(a, z) //会默认调用其String()接口，
}
```

#### Errors

​    Go programs express error state with `error` values.   

​    The `error` type is a built-in interface similar to `fmt.Stringer`:   (和上面的String接口一样，需要时自己进行定义)

```
type error interface {
    Error() string
}
```

​    (As with `fmt.Stringer`, the `fmt` package looks for the `error` interface when    printing values.)  

​    Functions often return an `error` value, and calling code should handle errors    by testing whether the error equals `nil`.  

```
i, err := strconv.Atoi("42")
if err != nil {
    fmt.Printf("couldn't convert number: %v\n", err)
    return
}
fmt.Println("Converted integer:", i)
```

​    A nil `error` denotes success; a non-nil `error` denotes failure.  

#### Readers

​    The `io` package specifies the `io.Reader` interface,    which represents the read end of a stream of data.  

​    The Go standard library contains [many implementations](https://golang.org/search?q=Read#Global) of these interfaces, including files, network connections, compressors, ciphers, and others.  

​    The `io.Reader` interface has a `Read` method:  

```
func (T) Read(b []byte) (n int, err error)
```

​    `Read` populates ***the given byte slice with data*** and returns the number of bytes    populated and an error value. It returns an `io.EOF` error when the stream    ends.  

​    The example code creates a     [`strings.Reader`](https://golang.org/pkg/strings/#Reader)    and consumes its output 8 bytes at a time.  

```go
package main

import (
	"fmt"
	"io"
	"strings"
)

func main() {
	r := strings.NewReader("Hello, Reader!")

	//var b []byte  // will --->process took too long
	b := make([]byte, 8)
	for {
		n, err := r.Read(b)
		fmt.Printf("n = %v err = %v b = %v\n", n, err, b)
		fmt.Printf("b[:n] = %q\n", b[:n])
		if err == io.EOF {
			break
		}
	}
}
```

```go
n = 8 err = <nil> b = [72 101 108 108 111 44 32 82]
b[:n] = "Hello, R"
n = 6 err = <nil> b = [101 97 100 101 114 33 32 82]
b[:n] = "eader!"
n = 0 err = EOF b = [101 97 100 101 114 33 32 82]
b[:n] = ""
```

