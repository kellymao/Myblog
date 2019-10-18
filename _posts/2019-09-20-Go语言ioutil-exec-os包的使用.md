---
layout: post
title: "Go 语言io/ioutil,os,exec包使用"
date: 2019-09-18 
description: "Go 语言io/ioutil,os,exec包使用"
tag: GO语言
---   
  


# Go 语言socket读写总结



## 字符串实现io.Reader 的方法：




    s := strings.NewReader("Hello World!")
    ra, _ := ioutil.ReadAll(s)
    fmt.Printf("%s", ra)
	


## io/ioutil包：



几个函数方法

|名称 	| 作用	| 备注
|------|------| 
|ReadAll	| 读取数据，返回读到的字节 slice	| 1
|ReadDir	| 读取一个目录，返回目录入口数组 []os.FileInfo, | 	2
|ReadFile	| 读一个文件，返回文件内容（字节slice）	| 3
|WriteFile	| 根据文件路径，写入字节slice	| 4
|TempDir	| 在一个目录中创建指定前缀名的临时目录，返回新临时目录的路径	| 5
|TempFile	| 在一个目录中创建指定前缀名的临时文件，返回 os.File	| 6


**1) func ReadAll(r io.Reader) ([]byte, error)** 

	r := strings.NewReader("Go is a general-purpose language designed with systems programming in mind.")

	b, err := ioutil.ReadAll(r)
	if err != nil {
		log.Fatal(err)
	}

	fmt.Printf("%s", b)

	/*output:
	Go is a general-purpose language designed with systems programming in mind.
	*/ 

	
**2) func ReadFile(filename string) ([]byte, error)**

	func main() {
		ra, _ := ioutil.ReadFile("/tmp/win.ini")
		fmt.Printf("%s", ra)
	}

**3) func WriteFile(filename string, data []byte, perm os.FileMode) error**
	
// WriteFile 向文件 filename 中写入数据 data

// 如果文件不存在，则以 perm 权限创建该文件

// 如果文件存在，则先清空文件，然后再写入

// 返回写入过程中遇到的任何错误

	func main() {
		fn := "/tmp/Test.txt"
		s := []byte("Hello World!")
		ioutil.WriteFile(fn, s, os.ModeAppend)
		rf, _ := ioutil.ReadFile(fn)
		fmt.Printf("%s", rf)
		// Hello World!
	}

**4) func ReadDir(dirname string) ([]os.FileInfo, error)**

// ReadDir 读取目录 dirmane 中的所有目录和文件（不包括子目录）

// 返回读取到的文件的信息列表和读取过程中遇到的任何错误

// 返回的文件列表是经过排序的

	func main() {
		rd, err := ioutil.ReadDir("C:\\Windows")
		for _, fi := range rd {
			fmt.Println("")
			fmt.Println(fi.Name())
			fmt.Println(fi.IsDir())
			fmt.Println(fi.Size())
			fmt.Println(fi.ModTime())
			fmt.Println(fi.Mode())
		}
		fmt.Println("")
		fmt.Println(err)
	}

示例：读取目录

	func main() {
		rd, err := ioutil.ReadDir("/")
		fmt.Println(err)
		for _, fi := range rd {
			if fi.IsDir() {
				fmt.Printf("[%s]\n", fi.Name())

			} else {
				fmt.Println(fi.Name())
			}
		}
	}	
	
**5) ioutil.Discard 相当于/dev/null**	

var Discard io.Writer = devNull(0)

Discard 是一个 io.Writer，对它进行的任何 Write 调用都将无条件成功。devNull优化的实现了ReadFrom，因此io.Copy到ioutil.Discard避免了不必要的工作，因此其一定会成功．但是ioutil.Discard不记录copy得到的数值．例子如下：

	package main

	import (
		"fmt"
		"io"
		"io/ioutil"
		"strings"
	)

	func main() {
		

		a := strings.NewReader("hello")
		p := make([]byte, 20)
		io.Copy(ioutil.Discard, a)
		ioutil.Discard.Write(p)
		fmt.Println(p)　　　　　　//[0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0]
	}

**6) func TempFile(dir, prefix string) (f *os.File, err error)**

// TempFile 在目录 dir 中创建一个临时文件并将其打开

// 文件名以 prefix 为前缀

// 返回创建的文件的对象和创建过程中遇到的任何错误

// 如果 dir 为空，则在系统的临时目录中创建临时文件

// 如果环境变量中没有设置系统临时目录，则在 /tmp 中创建临时文件

// 调用者可以通过 f.Name() 方法获取临时文件的完整路径

// 调用 TempFile 所创建的临时文件，应该由调用者自己移除

	func main() {
		dn := "C:\\"
		f, _ := ioutil.TempFile(dn, "Test")
		fmt.Printf("%s", f.Name())
	}


**6) func TempDir(dir, prefix string) (name string, err error)**

// TempDir 功能同 TempFile，只不过创建的是目录

// 返回值也只返目录的完整路径

	func main() {
		dn := "C:\\"
		f, _ := ioutil.TempDir(dn, "Test")
		fmt.Printf("%s", f)
	}

	// 完整的例子 
	
	func main() {
	// 创建临时目录
	dir, err := ioutil.TempDir("", "Test")
	if err != nil {
		fmt.Println(err)
	}
	defer os.Remove(dir) // 用完删除
	fmt.Printf("%s\n", dir)

	// 创建临时文件
	f, err := ioutil.TempFile(dir, "Test")
	if err != nil {
		fmt.Println(err)
	}
	defer os.Remove(f.Name()) // 用完删除
	fmt.Printf("%s\n", f.Name())
	}
	
<br> 

参考： 

[ioutil总结](https://www.cnblogs.com/msnsj/p/4242600.html)

[ioutil临时目录文件](https://blog.csdn.net/zf766045962/article/details/89243709)

##  bufio 总结：

	r = bufio.NewReader()

	buf:=[512]byte
	n,err:=r.Read(buf) 
	fmt.Println(buf[0:n]

	data,err:=r.ReadString('\n')
	fmt.Println(string(data))


	w = bufio.NewWriter() 
	w.WriteString("xxxxxxx\n") 
	w.Write([]byte("hello")) 



	
##  io 总结： 



	io.WriteString(w,"xxxxxx\n") 


	io.Copy(w,r) 




# golang语言中os包的学习与使用(文件，目录，进程的操作)




## os 常用的函数

	package main;
	 
	import (
		"os"
		"fmt"
		"time"
		"strings"
	)
	 
	//os包中的一些常用函数
	 
	func main() {
	 
		//获取主机名
		fmt.Println(os.Hostname());
	 
		//获取当前目录
		fmt.Println(os.Getwd());
	 
		//获取用户ID
		fmt.Println(os.Getuid());
	 
		//获取有效用户ID
		fmt.Println(os.Geteuid());
	 
		//获取组ID
		fmt.Println(os.Getgid());
	 
		//获取有效组ID
		fmt.Println(os.Getegid());
	 
		//获取进程ID
		fmt.Println(os.Getpid());
	 
		//获取父进程ID
		fmt.Println(os.Getppid());
	 
		//获取环境变量的值
		fmt.Println(os.Getenv("GOPATH"));
	 
		//设置环境变量的值
		os.Setenv("TEST", "test");
	 
		//改变当前工作目录
		os.Chdir("C:/");
		fmt.Println(os.Getwd());
	 
		//创建文件
		f1, _ := os.Create("./1.txt");
		defer f1.Close();
	 
		//修改文件权限
		if err := os.Chmod("./1.txt", 0777); err != nil {
			fmt.Println(err);
		}
	 
		//修改文件所有者
		if err := os.Chown("./1.txt", 0, 0); err != nil {
			fmt.Println(err);
		}
	 
		//修改文件的访问时间和修改时间
		os.Chtimes("./1.txt", time.Now().Add(time.Hour), time.Now().Add(time.Hour));
	 
		//获取所有环境变量
		fmt.Println(strings.Join(os.Environ(), "\r\n"));
	 
		//把字符串中带${var}或$var替换成指定指符串
		fmt.Println(os.Expand("${1} ${2} ${3}", func(k string) string {
			mapp := map[string]string{
				"1": "111",
				"2": "222",
				"3": "333",
			};
			return mapp[k];
		}));
	 
		//创建目录
		os.Mkdir("abc", os.ModePerm);
	 
		//创建多级目录
		os.MkdirAll("abc/d/e/f", os.ModePerm);
	 
		//删除文件或目录
		os.Remove("abc/d/e/f");
	 
		//删除指定目录下所有文件
		os.RemoveAll("abc");
	 
		//重命名文件
		os.Rename("./2.txt", "./2_new.txt");
	 
		//判断是否为同一文件
		//unix下通过底层结构的设备和索引节点是否相同来判断
		//其他系统可能是通过文件绝对路径来判断
		fs1, _ := f1.Stat();
		f2, _ := os.Open("./1.txt");
		fs2, _ := f2.Stat();
		fmt.Println(os.SameFile(fs1, fs2));
	 
		//返回临时目录
		fmt.Println(os.TempDir());
	}

## 文件的操作

	package main;
	 
	import (
		"os"
		"fmt"
		"strconv"
	)
	 
	//os包中关于文件的操作函数
	 
	func main() {
		//创建文件，返回一个文件指针
		f3, _ := os.Create("./3.txt");
		defer f3.Close();
	 
		//以读写方式打开文件，如果不存在则创建文件，等同于上面os.Create
		f4, _ := os.OpenFile("./4.txt", os.O_RDWR|os.O_CREATE|os.O_TRUNC, 0666);
		defer f4.Close();
	 
		//打开文件，返回文件指针
		f1, _ := os.Open("./1.txt");
		defer f1.Close();
	 
		//修改文件权限，类似os.chmod
		f1.Chmod(0777);
	 
		//修改文件所有者，类似os.chown
		f1.Chown(0, 0);
	 
		//返回文件的句柄，通过NewFile创建文件需要文件句柄
		fmt.Println(f1.Fd());
	 
		//从文件中读取数据
		buf := make([]byte, 128);
		//read每次读取数据到buf中
		for n, _ := f1.Read(buf); n != 0; n, _ = f1.Read(buf) {
			fmt.Println(string(buf[:n]));
		}
	 
		//向文件中写入数据
		for i := 0; i < 5; i++ {
			f3.Write([]byte("写入数据" + strconv.Itoa(i) + "\r\n"));
		}
	 
		//返回一对关联的文件对象
		//从r中可以读取到从w写入的数据
		r, w, _ := os.Pipe();
		//向w中写入字符串
		w.WriteString("写入w");
		buf2 := make([]byte, 128);
		//从r中读取数据
		n, _ := r.Read(buf);
		fmt.Println(string(buf2[:n]));
	 
		//改变工作目录
		os.Mkdir("a", os.ModePerm);
		dir, _ := os.Open("a");
		//改变工作目录到dir，dir必须为一个目录
		dir.Chdir();
		fmt.Println(os.Getwd());
	 
		//读取目录的内容，返回一个FileInfo的slice
		//参数大于0，最多返回n个FileInfo
		//参数小于等于0，返回所有FileInfo
		fi, _ := dir.Readdir(-1);
		for _, v := range fi {
			fmt.Println(v.Name());
		}
	 
		//读取目录中文件对象的名字
		names, _ := dir.Readdirnames(-1);
		fmt.Println(names);
	 
		//获取文件的详细信息，返回FileInfo结构
		fi3, _ := f3.Stat();
		//文件名
		fmt.Println(fi3.Name());
		//文件大小
		fmt.Println(fi3.Size());
		//文件权限
		fmt.Println(fi3.Mode());
		//文件修改时间
		fmt.Println(fi3.ModTime());
		//是否是目录
		fmt.Println(fi3.IsDir());
	}	

	
参考：

[go语言一些模块的使用](https://www.cnblogs.com/jkko123/category/991658.html)	


# golang语言中os/exec包的学习与使用

**1) func LookPath(file string) (string, error)**

在环境变量PATH指定的目录中搜索可执行文件，如file中有斜杠，则只在当前目录搜索。

**2) func Command(name string, arg ...string) *Cmd**

函数返回一个*Cmd，用于使用给出的参数执行name指定的程序。返回值只设定了Path和Args两个参数。

如果name不含路径分隔符（如果不是相对路径），将使用LookPath获取完整路径（就是用默认的全局变量路径）；否则直接使用name。参数arg不应包含命令名。

	cmd := exec.Command("go","version")
	fmt.Println(cmd.Args, cmd.Path)
	
注：在调用命令执行封装时，如果不提供相对路径，系统会使用LookPath获取完整路径；即这里可以给一个相对路径。

**管道的使用:我们可以通过指定一个对象连接到对应的管道进行传输参数（stdinpipe）,获取输出（stdoutpipe）,获取错误（stderrpipe）**

**3) func (c *Cmd) StdinPipe() (io.WriteCloser, error)　　//err 返回的是执行函数时的错误**

	package main

	import (
		"fmt"
		"os"
		"os/exec"
	)

	func main() {
		cmd := exec.Command("cat")
		stdin, err := cmd.StdinPipe()       //指定stdin连接StdinPipe,然后操作stdin就实现了对command的参数传递
		if err != nil {
			fmt.Println(err)
		}
		_, err = stdin.Write([]byte("tmp.txt"))   //字节切片
		if err != nil {
			fmt.Println(err)
		}
		stdin.Close()
		cmd.Stdout = os.Stdout     //终端标准输出tmp.txt
		cmd.Start()
	}

StdinPipe方法返回一个在命令Start后与命令标准输入关联的管道。Wait方法获知命令结束后会关闭这个管道。必要时调用者可以调用Close方法来强行关闭管道，例如命令在输入关闭后才会执行返回时需要显式关闭管道。

**4) func (c *Cmd) StdoutPipe() (io.ReadCloser, error)　　　　//err 返回的是执行函数时的错误**

	func main() {
		cmd := exec.Command("ls")
		stdout, err := cmd.StdoutPipe()　　//指向cmd命令的stdout，然后就可以从stdout读出信息
		cmd.Start()
		content, err := ioutil.ReadAll(stdout)
		if err != nil {
			fmt.Println(err)
		}
		fmt.Println(string(content))     //输出ls命令查看到的内容
	}

StdoutPipe方法返回一个在命令Start后与命令标准输出关联的管道。Wait方法获知命令结束后会关闭这个管道，一般不需要显式的关闭该管道。但是在从管道读取完全部数据之前调用Wait是错误的；同样使用StdoutPipe方法时调用Run函数也是错误的。	
	
	
**5) func (c *Cmd) StderrPipe() (io.ReadCloser, error)   //err 返回的是执行函数时的错误**

	import (
	"fmt"
	"io/ioutil"
	"os/exec"
	)

	func main() {
		c := exec.Command("mv", "hello")
		i, err := c.StderrPipe()
		if err != nil {
			fmt.Printf("Error: %s\n", err)
			return
		}
		if err = c.Start(); err != nil {　　　　//表示命令正确执行了
			fmt.Printf("Error: %s\n", err)
		}
		b, _ := ioutil.ReadAll(i)　　　　　　　　//读取命令执行的返回结果（命令执行结果的错误信息）
		if err := c.Wait(); err != nil {
			fmt.Printf("Error: %s\n", err) 　　//Error: exit status 1 mv: missing file argument Try `mv --help' for more information.
		}                                                                 
		fmt.Println(string(b))
	}

StderrPipe方法返回一个在命令Start后与命令标准错误输出关联的管道。Wait方法获知命令结束后会关闭这个管道，一般不需要显式的关闭该管道。但是在从管道读取完全部数据之前调用Wait是错误的；同样使用StderrPipe方法时调用Run函数也是错误的

**6) func (c *Cmd) Run() error**

**Run执行c包含的命令，并阻塞直到完成。**

如果命令成功执行，stdin、stdout、stderr的转交没有问题，并且返回状态码为0，方法的返回值为nil【执行Run函数的返回状态，正确执行Run函数，并不代表正确执行了命令】；如果函数没有执行或者执行失败，会返回*ExitError类型的错误；否则返回的error可能是表示I/O问题。

即：该命令只会执行且阻塞到执行结束，如果执行函数有错则返回报错信息，没错则返回nil，并不会返回执行结果。


**7) func (c *Cmd) Start() error**

Start开始执行c包含的命令，但并不会等待该命令完成即返回。Wait方法会返回命令的返回状态码并在命令返回后释放相关的资源。


**8) func (c *Cmd) Wait() error**

Wait会阻塞直到该命令执行完成，该命令必须是被Start方法开始执行的。


**9) func (c *Cmd) Output() ([]byte, error)**

执行命令并返回标准输出的切片。不用通过pipe方式获取命令的执行结果

	
	import (
	"fmt"
	"os/exec"
	)

	func main() {
		c, err := exec.Command("date").Output()
		if err != nil {
			fmt.Println(err)
		}
		fmt.Println(string(c)) //    Sat Jan 4 17:07:36 2014 这个是标准库里的例子
	}

**10) func (c *Cmd) CombinedOutput() ([]byte, error)**

**执行命令并返回标准输出和错误输出合并的切片.**


参考：

[https://www.cnblogs.com/sunailong/p/7852216.html](https://www.cnblogs.com/sunailong/p/7852216.html)

[https://www.cnblogs.com/jkko123/p/7144938.html](https://www.cnblogs.com/jkko123/p/7144938.html)	
	
一个具体的例子： 

	
	package main;
	 
	import (
		"os/exec"
		"fmt"
		"io/ioutil"
		"bytes"
	)
	 
	func main() {
		//在环境变量path中查找可执行二进制文件
		//返回完整路径或者相对于当前目录的一个相对路径
		file, _ := exec.LookPath("go");
		fmt.Println(file);
	 
		//返回一个cmd
		cmd := exec.Command("go", "version");
		//执行命令，并返回标准输出和错误输出
		out, _ := cmd.CombinedOutput();
		fmt.Println(string(out));
	 
		//创建一个cmd
		cmd2 := exec.Command("ping", "www.baidu.com");
		buf := bytes.Buffer{};
		//将cmd2的标准输出设置为buf
		cmd2.Stdout = &buf;
		//运行命令，阻塞直到完成
		cmd2.Run();
		fmt.Println(buf.String());
	 
		//创建一个cmd
		cmd3 := exec.Command("ping", "www.baidu.com");
		//获取命令在start后标准输出管道
		out3, _ := cmd3.StdoutPipe();
		//执行命令
		cmd3.Start();
		//读取管道中所有数据
		data3, _ := ioutil.ReadAll(out3);
		//等待命令执行完成
		cmd3.Wait();
		fmt.Println(string(data3));
	}


获取命令的返回码：

[https://studygolang.com/articles/11128?fr=sidebar](https://studygolang.com/articles/11128?fr=sidebar)
	
	
	
python celery 定时任务添加：

https://www.cnblogs.com/52forjie/p/9364136.html 


### 输出文件到终端

	func print1(){
	
		/*
	
		延迟输出，最后统一输出结果
	
		 */
	
		//创建一个cmd
		cmd3 := exec.Command("ping","-c","3", "www.baidu.com")
		//获取命令在start后标准输出管道
		out3, err := cmd3.StdoutPipe()
	
		if err != nil {
			fmt.Printf("stdoutpipe Error: %s\n", err)
			return
		}
		//执行命令
		if err = cmd3.Start(); err != nil { //表示命令正确执行了
			fmt.Printf("startError: %s\n", err)
		}
		//读取管道中所有数据
		data3, _ := ioutil.ReadAll(out3)
		//等待命令执行完成
		if err := cmd3.Wait(); err != nil {
			fmt.Printf("waitError: %s\n", err)
		}
		fmt.Println(string(data3))
	
	}
	
	
	
	func print2(){
		/*
	
		实时的输出
	
	
		 */
	
		//创建一个cmd
		cmd3 := exec.Command("ping","-c","3", "www.google.com")
		//获取命令在start后标准输出管道
		//out3, err := cmd3.StdoutPipe()
	
		cmd3.Stdout = os.Stdout
		cmd3.Stderr = os.Stderr
	
	
		//执行命令
		if err := cmd3.Start(); err != nil { //表示命令正确执行了
			fmt.Printf("startError: %s\n", err)
		}
	
		if err := cmd3.Wait(); err != nil {
			fmt.Printf("waitError: %s\n", err)
		}
	
	
	   //https://blog.csdn.net/zhuxinquan61/article/details/89716301
	
	}


实时输出exec结果到文件或终端:

[https://blog.csdn.net/zhuxinquan61/article/details/89716301](https://blog.csdn.net/zhuxinquan61/article/details/89716301)

