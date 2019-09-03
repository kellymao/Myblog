---
layout: post
title: "Go语言接口（interface）"
date: 2019-09-03 
description: "Go语言接口（interface）"
tag: GO语言
---   
  


### Go语言接口声明（定义）

### 1. 接口被实现的条件

#####（1）接口的方法与实现接口的类型方法格式一致

#####（2）接口中所有方法均被实现： 实现了接口里面所有的方法才叫实现了接口。只包含部分方法就没有实现这个接口 


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



### Go语言类型与接口的关系

#####（1）每一个struct 类型结构体可以实现多个接口

一个类型可以同时实现多个接口，而接口间彼此独立，不知道对方的实现。


#####（2）多个类型可以实现相同的接口

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


##### Service 接口定义了两个方法：一个是开启服务的方法（Start()），一个是输出日志的方法（Log()）。使用 GameService 结构体来实现 Service，GameService 自己的结构只能实现 Start() 方法，而 Service 接口中的 Log() 方法已经被一个能输出日志的日志器（Logger）实现了，无须再进行 GameService 封装，或者重新实现一遍。所以，选择将 Logger 嵌入到 GameService 能最大程度地避免代码冗余


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


### Go语言类型断言


##### Go语言中有四种接口相关的类型转换情形：
##### ● 将一个非接口值转换为一个接口类型。在这样的转换中，此非接口值的类型必须实现了此接口类型。
##### ● 将一个接口值转换为另一个接口类型（前者接口值的类型实现了后者目标接口类型）。
##### ● 将一个接口值转换为一个非接口类型（此非接口类型必须实现了此接口值的接口类型）。
##### ● 将一个接口值转换为另一个接口类型（前者接口值的类型可以实现了也可以未实现后者目标接口类型）。


本节将介绍后面两种情形。这两种情形的合法性是在运行时刻通过类型断言来验证的。 事实上，类型断言同样也适用于上面列出的第二种情形。

##### 断言的语法

##### 1. 断言被用做一个单值表达式 ，断言成功返回值。断言失败会引起恐慌

##### 2. 断言返回两个结果的多值表达式，断言成功返回值，和一个true 的bool类型， 断言失败返回零值 nil 接口值 ，和 flase 


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


##### 1. 将接口转换为其他接口

	var obj interface = new(bird)
	f, isFlyer := obj.(Flyer)
	
代码中，new(bird) 产生 *bird 类型的 bird 实例，这个实例被保存在 interface{} 类型的 obj 变量中。使用 obj.(Flyer) 类型断言，将 obj 转换为 Flyer 接口。f 为转换成功时的 Flyer 接口类型，isFlyer 表示是否转换成功，类型就是 bool。

	
##### 2. 将接口转换为其他类型

可以实现将接口转换为普通的指针类型。例如将 Walker 接口转换为 *pig 类型，请参考下面的代码：

	p1 := new(pig)
	var a Walker = p1
	p2 := a.(*pig)
	fmt.Printf("p1=%p p2=%p", p1, p2)


	
### Go语言接口的嵌套组合


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
