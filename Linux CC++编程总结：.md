# Linux C/C++编程总结

## 基础知识：

### 1.GCC部分：

gcc是是由GNU开发的编译器，现已被大多数类Unix操作系统采纳为标准的编译器。

gcc 与 g++ 分别是 gnu 的 c & c++ 编译器 gcc/g++ 在执行编译工作的时候，总共需要4步：

- 1、预处理,生成 .i 的文件[预处理器cpp]
- 2、将预处理后的文件转换成汇编语言, 生成文件 .s [编译器egcs]
- 3、有汇编变为目标代码(机器代码)生成 .o 的文件[汇编器as]
- 4、连接目标代码, 生成可执行程序 [链接器ld]

gcc 是 GCC 编译器的通用编译指令，因为根据程序文件的后缀名，gcc 指令可以自行判断出当前程序所用编程语言的类别。而g++ 指令，则无论目标文件的后缀名是什么，该指令都一律按照编译 C++ 代码的方式编译该文件。

### gcc 命令的常用选项

| 选项       | 解释                                        |
| :--------- | ------------------------------------------- |
| -c         | 只编译并生成目标文件。                      |
| -E         | 只运行 C 预编译器预处理源文件，不进行汇编。 |
| -g         | 生成调试信息。GNU 调试器可利用该信息。      |
| -l LIBRARY | 连接时搜索指定的函数库LIBRARY。             |
| -o FILE    | 生成指定的输出文件。用在生成可执行文件时。  |
| -O0        | 不进行优化处理。                            |
| -O 或 -O1  | 优化生成代码。                              |
| -O2        | 进一步优化。                                |
| -O3        | 比 -O2 更进一步优化，包括 inline 函数。     |
| -shared    | 生成共享目标文件。通常用在建立共享库时。    |
| -static    | 禁止使用共享连接。                          |
| -w         | 不生成任何警告信息。                        |
| -Wall      | 生成所有警告信息。                          |

### 2.Makefile文件：

Makefile文件可以实现工程的自动化编译，定义了一系列规则来指定哪些文件需要先编译，哪些文件需要后编译，哪些文件需要重新编译，甚至于进行更复杂的功能操作，并且通过一个make命令即可执行，极大的提高了软件开发的效率。Makefile来告诉make命令如何编译和链接这几个文件，其规则为：

​      **1.如果这个工程没有编译过，那么我们的所有C文件都要编译并被链接。**

​      **2.如果这个工程的某几个C文件被修改，那么我们只编译被修改的C文件，并链接目标程序。**

​      **3.如果这个工程的头文件被改变了，那么我们需要编译引用了这几个头文件的C文件，并链接目标程序**

关于Makefile的语法规则，由目标，依赖和命令三个部分组成，下面举例：

~~~makefile
main:main.o test1.o test2.o
	gcc main.o test1.o test2.o -o main
#通配符的使用：
OBJ=*.c
test:$(OBJ)
    gcc -o $@ $^
#变量的定义和使用
#调用变量的时候可以用"$(VALUE_LIST)"或者是"${VALUE_LIST}"来替换\:
OBJ=main.o test.o test1.o test2.o
test:$(OBJ)
      gcc -o test $(OBJ)
~~~

如果要定义一系列比较类似的文件，可以通过使用通配符来简化代码。关于通配符的使用规则：

| 通配符 | 使用说明                           |
| ------ | ---------------------------------- |
| *      | 匹配0个或者是任意个字符            |
| ？     | 匹配任意一个字符                   |
| []     | 我们可以指定匹配的字符放在 "[]" 中 |

除了这三个通配符外，波浪号（“~”）字符在文件名中也有比较特殊的用途。如果是“~/test”，这就表示当前用户的$HOME目录下的test目录。而“~hchen/test”则表示用户hchen的宿主目录下的test目录。而在Windows或是MS-DOS下，用户没有宿主目录，那么波浪号所指的目录则根据环境变量“HOME”而定。

在 Makefile中的定义的变量，类似于是C/C++语言中宏，代表了一个文本字串，在Makefile中执行的时候其会自动原样地展开在所使用的地方。其与C/C++所不同的是，你可以在Makefile中改变其值。变量的命名可以包含字符、数字，下划线（可以是数字开头），但不应该含有“:”、“#”、“=”或是空字符（空格、回车等）。除此之外，还有一些变量是特殊的字符串串，如“$<”、“$@”等，这些是自动化变量，代表不同文件。

变量的基本赋值：

-  简单赋值 ( := ) 编程语言中常规理解的赋值方式，只对当前语句的变量有效。
-  递归赋值 ( = ) 赋值语句可能影响多个变量，所有目标变量相关的其他变量都受影响。

~~~makefile
x=foo		y=$(x)b		x=new   #这种情况下x=new, y=newb，若是简单赋值则y值不变仍为foob
~~~

-  条件赋值 ( ?= ) 如果变量未定义，则使用符号中的值定义变量。如果该变量已经赋值，则该赋值语句无效。
-  追加赋值 ( += ) 原变量用空格隔开的方式追加一个新值。

自动化变量的表示规则：

| 变量 | 规则说明                                                     |
| ---- | ------------------------------------------------------------ |
| $@   | 表示规则的**目标文件**名。如果目标是一个文档文件（Linux 中，一般成 .a 文件为文档文件，也成为静态的库文件）， 那么它代表这个文档的文件名。在多目标模式规则中，它代表的是触发规则被执行的文件名。 |
| $%   | 当目标文件是一个静态库文件时，代表**静态库的一个成员名**。   |
| $<   | 规则的**第一个依赖的文件名**。如果是一个目标文件使用隐含的规则来重建，则它代表由隐含规则加入的第一个依赖文件。 |
| $?   | 所有比目标文件更新的依赖文件列表，即**被修改过的文件**，空格分隔。如果目标文件时静态库文件，代表的是库文件（.o 文件）。 |
| $^   | 代表的是**所有依赖文件列表**，使用空格分隔。如果目标是静态库文件，它所代表的只能是所有的库成员（.o 文件）名。 一个文件可重复的出现在目标的依赖中，变量“$^”只记录它的第一次引用的情况。就是说变量“$^”会去掉重复的依赖文件。 |
| $+   | 类似“$^”，但是它**保留了依赖文件中重复出现的文件**。主要用在程序链接时库的交叉引用场合。 |

除了变量和通配符，Makefile文件还支持函数的定义与使用。函数的语法类似于变量，也是以“$”来标识的，其格式为$(<function> <arguments> )，下面以字符串处理函数（subst）为例：

~~~makefile
#$(subst <from>,<to>,<text> )
	#名称：字符串替换函数——subst。
	#功能：把字串<text>中的<from>字符串替换成<to>。
	#返回：函数返回被替换过后的字符串。

$(subst ee,EE,feet on the street),

#返回结果是“fEEt on the strEEt
~~~

### 3.文件操作和目录操作相关：

**open函数：**

~~~c++
#include<sys/types.h> 
#include<sys/stat.h>	
#include<fcntl.h>
//打开一个已经存在的文件
int open(const char *pathname, int flags);
	//- pathname：要打开的文件路径
	//- flags：对文件的操作权限设置还有其他的设置
	//O_RDONLY,  O_WRONLY,  O_RDWR  这三个设置是互斥的
	//O_APPEND 每次写时都加到文件的尾端
	//O_CREAT  若文件不存在则创建它,同时说说明mode字段,写明该文件的权限
	//返回值：返回一个新的文件描述符，如果调用失败，返回-1
//创建一个新的文件
int open(const char *pathname, int flags, mode_t mode);
	//flags中要有O_CREAT，若文件不存在，则创建一个新文件
	//参数mode通常会和umask掩码做与运算，用来控制创建文件的权限
	//mask设置了用户创建文件的默认权限和mode做与运算是为了防止赋予文件不应该有的权限
~~~

**读写函数：**

~~~c++
#include<unistd.h>
ssize_t read(int fd, void *buf, size_t count);
    //- fd：文件描述符，open得到的，通过这个文件描述符操作某个文件
    //- buf：需要读取数据存放的地方，数组的地址（传出参数）
    //- count：指定的数组的大小
    //返回值：- 成功：>0: 返回实际的读取到的字节数 或者 =0：文件已经读取完了
		  //- 失败：-1 ，并且设置errno
//write函数同read函数参数相同，实际操作相反。
ssize_t write(int fd, const void *buf, size_t count);
~~~

**文件指针偏移函数：Lseek**

~~~c++
#include<unistd.h>	
#include <sys/types.h>
off_t lseek(int fd, off_t offset, int whence);
	//- offset：偏移量
	//- whence:有三个宏定义，分别是
		//SEEK_SET——设置文件指针的偏移量
    	//SEEK_CUR——设置偏移量：当前位置 + 第二个参数offset的值
    	//SEEK_END——设置偏移量：文件大小 + 第二个参数offset的值
    //返回值：返回文件指针的位置
//应用举例：
//1.移动文件指针到文件头
lseek(fd, 0, SEEK_SET);
//2.获取当前文件指针的位置
lseek(fd, 0, SEEK_CUR);
//3.获取文件长度
lseek(fd, 0, SEEK_END);
//4.拓展文件的长度，当前文件10b, 110b, 增加了100个字节
lseek(fd, 100, SEEK_END)
~~~

**文件属性操作函数：**

~~~c++
#include <unistd.h>
int access(const char *pathname, int mode);
//作用：判断某个文件是否有某个权限，或者判断文件是否存在
	//- pathname: 文件路径
    //- mode:
		//R_OK: 判断是否有读权限  W_OK: 判断是否有写权限
        //X_OK: 判断是否有执行权限  F_OK: 判断文件是否存在
    //返回值：成功返回0， 失败返回-1

//头文件	#include <sys.stat.h>
int chmod(const char *pathname, mode_t mode);
//修改文件的权限 - mode:需要修改的权限值，八进制的数

//头文件	#inclide <unistd.h> 	#include<sys/types.h>
int truncate(const char *path, off_t length);
//作用：缩减或者扩展文件的尺寸至指定的大小
	//- path: 需要修改的文件的路径
	//- length: 需要最终文件变成的大小
    //成功返回0， 失败返回-1
~~~

**目录操作相关函数：**

~~~c++
//创建目录
int mkdir(const char *pathname, mode_t mode);
//修改当前工作目录
int chdir(const char *path);
//获取当前工作目录，返回的指向的一块内存，这个数据就是第一个参数
char *getcwd(char *buf, size_t size);
//重命名目录
int rename(const char *oldpath, const char *newpath);
~~~

**文件描述符相关函数：**

~~~c++
#include<unistd.h>
int dup(int oldfd);
7//作用：复制一个新的文件描述符
fd=3; int fd1 = dup(fd);
//fd指向的是a.txt, fd1也是指向a.txt
//从空闲的文件描述符表中找一个最小的，作为新的拷贝的文件描述符
int dup2(int oldfd, int newfd);
//作用：重定向文件描述符
	//--oldfd 指向 a.txt, newfd 指向 b.txt
    //调用函数成功后：newfd 和 b.txt 做close, newfd 指向了 a.txt
    //oldfd 必须是一个有效的文件描述符
    //若oldfd和newfd值相同，相当于什么都没有做
int fcntl(int fd, int cmd, ...);
//作用：复制文件描述符、设置/获取文件的状态标志等
	//fd : 表示需要操作的文件描述符
    //cmd: 表示对文件描述符进行如何操作
    	//- F_DUPFD : 复制文件描述符,复制的是第一个参数fd，得到一个新的文件描述符（返回值）
		//- F_GETFL : 获取指定的文件描述符文件状态flag，和open函数中flag参数相同。
		//- F_SETFL : 设置文件描述符文件状态flag
				//必选项：O_RDONLY, O_WRONLY, O_RDWR 不可以被修改
				//可选项：O_APPEND, O)NONBLOCK
				//O_APPEND 表示追加数据、NONBLOK 设置成非阻塞
//应用举例：修改文件描述符状态为给flag，并加入O_APPEND这个标记
int flag = fcntl(fd, F_GETFL);
flag |= O_APPEND;   
int ret = fcntl(fd, F_SETFL, flag);
~~~

## 进程

### 1.进程的创建与销毁

~~~c++
#include <sys/types.h>	
#include <unistd.h>
pid_t fork(void);
//创建线程 返回值：成功：子进程中返回 0，父进程中返回子进程 ID 失败：返回 -1
//进程创建不一定成功，失败的两个主要原因：
	//（1）当前系统的进程数已经达到了系统规定的上限，这时 errno 的值被设置为 EAGAIN
	//（2）系统内存不足，这时 errno 的值被设置为 ENOMEM

//进程号相关
pid_t getpid(void);			  //返回目前进程的进程识别码
pid_t getppid(void);	 	  //返回父进程标识
pid_t getpgid(pid_t pid);	  //返回组进程标识

//进程退出	
#include <stdlib.h>
void exit(int status);   //属于标准库函数
//进程退出	#include <unistd.h>
void _exit(int status);  //属于系统调用
//参数为 status 返回给父进程的参数(低8位有效)，两个函数的功能相同，库函数不同

//进程回收
//头文件	#include <sys/types.h>	#include <wait.h>
int wait(int *status)
//父进程一旦调用了wait就立即阻塞自己，由wait自动分析是否当前进程的某个子进程已经退出，如果让它找到了这样一个已经变成僵尸的子进程，wait就会收集这个子进程的信息，并把它彻底销毁后返回；如果没有找到这样一个子进程，wait就会一直阻塞在这里，直到有一个出现为止。若不执行wait函数则可能会出现父进程结束子进程还在运行的情况，子进程会无法回收变成僵尸进程。
//返回值：如果执行成功则返回子进程识别码(PID), 如果有错误发生则返回-1. 失败原因存于errno 中。
pid_t waitpid(pid_t pid, int * status, int options);
//pid表示要等待的进程
	//pid<-1 等待进程组识别码为pid 绝对值的任何子进程.pid=-1 等待任何子进程, 相当于wait()。
	//pid=0 等待进程组识别码与目前进程相同的任何子进程.pid>0 等待任何子进程识别码为pid 的子进程。
//options可以为0 或下面的 OR 组合
	//WIFEXITED(status) 非0，进程正常退出
	//WEXITSTATUS(status) 如果上宏为真，获取进程退出的状态（exit的参数）
    //WIFSIGNALED(status) 非0，进程异常终止
    //WTERMSIG(status) 如果上宏为真，获取使进程终止的信号编号
    //WIFSTOPPED(status) 非0，进程处于暂停状态
    //WSTOPSIG(status) 如果上宏为真，获取使进程暂停的信号的编号
    //WIFCONTINUED(status) 非0，进程暂停后已经继续运行
~~~

### 2.线程同步：

#### （1）匿名管道pipe：

pipe提供两个文件描述符来操作管道。其中一个对管道进行写操作，另一个对管道进行读操作。对管道的读写与一般的IO系统函数一致，使用write()函数写入数据，使用read()读出数据。匿名管道没有文件实体，本质是一个在内核内存中维护的缓冲器，其内数据的传递方向是单向的，一端用于写入，一端用于读取，管道是半双工的。

~~~c++
#include <unistd.h>
int pipe(int pipefd[2]);
//pipefd[0] 对应的是管道的读端；pipefd[1] 对应的是管道的写端
int pipefd[2];
pipe(pipefd);
//父进程读取
close(pipefd[1]);		//关闭写端
char buf[1024] = {0};	
read(pipefd[0], buf, sizeof(buf));
//子进程
close(pipefd[0]);		//关闭读端
char * str = "hello,i am child";
write(pipefd[1], str, strlen(str));
~~~

#### （2）有名管道FIFO：

有名管道提供了一个路径名与之关联，以 FIFO 的文件形式存 在于文件系统中，并且其打开方式与打开一个普通文件是一样的。与匿名管道pipe只能用于亲缘关系的进程间通信不同， 即使与 FIFO 的创建进程不存在亲缘关系的进程，只要可以访问该路径，就能够彼此通过 FIFO 相互通信。

~~~c++
#include <sys/types.h>
#include <sys/stat.h>
//通过函数创建有名管道
int mkfifo(const char *pathname, mode_t mode);
~~~

一旦使用 mkfifo 创建了一个 FIFO，就可以使用 open 打开它，常见的文件I/O 函数都可用于 FIFO。如： close、read、write、unlink 等。 FIFO 就像其名字，遵循先进先出原则，对管道及 FIFO 的读总是从开始处返回数据，对它们的写则把数据添加到末尾。FIFO不支持诸如 lseek() 等文件定位操作。

#### （3）内存映射：

内存映射是一种内存映射技术。将磁盘文件的数据映射到内存，使用户通过修改内存就能修改磁盘文件。

~~~c++
#include <sys/mman.h>
s
//将一个文件或者设备的数据映射到内存中
void *mmap(void *addr, size_t length, int prot, int flags, int fd, off_t offset);
//- void *addr: NULL, 由内核指定
//- length : 要映射的数据的长度，一般使用文件的长度。
//- prot : 对申请的内存映射区的操作权限
      //-PROT_EXEC ：可执行的权限	-PROT_READ ：读权限
      //-PROT_WRITE ：写权限	-PROT_NONE ：没有权限
      //要操作映射内存，必须要有读的权限。
//- flags :
      //- MAP_SHARED : 映射区的数据会自动和磁盘文件进行同步，进程间通信，必须要设置这个选项
      //- MAP_PRIVATE ：不同步，内存映射区数据改变后，不修改原来文件，而是重新创建一个新的文件。
//- fd: 需要映射的那个文件的文件描述符
      //- 通过open得到，open的是一个磁盘文件
      //- 注意：文件的大小不能为0，open指定的权限不能和prot参数有冲突。
//- 返回值：成功返回创建的内存的首地址，失败返回MAP_FAILED，(void *) -1

//释放内存映射
int munmap(void *addr, size_t length);
~~~

在内存映射中，参数flags除了上述两种宏定义外，还有一个宏定义为MAP_ANONYMOUS，表示匿名映射，此时的文件映射会将一个内核创建的全为进制零文件映射到虚拟内存中，不需要真正的文件实体。

~~~c++
//创建匿名内存映射区
int len = 4096;
void * ptr = mmap(NULL, len, PROT_READ | PROT_WRITE, MAP_SHARED | MAP_ANONYMOUS, -1, 0);
pid_t pid = fork();
if(pid > 0) {
    // 父进程
    strcpy((char *) ptr, "hello, world");
    wait(NULL);
}else if(pid == 0) {
    // 子进程
    sleep(1);
    printf("%s\n", (char *)ptr);
}
munmap(ptr, len);		//释放内存映射区
~~~

#### （4）信号：

信号是事件发生时对进程的通知机制，信号可以导致⼀个正在运⾏ 的进程被另⼀个正在运⾏的异步进程中断，转⽽处理某⼀个突发事件。信号可以被理解为软件中断，是在软件层次上对中断机制的一种模拟，是一种异步通信的方式。其工作流程为：信号产生--->信号的注册和注销--->执行信号处理函数。

**信号产生函数：**kill、raise、abort、alarm、setitimer

~~~c++
#include <sys/types.h>
#include <signal.h>

//给指定进程发送指定信号(不⼀定杀死).
int kill(pid_t pid, int sig);
//pid > 0: 将信号传送给进程 ID 为pid的进程; pid = 0 : 将信号传送给当前进程所在进程组中的所有进程;
//pid = -1 : 将信号传送给系统内所有的进程; pid < -1 : 将信号传给指定进程组的所有进程.
//sig 为信号的编号，这⾥可以填数字编号，也可以填信号的宏定义.

//给当前进程发送指定信号(⾃⼰给⾃⼰发)，等价于 kill(getpid(), sig).
int raise(int sig);

#include <stdlib.h>
//给当前进程发送异常终⽌信号 6) SIGABRT，并产⽣core⽂件，等价于kill(getpid(), SIGABRT).
void abort(void);

#include <unistd.h>
//设置定时器(闹钟)。在指定seconds后，内核会给当前进程发送14）SIGALRM信号。
//进程收到该信号，默认动作终⽌。每个进程都有且只有唯⼀的⼀个定时器;
unsigned int alarm(unsigned int seconds);

//设置定时器函数。 可代替alarm函数。精度微秒us，可以实现周期定时。
#include <sys/time.h>
struct itimerval {
 struct timerval it_interval; // 闹钟触发周期
 struct timerval it_value; // 闹钟触发时间
};
struct timeval {
 long tv_sec; // 秒
 long tv_usec; // 微秒
}
//which 指定定时⽅式:
	//⾃然定时：ITIMER_REAL → 14）SIGALRM计算⾃然时间;
	//虚拟空间计时(⽤户空间)：ITIMER_VIRTUAL → 26）SIGVTALRM 只计算进程占⽤cpu的时间;
//new_value 负责设定timeout时间.
//old_value 存放旧的timeout值，⼀般指定为NULL.
//返回值 成功: 0; 失败: -1.
int setitimer(int which, const struct itimerval *new_value, struct itimerval *old_value);
// itimerval.it_value： 设定第⼀次执⾏function所延迟的秒数
// itimerval.it_interval： 设定以后每⼏秒执⾏function
~~~

信号集：一组信号的集合，记录了集合中每一个信号的属性。由sigset_t数据类型来存储，本质上是位图，用0和1代表该位信号是否阻塞或者是否为未决状态。

~~~c++
#include <signal.h> 
int sigemptyset(sigset_t *set); // 将set集合置空
int sigfillset(sigset_t *set); // 将所有信号加⼊set集合
int sigaddset(sigset_t *set, int signo); // 将signo信号加⼊到set集合
int sigdelset(sigset_t *set, int signo); // 从set集合中移除signo信号
int sigismember(const sigset_t *set, int signo); // 判断信号是否存在
~~~

当一个信号产生并被捕获后，和阻塞信号集中的标志位进行查询。若没有阻塞，这个信号就被处理；若阻塞，这个信号就继续处于未决状态，直到阻塞解除，这个信号就被处理。通常使用sigprocmask函数 设置阻塞集：

~~~c++
#include <signal.h>

//检查或修改信号阻塞集，根据 how 指定的⽅法对进程的阻塞集合进⾏修改，
//新的信号阻塞集由 set 指定，⽽原先的信号阻塞集合由 oldset 保存.
// how 信号阻塞集合的修改⽅法，有 3 种情况:
	//SIG_BLOCK：向信号阻塞集合中添加 set 信号集，
	//新的信号掩码是set和旧信号掩码的并集。相当于 mask = mask|set;
	//SIG_UNBLOCK：从信号阻塞集合中删除 set 信号集，
	//从当前信号掩码中去除 set 中的信号。相当于 mask = mask & ~ set;
	//SIG_SETMASK：将信号阻塞集合设为 set 信号集，
	//相当于原来信号阻塞集的内容清空，然后按照 set 中的信号᯿新设置信号阻塞集。相当于mask = set.
//set 要操作的信号集地址,若set为 NULL，则不改变信号阻塞集合，函数只把当前信号阻塞集合保存到oldset中。
//oldset 保存原先信号阻塞集地址。
//返回值 成功: 0; 失败: -1，失败时错误代码只可能是 EINVAL，表示参数 how 不合法。
int sigprocmask(int how, const sigset_t *set, sigset_t *oldset);

#include <signal.h>

//读取当前进程的未决信号集。
//set 未决信号集,返回值：成功: 0; 失败: -1。
int sigpending(sigset_t *set);
~~~

**信号的捕捉和处理：**

~~~c++
#include <signal.h>

struct sigaction {
 void(*sa_handler)(int); //旧的信号处理函数指针
 void(*sa_sigaction)(int, siginfo_t *, void *); //新的信号处理函数指针
 sigset_t sa_mask; //信号阻塞集
 int sa_flags; //信号处理的⽅式
 void(*sa_restorer)(void); //已弃⽤
};

//检查或修改指定信号的设置（或同时执⾏这两种操作）。
//signum 要操作的信号。
//act 要设置的对信号的新处理⽅式（传⼊参数）。
//oldact：原来对信号的处理⽅式（传出参数）。
	//如果 act 指针⾮空，则要改变指定信号的处理⽅式（设置），
	//如果 oldact 指针⾮空，则系统将此前指定信号的处理⽅式存⼊ oldact。
//返回值 成功: 0; 失败: -1。
int sigaction(int signum, const struct sigaction *act, struct sigaction *oldact);
~~~

**一些信号的属性和对应事件：**

| 编号    | 信号宏定义           | 对应事件                                                     | 默认动作                   |
| ------- | -------------------- | ------------------------------------------------------------ | -------------------------- |
| 1       | SIGHUP               | 用户退出shell时，由该shell启动的所有进程将收到这个信号       | 终止进程                   |
| 2       | SIGINT               | 当用户按下了<Ctrl+C>组合键时，用户终端向正在运行中的由该终端启动的程序发出此信号 | 终止进程                   |
| 3       | SIGQUIT              | 用户按下<Ctrl+>组合键时产生该信号，用户终端向正在运 行中的由该终端启动的程序发出些信号 | 终止进程                   |
| 9       | SIGKILL              | 无条件终止进程。该信号不能被忽略，处理和阻塞                 | 终止进程，可以杀死任何进程 |
| 11      | SIGSEGV              | 指示进程进行了无效内存访问(段错误)                           | 终止进程并产生 core文件    |
| 13      | SIGPIPE              | Broken pipe向一个没有读端的管道写数据                        | 终止进程                   |
| 17      | SIGCHLD              | 子进程结束时，父进程会收到这个信号                           | 忽略这个信号               |
| 18      | SIGCONT              | 如果进程已停止，则使其继续运行                               | 继续/忽略                  |
| 19      | SIGSTOP              | 停止进程的执行。信号不能被忽略，处理和阻塞                   | 终止进程                   |
| 34 ~ 64 | SIGRTMIN ～ SIGRTMAX | LINUX的实时信号，它们没有固定的含义（可以由用户自 定义）     | 终止进程                   |







## 线程