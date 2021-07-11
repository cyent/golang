用interface.method和直接用自定义TYPE.method()有一点不同要注意：

如果用interface.method()，那么method的receiver参数类型必须和实现interface时候的类型相同，比如：

```go
type I interface {
    M()
}

type T struct {
    S string
}

func (t *T) M() {
    fmt.Println(t.S)
}

func main() {
    var i I
    i = &T{"hello"} //不能写成i = T{"hello"}
    i.M()
}
```

但如果是直接用method()，就不用类型一致，会自动判断，比如：

```go
type T struct {
    S string
}

func (t T) M() {
    fmt.Println(t.S)
}

func main() {
    i := T{"hello"} //也可以写成i := &T{"hello"}
    i.M()
}
```

这2个例子的输出都是hello
