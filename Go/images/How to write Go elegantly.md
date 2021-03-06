### 模块拆分



Go 语言项目中的每一个文件目录都代表着一个**独立的命名空间**，也就是一个单独的包，当我们想要引用其他文件夹的目录时，首先需要使用 `import` 关键字引入相应的文件目录，再通过 `pkg.xxx` 的形式引用其他目录定义的结构体、函数或者常量，这种划分层级的方法在 Go 语言中会显得非常冗余，并且如果对项目依赖包的管理不够谨慎时，很容易发生引用循环，出现这些问题的最根本原因其实也非常简单：

1. Go 语言对同一个项目中不同目录的命名空间做了隔离，整个项目中定义的类和方法并不是在同一个命名空间下的，这也就需要工程师自己维护不同包之间的依赖关系；
2. 按照职责垂直拆分的方式在单体服务遇到瓶颈时非常容易对微服务进行拆分，我们可以直接将一个负责独立功能的 `package` 拆出去，对这部分性能热点单独进行扩容；

