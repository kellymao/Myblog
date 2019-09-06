---
layout: post
title: "Go语言反射（reflection）简述"
date: 2019-09-05 
description: "Go语言反射（reflection）简述"
tag: GO语言
---   
  
<br/>

## 反射的基本概念

<br/>

Go语言中的反射与其他语言有比较大的不同。首先我们要理解两个基本概念 Type 和 Value，它们也是 Go语言包中 reflect 空间里最重要的两个类型。我们先看一下下面的定义：


	type MyReader struct {
		Name string
	}
	func (r MyReader)Read(p []byte) (n int, err error) {
		// 实现自己的Read方法
	}
	
	
因为 MyReader 类型实现了 io.Reader 接口的所有方法（其实就是一个 Read() 函数），所以 MyReader 实现了接口 io.Reader。我们可以按如下方式来进行 MyReader 的实例化和赋值：


	var reader io.Reader
	reader = &MyReader{"a.txt"}
	
	
现在我们可以再来解释一下什么是 Type，什么是 Value。


对所有接口进行反射，都可以得到一个包含 Type 和 Value 的信息结构。比如我们对上例的 reader 进行反射，也将得到一个 Type 和 Value，Type 为 io.Reader，Value 为 MyReader{"a.txt"}。顾名思义，Type 主要表达的是被反射的这个变量本身的类型信息，而 Value 则为该变量实例本身的信息。

<br/>

## 反射规则浅析

<br/>

前面讲解了 Value 和 Type 的基本概念，本节重点讲解反射对象 Value、Type 和类型实例之间的相互转化。实例、Value、Type 三者之间的转换关系如下图所示。


![](http://c.biancheng.net/uploads/allimg/190813/4-1ZQ31336163P.gif)


##### **反射 API 的分类总结如下:** 


##### **1. 通过实例获取 Value 对象，直接使用 reflect.ValueOf() 函数**

##### **2. 通过实例获取反射对象的 Type，直接使用 reflect.TypeOf() 函数** 

##### **3. 从 Value 获取反射对象的 Type，可以直接调用 Value.Type()方法，因为 Value 内部存放着到 Type 类型的指针** 

##### **4. Value 到实例的转换，Value 本身就包含类型和值信息，reflect 提供了丰富的方法来实现从 Value 到实例的转换** 

	//该方法最通用，用来将 Value 转换为空接口，该空接口内部存放具体类型实例
	
	func (v Value) Interface() （i interface{})

	//Value 自身也提供丰富的方法，直接将 Value 转换为简单类型实例，如果类型不匹配，则直接引起 panic
	
	func (v Value) Bool () bool
	func (v Value) Float() float64
	func (v Value) Int() int64
	func (v Value) Uint() uint64

##### **5. 从一个指针类型的 Value 获得值类型 Value， 直接调用 Elem() 方法**

##### **6. 从一个指针类型的 Type 获得值类型 Type ，也是直接调用 Elem() 方法**

##### **7. 判断Value 值的可修改性 CanSet() 方法，修改value的值  Set(x Value) 方法**


<br/>

## 理解反射的类型（Type）与种类（Kind）


<br/>

### 1) 反射种类（Kind）的定义

	种类（Kind）指的是对象归属的品种，在 reflect 包中有如下定义：

	type Kind uint
	const (
		Invalid Kind = iota  // 非法类型
		Bool                 // 布尔型
		Int                  // 有符号整型
		Int8                 // 有符号8位整型
		Int16                // 有符号16位整型
		Int32                // 有符号32位整型
		Int64                // 有符号64位整型
		Uint                 // 无符号整型
		Uint8                // 无符号8位整型
		Uint16               // 无符号16位整型
		Uint32               // 无符号32位整型
		Uint64               // 无符号64位整型
		Uintptr              // 指针
		Float32              // 单精度浮点数
		Float64              // 双精度浮点数
		Complex64            // 64位复数类型
		Complex128           // 128位复数类型
		Array                // 数组
		Chan                 // 通道
		Func                 // 函数
		Interface            // 接口
		Map                  // 映射
		Ptr                  // 指针
		Slice                // 切片
		String               // 字符串
		Struct               // 结构体
		UnsafePointer        // 底层指针
	)

### 2) 类型（Type）的定义

Go 程序中的类型（Type）指的是系统原生数据类型，如 int、string、bool、float32 等类型，以及使用 type 关键字定义的类型，这些类型的名称就是其类型本身的名称。例如使用 type A struct{} 定义结构体时，A 就是 struct{} 的类型。



## Go语言通过反射获取 Type 的相关方法 

### 基本的方法： 


t_ptr := reflect.TypeOf(&a)  获取反射对象的 Type

t := t_ptr.Elem() 从指针获取值 ，得到reflect.Type 对象

t.Name()  返回的是类型 

t.Kind()  返回是种类，const定义的常量 

t.Kind() == reflect.Float64  可以直接进行比较，结果为true 或者 false的布尔值 

t.String() 跟直接fmt.Println(t) 结果是一样的

t.NumField() 返回字段的个数 

t.Field(i) 返回某一个字段的 StructField 结构

fieldType := t.Field(i) ; fieldType.Name, fieldType.Tag 返回某一个字段的名字 和 tag信息 

fieldType := t.Field(i) ; fieldType.Tag.Get("json")  从tag中获取特定的tag信息 

t_ptr.NumMethod() 返回方法的数量 （此处要注意：结构体的方法使用指针定义的，所以用指针来获取）

t_ptr.Method(i) 返回某一个方法的 reflect.Method , 是string类型

method_name := t_ptr.Method(i) ; method_name.Name  返回某一个方法的名字

### 获取结构体的成员的方法

|方法	|说明|
|---------|-------|
|Field(i int) StructField	|根据索引，返回索引对应的结构体字段的信息。当值不是结构体或索引超界时发生宕机|
|NumField() int	|返回结构体成员字段数量。当类型不是结构体或索引超界时发生宕机|
|FieldByName(name string) (StructField, bool)	|根据给定字符串返回字符串对应的结构体字段的信息。没有找到时 bool 返回 false，当类型不是结构体或索引超界时发生宕机|
|FieldByIndex(index []int) StructField	|多层成员访问时，根据 []int 提供的每个结构体的字段索引，返回字段的信息。没有找到时返回零值。当类型不是结构体或索引超界时 发生宕机|
|FieldByNameFunc( match func(string) bool) (StructField,bool)	|根据匹配函数匹配需要的字段。当值不是结构体或索引超界时发生宕机|




### 获取结构体标签（Struct Tag）的方法：

使用 StructField 中 Tag 的 Get() 方法


    typeOfCat := reflect.TypeOf(cat{})
    if catType, ok := typeOfCat.FieldByName("Type"); ok {
        fmt.Println(catType.Tag.Get("json"))
    }

## Go语言通过反射获取 Value 的相关方法 

### 基本的方法：

v_ptr := reflect.ValueOf(&x) 返回一个Value 值类型

v := v_ptr.Elem() 从指针获取值 , 返回一个reflect.Value对象 

v.Kind()  返回const定义的常量 

v.Type()  获取反射对象的 Type

v.Kind() == reflect.Float64  可以直接进行比较，结果为true 或者 false的布尔值 

v.Field(i)  返回一个Value对象 

v.NumField() int	返回结构体成员字段数量

v.NumMethod()  返回方法的数量 

v_ptr.Method(i)   返回一个reflect.Value对象 

me := v_ptr.Method(i) ; me.Type()  返回一个reflect.Type 类型 ，拥有上面介绍的type的所有方法

me := v_ptr.Method(0); v_ptr.Method(0).Call() 

### 从反射值对象获取被包装的值

|方法名	|说  明|
|--------|------|
|Interface() interface {}	|将值以 interface{} 类型返回，可以通过类型断言转换为指定类型|
|Int() int64	|将值以 int 类型返回，所有有符号整型均可以此方式返回|
|Uint() uint64	|将值以 uint 类型返回，所有无符号整型均可以此方式返回|
|Float() float64	|将值以双精度（float64）类型返回，所有浮点数（float32、float64）均可以此方式返回|
|Bool() bool	|将值以 bool 类型返回|
|Bytes() []bytes	|将值以字节数组 []bytes 类型返回|
|String() string	|将值以字符串类型返回|

### 反射访问结构体成员的值

|方  法|	备  注|
|--------|------|
|Field(i int) Value	|根据索引，返回索引对应的结构体成员字段的反射值对象。当值不是结构体或索引超界时发生宕机|
|NumField() int	|返回结构体成员字段数量。当值不是结构体或索引超界时发生宕机|
|FieldByName(name string) Value	|根据给定字符串返回字符串对应的结构体字段。没有找到时返回零值，当值不是结构体或索引超界时发生宕机|
|FieldByIndex(index []int) Value	|多层成员访问时，根据 []int 提供的每个结构体的字段索引，返回字段的值。 没有找到时返回零值，当值不是结构体或索引超界时发生宕机|
|FieldByNameFunc(match func(string) bool) Value	|根据匹配函数匹配需要的字段。找到时返回零值，当值不是结构体或索引超界时发生宕机|




## GO语言通过反射修改变量的值

修改普通变量值得例子：

	package main
	import (
		"fmt"
		"reflect"
	)
	func main() {
		// 声明整型变量a并赋初值
		var a int = 1024
		// 获取变量a的反射值对象(a的地址)
		valueOfA := reflect.ValueOf(&a)
		// 取出a地址的元素(a的值)
		valueOfA = valueOfA.Elem()
		// 修改a的值为1
		valueOfA.SetInt(1)
		// 打印a的值
		fmt.Println(valueOfA.Int())
	}

修改结构体值得例子：


	package main
	import (
		"reflect"
		"fmt"
	)
	func main() {
		type dog struct {
				LegCount int
		}
		// 获取dog实例地址的反射值对象
		valueOfDog := reflect.ValueOf(&dog{})
		// 取出dog实例地址的元素
		valueOfDog = valueOfDog.Elem()
		// 获取legCount字段的值
		vLegCount := valueOfDog.FieldByName("LegCount")
		// 尝试设置legCount的值(这里会发生崩溃)
		vLegCount.SetInt(4)
		fmt.Println(vLegCount.Int())
	
	
## Go语言通过反射调用函数


如果反射值对象（reflect.Value）中值的类型为函数时，可以通过 reflect.Value 调用该函数。使用反射调用函数时，需要将参数使用反射值对象的切片 []reflect.Value 构造后传入 Call() 方法中，调用完成时，函数的返回值通过 []reflect.Value 返回。

下面的代码声明一个加法函数，传入两个整型值，返回两个整型值的和。将函数保存到反射值对象（reflect.Value）中，然后将两个整型值构造为反射值对象的切片（[]reflect.Value），使用 Call() 方法进行调用。


反射调用函数：


	package main
	import (
		"fmt"
		"reflect"
	)
	// 普通函数
	func add(a, b int) int {
		return a + b
	}
	func main() {
		// 将函数包装为反射值对象
		funcValue := reflect.ValueOf(add)
		// 构造函数参数, 传入两个整型值
		paramList := []reflect.Value{reflect.ValueOf(10), reflect.ValueOf(20)}
		// 反射调用函数
		retList := funcValue.Call(paramList)
		// 获取第一个返回值, 取整数值
		fmt.Println(retList[0].Int())
	}


代码说明如下：

第 9～12 行，定义一个普通的加法函数。

第 17 行，将 add 函数包装为反射值对象。

第 20 行，将 10 和 20 两个整型值使用 reflect.ValueOf 包装为 reflect.Value，再将反射值对象的切片 []reflect.Value 作为函数的参数。

第 23 行，使用 funcValue 函数值对象的 Call() 方法，传入参数列表 paramList 调用 add() 函数。

第 26 行，调用成功后，通过 retList[0] 取返回值的第一个参数，使用 Int 取返回值的整数值。

提示

反射调用函数的过程需要构造大量的 reflect.Value 和中间变量，对函数参数值进行逐一检查，还需要将调用参数复制到调用函数的参数内存中。调用完毕后，还需要将返回值转换为 reflect.Value，用户还需要从中取出调用值。因此，反射调用函数的性能问题尤为突出，不建议大量使用反射函数调用。





## 一个完整的例子：


	type Stu struct{


		Name string `json:"stu_name"`
		Age int
	}

	func (s *Stu) Printstu() {

		fmt.Println("xxxxx",s)
	}


	func (s *Stu) Printage(a int, b int ) int {

		fmt.Println("xxxxx ",s)
		fmt.Println("result is ", a+b)

		return a+b
	}

	func test_reflect(){




		var stu *Stu = &Stu{"zhangsan",10,}
		stu.Printstu()

		t_ptr := reflect.TypeOf(stu)

		t:=t_ptr.Elem()

		fmt.Println(t)
		fmt.Printf("t Name() is %v",t.Name())
		fmt.Println()
		fmt.Printf("t Kind() is %v\n",t.Kind())

		fmt.Printf("t String() is %v",t.String())
		fmt.Println()
		fmt.Printf("t NumField() is %v",t.NumField())
		fmt.Println()
		fmt.Printf("t_ptr NumMethod() is %v \n",t_ptr.NumMethod())


		fmt.Printf("t_ptr Method(0) is %T %v \n ",t_ptr.Method(0),t_ptr.Method(0))

		fmt.Printf("t_ptr Method(0).Name is %v \n ",t_ptr.Method(0).Name)  //Printstu

		fmt.Println()
		fmt.Printf("t Field(1) is %T %v",t.Field(0),t.Field(0))
		fmt.Println()
		fmt.Printf("t Field(1).Name is %v",t.Field(0).Name)
		fmt.Println()
		fmt.Printf("t Field(1).Tag is %s",t.Field(0).Tag)
		fmt.Println()


		fmt.Printf("t Field(1).Tag.Get('json') is %s \n ",t.Field(0).Tag.Get("json"))

		s,ok:=t.FieldByName("Name")
		if ok {

			fmt.Println(s.Name )
			fmt.Println(s.Tag )
		}


		v_ptr := reflect.ValueOf(stu)

		v:=v_ptr.Elem()

		fmt.Println(v)

		fmt.Println()
		fmt.Printf("v Type() is %T %v \n",v.Type(),v.Type())

		fmt.Printf("v Type().Field(0) is %T %v %v\n",v.Type().Field(0),v.Type().Field(0),v.Type().Field(0).Name)

		fmt.Printf("v Kind() is %v",v.Kind())
		fmt.Println()
		fmt.Printf("v NumField() is %v",v.NumField())
		fmt.Println()
		fmt.Printf("v_ptr NumMethod() is %v \n ",v_ptr.NumMethod())

		fmt.Printf("v_ptr Method(0) is %T %v \n",v_ptr.Method(0),v_ptr.Method(0))



		fmt.Printf("v_ptr Method(0).Type() is %T %v \n",v_ptr.Method(0).Type(),v_ptr.Method(0))

		fmt.Printf("v_ptr Method(0).Type().Name() String() is %v %v \n",v_ptr.Method(0).Type().Name(),v_ptr.Method(0).Type().String())



		var paramList0  []reflect.Value   // 没有值得情况

		retList0 :=v_ptr.Method(1).Call(paramList0)
		fmt.Println(retList0)

		paramList1 := []reflect.Value{reflect.ValueOf(10),reflect.ValueOf(20)}  // 传值的情况

		retList1  := v_ptr.Method(0).Call(paramList1)

		fmt.Println(retList1)

		fmt.Println(retList1[0].Int())  // 获取第一个返回值, 取整数值

		fmt.Println()
		fmt.Printf("v Field(1) is %T %v",v.Field(0),v.Field(0))
		fmt.Println()
		fmt.Printf("v Field(1).CanSet is %v",v.Field(0).CanSet())
		fmt.Println()
		fmt.Printf("v Field(1).Interface is %s",v.Field(0).Interface())
		fmt.Println()

		fmt.Printf("v FieldByName('Name').Interface is %T %s \n",v.FieldByName("Name"),v.FieldByName("Name").Interface())





	}
	
执行的结果：

	main.Stu
	t Name() is Stu
	t Kind() is struct
	t String() is main.Stu
	t NumField() is 2
	t_ptr NumMethod() is 2 
	t_ptr Method(0) is reflect.Method {Printage  func(*main.Stu, int, int) int <func(*main.Stu, int, int) int Value> 0} 
	 t_ptr Method(0).Name is Printage 
	 
	t Field(1) is reflect.StructField {Name  string json:"stu_name" 0 [0] false}
	t Field(1).Name is Name
	t Field(1).Tag is json:"stu_name"
	t Field(1).Tag.Get('json') is stu_name 
	 Name
	json:"stu_name"
	{zhangsan 10}

	v Type() is *reflect.rtype main.Stu 
	v Type().Field(0) is reflect.StructField {Name  string json:"stu_name" 0 [0] false} Name
	v Kind() is struct
	v NumField() is 2
	v_ptr NumMethod() is 2 
	 v_ptr Method(0) is reflect.Value 0x49d570 
	v_ptr Method(0).Type() is *reflect.rtype 0x49d570 
	v_ptr Method(0).Type().Name() String() is  func(int, int) int 
	xxxxx &{zhangsan 10}
	[]
	xxxxx  &{zhangsan 10}
	result is  30
	[<int Value>]
	30

	v Field(1) is reflect.Value zhangsan
	v Field(1).CanSet is true
	v Field(1).Interface is zhangsan
	v FieldByName('Name').Interface is reflect.Value zhangsan 	
