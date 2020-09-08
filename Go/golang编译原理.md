# 一、概述

**Go编译器特点**

* Go 语言的编译器完全用 Go 语言本身来实现 
*  经典的递归下降算法、经典的 SSA 格式的 IR 和 CFG、经典的优化算法、经典的 Lower 和代码生成 
*  除了常规使用，还可以学习到Go运行时（垃圾收集器、并发调度机制等）、标准库和工具链，链接器

## 1.1  编译原理

 编译器的前端和后端，编译器的前端一般承担着词法分析、语法分析、类型检查和中间代码生成几部分工作，而编译器后端主要负责目标代码的生成和优化，也就是将中间代码翻译成目标机器能够运行的二进制机器码。 

词法、语法分析--->类型检查--->AST转换--->SSA生成--->机器码生成

![](D:\gitMyBook\Go\编译原理的核心过程.png)

 [抽象语法树](https://en.wikipedia.org/wiki/Abstract_syntax_tree)（AST） 

![](D:\gitMyBook\Go\抽象语法树.png)

 [静态单赋值](https://en.wikipedia.org/wiki/Static_single_assignment_form)（Static Single Assignment, SSA）是中间代码的一个特性，如果一个中间代码具有静态单赋值的特性，那么每个变量就只会被赋值一次。 

```go
x := 1
x := 2
y := x
```



## 1.2  编译器入口

 Go 语言的编译器入口在 [`src/cmd/compile/internal/gc/main.go`](https://github.com/golang/go/blob/master/src/cmd/compile/internal/gc/main.go) 



# 二、词法、语法分析

# 三、类型检查

# 四、中间代码生成

# 五、机器码生成

# 六、Go交叉编译详解

**Go 可移植性**

- 语言自身被移植到不同平台的容易程度

- 通过这种语言编译出来的应用程序对平台的适应性(Go独立实现了runtime， runtime是支撑程序运行的基础)

  ![](D:\gitMyBook\Go\auto-go-runtime-vs-c-runtime.png)

  libc（C运行时）是目前主流操作系统上应用最普遍的，运行时通常以动态链接库的形式(比如：/lib/x86_64-linux-gnu/libc.so.6)随着系统一并发布，它的功能大致有如下几个：

  - 提供基础库函数调用，比如：strncpy；
  - 封装syscall（注:syscall是操作系统提供的API口，当用户层进行系统调用时，代码会trap(陷入)到内核层面执行），并提供同语言的库函数调用，比如：malloc、fread等；
  - 提供程序启动入口函数，比如：linux下的__libc_start_main。

  独立实现的go runtime层将Go user-level code与OS syscall解耦，把Go porting到一个新平台时，将runtime与新平台的syscall对接即可(当然porting工作不仅仅只有这些)；同时，runtime层的实现基本摆脱了Go程序对libc的依赖，这样静态编译的Go程序具有很好的平台适应性。比如：一个compiled for linux amd64的Go程序可以很好的运行于不同linux发行版（centos、ubuntu）下。 

**查看Go支持OS和平台列表**

```
$ go tool dist list
android/386
android/amd64
android/arm
android/arm64
darwin/386
darwin/amd64
darwin/arm
darwin/arm64
dragonfly/amd64
freebsd/386
freebsd/amd64
freebsd/arm
linux/386
linux/amd64
linux/arm
linux/arm64
linux/mips
linux/mips64
linux/mips64le
linux/mipsle
linux/ppc64
linux/ppc64le
linux/s390x
nacl/386
nacl/amd64p32
nacl/arm
netbsd/386
netbsd/amd64
netbsd/arm
openbsd/386
openbsd/amd64
openbsd/arm
plan9/386
plan9/amd64
plan9/arm
solaris/amd64
windows/386
windows/amd64
```

**查看硬件信息**

```bash
$ uname -m
x86_64  //x86 是目前比较常见的指令集
```

**Go 交叉编译**

```bash
// GOOS(目标平台的操作系统)可选darwin(mac)、freebsd(unix)、linux、windows
// GOARCH(目标平台的体系架构)可选386、amd64、arm
$ GOARCH=wasm GOOS=js go build -o lib.wasm main.go
$ GOARCH=amd64 GOOS=linux go build main.go
```

**Go 语言支持的架构** 

```go
# https://github.com/golang/go/blob/master/src/go/build/syslist.go
package build
const goosList = "android darwin dragonfly freebsd js linux nacl netbsd openbsd plan9 solaris windows zos "
const goarchList = "386 amd64 amd64p32 arm armbe arm64 arm64be ppc64 ppc64le mips mipsle mips64 mips64le mips64p32 mips64p32le
```

![](D:\gitMyBook\Go\Go 语音支持的架构.png)

**Mac 下编译 Linux和 Windows 64位可执行程序（交叉编译不支持 CGO 所以要禁用它）**

```bash
$ CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build 
$ CGO_ENABLED=0 GOOS=windows GOARCH=amd64 go build 
```

**Linux下编译 Mac 和 Windows 64位可执行程序**

```bash
$ CGO_ENABLED=0 GOOS=darwin GOARCH=amd64 go build 
$ CGO_ENABLED=0 GOOS=windows GOARCH=amd64 go build
```

**Windows 下编译 Mac 和 Linux 64位可执行程序 **

```bash
SET CGO_ENABLED=0 SET GOOS=darwin SET GOARCH=amd64 go build main.go
SET CGO_ENABLED=0 SET GOOS=linux SET GOARCH=amd64 go build main.go
```

**Go默认静态链接**

* 对比 hello.c和hello.go 

  ```c
  //hello.c
  #include <stdio.h>
   
  int main() {
          printf("%s\n", "hello, portable c!");
          return 0;
  }
   
  //hello.go
  package main
   
  import "fmt"
   
  func main() {
      fmt.Println("hello, portable go!")
  }
  ```

*  采用“默认”方式分别编译 (helloc和hellogo的size对比)

  ```bash
  $ cc -o helloc hello.c
  $ go build -o hellogo hello.go
   
  $ ls -l
  ```

*  查看一下两个文件的对外部动态库的依赖情况 

  ```bash
  $ ldd --help
  $ ldd -v helloc
  $ ldd -v hellogo
  ```

*  通过nm工具可以查看到helloc具体是哪个函数符号需要由外部动态库提供(未定义的对应的前缀符号是U)

  ```bash
  $ nm helloc |grep " U "
  $ nm hellogo |grep " U "
  ```

* Go默认编译中也有动态链接(外部依赖)

  ```go
  //server.go
  package main
   
  import (
      "log"
      "net/http"
      "os"
  )
   
  func main() {
      cwd, err := os.Getwd()
      if err != nil {
          log.Fatal(err)
      }
   
      srv := &http.Server{
          Addr:    ":8000", // Normally ":443"
          Handler: http.FileServer(http.Dir(cwd)),
      }
      log.Fatal(srv.ListenAndServe())
  }
  ```

  ```bash
  $ go build server.go
  $ ldd -u server
  $ nm server |grep " U "
  ```

**cgo对可移植性的影响**

 默认情况下，Go的runtime环境变量CGO_ENABLED=1，即默认开始cgo，允许你在Go代码中调用C代码，Go的pre-compiled标准库的.a文件也是在这种情况下编译出来的。.a路径：$GOROOT/pkg/XXX_amd64。

在CGO_ENABLED=1，即cgo开启的情况下，os/user包中的XXX系列函数采用了c版本的实现，我们看到在$GOROOT/src/os/user/XXX.go中的build tag中包含了 **build cgo**。这样一来，在CGO_ENABLED=1，该文件将被编译，该文件中的c版本实现的XXX将被使用。

```bash
$ which go
$ nm os.a |grep " U "
=> os/user.a
                 U __cgo_topofstack
                 U _free
                 U _getgrgid_r
                 U _getgrnam_r
                 U _getgrouplist
                 U _getpwnam_r
                 U _getpwuid_r
                 U _malloc
                 U _realloc
                 U _sysconf
                 
$ find ./ -name "*.go" |xargs grep "build cgo"
```

 凡是依赖上述包的Go代码最终编译的可执行文件都是要有外部依赖的。不过我们依然可以通过disable CGO_ENABLED来编译出纯静态的Go程序 ，CGO_ENABLED=0

```bash
$ CGO_ENABLED=0 go build -o server_cgo_disabled server.go
$ ldd -v server_cgo_disabled
server_cgo_disabled:
$ nm server_cgo_disabled |grep " U "
```

**internal linking和external linking**

在CGO_ENABLED=1这个默认值的情况下，也可以实现纯静态连接。在$GOROOT/cmd/cgo/doc.go中，文档介绍了cmd/link的两种工作模式：internal linking和external linking。 

* ### internal linking

   internal linking的大致意思是若用户代码中仅仅使用了net、os/user等几个标准库中的依赖cgo的包时，cmd/link默认使用internal linking，而无需启动外部external linker(如:gcc、clang等)，不过由于cmd/link功能有限，仅仅是将.o和pre-compiled的标准库的.a写到最终二进制文件中。因此如果标准库中是在CGO_ENABLED=1情况下编译的，那么编译出来的最终二进制文件依旧是动态链接的，即便在go build时传入 -ldflags '-extldflags "-static"'亦无用，因为根本没有使用external linker：

  ```bash
  $ go build -o server-fake-static-link  -ldflags '-extldflags "-static"' server.go
  $ nm server-fake-static-link |grep " U "
  ```

* ### external linking

   external linking机制则是cmd/link将所有生成的.o都打到一个.o文件中，再将其交给外部的链接器，比如gcc或clang去做最终链接处理。如果此时，我们在cmd/link的参数中传入 -ldflags '-linkmode "external" -extldflags "-static"'，那么gcc/clang将会去做静态链接，将.o中undefined的符号都替换为真正的代码。我们可以通过-linkmode=external来强制cmd/link采用external linker：

  ```bash
  $ go build -o server-static-link  -ldflags '-linkmode "external" -extldflags "-static"' server.go
  $ nm server-static-link |grep " U "
  ```

   上述命令，在CGO_ENABLED=1的情况下，也编译构建出了一个纯静态链接的Go程序。 

**小结**

- 程序用了哪些标准库包？如果仅仅是非net、os/user等的普通包，那么你的程序默认将是纯静态的，不依赖任何libc等外部动态链接库；
- 如果使用了net这样的包含cgo代码的标准库包，那么CGO_ENABLED的值将影响你的程序编译后的属性：是静态的还是动态链接的；
- CGO_ENABLED=0的情况下，Go采用纯静态编译；
- 如果CGO_ENABLED=1，但依然要强制静态编译，需传递-linkmode=external给cmd/link。

































参考

https://johng.cn/cgo-enabled-affect-go-static-compile/