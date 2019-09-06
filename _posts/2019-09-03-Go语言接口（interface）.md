---
layout: post
title: "Go语言接口（interface）"
date: 2019-09-03 
description: "Go语言接口（interface）"
tag: GO语言
---   
  


## Go语言接口声明（定义）

##  接口被实现的条件

##### **1. 接口的方法与实现接口的类型方法格式一致**

##### **2. 接口中所有方法均被实现： 实现了接口里面所有的方法才叫实现了接口。只包含部分方法就没有实现这个接口**


	// 接口的声明

	type Writer interface{

		writedata(v interface{})(n int ,err error)

	}
	type Reader interface{

		readdata(v interface{})

	}

	type File struct{}


	func (f *File) writedata(v interface{})(n int ,err error){

		fmt.Println("writedata:", v)
		return

	}

	func (f *File) readdata(v interface{}){

		fmt.Println("readedata:", v)
		return             // 函数的裸返回，跟定义的返回值变量名一样，没定义就返回空 


		fmt.Println("end:")
	}


	func e1(){

		var write Writer = &File{}
		fmt.Println(write)

		n,err:=write.writedata("write a few data")

		fmt.Println(n,err)


		var read Reader = &File{}

		read.readdata("read a few data")

	}

	运行的结果：

	&{}
	writedata: write a few data
	0 <nil>
	readedata: read a few data



## Go语言类型与接口的关系

##### **1. 每一个struct 类型结构体可以实现多个接口**

一个类型可以同时实现多个接口，而接口间彼此独立，不知道对方的实现。


##### **2. 多个类型可以实现相同的接口**

一个接口的方法，不一定需要由一个类型完全实现，接口的方法可以通过在类型中嵌入其他类型或者结构体来实现。也就是说，使用者并不关心某个接口的方法是通过一个类型完全实现的，还是通过多个结构嵌入到一个结构体中拼凑起来共同实现的。



##### 把 Socket 能够写入数据和需要关闭的特性使用接口来描述，请参考下面的代码：


	type writer interface{

		writedata([]byte)(n int,err error) 
	}


	  


	type closer interface{

		close()(err error) 
	}

	type socket struct{} 


	func (s *socket) writedata(p []byte)(n int,err error) {

		fmt.Println(string(p)) 
		return 0 , nil 




	}


	func (s *socket) close() error{

		return nil 
	}


	func usingwriter(write writer){

		write.writedata([]byte("hello world"))

	}


	func usingcloser(close  closer){

		close.close() 
	}

	func main(){

	// 实例化socket 
	var i *socket = &socket{}


	// socket 类型实现了writer 和 closer 两个接口 
	usingwriter(i) 
	usingcloser(i) 


	}


	usingWriter() 和 usingCloser() 完全独立，互相不知道对方的存在，也不知道自己使用的接口是 Socket 实现的。


Service 接口定义了两个方法：一个是开启服务的方法（Start()），一个是输出日志的方法（Log()）。使用 GameService 结构体来实现 Service，GameService 自己的结构只能实现 Start() 方法，而 Service 接口中的 Log() 方法已经被一个能输出日志的日志器（Logger）实现了，无须再进行 GameService 封装，或者重新实现一遍。所以，选择将 Logger 嵌入到 GameService 能最大程度地避免代码冗余


	type Service interface{

		Start()
		Log() 

	}

	type Logger struct{

	}

	func (l *Logger) Log(){



	}


	type GameService struct{
		Logger 

	}

	func (g *GameService) Start(){


	}


	func main(){

	var service  Service = &GameService{}
	service.Start()
	service.Log()

	}


	service 就可以使用 Start() 方法和 Log() 方法，其中，Start() 由 GameService 实现，Log() 方法由 Logger 实现。


## Go语言类型断言


##### **Go语言中有四种接口相关的类型转换情形：**
##### ● **将一个非接口值转换为一个接口类型。在这样的转换中，此非接口值的类型必须实现了此接口类型。**
##### ● **将一个接口值转换为另一个接口类型（前者接口值的类型实现了后者目标接口类型）。**
##### ● **将一个接口值转换为一个非接口类型（此非接口类型必须实现了此接口值的接口类型）。**
##### ● **将一个接口值转换为另一个接口类型（前者接口值的类型可以实现了也可以未实现后者目标接口类型）。**


本节将介绍后面两种情形。这两种情形的合法性是在运行时刻通过类型断言来验证的。 事实上，类型断言同样也适用于上面列出的第二种情形。

### 断言的语法

##### **1. 断言被用做一个单值表达式 ，断言成功返回值。断言失败会引起恐慌**

##### **2. 断言返回两个结果的多值表达式，断言成功返回值，和一个true 的bool类型， 断言失败返回零值 nil 接口值 ，和 flase**


【示例 1】断言类型为非接口类型 : 将一个接口值转换为一个非接口类型



	func main() {
		var x interface{} = "123"  //声明一个接口类型

		rel,ok := x.(int)          //断言类型为非接口类型 
		fmt.Println(rel,ok)        // 123, true 
		
		
		rel := x.(int)  
		fmt.Println(rel)  		   // 123 
		
		rel,ok := x.(float32)      //断言类型为非接口类型 
		fmt.Println(rel,ok)        // 0, false 
		
		
		rel:= x.(float32)          
		fmt.Println(rel)           //断言失败单值表达式 会产生恐慌 


	}


【示例 2】断言类型为接口类型：


	type Write interface{

		write() 
	}



	type Summerywrite struct {

	}

	func ( s *Summerywrite) write(){} 




	func main(){

		var a interface{}  = Summerywrite{}

		var a1 interface{}  = &Summerywrite{}


		var b interface{}  = "hello"  // b的动态类型为内置类型string


		var c Write = &Summerywrite{}


		rel1,ok1 := a.(Write)
		fmt.Println(rel1,ok1)              // <nil> false


		rel2,ok2 := a1.(Write)
		fmt.Println(rel2,ok2)              //  &{} true

		rel3,ok3 := b.(Write)
		fmt.Println(rel3,ok3)             // <nil> false


		var result interface{}              //先定义类型，在进行断言测试
		var is_boo bool
		result,is_boo = c.(interface{})     //c 实现了interface() 和 Write 两个接口

		fmt.Println(result,is_boo)           //  &{} true 

		rel:= b.(Write)                     // 断言失败出现恐慌
		fmt.Println(rel)                    // panic: interface conversion: string is not main.Write: missing method write




	}
	
	
	
	
	
### 断言类型实现接口转换

	var w io.Writer
	w = os.Stdout
	f := w.(*os.File) // 成功: f == os.Stdout
	c := w.(*bytes.Buffer) // 死机：接口保存*os.file，而不是*bytes.buffer


##### **1. 将接口转换为其他接口**

	var obj interface = new(bird)
	f, isFlyer := obj.(Flyer)
	
代码中，new(bird) 产生 *bird 类型的 bird 实例，这个实例被保存在 interface{} 类型的 obj 变量中。使用 obj.(Flyer) 类型断言，将 obj 转换为 Flyer 接口。f 为转换成功时的 Flyer 接口类型，isFlyer 表示是否转换成功，类型就是 bool。

	
##### **2. 将接口转换为其他类型**

可以实现将接口转换为普通的指针类型。例如将 Walker 接口转换为 *pig 类型，请参考下面的代码：

	p1 := new(pig)
	var a Walker = p1
	p2 := a.(*pig)
	fmt.Printf("p1=%p p2=%p", p1, p2)


	
## Go语言接口的嵌套组合


##### 例一


	type Reader interface{

		Read() 

	}

	type Writer interface{

		Writer() 
	}


	type ReadWriter interface{
		Reader 
		Writer 


	}

	type File struct{


	}

	func (f *File) Read(){}

	func (f *File) Writer(){}

	func test(rw ReadWriter){

		rw.Read() 
		rw.Writer()
	}

	test(&File{})


##### 例二


	// io 包
	type Writer interface {
		Write(p []byte) (n int, err error)
	}
	type Closer interface {
		Close() error
	}
	type WriteCloser interface {
		Writer
		Closer
	}
	
	
	
	// main 包
	package main

	import (
		"io"
	)

	
	
	// 声明一个设备结构
	type device struct {
	}

	// 实现io.Writer的Write()方法
	func (d *device) Write(p []byte) (n int, err error) {
		return 0, nil
	}

	// 实现io.Closer的Close()方法
	func (d *device) Close() error {
		return nil
	}

	func main() {

		// 声明写入关闭器, 并赋予device的实例
		var wc io.WriteCloser = new(device)

		// 写入数据
		wc.Write(nil)

		// 关闭设备
		wc.Close()

		// 声明写入器, 并赋予device的新实例
		var writeOnly io.Writer = new(device)

		// 写入数据
		writeOnly.Write(nil)

	}
	
	
##  Go语言空接口类型（interface{}）	

#####  1. 将值保存到空接口

	var any interface{}  

	any = 1 
	fmt.Println(any) 


	any = true 
	fmt.Println(any) 

	any = false 
	fmt.Println(any) 


#####  2. 从空接口获取值 

	var i interface{} = 1 

	var b int 
	b = i              // 触发宕机了，空接口不能直接转换为int

	b = i.(int)       // 通过断言进行类型转换

##  Go语言类型分支（switch判断空接口中变量的类型）

一个 type-switch 流程控制代码块的语法如下所示：


	switch t := areaIntf.(type) {
	case *Square:
		fmt.Printf("Type Square %T with value %v\n", t, t)
	case *Circle:
		fmt.Printf("Type Circle %T with value %v\n", t, t)
	case nil:
		fmt.Printf("nil value: nothing to check?\n")
	default:
		fmt.Printf("Unexpected type %T\n", t)
	}


下面的例子将一个 interface{} 类型的参数传给 printType() 函数，通过 switch 判断 v 的类型，然后打印对应类型的提示，代码如下：


	package main
	import (
		"fmt"
	)
	func printType(v interface{}) {
		switch v.(type) {
		case int:
			fmt.Println(v, "is int")
		case string:
			fmt.Println(v, "is string")
		case bool:
			fmt.Println(v, "is bool")
		}
	}


	func main() {
		printType(1024)
		printType("pig")
		printType(true)
	}

##  Go语言排序（借助sort.Interface接口） 

##### **原理： sort.Sort(v sort.Interface) 函数实现排序,接受一个sort.Interface 的参数**

##### **一个内置的排序算法需要知道三个东西：序列的长度，表示两个元素比较的结果，一种交换两个元素的方式；这就是sort.Interface 的三个方法：**

	package sort
	type Interface interface {
		Len() int            // 获取元素数量
		Less(i, j int) bool // i，j是序列元素的指数。
		Swap(i, j int)        // 交换元素
	}

### 自定义切片 []string 的排序方法



### 内置的切片排序方法

使用 sort 包的 StringSlice 类型就可以更简单快速地进行字符串切片排序 ：

	func main(){
		var str sort.StringSlice

		str = sort.StringSlice{
			"apple","banana","taozi","xigua","why","cm",
		}


		sort.Sort(str)

		fmt.Println(str)  // [apple banana cm taozi why xigua]

	}	

	
使用 sort 包的sort.IntSlice 类型进行整型切片的排序


	func sort_int(){

		var i sort.IntSlice  = sort.IntSlice{6,8,9,3,2,7}

		sort.Sort(i)

		fmt.Println(i) // [2 3 6 7 8 9]


	}


使用 sort.Strings 函数直接对字符串切片进行排序 

	func sort_str1(){

		var str []string = []string{"apple","banana","taozi","xigua","why","cm",}

		sort.Strings(str)

		fmt.Println(str)  // [apple banana cm taozi why xigua]


	}

使用 sort.Ints 函数直接对字符串切片进行排序 


	func sort_int1(){


		var i []int = []int{6,8,9,3,2,7}

		sort.Ints(i)

		fmt.Println(i)

	}




Go语言中的 sort 包中定义了一些常见类型的排序方法，如下表所示。

|sort 包中内建的类型排序接口：                                          |
|---------------------------------|
|字符串（String）：     |	StringSlice	  |  sort.Strings(a [] string)	|
|整型（int）：	        |IntSlice	    |sort.Ints(a []int)	|
|双精度浮点（float64）：|	Float64Slice	|sort.Float64s(a []float64)	|


编程中经常用到的 int32、int64、float32、bool 类型并没有由 sort 包实现，使用时依然需要开发者自己编写。
	 
### 自定义结构体的排序方法



### 内置的sort.Slice 进行结构体排， 使用sort.Slice进行切片元素排序

sort.Slice() 函数的定义如下：

	func Slice(slice interface{}, less func(i, j int) bool)

将一批英雄名单使用结构体定义，英雄名单的结构体中定义了英雄的名字和分类。排序时要求按照英雄的分类进行排序，相同分类的情况下按名字进行排序，详细代码实现过程如下。


	func sort_hero(){

		type HeroKind int
		const (
			None = iota
			Tank
			Assassin
			Mage
		)


		type Hero struct{
			name string
			kind HeroKind


		}

		var hero_se []*Hero = []*Hero{

			&Hero{"张飞",Mage},
			&Hero{"吕布",Assassin},
			&Hero{"关于",Tank},
			&Hero{"曹操",Assassin},
			&Hero{"张辽",Mage},
			&Hero{"孙权",Assassin},
			&Hero{"周瑜",Tank},




		}

		sort.Slice(hero_se,func(i, j int)bool{

			return  hero_se[i].kind >  hero_se[j].kind


		})

		for index,v := range hero_se{

			fmt.Println(index, *v )

		}





	}

		
	运行的结果：
	0 {张飞 3}
	1 {张辽 3}
	2 {吕布 2}
	3 {曹操 2}
	4 {孙权 2}
	5 {关于 1}
	6 {周瑜 1}

##  Go语言error接口：


error 类型，而且没有解释它究竟是什么。实际上它就是 interface 类型，这个类型有一个返回错误信息的单一方法：


	type error interface {
		Error() string
	}
	
创建一个 error 的方法

##### **1. 调用 errors.New 函数**

	errors.New("this is a error") 

##### **2. 调用 fmt.Errorf，它还会处理字符串格式化。** 	

	fmt.Errorf("this is a error %d %s" , a, err) 
	
##  Go语言Writer和Reader接口简述


	type Writer interface {
		Write(p []byte) (n int, err error)
	}
	
	
上述代码中展示了 io.Writer 接口的声明。这个接口声明了唯一一个方法 Write，这个方法接受一个 byte 切片，并返回两个值。第一个值是写入的字节数，第二个值是 error 错误值。


	type Reader interface {
		Read(p []byte) (n int, err error)
	}
	
	
上述代码中的 io.Reader 接口声明了一个方法 Read，这个方法接受一个 byte 切片，并返回两个值。第一个值是读入的字节数，第二个值是 error 错误值。




