reflect.Type和reflect.Value都有许多方法用来检验和操作它们

## **reflect.Type**

---

reflect.Type：是一个interface，对应的struct是reflect.\*rtype，在go源码/src/reflect/type.go里定义：

```text
// Type is the representation of a Go type.
```

!!! note
	本文提到的reflect.Type实际均是reflect.\*rtype

!!! warning
	这个reflect.Type是接口hold的变量的类型，而不是追根溯源的最底层类型。比如说

	```text
	type MyStruct struct {}
	```

	这里reflect.Type就是"MyStruct"，而非"struct"。如果想获得struct只能使用Kind()

### **String()**

reflect.Type(实际是reflect.\*rtype)的String()方法：将接口变量hold的变量的类型名转为字符串输出，即输出的变量是类型为string，值为hold的类型名

### **Kind()**

Type和Value都有Kind()方法用来返回一个常量（Type和Value返回的是相同的）

这个常量指示了所hold的值是什么类型 **（这个类型是最终类型，而不是中间类型）**

例如

```go
package main

import (
	"fmt"
	"reflect"
)

type AInt int
type BInt AInt

func main() {
	var e interface{} = BInt(10)
	fmt.Println(reflect.TypeOf(e).Kind())
}
```

输出

```text
int
```

!!! note
	目前暂时没有找到办法获得中间类型

Kind()方法返回的变量类型是reflect.Kind，其值是uint类型的数字，每个数字表示一种go内置类型

**Kind底层研究**

```go hl_lines="13"
package main

import (
	"fmt"
	"reflect"
)

func main() {
	var x float64 = 3.4
	v := reflect.ValueOf(x)
	fmt.Printf("v.Kind(): %T %v\n", v.Kind(), v.Kind())
	fmt.Printf("reflect.Float64: %T %v\n", reflect.Float64, reflect.Float64)
	fmt.Println(v.Kind() == reflect.Float64)
	fmt.Println(v.Kind().String() == "float64")
}
```

输出

```text
v.Kind(): reflect.Kind float64
reflect.Float64: reflect.Kind float64
true
true
```

为什么Kind()可以等于reflect.Float64，是因为同时满足了2点：

1. Kind()返回的类型与reflect.Float64类型一致

2. Kind()返回的值与reflect.Float64值一致

下面一一解答：

1. Kind()返回的是Kind类型，这个Kind类型是reflect包里定义的一个uint的type

	??? note "附Kind定义"
		type Kind是在Type和Value的Kind()方法中都有提及的核心部分

		```go
		// A Kind represents the specific kind of type that a Type represents.
		// The zero Kind is not a valid kind.
		type Kind uint

		const (
		    Invalid Kind = iota
		    Bool
		    Int
		    Int8
		    Int16
		    Int32
		    Int64
		    Uint
		    Uint8
		    Uint16
		    Uint32
		    Uint64
		    Uintptr
		    Float32
		    Float64
		    Complex64
		    Complex128
		    Array
		    Chan
		    Func
		    Interface
		    Map
		    Ptr
		    Slice
		    String
		    Struct
		    UnsafePointer
		)

		func (k Kind) String() string {
			if int(k) < len(kindNames) {
				return kindNames[k]
			}
			return "kind" + strconv.Itoa(int(k))
		}

		var kindNames = []string{
			Invalid:       "invalid",
			Bool:          "bool",
			Int:           "int",
			Int8:          "int8",
			Int16:         "int16",
			Int32:         "int32",
			Int64:         "int64",
			Uint:          "uint",
			Uint8:         "uint8",
			Uint16:        "uint16",
			Uint32:        "uint32",
			Uint64:        "uint64",
			Uintptr:       "uintptr",
			Float32:       "float32",
			Float64:       "float64",
			Complex64:     "complex64",
			Complex128:    "complex128",
			Array:         "array",
			Chan:          "chan",
			Func:          "func",
			Interface:     "interface",
			Map:           "map",
			Ptr:           "ptr",
			Slice:         "slice",
			String:        "string",
			Struct:        "struct",
			UnsafePointer: "unsafe.Pointer",
		}
		```

		上面这个例子const部分用到了类似枚举功能，详见[枚举章节](./../../other/enum/)

		注意：上面reflect.Uint、reflect.Struct这种的类型是reflect.Kind，而值就是uint、struct（不是字符串"uint"、"struct"，而是go的类型uint、struct）

	其次，点击打开上面的"附Kind定义"可以看到：reflect.Float64的类型是reflect.Kind，其值是14(uint的14，从Invalid往下数第14个)。

	所以Kind()返回的类型与reflect.Float64类型一致

2. Kind()返回的值实际是uint数字，例子中为14（reflect.Float64），reflect.Float64返回的值也是uint数字，也为14，所以是相等的，为true

	!!! note
		fmt打印Kind值的时候，调用了Kind的String()方法，因此打印出"float64"而不是数字14

		因此`v.Kind().String() == "float64"`也为true

		**因此今后如果要查询一个变量的最终类型，可以v.Kind()，如果想获得类型的字符串形式，就v.Kind().String**

### **Field().Name**

查询struct元素值，比如.Field(0)表示查询struct中第一个元素的值，.FieldByName("X")表示查询struct中元素名为X的值

Type和Value都有Field()和FieldByName()，但有所不同，并且有的支持.Name，有的不支持，这里做了个测试，贴出结果

```go
fmt.Println(v.Field(i))
fmt.Println(v.Field(i).Name) // panic
fmt.Println(v.FieldByName("A"))
fmt.Println(v.FieldByName("A").Name) // panic

fmt.Println(t.Field(i))
fmt.Println(t.Field(i).Name)
fmt.Println(t.FieldByName("A"))
fmt.Println(t.FieldByName("A").Name) // panic
```

- i是个数字，表示第几个元素，"A"表示结构体中有一个元素为"A"

- v表示Value类型

- t表示Type类型(即\*rtype)

注释掉的表示会报错，因此可以看出，Type和Value都支持Field()和FieldByName()，但只有Type的Field()支持.Name

### **FieldByName()**

通过结构体的元素名查询对应的元素值，不支持.Name、.Type()、.Interface()

和Value的FieldByName支持的有所不同，详见[Value的FieldByName](./../../reflect/pkg_type_value/#fieldbyname_1)

## **reflect.Value**

---

reflect.Value：是一个struct，在go源码/src/reflect/value.go里定义：

```text
// Value is the reflection interface to a Go value.
```

### **Kind()**

[同reflec.Type.Kind()](./../../reflect/pkg_type_value/#kind)

### **Type()**

reflect.Value有Type()方法，用于得到reflect.Type

```go
package main

import (
	"fmt"
	"reflect"
)

type MyStruct struct {
	X, Y int
}

func main() {
	var e interface{} = MyStruct{3, 4}
	t := reflect.ValueOf(e).Type()
	fmt.Printf("%T %v\n", t, t)
}
```

输出

```text
*reflect.rtype main.MyStruct
```

### **获得值**

#### **String()**

reflect.Value的String()方法:

- 若hold的变量类型是string则输出值

- 若hold的变量类型不是string则输出`<接口hold的变量的类型名 Value>`

	??? note "例子"
		```text
		var x float64 = 3.4
		fmt.Println("value:", reflect.ValueOf(x).String())
		```

		输出

		```text
		value: <float64 Value>
		```

	??? note "附String()定义"
		```go
		// String returns the string v's underlying value, as a string.
		// String is a special case because of Go's String method convention.
		// Unlike the other getters, it does not panic if v's Kind is not String.
		// Instead, it returns a string of the form "<T value>" where T is v's type.
		// The fmt package treats Values specially. It does not call their String
		// method implicitly but instead prints the concrete values they hold.
		func (v Value) String() string {
		    switch k := v.kind(); k {
		    case Invalid:
		        return "<invalid Value>"
		    case String:
		        return *(*string)(v.ptr)
		    }
		    // If you call String on a reflect.Value of other type, it's better to
		    // print something than to panic. Useful in debugging.
		    return "<" + v.Type().String() + " Value>"
		}
		```

	为什么要输出`<接口hold的变量的类型名 Value>`，是因为在go里非string类型无法转化为string，若直接panic又不太友好，同时为了便于调试，干脆就打印类型好了（即调用`.Type()`）

**String()例子，包含reflect.Type.String()和reflect.Value.String()**

```go
package main

import (
	"fmt"
	"reflect"
)

type Adder interface {
	Add()
}

type MyStruct struct {
	X, Y int
}

func (this MyStruct) Add() {
	fmt.Println(this.X + this.Y)
}

func main() {
	var i Adder = MyStruct{3, 4}
	t1 := reflect.TypeOf(i)
	v1 := reflect.ValueOf(i)
	st1 := t1.String()
	sv1 := v1.String()

	fmt.Printf("i:\t%T\t%v\n", i, i)
	fmt.Printf("t1:\t%T\t%v\n", t1, t1)
	fmt.Printf("v1:\t%T\t%v\n", v1, v1)
	fmt.Printf("st1:\t%T\t\t%v\n", st1, st1)
	fmt.Printf("sv1:\t%T\t\t%v\n", sv1, sv1)

	fmt.Println("-----")

	var e interface{} = "hello, world"
	v2 := reflect.ValueOf(e)
	sv2 := v2.String()
	fmt.Printf("e:\t%T\t\t%v\n", e, e)
	fmt.Printf("v2:\t%T\t%v\n", v2, v2)
	fmt.Printf("sv2:\t%T\t\t%v\n", sv2, sv2)
}
```

输出

```text
i:		main.MyStruct	{3 4}
t1:		*reflect.rtype	main.MyStruct
v1:		reflect.Value	{3 4}
st1:	string			main.MyStruct
sv1:	string			<main.MyStruct Value>
-----
e:		string			hello, world
v2:		reflect.Value	hello, world
sv2:	string			hello, world
```

#### **Int()、Float()**

Value类型变量还拥有一些名如Int和Float的方法用来获取内部存储的值

!!! warning
	返回的是int64类型以及float64类型的变量，而不是int8、float32

有符号整型为何只有Int()，而没有Int8()、Int32()，是为了保持API的简洁性，获得值和设置值的方法有一定的简化考虑，用最大的类型来包含相关类型，以便存储值。例如使用int64来表示所有有符号整型（int64最大，存储的值能涵盖int8、int16、int32），而Int()是Value获得有符号整型方法，返回的也是int64

```go
package main

import (
	"reflect"
	"fmt"
)

func main() {
	var f32 float32 = 3.4
	fmt.Println(f32)
	vf32 := reflect.ValueOf(f32)
	fmt.Printf("%T %v\n", vf32.Float(), vf32.Float())

	fmt.Println("-----")

	var f64 float64 = 3.4
	fmt.Println(f64)
	vf64 := reflect.ValueOf(f64)
	fmt.Printf("%T %v\n", vf64.Float(), vf64.Float())

	fmt.Println("=====")

	var i8 int8 = 10
	fmt.Println(i8)
	vi8 := reflect.ValueOf(i8)
	fmt.Printf("%T %v\n", vi8.Int(), vi8.Int())

	fmt.Println("-----")

	var i64 int64 = 10
	fmt.Println(i64)
	vi64 := reflect.ValueOf(i64)
	fmt.Printf("%T %v\n", vi64.Int(), vi64.Int())
}
```

输出

```text
3.4
float64 3.4000000953674316
-----
3.4
float64 3.4
=====
10
int64 10
-----
10
int64 10
```

若将int64类型用Float()方法打印会报错，同样，float64用Int()打印也会报错，不过有一个特殊的，就是上面提到的String()，按理来说应该也会报错，但这样意义不大，还不如输出点什么有意义的，便于调试，于是就输出`<float64 Value>`好了

### **设置值**

#### SetInt()、SetFloat()

Value类型变量还拥有一些名如SetInt和SetFloat的方法用来改变内部存储的值，但要注意仅对指针这种地址传递的才有效，并且要用上[Elem()](./../../reflect/pkg_type_value/#elem)才行

SetInt()例子

```go
package main

import (
	"reflect"
	"fmt"
)

type MyInt int

func main() {
	m := MyInt(10)
	v := reflect.ValueOf(&m)
	e := v.Elem()
	e.SetInt(20)
	fmt.Printf("m: %T %v\n", m, m)
	fmt.Printf("e: %T %v\n", e, e)
}
```

输出

```text
m: main.MyInt 20
e: reflect.Value 20
```

SetFloat()例子

```go
package main

import (
	"reflect"
	"fmt"
)

func main() {
	var f float64 = 3.4
	v := reflect.ValueOf(&f)
	e := v.Elem()
	e.SetFloat(7.1)
	fmt.Printf("f: %T %v\n", f, f)
	fmt.Printf("e: %T %v\n", e, e)
}
```

输出

```text
f: float64 7.1
e: reflect.Value 7.1
```

### **Interface()**

给定一个reflect.Value类型的对象我们可以通过Interface方法来将其反转回接口变量。将其类型和值重新打包回一个接口变量中，只不过这个接口变量是个空接口

```go
package main

import (
	"reflect"
	"fmt"
)

type MyInt int

func main() {
	m := MyInt(10)
	v := reflect.ValueOf(m)
	i := v.Interface()
	fmt.Printf("m: %T %v\n", m, m)
	fmt.Printf("v: %T %v\n", v, v)
	fmt.Printf("i: %T %v\n", i, i)
}
```

输出

```text
m: main.MyInt 10
v: reflect.Value 10
i: main.MyInt 10
```

由于是空接口，因此如果类型（比如上面的MyInt）有方法的话是无法执行的，若想可以执行方法，可以用类型断言，详见[类型断言实现接口](./../../method/interface_implement_interface/#_5)，例如：

```go hl_lines="22"
package main

import (
	"reflect"
	"fmt"
)

type MyInt int

func (this MyInt) Add() {
	fmt.Println(this + 3)
}

func main() {
	m := MyInt(10)
	v := reflect.ValueOf(m)
	i := v.Interface()
	fmt.Printf("m: %T %v\n", m, m)
	fmt.Printf("v: %T %v\n", v, v)
	fmt.Printf("i: %T %v\n", i, i)

	e := i.(MyInt)
	e.Add()
}
```

输出

```text
m: main.MyInt 10
v: reflect.Value 10
i: main.MyInt 10
13
```

简单来说，Interface()是ValueOf()的反函数

### **CanSet()**

reflect.Value有一个CanSet()方法可以用来检测Value类型的可设置性（返回一个bool类型的true或false）

??? note "例：非指针不具备可设置性"
	```go
	package main

	import (
		"reflect"
		"fmt"
	)

	func main() {
		var x float64 = 3.4
		v := reflect.ValueOf(x)
		c := v.CanSet()
		fmt.Println(c)
	}
	```

	输出

	```text
	false
	```

??? note "例：指针具备可设置性"
	```go
	package main

	import (
		"reflect"
		"fmt"
	)

	func main() {
		var x float64 = 3.4
		v := reflect.ValueOf(&x)
		c := v.CanSet()
		fmt.Println(c)
	}
	```

	输出

	```text
	false
	```

	虽然用了指针，但还是不可设置性，因为要取它的指向的值才可以，详见[Elem()章节](./../../reflect/pkg_type_value/#elem)

??? note "附: CanSet()定义"
	```go
	func (v Value) CanSet() bool {
		return v.flag&(flagAddr|flagRO) == flagAddr
	}
	```

### **Elem()**

用于获得Value变量内部存储的值，注意仅针对指针这种传递地址的才有效

```go
package main

import (
	"reflect"
	"fmt"
)

func main() {
	var x float64 = 3.4
	v := reflect.ValueOf(&x)
	e := v.Elem()
	c := e.CanSet()
	fmt.Println(c)
}
```

输出

```text
true
```

注意：不能对非指针使用Elem()，会报错

```go
package main

import (
	"reflect"
	"fmt"
)

func main() {
	var x float64 = 3.4
	v := reflect.ValueOf(x)
	e := v.Elem()
	c := e.CanSet()
	fmt.Println(c)
}
```

报错

```text
panic: reflect: call of reflect.Value.Elem on float64 Value
```

??? note "附: Elem()定义"
	```go
	// Elem returns the value that the interface v contains
	// or that the pointer v points to.
	// It panics if v's Kind is not Interface or Ptr.
	// It returns the zero Value if v is nil.
	func (v Value) Elem() Value {
		k := v.kind()
		switch k {
		case Interface:
			var eface interface{}
			if v.typ.NumMethod() == 0 {
				eface = *(*interface{})(v.ptr)
			} else {
				eface = (interface{})(*(*interface {
					M()
				})(v.ptr))
			}
			x := unpackEface(eface)
			if x.flag != 0 {
				x.flag |= v.flag & flagRO
			}
			return x
		case Ptr:
			ptr := v.ptr
			if v.flag&flagIndir != 0 {
				ptr = *(*unsafe.Pointer)(ptr)
			}
			// The returned value's address is v's value.
			if ptr == nil {
				return Value{}
			}
			tt := (*ptrType)(unsafe.Pointer(v.typ))
			typ := tt.elem
			fl := v.flag&flagRO | flagIndir | flagAddr
			fl |= flag(typ.Kind())
			return Value{typ, ptr, fl}
		}
		panic(&ValueError{"reflect.Value.Elem", v.kind()})
	}
	```

### **NumField()**

查询struct结构的元素个数

```go
package main

import (
	"fmt"
	"reflect"
)

type MyStruct struct {
	X, Y int
}

func main() {
	s := MyStruct{3, 4}

	v := reflect.ValueOf(&s)
	e := v.Elem()
	n := e.NumField()

	fmt.Printf("%T %v\n", n, n)
}
```

输出

```text
int 2
```

### **Field()**

打印指定下标的元素（下标从0开始），比如

```go hl_lines="17 18"
package main

import (
	"fmt"
	"reflect"
)

type MyStruct struct {
	X, Y int
}

func main() {
	s := MyStruct{3, 4}

	v := reflect.ValueOf(&s)
	e := v.Elem()
	f0 := e.Field(0)
	f1 := e.Field(1)

	fmt.Printf("f0: %T %v\n", f0, f0)
	fmt.Printf("f1: %T %v\n", f1, f1)
}
```

输出

```
f0: reflect.Value 3
f1: reflect.Value 4
```

### **Field().Type()和Field().Interface()**

Field()是查询第几个元素，比如.Field(0)表示查询第1个元素

但由于Type和Value都有Field()，但有所不同，并且有的支持.Type()和.Interface()，有的不支持，这里做了个测试，贴出结果

```go
fmt.Println(v.Field(i))
fmt.Println(v.Field(i).Type())
fmt.Println(v.Field(i).Interface())

fmt.Println(t.Field(i))
fmt.Println(t.Field(i).Type()) // panic
fmt.Println(t.Field(i).Interface()) // panic
```

- i是个数字，表示第几个元素

- v表示Value类型

- t表示Type类型(即\*rtype)

注释掉的表示会报错，因此可以看出，Type和Value都支持Field()，但只有Value的Field()支持.Type()和.Interface()

**完整例子**，包含Type的Field()

```go
package main

import (
	"fmt"
	"reflect"
)

type T struct {
	A int
	B string
}

func main() {
	t := T{23, "skidoo"}
	s := reflect.ValueOf(&t).Elem()
	typeOfT := s.Type()

	for i := 0; i < s.NumField(); i++ {
		f := s.Field(i)
		fmt.Printf("%d: %s %s = %v\n", i, typeOfT.Field(i).Name, f.Type(), f.Interface())
	}
}
```

输出

```text
0: A int = 23
1: B string = skidoo
```

### **FieldByName()**

通过结构体的元素名查询对应的元素值，不支持.Name，但支持.Type()、.Interface()。

和Type的FieldByName支持的有所不同，详见[Type的FieldByName](./../../reflect/pkg_type_value/#fieldbyname)
