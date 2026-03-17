## 操作系统及计算机组成原理 (二) - 进程和线程 
### 进程和线程

**进程概念**

进程 (Proces) 是正在执行的程序的实例，包含代码、数据、资源（如内存、文件句柄）和运行时状态。

进程 ≠ 程序：程序是静态的代码文件，进程是动态的执行实体。

进程是操作系统进行`资源分配基本单位`，它为应用程序提供了`独立的运行环境`，包括独立的`内存空间`、`系统资源`等。应用程序在运行时，操作系统会为其创建一个或多个进程来执行相应的任务。

**线程概念**

线程是进程内的一个独立执行流，是操作系统调度的最小单位。

### 进程镜像

**进程镜像（Process Image）** 是进程在`内存中的完整表示`，包含其代码、数据、堆栈以及操作系统管理进程所需的所有信息。它是进程运行时在内存中的“快照”，决定了进程的执行环境和资源布局。

#### 核心组成

进程镜像由多个逻辑段（Segments）组成，每个段负责不同的功能：

| **段名**          | **存储内容**                                  | **权限**         | **生命周期**       |
|--------------------|---------------------------------------------|------------------|--------------------|
| **代码段（Text）** | 可执行指令（机器码）                         | 只读、可执行      | 进程启动时加载      |
| **数据段（Data）** | 已初始化的全局变量、静态变量                 | 读/写             | 进程启动时初始化    |
| **BSS 段**         | 未初始化的全局变量、静态变量（初始化为 0）     | 读/写             | 进程启动时清零      |
| **堆（Heap）**     | 动态分配的内存（如 `malloc`、`new`）         | 读/写             | 运行时动态扩展/收缩 |
| **栈（Stack）**    | 函数调用时的局部变量、返回地址、参数         | 读/写             | 函数调用时自动管理  |
| **内存映射区**     | `共享库`、`文件映射`（如 `mmap`）                | 依映射类型而定     | 动态加载/释放       |

#### 示例（Linux）

通过 `pmap` 命令查看进程的内存布局（以 PID 1234 为例）：

```bash
pmap 1234
```

输出示例：

``` bash
Address           Perm   Size    Offset  Device  Mapping
00400000         r-xp   00004000 00:00  0       /bin/ls    # 代码段
00601000         r--p   00001000 00:00  0       /bin/ls    # 数据段
00602000         rw-p   00002000 00:00  0       /bin/ls    # BSS 段
7f8e1a200000      rw-p   00000000 00:00  0                  # 堆
7f8e1a400000      r-xp   0001f000 00:00  0       /lib/libc-2.31.so  # 共享库代码段
7ffeefbff000      rw-p   00000000 00:00  0                  # 栈
```

> mac 使用 vmmap -summary 27961

### 进程控制块 (PCB)

`进程控制块`（Process Control Block, PCB） 是操作系统为每个`进程维护`的核心`数据结构`，用于`存储进程`的所有`状态信息`。它是操作系统实现多任务、进程调度和资源管理的基础。

以下是 PCB 的详细解析：

#### 作用

1. **保存进程状态**：当进程被切换出 CPU 时，保存其运行状态（如`寄存器值`、`PC` 等）。  
2. **资源管理**：记录进程占用的资源（如`内存`、`文件句柄`、`设备`）。  
3. **调度依据**：为进程`调度`器提供`决策信息`（如优先级、状态）。  
4. **进程隔离**：确保不同进程的地址空间和资源相互独立。

#### 存储位置

- **内存中的内核空间**：PCB 由操作系统内核管理，存储在**内核态内存**中，用户程序无法直接访问。  
- **组织方式**：通常以`链表或树`的形式组织（如 Linux 的 `task_struct` 链表）。

#### 主要内容

**1. 进程标识信息**

- **进程 ID（PID）**：唯一标识进程的整数（如 Linux 中 PID=1 是 `init` 进程）。  
- **父进程 ID（PPID）**：创建该进程的父进程 PID。  
- **用户 ID（UID）** 和 **组 ID（GID）**：进程的权限归属。

**2. 进程状态信息**

- **进程状态**：`运行`（Running）、`就绪`（Ready）、`阻塞`（Blocked）、`僵尸`（Zombie）等。  
- **调度信息**：优先级、时间片剩余量、调度策略（如 FIFO、轮转）。  
- **退出状态码**：进程终止时的返回值（供父进程查询）。

**3. CPU 上下文（Context）**

- **寄存器值**：通用寄存器（如 x86 的 EAX、EBX）、`PC`、`栈指针`（SP）、`状态寄存器`（FLAGS）。  
- **浮点/向量寄存器**：如 x87 FPU 状态、SSE/AVX 寄存器（若支持 SIMD）。

**4. 内存管理信息**

- **页表基址**：如 x86 的 CR3 寄存器值，`指向进程的页表`。  
- **内存映射**：代码段、数据段、堆、栈的地址范围。  
- **共享内存**：进程间共享的内存区域信息。

**5. 文件与 I/O 资源**

- **打开文件表**：记录进程打开的文件描述符（如`文件句柄`、`读写位置`）。  
- **工作目录**：进程的当前文件系统路径。  
- **I/O 设备**：占用的设备（如打印机、网络套接字）。

**6. 其他控制信息**

- **信号处理表**：注册的信号处理器（如 SIGINT、SIGKILL）。  
- **进程间通信（IPC）**：`消息队列`、`信号量`、`共享内存`的标识符。  
- **资源限制**：CPU 时间、内存用量、文件打开数等限制（如 Linux 的 `ulimit`）。

#### 生命周期

1. **创建**：进程通过 `fork()` 或 `CreateProcess()` 创建时，操作系统分配并初始化 PCB。  
2. **运行**：进程执行期间，PCB 中的状态和资源信息动态更新。  
3. **切换**：进程被挂起时，CPU 上下文保存到 PCB；恢复时从 PCB 加载。  
4. **终止**：进程退出后，PCB 保留至父进程读取退出状态（`wait()`），最终由内核释放。

#### 实际示例（Linux 的 `task_struct`）

Linux 内核中，PCB 对应 `task_struct` 结构体，包含数百个字段。以下是部分关键字段：

```c
struct task_struct {
    // 进程标识
    pid_t pid;             // 进程 ID
    pid_t tgid;            // 线程组 ID（主线程 PID）
    
    // 状态与调度
    volatile long state;   // 进程状态（TASK_RUNNING 等）
    int prio;              // 动态优先级
    unsigned int policy;   // 调度策略（SCHED_NORMAL, SCHED_FIFO 等）
    
    // CPU 上下文
    struct thread_struct thread; // 保存寄存器的结构体
    
    // 内存管理
    struct mm_struct *mm;  // 内存描述符（页表、地址空间等）
    
    // 文件与 I/O
    struct files_struct *files; // 打开的文件表
    
    // 信号处理
    struct signal_struct *signal; // 信号处理器表
    
    // 父进程与子进程
    struct task_struct *parent;  // 父进程
    struct list_head children;   // 子进程链表
};
```

#### 访问与安全性

- **内核特权**：只有操作系统内核可以读写 PCB，用户程序无法直接访问。  
- **隔离性**：不同进程的 PCB 完全隔离，防止恶意篡改（如修改其他进程的 PC）。

### 线程控制块 (TCB)

- **定义**：线程控制块（Thread Control Block, TCB）是操作系统为每个线程维护的数据结构，用于存储线程的独立状态和资源信息。  
- **核心作用**：  
  1. **保存线程上下文**：线程切换时保存/恢复寄存器、栈指针（SP）、程序计数器（PC）等。  
  2. **管理线程资源**：记录线程的栈、优先级、同步状态等。  
  3. **支持线程调度**：为调度器提供线程状态、优先级等信息。  

#### 主要内容

TCB 包含线程的**独立执行状态**和**共享资源引用**，典型字段如下：  

**1. 线程标识信息**  

- **线程 ID（TID）**：唯一标识线程的整数（如 Linux 的 `pthread_t`）。  
- **所属进程 ID（PID）**：线程归属的进程标识。

**2. 线程上下文（Thread Context）**  

- **寄存器状态**：通用寄存器（EAX、EBX）、PC、SP、状态寄存器（FLAGS）。  
- **浮点/向量寄存器**：如 SSE、AVX 寄存器状态（若支持 SIMD）。  

**3. 栈信息**  

- **栈指针（SP）**：指向线程的私有栈顶。  
- **栈基址与大小**：定义线程栈的内存范围（防止溢出）。  

**4. 线程状态与调度信息**  

- **线程状态**：`运行`（Running）、`就绪`（Ready）、`阻塞`（Blocked）、`终止`（Terminated）。  
- **优先级**：静态或动态优先级（影响调度顺序）。  
- **时间片剩余量**：轮转调度中剩余的 CPU 时间。  

**5. 同步与通信机制**  

- **信号量/互斥锁**：线程持有的同步对象。  
- **等待队列**：线程因等待条件变量或 I/O 而阻塞时的队列指针。  

**6. 资源指针**  

- **指向 PCB 的指针**：线程所属进程的 PCB（共享进程级资源，如内存、文件）。  
- **线程本地存储（TLS）**：线程独有的数据区指针。

#### 线程切换与 TCB 的工作流程

**1. 触发线程切换**  

- 主动让出（如调用 `pthread_yield()`）。  
- 时间片耗尽、等待 I/O 或同步对象（如锁、条件变量）。  

**2. 保存当前线程上下文**  

- 将寄存器值（PC、SP 等）保存到当前线程的 TCB 中。  

**3. 选择下一个线程**  

- 调度器根据优先级、状态等从就绪队列中选择目标线程。  

**4. 加载目标线程上下文**  

- 从目标线程的 TCB 中恢复寄存器、栈指针等状态。  

**5. 执行目标线程**  

- CPU 从恢复的 PC 地址继续执行目标线程。  

#### 实际实现示例

**1. Linux 的线程实现（基于 `task_struct`）**  

- Linux 将线程视为“轻量级进程”，复用 `task_struct` 结构（即 PCB），但共享进程资源：  
  - **共享部分**：内存描述符（`mm_struct`）、打开文件表（`files_struct`）。  
  - **独立部分**：线程 ID（`pid`）、寄存器状态、栈（`thread_info`）。  

**2. Windows 的 ETHREAD 结构**  

- Windows 内核使用 `ETHREAD` 结构管理线程：  
  - **TEB（Thread Environment Block）**：用户态线程信息（如 TLS、异常处理链）。  
  - **KTHREAD**：内核态线程信息（如优先级、调度状态）。  

**3. 用户级线程（如 POSIX Pthreads）**  

- TCB 存储在用户空间，由线程库（如 glibc）管理：  
  - **优点**：切换无需内核介入，速度快。  
  - **缺点**：一个线程阻塞会导致整个进程阻塞。  

#### 性能与优化

- **上下文切换开销**：  
  - 内核线程切换需陷入内核（约 1-10 μs）。  
  - 用户线程切换仅用户态操作（约 100 ns）。  
- **栈大小管理**：  
  - 默认栈大小（如 Linux 中 8 MB）可能导致内存浪费，可自定义栈大小。  
- **线程池技术**：  
  - 预创建线程并复用 TCB，避免频繁创建/销毁开销。

#### 总结

- **TCB 是线程执行的“快照”**，保存了线程独立的执行状态。
- **与 PCB 协同工作**：TCB 管理线程私有状态，PCB 管理进程共享资源。  
- **设计权衡**：用户级线程灵活但功能受限，内核级线程功能全面但开销较高。  

理解 TCB 是掌握多线程编程、调试线程同步问题（如死锁、竞态条件）的基础，也是优化并发程序性能的关键。

### PCB vs TCB

| **特性**         | **TCB（线程控制块）**                | **PCB（进程控制块）**                |
|------------------|------------------------------------|------------------------------------|
| **管理对象**     | 线程（轻量级执行流）                | 进程（资源分配单位）                |
| **资源归属**     | 共享进程资源（内存、文件）           | 独占资源（地址空间、全局变量）       |
| **存储内容**     | 线程私有状态（栈、寄存器）           | 进程全局状态（页表、文件表）         |
| **数量关系**     | 一个进程可包含多个 TCB              | 一个进程对应一个 PCB                |
| **切换开销**     | 低（仅保存寄存器、栈）              | 高（需切换地址空间、文件表等）       |

- **PCB**：管理进程级资源（如地址空间、文件）。  
- **TCB**：管理线程级资源（如栈、寄存器），多个线程共享同一进程的 PCB。  
- **示例**：  
  - 进程的 PCB 包含页表基址（CR3）、打开文件表。  
  - 线程的 TCB 包含栈指针（SP）、程序计数器（PC）

资源归属：线程`共享进程的内存空间`（代码段、数据段、堆、打开的文件等），但拥有`独立的栈`和`寄存器状态`。

执行粒度：进程是`资源分配`的单位，线程是 `CPU 调度`的单位。

开销：线程的创建、切换、通信成本远低于进程。

### 子进程和子线程

#### 创建子进程

在 C 语言中，创建子进程主要通过 `fork()` 系统调用实现。子进程会`复制父进程的内存空间`，并在`独立的地址空间`中运行。

```c
#include <stdio.h>
#include <unistd.h> // fork(), getpid()
#include <sys/wait.h> // wait()

int main() {
    pid_t pid = fork(); // 创建子进程

    if (pid < 0) {
        // fork 失败
        perror("fork failed");
        return 1;
    } else if (pid == 0) {
        // 子进程代码
        printf("Child Process: PID = %d\n", getpid());
        // 子进程可以执行其他程序（如 exec 系列函数）
        // execlp("ls", "ls", "-l", NULL); // 示例：执行 ls -l
    } else {
        // 父进程代码
        printf("Parent Process: PID = %d, Child PID = %d\n", getpid(), pid);
        wait(NULL); // 等待子进程结束
    }

    return 0;
}
```

关键点:

- fork() 返回两次：父进程返回子进程的 PID，子进程返回 0。
- 子进程可通过 exec 系列函数（如 execlp）加载新程序。
- wait() 用于父进程等待子进程结束，避免僵尸进程。

#### 创建子线程

在 C 语言中，创建子线程需使用 POSIX 线程库（`pthread`），编译时需链接 `-lpthread`。

**代码示例**

```c
#include <stdio.h>
#include <pthread.h> // pthread 库

// 线程函数原型：void* (*start_routine)(void*)
void* thread_function(void* arg) {
    char* message = (char*)arg;
    printf("Thread: Message = %s\n", message);
    return NULL;
}

int main() {
    pthread_t thread_id;
    char* message = "Hello from the main thread!";

    // 创建线程
    int ret = pthread_create(
        &thread_id,    // 线程 ID 指针
        NULL,          // 线程属性（默认 NULL）
        thread_function, // 线程函数
        (void*)message // 传递给线程的参数
    );

    if (ret != 0) {
        perror("pthread_create failed");
        return 1;
    }

    printf("Main Thread: Created thread ID = %lu\n", thread_id);

    // 等待线程结束
    pthread_join(thread_id, NULL);

    return 0;
}
```

关键点：

- `pthread_create` 的四个参数：线程 ID 指针、属性、线程函数、参数。
- 线程函数必须为 `void* func(void*)` 格式。
- `pthread_join` 用于主线程等待子线程结束。

**编译与运行**

**1. 编译命令**

- **子进程程序**（假设文件名为 `fork_demo.c`）：

  ```bash
  gcc fork_demo.c -o fork_demo
  ./fork_demo
  ```
  预期输出:
  ``` bash
  Parent Process: PID = 1234, Child PID = 1235
  Child Process: PID = 1235
  ```

- **子线程程序**（假设文件名为 `thread_demo.c`）：

  ```bash
  gcc thread_demo.c -o thread_demo -lpthread
  ./thread_demo
  ```
  预期输出:
  ``` bash
  Main Thread: Created thread ID = 140123456789760
  Thread: Message = Hello from the main thread!
  ```

#### 注意事项

1. **进程间同步**：进程需通过 IPC（如信号量、共享内存）同步。  
2. **线程安全**：线程共享数据时需使用互斥锁（`pthread_mutex_t`）。  
3. **资源释放**：线程应避免访问已释放的内存（如主线程栈变量）。

#### fork 和 exec

`exec` 是类 Unix 系统（如 Linux、macOS）中一组用于**替换当前进程映像**的系统调用。它允许一个进程加载并执行另一个全新的程序，覆盖当前进程的代码段、数据段和堆栈，但保留进程 ID（PID）和部分资源（如文件描述符）。

##### 核心作用

- **替换进程映像**：终止当前进程的代码，加载并执行新程序。  
- **保留 PID**：新程序继承原进程的 PID、文件描述符、环境变量等。  
- **不创建新进程**：与 `fork()` 不同，`exec` **不会创建新进程**，而是重用现有进程的“外壳”运行新程序。

`exec` 并非单一函数，而是一组功能相似的函数，区别在于参数传递方式和搜索路径规则：

| **函数名**       | **参数传递方式**       | **是否使用 PATH 环境变量** | **示例用法**                              |
|------------------|-----------------------|--------------------------|-----------------------------------------|
| `execl`          | **可变参数列表**       | 否（需完整路径）          | `execl("/bin/ls", "ls", "-l", NULL);`   |
| `execv`          | **参数数组（argv）**   | 否                       | `char *args[] = {"ls", "-l", NULL}; execv("/bin/ls", args);` |
| `execlp`         | 可变参数列表           | 是                       | `execlp("ls", "ls", "-l", NULL);`       |
| `execvp`         | 参数数组               | 是                       | `char *args[] = {"ls", "-l", NULL}; execvp("ls", args);` |
| `execle`         | 可变参数 + 环境变量    | 否                       | `execle("/bin/ls", "ls", "-l", NULL, env_vars);` |
| `execvpe`        | 参数数组 + 环境变量    | 是                       | `char *args[] = {"ls", "-l", NULL}; execvpe("ls", args, env_vars);` |

**`exec` 的典型使用场景**

**1. 结合 `fork()` 创建子进程并执行新程序**

```c
#include <unistd.h>
#include <sys/wait.h>

int main() {
    pid_t pid = fork();

    if (pid == 0) {
        // 子进程执行 ls -l
        execlp("ls", "ls", "-l", NULL);
        // 若 execlp 成功，以下代码不会执行
        perror("execlp failed");
        return 1;
    } else {
        // 父进程等待子进程结束
        wait(NULL);
    }
    return 0;
}
```

**2. 替换当前进程（如 Shell 执行命令）**

```c
// 直接替换当前进程为 /bin/bash
execl("/bin/bash", "bash", "-c", "echo Hello, World!", NULL);
```

##### 关键特性

1. **无返回值（除非失败）**：  
   - 若 `exec` 成功，原进程的代码被完全替换，**不会返回**。  
   - 若失败（如路径错误），返回 `-1` 并设置 `errno`（需检查错误）。

2. **继承资源**：  
   - **保留**：PID、PPID、文件描述符（除非标记为 `CLOEXEC`）、`环境变量`（除非显式覆盖）。  
   - **重置**：信号处理器恢复为默认行为。

3. **参数列表规范**：  
   - 参数列表必须以 `NULL` 结束（如 `"ls", "-l", NULL`）。

##### 常见问题

**1. 为什么 `exec` 常与 `fork()` 配合使用？**

- `fork()` 创建子进程，`exec()` 让子进程执行新程序，而父进程保持原逻辑。  
- 这是 Unix/Linux 中运行新程序的标准模式（如 Shell 执行命令）。

**2. `exec` 后原进程的代码是否还在？**

- **完全替换**：原进程的代码、数据、堆栈均被新程序覆盖，但 PID 不变。  
- **内存释放**：原进程的内存空间由操作系统回收，分配给新程序。

**3. 如何传递环境变量？**

- 使用 `execle` 或 `execvpe` **显式指定**环境变量数组：  

  ```c
  char *env[] = {"PATH=/usr/bin", "USER=root", NULL};
  execle("/bin/ls", "ls", "-l", NULL, env);
  ```

### 进程间通信 (IPC)

**IPC（Inter-Process Communication）** 是不同进程之间交换数据或协调操作的机制，主要用于以下场景：

- **数据传输**：进程间共享数据（如生产者-消费者模型）。
- **资源共享**：多个进程访问同一资源（如文件、硬件）。
- **协调同步**：避免竞态条件（如共享内存的互斥访问）。
- **通知事件**：异步信号通知（如进程终止、用户中断）。

#### 主要方式

**1. 管道（Pipe）**

- **匿名管道**  
  - **特点**：单向通信，仅限有亲缘关系的进程（如父子进程）。  
  - **实现**：通过 `pipe()` 系统调用创建，内核缓冲区传递数据。  
  - **示例**：  
    ```c
    int fd[2];
    pipe(fd); // 创建管道
    if (fork() == 0) {
        close(fd[0]); // 子进程关闭读端
        write(fd[1], "Hello", 6);
    } else {
        close(fd[1]); // 父进程关闭写端
        char buf[6];
        read(fd[0], buf, 6);
    }
    ```

- **命名管道（FIFO）**  
  - **特点**：通过文件系统路径标识，支持无亲缘关系的进程。  
  - **实现**：`mkfifo` 命令或 `mkfifo()` 函数创建管道文件。  
  - **示例**：  
    ```bash
    mkfifo my_pipe    # 创建管道文件
    echo "Data" > my_pipe &  # 进程A写入
    cat < my_pipe     # 进程B读取
    ```

**2. 消息队列（Message Queue）**

- **特点**：  
  - 消息按类型存储，接收方可选择读取特定类型。  
  - 解耦发送与接收进程，支持异步通信。  
- **实现**：通过 `msgget()`, `msgsnd()`, `msgrcv()` 系统调用操作。  
- **示例**：  
  ```c
  struct msg_buf {
      long mtype;
      char mtext[100];
  };
  int msgid = msgget(IPC_PRIVATE, 0666); // 创建队列
  msgsnd(msgid, &msg, sizeof(msg), 0);   // 发送消息
  msgrcv(msgid, &msg, sizeof(msg), 1, 0); // 接收类型1的消息
  ```

**3. 共享内存（Shared Memory）**

- **特点**：  
  - 最高效的 IPC 方式（直接内存访问，无数据拷贝）。  
  - 需配合信号量或锁实现同步。  
- **实现**：  
  - **POSIX**：`shm_open()`, `mmap()`。  
  - **System V**：`shmget()`, `shmat()`, `shmdt()`。  
- **示例**（POSIX）：  
  ```c
  int fd = shm_open("/my_shm", O_CREAT | O_RDWR, 0666);
  ftruncate(fd, SIZE);
  char *ptr = mmap(NULL, SIZE, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
  sprintf(ptr, "Shared Data"); // 写入共享内存
  ```

**4. 信号量（Semaphore）**

- **特点**：  
  - 用于同步进程对共享资源的访问（如互斥锁）。  
  - 分为二进制信号量（0/1）和计数信号量。  
- **实现**：  
  - **POSIX**：`sem_open()`, `sem_wait()`, `sem_post()`.  
  - **System V**：`semget()`, `semop()`.  
- **示例**（POSIX）：  
  ```c
  sem_t *sem = sem_open("/my_sem", O_CREAT, 0666, 1);
  sem_wait(sem); // 进入临界区
  // 操作共享资源
  sem_post(sem); // 离开临界区
  ```

**5. 套接字（Socket）**

- **特点**：  
  - 支持网络通信和本地进程通信（Unix 域套接字）。  
  - 双向通信，灵活但开销较大。  
- **示例**（本地 Unix 域套接字）：  
  ```c
  int sockfd = socket(AF_UNIX, SOCK_STREAM, 0);
  struct sockaddr_un addr;
  addr.sun_family = AF_UNIX;
  strcpy(addr.sun_path, "/tmp/my_socket");
  bind(sockfd, (struct sockaddr*)&addr, sizeof(addr));
  // 监听、连接、发送/接收数据...
  ```

**6. 信号（Signal）**

- **特点**：  
  - 异步通知进程特定事件（如 `SIGINT` 终止信号）。  
  - 仅传递信号编号，无法携带复杂数据。  
- **示例**：  
  ```c
  void handler(int sig) {
      printf("Received signal: %d\n", sig);
  }
  signal(SIGINT, handler); // 注册信号处理器
  ```

#### 不同 IPC 方式的对比

| **方式**        | **传输方向** | **速度** | **同步需求** | **适用场景**               |
|-----------------|--------------|----------|--------------|----------------------------|
| **匿名管道**    | 单向         | 快       | 无需         | 父子进程简单数据流         |
| **命名管道**    | 单向         | 快       | 无需         | 无亲缘进程的简单通信       |
| **消息队列**    | 双向         | 中       | 无需         | 解耦的多进程通信           |
| **共享内存**    | 双向         | 最快     | 需同步       | 高频数据交换（如数据库）   |
| **信号量**      | -            | 快       | 需同步       | 共享资源互斥访问           |
| **套接字**      | 双向         | 较慢     | 无需         | 跨网络或本地复杂通信       |
| **信号**        | 单向         | 快       | 异步         | 事件通知（如进程终止）     |

**选择 IPC 方式的考虑因素**

1. **数据量**：共享内存适合大数据，信号适合小事件通知。  
2. **性能要求**：共享内存最快，套接字开销最大。  
3. **进程关系**：管道需亲缘关系，消息队列和套接字无限制。  
4. **同步需求**：共享内存需额外同步机制，消息队列自带缓冲。  
5. **跨平台性**：套接字最通用，共享内存和信号量依赖操作系统。  

**总结**

- **管道**：适合简单、单向的数据流（如 Shell 管道符 `|`）。  
- **消息队列**：适合解耦的异步通信（如`任务分发系统`）。  
- **共享内存**：适合高性能数据共享（`结合信号量同步`）。  
- **套接字**：适合`分布式系统`或`复杂通信`需求。  
- **信号**：适合`轻量级事件`通知（如`进程终止`）。

### 线程间通信

**特点**

线程属于同一进程，**共享内存空间**，因此通信方式更高效，但需解决**同步问题**（避免数据竞争）：

- **直接共享数据**：通过`全局变量`、`堆内存`等共享数据。
- **无需内核介入**：`同步机制`在用户态实现（如`互斥锁`、`条件变量`）。
- **低开销**：相比进程间通信（IPC），线程通信无需系统调用或数据拷贝。

#### **核心方法**

**1. 共享内存（Shared Memory）**

- **原理**：线程直接读写进程内的`全局变量`或`堆内存`。  
- **优点**：速度最快，无数据拷贝。  
- **风险**：需`同步机制`避免竞态条件（Race Condition）。  
- **示例**（C语言）：  
  ```c
  #include <pthread.h>
  int counter = 0; // 全局变量，共享数据

  void* thread_func(void* arg) {
      for (int i = 0; i < 100000; i++) counter++;
      return NULL;
  }

  int main() {
      pthread_t t1, t2;
      pthread_create(&t1, NULL, thread_func, NULL);
      pthread_create(&t2, NULL, thread_func, NULL);
      pthread_join(t1, NULL);
      pthread_join(t2, NULL);
      printf("Counter = %d\n", counter); // 可能小于 200000（竞态条件）
      return 0;
  }
  ```

**2. 互斥锁（Mutex）**

- **原理**：通过锁机制确保同一时间只有一个线程访问共享资源。  
- **操作**：  
  - `pthread_mutex_lock()`：获取锁（阻塞直到锁可用）。  
  - `pthread_mutex_unlock()`：释放锁。  
- **示例**（修复上述竞态条件）：  
  ```c
  pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;

  void* thread_func(void* arg) {
      for (int i = 0; i < 100000; i++) {
          pthread_mutex_lock(&mutex);
          counter++;
          pthread_mutex_unlock(&mutex);
      }
      return NULL;
  }
  ```

**3. 条件变量（Condition Variable）**

- **原理**：允许线程在特定条件成立前挂起等待，条件满足时被唤醒。  
- **操作**：  
  - `pthread_cond_wait()`：释放锁并等待条件。  
  - `pthread_cond_signal()`：唤醒一个等待线程。  
  - `pthread_cond_broadcast()`：唤醒所有等待线程。  
- **示例**（生产者-消费者模型）：  
  ```c
  pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
  pthread_cond_t cond = PTHREAD_COND_INITIALIZER;
  int buffer = 0; // 共享缓冲区

  void* producer(void* arg) {
      pthread_mutex_lock(&mutex);
      buffer = 1; // 生产数据
      pthread_cond_signal(&cond); // 通知消费者
      pthread_mutex_unlock(&mutex);
      return NULL;
  }

  void* consumer(void* arg) {
      pthread_mutex_lock(&mutex);
      while (buffer == 0) pthread_cond_wait(&cond, &mutex); // 等待数据
      printf("Consumed: %d\n", buffer);
      pthread_mutex_unlock(&mutex);
      return NULL;
  }
  ```

**4. 信号量（Semaphore）**

- **原理**：通过计数器控制资源访问，支持多线程同步。  
- **操作**：  
  - `sem_wait()`：减少信号量（若为0则阻塞）。  
  - `sem_post()`：增加信号量。  
- **示例**（限制并发线程数）：  
  ```c
  #include <semaphore.h>
  sem_t sem;

  void* thread_func(void* arg) {
      sem_wait(&sem); // 获取信号量
      // 访问共享资源
      sem_post(&sem); // 释放信号量
      return NULL;
  }

  int main() {
      sem_init(&sem, 0, 3); // 初始值3，允许3个线程并发
      // 创建多个线程...
      sem_destroy(&sem);
      return 0;
  }
  ```

**5. 原子操作（Atomic Operations）**

- **原理**：硬件支持的不可中断操作，适用于简单变量（如计数器）。  
- **优点**：无锁，性能高。  
- **示例**（C11原子变量）：  
  ```c
  #include <stdatomic.h>
  atomic_int counter = ATOMIC_VAR_INIT(0);

  void* thread_func(void* arg) {
      for (int i = 0; i < 100000; i++) atomic_fetch_add(&counter, 1);
      return NULL;
  }
  ```

#### 同步问题

**1. 死锁（Deadlock）**

- **成因**：多个线程互相等待对方释放锁。  
- **避免方法**：  
  - 按固定顺序获取锁。  
  - 使用超时机制（如 `pthread_mutex_trylock()`）。  

**2. 虚假唤醒（Spurious Wakeup）**

- **成因**：条件变量可能在未收到信号时被唤醒。  
- **解决方法**：始终在循环中检查条件：  
  ```c
  while (condition_is_false) pthread_cond_wait(&cond, &mutex);
  ```

**3. 优先级反转（Priority Inversion）**

- **成因**：低优先级线程持有高优先级线程所需的锁。  
- **解决方法**：使用优先级继承协议（如 `pthread_mutexattr_setprotocol()`）。

#### 不同方法的对比

| **方法**         | **适用场景**                      | **优点**          | **缺点**               |
|------------------|-----------------------------------|-------------------|------------------------|
| **共享内存**     | 高频数据交换（需同步）            | 极快              | 需手动同步             |
| **互斥锁**       | 保护临界区（如全局变量）          | 简单易用          | 可能引发死锁           |
| **条件变量**     | 线程间事件通知（如生产者-消费者）  | 高效等待/通知     | 需与互斥锁配合         |
| **信号量**       | 控制资源并发访问（如连接池）      | 灵活控制并发数    | 复杂场景不易管理       |
| **原子操作**     | 简单计数器、标志位                | 无锁，高性能      | 仅支持简单数据类型     |

**总结**

- **线程间通信的核心是共享内存 + 同步机制**。
- **互斥锁和条件变量**是解决复杂同步问题的基石。  
- **原子操作和轻量级锁**（如自旋锁）适用于高性能场景。  
- **避免竞态和死锁**是设计多线程程序的关键挑战。
