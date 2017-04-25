### Go语言核心之美阅读笔记1

#### 变量可见性

1. 声明在函数内部，是函数的本地值，类似private
2. 声明在函数外部，是对当前包可见(包内所有.go文件都可见)的全局值，类似protect
3. 声明在函数外部且首字母大写是所有包可见的全局值,类似public

#### 命名方法

建议使用驼峰命名法：**第一个单词以小写字母开始，第二个单词的首字母大写或每一个单词的首字母都采用大写字母**

缩写短语要么全部大写，要么全部小写

#### 声明

Go的程序是保存在多个.go文件中，文件的第一行就是package XXX声明，用来说明该文件属于哪个包(package)，package声明下来就是import声明，再下来是无关吮吸的类型，变量，常量，函数的声明

#### 初始化

1. 变量也可以通过函数的返回值来初始化

   ```go
   var f, err = os.Open(name) // os.Open returns a file and an error  
   ```

2. 就像var声明一样，短声明也可以并行初始化.

   ```go
   i, j := 0, 1
   ```

3. 要谨记的是，:=是一个声明，=是一个赋值，因此在需要赋值的场所不能使用 :=

4. 可以利用并行赋值的特性来进行值交换

   ```go
   i, j = j, i // swap values of i and j  
   ```

   ​

5. 指针值是一个变量的存储地址。注意：不是所有的值都有地址，但是变量肯定是有地址的！这个概念一定要搞清楚！  通过指针，我们可以间接的去访问一个变量，甚至不需要知道变量名。

6. 在函数中返回一个本地变量的地址是很安全的。例如以下代码，本地变量v是在f中创建的，从f返回后依然会存在，指针p仍然会去引用v

   ```go
   var p = f()  
   fmt.Println(*p) //output:1  
   func f() *int {  
      v := 1  
      return &v  
   }  
   ```

   ​	那么GC是怎么判断一个变量应该被回收呢？完整的机制是很复杂的，但是基本的思想是寻找每个变量的过程路径，如果找不到这样的路径，那么变量就是**不可到达**的，因此就是可以被回收的。

   ​	一个变量的生命期只取决于变量是否是可到达的，因此一个本地变量可以在循环之外依然存活，甚至可以在函数return后依然存活。编译器会选择在堆上或者栈上去分配变量，但是请记住：编译器的选择并不是由var或者new这样的声明方式决定的。

   ```go
       var global *int  
         
       func f() {                      func g() {  
           var x int                       y := new(int)  
           x = 1                           *y = 1  
           global = &x                 }  
       }  
   ```

   ​	上面代码中，x是在**堆上**分配的变量，因为在f返回后，x也是可到达的(global指针)。这里x是f的本地变量，因此，这里我们说x从f中**逃逸**了。相反，当g返回时，变量*y就变为不可到达的，然后会被垃圾回收。因为*y没有从g中逃逸，所以编译器将*y分配在栈上(即使是用new分配的)。在绝大多数情况下，我们都不用担心变量逃逸的问题，只要在做性能优化时意识到：每一个逃逸的变量都需要进行一次额外的内存分配。

   ​	尽管自动GC对于写现代化的程序来说，是一个巨大的帮助，但是我们也要理解go语言的内存机制。程序不需要显式的内存分配或者回收，可是为了写出高效的程序，我们仍然需要清楚的知道变量的生命期。例如，在长期对象(特别是全局变量)中持有指向短期对象的指针，会阻止GC回收这些短期对象，因为在这种情况下，短期对象是可以到达的！！

7. go的函数默认是值传递，所有的值类型都会进行内存拷贝

8. 指针在flag包中是很重要的。flag会读取程序命令行的参数，然后设置程序内部的变量。下面的例子中，我们有两个命令行参数：-n，不打印换行符；-s sep，使用自定义的字符串分隔符进行打印.

   ```go
   package main  
     
   import (  
       "flag"  
       "fmt"  
       "strings"  
   )  
     
   var n = flag.Bool("n", false, "忽略换行符")  
   var sep = flag.String("s", " ", "分隔符")  
     
   func main() {  
       flag.Parse()  
       fmt.Print(strings.Join(flag.Args(), *sep))  
       if !*n {  
             fmt.Println()  
       }  
   } 
   ```

   flag.Bool会创建一个bool类型的flag变量，flag.Bool有三个参数：flag的名字，命令行没有传值时默认的flag值(false),flag的描述信息( 当用户传入一个非法的参数或者-h、 -help时，会打印该描述信息)。变量sep和n 都是flag变量的指针，因此要通过*sep和*n来访问原始的flag值。

   ​	当程序运行时，在使用flag值之前首先要调用flag.Parse。非flag参数可以通过args := flag.Args()来访问，args的类型是[]string(见后续章节)。如果flag.Parse报错，那么程序就会打印出一个使用说明，然后调用os.Exit(2)来结束。

9. 还可以通过内建(built-in)函数new来创建变量。new(T)会初始化一个类型为T的变量，值为类型T对应的零值，然后返回一个指针：*T。

10. 因为new是预定义的函数名(参见上一节的保留字),不是语言关键字，因此可以用new做函数内的变量名

#### 赋值及类型声明

1. 元组(tuple) 赋值

   ```go
   x, y = y, x  
   a[i], a[j] = a[j], a[i]  
   x, y = y, x%y  
   ```

   但是如果表达式较为复杂时，应该尽量避免元组赋值，分开赋值的可读性会更好.

   函数会利用这种多返回值特性来额外返回一个值

2. 使用type就可以声明具名类型，这样就可以在底层类型之上构建自己的需要的时间戳，文件描述符等类型

   为了解释类型声明，这里把不同的温度计量单位设置为不同的类型：

   ```go
   package tempconv  
     
   import "fmt"  
     
   type Celsius float64  
   type Fahrenheit float64  
     
   const (  
       AbsoluteZeroC Celsius = -273.15  
       FreezingC     Celsius = 0  
       BoilingC      Celsius = 100  
   )  
     
   func CToF(c Celsius) Fahrenheit { return Fahrenheit(c*9/5 + 32) }  
     
   func FToC(f Fahrenheit) Celsius { return Celsius((f - 32) * 5 / 9) }
   ```

   上面定义了两个类型Celsius(摄氏度)和Fahrenheit(华氏度)，作为温度的两种不同单位。***即使两个类型的底层类型是相同的float64，但是它们是完全不同的类型，所以它们彼此之间不能比较，也不能在算术表达式中结合使用，这种规则可以避免错误的使用两种不同的温度单位***，因为两个温度单位的类型是不同的，CToF和FToC返回的两个值也是完全不同的。

3. 具名类型的底层类型决定了它的结构和表现形式，也决定了它支持的基本操作(可以理解为继承自底层类型),就好像直接使用底层类型一样。因此对于Celsius和Fahrenheit来说，float64支持的算术操作，它们都支持

4. 如果两个值有同样的具名类型，那么就可以用比较操作符==和<进行比较；或者两个值，一个是具名类型，一个是具名类型的底层类型，也可以进行比较。但是两个不同的具名类型是不可以直接比较的

#### 包和文件

1. Go语言中的包(Package)就像其它语言的库(Library)或模块(Module)一样，支持模块化，封装性，可重用性，单独编译等特点。包的源码是由数个.go文件组成，这些文件所在的目录名是import路径的最后一个词

2. 每个包都有独立的命名空间。例如，在image包中的Decode和unicode/utf16中的Decode是完全不同的函数。如果要引用第三方库的函数，我们要使用package.Func的形式，例如image.Decode和utf16.Decode。

3. 包也允许我们自己控制包内变量、函数的可见性。在Go语言中，变量、函数等的导出只取决于一个因素：名字首字母的大小写。

   例子：

   ```go
   // temconv.go
   	package tempconv  
         
       import "fmt"  
         
       type Celsius float64  
       type Fahrenheit float64  
         
       const (  
           AbsoluteZeroC Celsius = -273.15  
           FreezingC     Celsius = 0  
           BoilingC      Celsius = 100  
       )  
         
       func (c Celsius) String() string    { return fmt.Sprintf("%g°C", c) }  
       func (f Fahrenheit) String() string { return fmt.Sprintf("%g°F", f) }  
   ```

   ```go
   //conv.go
   //温度转换  
   package tempconv  
     
   // 将Celsius温度转换为Fahrenheit  
   func CToF(c Celsius) Fahrenheit { return Fahrenheit(c*9/5 + 32) }  
     
   // 将Fahrenheit温度转换为Celsius.  
   func FToC(f Fahrenheit) Celsius { return Celsius((f - 32) * 5 / 9) } 
   ```

   ​	每个.go文件的第一行都是package tempconv的声明，表示该文件属于哪个包。当包被导入后，可以这样调用它的成员：tempconv.CToF等等。包级别的变量，例如类型、常量等，对同一个包内的所有文件都是可见的，就好像所有代码定义在同一个文件中一样。注意这里temconv.go导入了fmt.但是conv.go没有，因为它没有用到fmt的任何成员。

   ​	上面代码中包级别的const变量，都是大写字母开头的，因此可以在temconv包的外部使用，tempconv.AbsoluteZeroC

   ​	我们会选择包内的某个.go文件进行包级别的注释，注释写在该文件的package声明前(见之前的conv.go)。一般来说，这里的注释是对文件进行概述的。一个包只有一个文件需要包级别的注释。这些注释一般会放在doc.go文件中，后续可以通过go
    doc tempconv来查看包注释。

4. 在Go程序中，每一个包都是通过一个唯一的字符串来标示的，被称为导入路径，这些包是在import声明中统一导入的。Go语言规范不会对导入的包名进行任何约定，这些是由Go的工具来完成解析的。当使用go的工具
    (tool)时，一个导入路径代表了一个文件夹，该文件夹内包含了组成包的.go文件。

5. 如果导入一个包后不去使用，那么就会报编译错误，编译器的这个检查可以帮助消除不需要的包引用，虽然在debug期间可能会比较蛋疼

6. 包的初始化时会按照声明的顺序初始化包级别的变量，除非变量间有依赖顺序

   ```go
       var a = b + c     // a 第三个初始化 3  
       var b = f()       // b 第二个初始化，调用了f  
       var c = 1         // c 第一个初始化  
         
       func f() int { return c + 1 }  
   ```

7. 每个包只会被初始化一次，首先是初始化依赖包，如果包p导入q，可以肯定的是：q在p初始化之前肯定会完全初始化。因为依赖包会先初始化，所以程序的初始化是自底向上的，main包肯定是最后一个初始化(包的组织类似一个树形结构，main是根节点)。这样在main函数开始时，所有的包都会初始化完毕！





