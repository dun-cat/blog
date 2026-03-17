## 操作系统及计算机组成原理 (二) - 信号量 
信号量（**Semaphore**, 英 /ˈseməfɔː(r)/；美 /ˈseməfɔːr/）是操作系统中用于**多线程/多进程同步**的核心工具，由`荷兰`计算机科学家 [Dijkstra](https://en.wikipedia.org/wiki/Edsger_W._Dijkstra) (/ˈdaɪkstrəz/, 1930-2002) 在 1965 年提出。它的本质是一个**计数器**，用于管理对共享资源的访问权限，`避免竞态条件`（Race Condition）和`死锁`（Deadlock）。

### 概念

**1. 核心作用**

- **协调多个线程/进程的`并行执行顺序`**
- **控制对共享资源的访问数量**（例如：`数据库连接池`、`打印机`等有限资源）

**2. 信号量的组成**

- **整数值（Count）**：表示可用资源的数量。
- **等待队列（Wait Queue）**：当资源不足时，阻塞并记录等待的线程/进程。
- **原子操作**：通过 `P()`（申请资源）和 `V()`（释放资源）操作修改计数值。
  - **P 操作**：也称为`等待操作`或`减 1 操作`，用于`申请资源`。当执行 P 操作时，信号量的值会`减 1`，如果`信号量的值 < 0`，则进程或线程会被`阻塞`，直到有其他进程或线程释放资源。
  - **V 操作**：也称为`释放操作`或`加 1 操作`，用于`释放资源`。当执行 V 操作时，信号量的值会`加 1`，如果`信号量的值 = 0`，则会`唤醒`一个正在`等待`该资源的进程或线程。

### 操作原理

**1. P() 操作（Proberen，荷兰语“尝试”）**

- **功能**：`申请资源`，若资源不足则阻塞。
- **伪代码逻辑**：

  ```plaintext
  P(semaphore S):
      if S.count > 0:
          S.count -= 1
      else:
          将当前线程加入 S 的等待队列，并阻塞
  ```

**2. V() 操作（Verhogen，荷兰语“增加”）**

- **功能**：`释放资源`，唤醒等待线程。
- **伪代码逻辑**：

  ```plaintext
  V(semaphore S):
      if S 的等待队列不为空:
          唤醒队列中的一个线程
      else:
          S.count += 1
  ```

### 类型

#### 二进制信号量

`二进制信号量`（Binary Semaphore）是一种特殊的信号量，它的值只能取 `0` 或 `1`。它主要用于`实现互斥`和`简单的同步`功能，确保在任何时刻只有一个进程能够访问共享资源，或者用于控制进程之间的执行顺序。

- **计数值范围**：`0` 或 `1`（类似`互斥锁` **Mutex**）。
- **用途**：保护单一共享资源（如`打印机`）。
- **示例**：

  ```c
  sem_t mutex;
  sem_init(&mutex, 0, 1); // 初始值为 1 , & 取址符

  // 线程进入临界区前
  sem_wait(&mutex);   // P 操作：申请资源
  // 访问共享资源...
  sem_post(&mutex);   // V 操作：释放资源
  ```

``` c
int sem_init(sem_t *sem, int pshared, unsigned int value);
```

`参数说明`：

- `sem`：指向要初始化的信号量对象的指针，sem_t 是信号量的数据类型。
- `pshared`：指定信号量的共享选项，有以下两种取值：
  - **0**：表示信号量是在当前进程的多个线程之间共享的。
  - **非零值**：表示信号量可以在`多个进程之间共享`。不过，要在进程间共享信号量，信号量必须存放在`共享内存区域`。
- `value`：信号量的初始值，该值必须是非负整数。信号量的值通常用于表示可用资源的数量。

`返回值:`

- 成功：返回 0。
- 失败：返回 -1，并设置 errno 来指示错误类型。

多个进程可能需要在不同条件下等待或唤醒以进行文件读写操作时，会使用信号量机制。例如，在文件服务器场景中，多个客户端进程可能同时请求访问服务器上的文件资源

#### 计数信号量

`计数信号量`（Counting Semaphore）的值可以是任意非负整数。它用于控制多个进程对一组有限资源的访问，记录可用资源的数量。

- **计数值范围**：非负整数（`≥0`）。
- **用途**：管理多个同类资源（如连接池中的 10 个`数据库连接`）。
- **示例**：

  ```c
  sem_t db_connections;
  sem_init(&db_connections, 0, 10); // 初始值为 10

  // 线程获取数据库连接
  sem_wait(&db_connections); // P 操作：计数值减 1
  // 使用连接...
  sem_post(&db_connections); // V 操作：计数值加 1
  ```

### POSIX 信号量

`POSIX`（Portable Operating System Interface）标准定义了一套用于操作信号量的系统调用，主要分为命名信号量和无名信号量。

#### 命名信号量

**命名信号量**：用于不同进程之间的`同步`和`互斥`。相关的系统调用包括：

- `sem_open`：用于创建或打开一个命名信号量。
- `sem_wait`：对应 P 操作，申请资源。
- `sem_post`：对应 V 操作，释放资源。
- `sem_close`：关闭一个命名信号量。
- `sem_unlink`：删除一个命名信号量。

**核心机制**

- `全局名称`：通过唯一的名称（如 /my_sem）标识信号量。
- `内核管理`：信号量由操作系统内核维护，独立于进程生命周期。

**示例**

进程A：创建信号量并等待

```c
#include <stdio.h>
#include <fcntl.h>
#include <sys/stat.h>
#include <semaphore.h>

#define SEM_NAME "/demo_sem"

int main() {
    sem_t *sem = sem_open(SEM_NAME, O_CREAT, 0644, 0);  // 初始值为 0
    if (sem == SEM_FAILED) {
        perror("sem_open");
        return 1;
    }

    printf("Process A: Waiting for signal...\n");
    sem_wait(sem);  // 阻塞直到信号量值 > 0
    printf("Process A: Received signal!\n");

    sem_close(sem);
    sem_unlink(SEM_NAME);  // 删除信号量
    return 0;
}
```

进程B：触发信号量

```c
#include <stdio.h>
#include <fcntl.h>
#include <sys/stat.h>
#include <semaphore.h>

#define SEM_NAME "/demo_sem"

int main() {
    sem_t *sem = sem_open(SEM_NAME, 0);  // 打开已存在的信号量
    if (sem == SEM_FAILED) {
        perror("sem_open");
        return 1;
    }

    printf("Process B: Sending signal...\n");
    sem_post(sem);  // 增加信号量值，唤醒进程A

    sem_close(sem);
    return 0;
}
```

编译：

```bash
gcc -o process_a process_a.c -lrt
gcc -o process_b process_b.c -lrt
```

#### 匿名信号量

**核心机制**

- `共享内存`：将信号量存储在共享内存中，供多个进程访问。

**示例**

共享内存和信号量定义（shared.h）

```c
// shared.h
#include <sys/shm.h>
#include <semaphore.h>

#define SHM_KEY 1234
#define SEM_SIZE sizeof(sem_t)

struct shared_data {
    sem_t sem;
    int counter;
};
```

进程 A：创建共享内存并初始化信号量

```c
#include "shared.h"
#include <stdio.h>
#include <stdlib.h>
#include <sys/shm.h>

int main() {
    // 创建共享内存
    int shmid = shmget(SHM_KEY, sizeof(struct shared_data), IPC_CREAT | 0666);
    if (shmid == -1) {
        perror("shmget");
        exit(1);
    }

    // 附加共享内存进程地址空间
    struct shared_data *shm = shmat(shmid, NULL, 0);
    if (shm == (void *)-1) {
        perror("shmat");
        exit(1);
    }

    // 初始化匿名信号量（跨进程共享需设置 pshared = 1）
    if (sem_init(&shm->sem, 1, 0) == -1) {
        perror("sem_init");
        exit(1);
    }

    printf("Process A: Waiting for signal...\n");
    sem_wait(&shm->sem);  // 阻塞等待
    printf("Process A: Counter = %d\n", shm->counter);

    // 清理
    sem_destroy(&shm->sem);
    shmdt(shm);
    shmctl(shmid, IPC_RMID, NULL);
    return 0;
}
```

进程 B：访问共享内存并触发信号量

```c
#include "shared.h"
#include <stdio.h>
#include <stdlib.h>
#include <sys/shm.h>

int main() {
    // 获取共享内存
    int shmid = shmget(SHM_KEY, sizeof(struct shared_data), 0666);
    if (shmid == -1) {
        perror("shmget");
        exit(1);
    }

    // 附加共享内存
    struct shared_data *shm = shmat(shmid, NULL, 0);
    if (shm == (void *)-1) {
        perror("shmat");
        exit(1);
    }

    // 修改数据并发送信号
    shm->counter = 42;
    sem_post(&shm->sem);  // 唤醒进程 A

    shmdt(shm);
    return 0;
}
```

- `shmget` 函数`创建`一个大小为 1024 字节的`共享内存段`。
- `shmat` 函数将共享内存段`附加`到`进程的地址空间`中，并向共享内存段写入一条消息。
- `shmdt` 函数将共享内存段从进程的地址空间中`分离`。

**关键注意事项**

1. **命名信号量的名称规则**：
   - Linux要求命名信号量以 `/` 开头（如 `/my_sem`），且名称长度通常有限制。
   - Windows 的命名信号量规则不同，需参考具体 API 文档。

2. **匿名信号量的共享内存权限**：
   - 使用 `shmget` 时需设置正确的权限（如 `0666`）。
   - 确保信号量在共享内存中正确对齐（避免内存访问错误）。

3. **信号量的销毁**：
   - 命名信号量需显式调用 `sem_unlink()` 防止资源泄漏。
   - 匿名信号量随共享内存销毁而释放。

### 经典应用场景

#### 生产者-消费者问题

一个过程（生产者）生成数据项，另一个过程（消费者）会接收并使用它们。它们使用最大尺寸 n 的队列进行交流，并受到以下条件的约束：

- 如果队列为`空`，消费者必须等待生产者生产一些东西。
- 如果队列`已满`，则生产者必须等待消费者消费。

##### 问题描述

- **生产者**：生成数据并放入缓冲区。
- **消费者**：从缓冲区取出数据并处理。
- **缓冲区**：固定大小的循环队列，需保证线程安全。
- **同步目标**：`防止缓冲区溢出`（生产者等待空位）或`消费空数据`（消费者等待数据）。

##### 代码实现

```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <semaphore.h>

#define BUFFER_SIZE 5
#define MAX_ITEMS 10

int buffer[BUFFER_SIZE];       // 共享缓冲区
int in = 0, out = 0;           // 缓冲区索引

sem_t mutex, empty, full;      // 信号量

void *producer(void *arg) {
    int item;
    for (int i = 0; i < MAX_ITEMS; i++) {
        item = rand() % 100;   // 模拟生产数据

        sem_wait(&empty);      // 等待空位（P(empty)）
        sem_wait(&mutex);      // 进入临界区（P(mutex)）

        buffer[in] = item;
        printf("生产者生产: %d 于位置 %d\n", item, in);
        in = (in + 1) % BUFFER_SIZE;

        sem_post(&mutex);      // 离开临界区（V(mutex)）
        sem_post(&full);       // 增加满位（V(full)）
    }
    return NULL;
}

void *consumer(void *arg) {
    int item;
    for (int i = 0; i < MAX_ITEMS; i++) {
        sem_wait(&full);       // 等待数据（P(full)）
        sem_wait(&mutex);      // 进入临界区（P(mutex)）

        item = buffer[out];
        printf("消费者消费: %d 自位置 %d\n", item, out);
        out = (out + 1) % BUFFER_SIZE;

        sem_post(&mutex);      // 离开临界区（V(mutex)）
        sem_post(&empty);      // 增加空位（V(empty)）
    }
    return NULL;
}

int main() {
    pthread_t prod_thread, cons_thread;

    // 初始化信号量
    sem_init(&mutex, 0, 1);    // 互斥信号量初始为1（二进制锁）
    sem_init(&empty, 0, BUFFER_SIZE);  // 空位初始为缓冲区大小
    sem_init(&full, 0, 0);     // 满位初始为0

    // 创建线程
    pthread_create(&prod_thread, NULL, producer, NULL);
    pthread_create(&cons_thread, NULL, consumer, NULL);

    // 等待线程结束
    pthread_join(prod_thread, NULL);
    pthread_join(cons_thread, NULL);

    // 销毁信号量
    sem_destroy(&mutex);
    sem_destroy(&empty);
    sem_destroy(&full);

    return 0;
}
```

 **代码解析**

1. **信号量初始化**：
   - `mutex`：初始值为 `1`，确保`缓冲区操作原子性`。
   - `empty`：初始值为缓冲区大小（`BUFFER_SIZE = 5`），表示初始空位数。
   - `full`：初始值为 `0`，表示初始无数据。

2. **生产者逻辑**：
   - **等待空位**：`sem_wait(&empty)` 保证缓冲区不满。
   - **获取锁**：`sem_wait(&mutex)` 进入临界区。
   - **生产数据**：向缓冲区写入数据，更新写入位置 `in`。
   - **释放锁**：`sem_post(&mutex)` 退出临界区。
   - **通知消费者**：`sem_post(&full)` 增加满位计数。

3. **消费者逻辑**：
   - **等待数据**：`sem_wait(&full)` 保证缓冲区不空。
   - **获取锁**：`sem_wait(&mutex)` 进入临界区。
   - **消费数据**：从缓冲区取出数据，更新读取位置 `out`。
   - **释放锁**：`sem_post(&mutex)` 退出临界区。
   - **通知生产者**：`sem_post(&empty)` 增加空位计数。

4. **缓冲区管理**：
   - 使用循环队列（`in` 和 `out` 索引模运算），避免浪费空间。

**运行结果示例**

``` bash
生产者生产: 83 于位置 0
生产者生产: 86 于位置 1
消费者消费: 83 自位置 0
消费者消费: 86 自位置 1
生产者生产: 77 于位置 2
生产者生产: 15 于位置 3
...
（交替执行，直到生产消费各完成10次）
```

**关键注意事项**

1. **信号量操作顺序**：
   - 必须先执行 `sem_wait(&empty)` 再 `sem_wait(&mutex)`，否则可能导致死锁。
   - 若颠倒顺序（先锁后等待资源），当缓冲区满时，生产者持锁但无法生产，消费者无法获锁消费。

2. **信号量类型**：
   - `mutex` 是二进制信号量（互斥锁）。
   - `empty` 和 `full` 是计数信号量。

3. **线程安全**：
   - `in` 和 `out` 的修改必须在互斥锁保护下进行。

4. **编译命令**：

   ```bash
   gcc -o prod_cons prod_cons.c -lpthread
   ```

##### 扩展场景：多生产者与多消费者

若需支持多个生产者和消费者，只需创建更多线程，信号量逻辑无需修改：

```c
// 创建 3 个生产者和 2 个消费者
pthread_t prod_threads[3], cons_threads[2];
for (int i = 0; i < 3; i++) {
    pthread_create(&prod_threads[i], NULL, producer, NULL);
}
for (int i = 0; i < 2; i++) {
    pthread_create(&cons_threads[i], NULL, consumer, NULL);
}
```

通过信号量的协调，生产者和消费者可以`安全`、`高效`地`共享缓冲区`，`避免竞态条件`和`资源浪费`。

#### **读者-写者问题**

- **问题**：多个读者可同时读数据，但写者必须独占访问。
- **信号量方案**：
  - **rw_mutex**：保护写操作的互斥锁（二进制信号量）。
  - **read_count_mutex**：保护读者计数的互斥锁。
  - **read_count**：当前活跃的读者数量。

### 信号量 vs 互斥锁（Mutex）

基本功能：二进制信号量和互斥锁都可以用于实现对`共享资源的互斥访问`，确保在同一时刻只有一个线程或进程能够访问临界区，防止数据冲突和不一致性。

| **特性**         | **信号量**                  | **互斥锁**                |
|------------------|----------------------------|--------------------------|
| **计数值**       | 任意非负整数               | 0 或 1（二进制）           |
| **所有权**       | 无归属，可由任意线程释放   | 必须由`加锁线程解锁`       |
| **用途**         | 同步、资源计数             | 保护临界区               |
| **灵活性**       | 更灵活（可控制多资源访问） | 简单，仅保护单一临界区   |

信号量主要关注资源的计数，而不是特定线程的所有权。

### 注意事项

1. **死锁风险**：错误的 `P()`/`V()` 顺序可能导致死锁（例如：`P(A); P(B);` 和 `P(B); P(A);` 并发执行）。
2. **优先级反转**：低优先级任务持有信号量时，可能阻塞高优先级任务。
3. **资源泄漏**：忘记调用 `V()` 会导致资源永久不可用。

### Java 多线程编程

在 Java 中，Semaphore 类位于 `java.util.concurrent` 包下。下面是一个简单的示例代码：

``` java
import java.util.concurrent.Semaphore;

public class SemaphoreExample {
    public static void main(String[] args) {
        // 创建一个信号量，初始许可数量为 2，表示最多允许 2 个线程同时访问资源
        Semaphore semaphore = new Semaphore(2);

        // 创建 5 个线程
        for (int i = 0; i < 5; i++) {
            final int threadId = i;
            new Thread(() -> {
                try {
                    // 获取许可
                    semaphore.acquire();
                    System.out.println("线程 " + threadId + " 已获取许可，开始执行任务");
                    // 模拟任务执行
                    Thread.sleep(2000);
                    System.out.println("线程 " + threadId + " 完成任务，释放许可");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    // 释放许可
                    semaphore.release();
                }
            }).start();
        }
    }
}
```

代码解释:

1. `创建 Semaphore 对象`：Semaphore semaphore = new Semaphore(2); 表示创建一个初始`许可数量`为 2 的信号量，即最多允许 2 个线程同时访问共享资源。
2. `获取许可`：semaphore.acquire(); 线程调用该方法来获取许可，如果没有可用许可，线程会被阻塞。
3. `释放许可`：semaphore.release(); 线程执行完任务后，需要调用该方法释放许可，以便其他线程可以获取许可。
