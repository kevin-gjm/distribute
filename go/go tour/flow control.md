#### For

​    Go has only one looping construct, the `for` loop.  

​    The basic `for` loop has three components separated by semicolons:  

- the init statement: executed before the first iteration
- the condition expression: evaluated before every iteration
- the post statement: executed at the end of every iteration

​    The init statement will often be a short variable declaration, and the    variables declared there are ***visible only in the scope of the `for`    statement***.  

​    The loop will stop iterating once the boolean condition evaluates to `false`.  

​    *Note*: Unlike other languages like C, Java, or Javascript there are no parentheses    surrounding the three components of the `for` statement and the braces `{ }` are    ***always required.***  

```go
for i := 0; i < 10; i++ {
		sum += i
	}
```

The init and post statement are optional. 

```go
for ; sum < 1000; {
		sum += sum
	}
```

#### For is Go's "while"

```go
for sum < 1000 {
		sum += sum
	}
```

#### if

Go's `if` statements are like its `for` loops; the expression _need not be surrounded by parentheses_ `( )` but ***the braces `{ }` are required***.

```go
if x< 10 {
  return false
}
```

#### If with a short statement

​    Like `for`, the `if` statement can start with a short statement to execute before the condition.  

​    Variables declared by the statement ***are only in scope*** until the end of the `if`.  

```go
if v:= math.Pow(x,n); v<lim {
  return v
}else{  //not spearate
  return -1
}
```

#### If and else

​    Variables declared inside an `if` short statement are also available inside any    of the `else` blocks.  

#### Switch

​    You probably knew what `switch` was going to look like.  

​    ***A case body breaks automatically, unless it ends with a `fallthrough` statement.***  

```go
switch os := runtime.GOOS; os {
	case "darwin":
		fmt.Println("OS X.")
```

#### Switch evaluation order

​    Switch cases evaluate cases from top to bottom, ***stopping when a case succeeds.***  

#### Switch with no condition

​    Switch without a condition is the same as `switch true`.  

​    This construct can be a clean way to write long if-then-else chains.  

```go
t := time.Now()
	switch {
	case t.Hour() < 12:
		fmt.Println("Good morning!")
	case t.Hour() < 17:
		fmt.Println("Good afternoon.")
	default:
		fmt.Println("Good evening.")
	}
```

#### Defer

​    A defer statement defers the execution of a function until the surrounding    function returns.  

​    The deferred call's ***arguments are evaluated immediately***, but the function call    is not executed until the surrounding function returns.  

#### Stacking defers

​    Deferred function calls are pushed onto a stack. When a function returns, its    deferred calls are executed in last-in-first-out order.  

```go
for i := 0; i < 10; i++ {
		defer fmt.Println(i)
	}
```

