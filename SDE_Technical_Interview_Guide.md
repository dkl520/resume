# 💻 SDE技术面试八股文手册
## Software Development Engineer Technical Interview Guide

> 📚 **使用说明**: 本文档涵盖SDE面试高频技术问题，中英双语对照，建议逐个背诵理解

---

## 📋 目录 Table of Contents

1. [操作系统 Operating System](#os)
2. [计算机网络 Computer Network](#network)
3. [数据结构与算法 Data Structures & Algorithms](#dsa)
4. [数据库 Database](#database)
5. [Java核心 Java Fundamentals](#java)
6. [并发编程 Concurrency](#concurrency)
7. [Spring框架 Spring Framework](#spring)
8. [分布式系统 Distributed Systems](#distributed)
9. [微服务架构 Microservices](#microservices)
10. [Redis缓存 Redis Cache](#redis)

---

<a name="os"></a>
## 🖥️ 操作系统 Operating System

### Q1: 进程和线程的区别？
### What's the difference between Process and Thread?

**🇨🇳 中文回答：**

| 维度 | 进程 Process | 线程 Thread |
|------|-------------|-------------|
| **定义** | 系统资源分配的最小单位 | CPU调度的最小单位 |
| **资源拥有** | 拥有独立的内存空间、文件描述符 | 共享进程的内存空间和资源 |
| **开销** | 创建和切换开销大（需要切换页表、刷新TLB） | 开销小（只需切换寄存器和栈） |
| **通信方式** | 进程间通信（IPC）：管道、消息队列、共享内存 | 直接读写共享变量（需要加锁） |
| **崩溃影响** | 一个进程崩溃不影响其他进程 | 一个线程崩溃可能导致整个进程崩溃 |

**核心区别：**
- 进程是资源分配单位，线程是调度执行单位
- 进程有独立地址空间，线程共享进程地址空间
- 线程切换比进程切换快10-100倍

**🇺🇸 English Answer:**

| Aspect | Process | Thread |
|--------|---------|--------|
| **Definition** | Minimum unit of resource allocation | Minimum unit of CPU scheduling |
| **Resources** | Independent memory space, file descriptors | Shares process memory and resources |
| **Overhead** | High (context switch involves page table, TLB flush) | Low (only registers and stack switch) |
| **Communication** | IPC: pipes, message queues, shared memory | Direct access to shared variables (requires locks) |
| **Crash Impact** | Isolated - one crash doesn't affect others | One thread crash may crash entire process |

**Key Differences:**
- Process = resource allocation unit; Thread = execution unit
- Processes have isolated address spaces; threads share address space
- Thread context switching is 10-100x faster than process switching

---

### Q2: 进程间通信（IPC）的方式有哪些？
### What are the methods of Inter-Process Communication (IPC)?

**🇨🇳 中文回答：**

#### 1️⃣ **管道 Pipe**
```bash
# 匿名管道（只能用于父子进程）
ps aux | grep java

# 命名管道（FIFO，可用于无关进程）
mkfifo mypipe
```
- ✅ 简单易用
- ❌ 单向通信，只能用于有亲缘关系的进程（匿名管道）

#### 2️⃣ **消息队列 Message Queue**
```c
// 发送消息
msgsnd(msgid, &msg, sizeof(msg), 0);
// 接收消息
msgrcv(msgid, &msg, sizeof(msg), 1, 0);
```
- ✅ 支持多对多通信，消息有类型区分
- ❌ 消息大小有限制，需要内核缓冲区

#### 3️⃣ **共享内存 Shared Memory** ⭐ 最快
```c
// 创建共享内存
shmid = shmget(key, size, IPC_CREAT);
// 映射到进程地址空间
ptr = shmat(shmid, NULL, 0);
```
- ✅ 速度最快（无需数据拷贝，直接读写内存）
- ❌ 需要同步机制（信号量）防止竞态条件

#### 4️⃣ **信号量 Semaphore**
```c
// P操作（等待）
sem_wait(&sem);
// V操作（释放）
sem_post(&sem);
```
- ✅ 用于进程同步，控制并发访问
- ❌ 本身不传递数据，只做同步

#### 5️⃣ **信号 Signal**
```bash
kill -9 <pid>  # SIGKILL
kill -15 <pid> # SIGTERM
```
- ✅ 异步通知机制
- ❌ 携带信息量少（只有信号类型）

#### 6️⃣ **套接字 Socket**
```python
# TCP Socket
server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server.bind(('localhost', 8080))
```
- ✅ 支持跨网络通信，最通用
- ❌ 性能开销相对较大

**🇺🇸 English Answer:**

#### IPC Methods Comparison

| Method | Speed | Use Case |
|--------|-------|----------|
| **Shared Memory** | ⭐ Fastest | High-performance local IPC |
| **Pipe** | Medium | Simple parent-child communication |
| **Message Queue** | Medium | Many-to-many messaging |
| **Socket** | Slowest | Network communication |

**💡 选择建议:**
- 性能优先 → 共享内存 (Performance → Shared Memory)
- 简单通信 → 管道 (Simple → Pipe)
- 跨网络 → Socket (Network → Socket)

---

### Q3: 虚拟内存是什么？为什么需要？
### What is Virtual Memory? Why do we need it?

**🇨🇳 中文回答：**

**定义：** 虚拟内存是操作系统提供的一种内存管理技术，让每个进程都认为自己拥有连续的、独占的地址空间。

**核心机制：**
```
虚拟地址 → [页表映射] → 物理地址
Virtual Address → [Page Table] → Physical Address
```

**为什么需要虚拟内存？**

#### 1️⃣ **内存隔离（安全性）**
```
进程A: 0x0000 - 0xFFFF  →  物理内存 0x1000 - 0x1FFF
进程B: 0x0000 - 0xFFFF  →  物理内存 0x2000 - 0x2FFF
```
每个进程有独立地址空间，互不干扰，防止非法访问

#### 2️⃣ **突破物理内存限制**
- 虚拟内存 = 物理内存 + 磁盘交换空间（Swap）
- 可以运行超过物理内存大小的程序（通过页面置换算法）

#### 3️⃣ **内存碎片管理**
```
物理内存（碎片化）：[已用][空闲][已用][空闲]
虚拟内存（连续）：  [连续的虚拟地址空间]
```
虚拟地址连续，物理地址可以不连续

#### 4️⃣ **按需分配（懒加载）**
```c
char *p = malloc(1GB);  // 只分配虚拟地址，不分配物理内存
p[0] = 'a';            // 访问时触发缺页中断，分配物理页
```

**关键组件：**
- **页表 Page Table**: 虚拟地址到物理地址的映射
- **TLB (Translation Lookaside Buffer)**: 页表缓存，加速地址转换
- **页面置换算法**: LRU、LFU、Clock算法

**🇺🇸 English Answer:**

**Why Virtual Memory?**

1️⃣ **Memory Isolation** - Each process has isolated address space, prevents unauthorized access
2️⃣ **Exceed Physical RAM** - Virtual Memory = RAM + Disk Swap, run programs larger than RAM
3️⃣ **Fragmentation Management** - Virtual addresses continuous, physical can be fragmented
4️⃣ **Demand Paging** - Allocate virtual addresses immediately, physical pages on first access

**Key Components:**
- **Page Table**: Virtual-to-physical mapping
- **TLB**: Page table cache for fast translation
- **Page Replacement**: LRU, LFU, Clock algorithms

---

### Q4: 死锁的四个必要条件和解决方法？
### What are the 4 necessary conditions for deadlock and solutions?

**🇨🇳 中文回答：**

**死锁定义：** 两个或多个进程互相持有对方需要的资源，永久等待的状态。

**四个必要条件（同时满足才会死锁）：**

#### 1️⃣ **互斥条件 Mutual Exclusion**
资源不能被多个进程同时使用

#### 2️⃣ **持有并等待 Hold and Wait**
进程持有至少一个资源，同时等待获取其他资源

#### 3️⃣ **不可剥夺 No Preemption**
资源不能被强制剥夺，只能由持有者主动释放

#### 4️⃣ **循环等待 Circular Wait**
存在进程等待环：P1 → P2 → P3 → P1

**经典死锁示例：**
```java
// ❌ 死锁代码
Thread1:
  synchronized(lockA) {
    Thread.sleep(100);
    synchronized(lockB) { /* work */ }
  }

Thread2:
  synchronized(lockB) {
    Thread.sleep(100);
    synchronized(lockA) { /* work */ }
  }
```

**解决方法：**

#### ✅ **预防 Prevention - 破坏四个条件之一**

**方法1: 破坏"持有并等待"**
```java
// 一次性获取所有资源
synchronized(lockManager) {
    acquire(lockA);
    acquire(lockB);
}
```

**方法2: 破坏"循环等待" - 资源排序** ⭐ 最常用
```java
// 规定锁的获取顺序（按ID大小）
Lock first = lockA.id < lockB.id ? lockA : lockB;
Lock second = lockA.id < lockB.id ? lockB : lockA;

synchronized(first) {
    synchronized(second) {
        // work
    }
}
```

**方法3: 破坏"不可剥夺"**
```java
// 使用tryLock()，获取失败则释放已持有的锁
if (lockA.tryLock()) {
    try {
        if (lockB.tryLock()) {
            try {
                // work
            } finally {
                lockB.unlock();
            }
        }
    } finally {
        lockA.unlock();
    }
}
```

**🇺🇸 English Answer:**

**Four Necessary Conditions:**
1. **Mutual Exclusion** - Resource can't be shared
2. **Hold and Wait** - Process holds resources while waiting
3. **No Preemption** - Resources can't be forcibly taken
4. **Circular Wait** - Cycle in resource dependency graph

**Solutions:**

✅ **Prevention - Break one condition:**
- **Resource Ordering** (most common): Always acquire locks in same order
- **tryLock()**: Release held locks if can't acquire all

✅ **Avoidance**: Banker's algorithm

✅ **Detection & Recovery**: Periodically detect cycles, terminate processes

---

<a name="network"></a>
## 🌐 计算机网络 Computer Network

### Q5: TCP三次握手和四次挥手详解
### Explain TCP Three-Way Handshake and Four-Way Termination

**🇨🇳 中文回答：**

#### 🤝 三次握手（建立连接）

```
Client                          Server
  |-------- SYN (seq=x) -------→ |  1️⃣ Client发起连接
  |←----- SYN-ACK (seq=y) -------|  2️⃣ Server确认并发起连接
  |      (ack=x+1)                |
  |-------- ACK (ack=y+1) -----→ |  3️⃣ Client确认
 [ESTABLISHED]               [ESTABLISHED]
```

**为什么是三次，不是两次？**

❌ **两次握手的问题：**
- 旧的SYN包延迟到达，Server误认为是新连接
- Server建立无效连接，浪费资源

✅ **三次握手的保障：**
- 第三次握手让Server确认Client收到了SYN-ACK
- 防止历史连接重复建立
- 双方同步初始序列号（ISN）

#### 👋 四次挥手（断开连接）

```
Client                          Server
  |-------- FIN (seq=u) -------→ |  1️⃣ Client主动关闭
  |←------- ACK (ack=u+1) -------|  2️⃣ Server确认
  |                          [半关闭状态]
  |←------- FIN (seq=v) ---------|  3️⃣ Server关闭连接
  |-------- ACK (ack=v+1) -----→ |  4️⃣ Client最终确认
 [TIME_WAIT 2MSL]             [CLOSED]
 [CLOSED]
```

**为什么是四次，不是三次？**

因为TCP是**全双工**通信：
- FIN1 + ACK1：关闭Client→Server方向
- FIN2 + ACK2：关闭Server→Client方向
- 中间的半关闭状态允许Server继续发送未完成的数据

**TIME_WAIT状态（2MSL）的作用：**
1. 确保最后的ACK送达（如果丢失，Server会重发FIN）
2. 防止旧连接干扰新连接（等待旧数据包全部消失）

**🇺🇸 English Answer:**

#### Three-Way Handshake

**Why 3-way, not 2-way?**
- Prevents old delayed SYN packets from establishing invalid connections
- Both sides confirm connection and synchronize sequence numbers

#### Four-Way Termination

**Why 4-way, not 3-way?**
- TCP is full-duplex: need to close both directions independently
- Half-close state allows server to finish sending data

**TIME_WAIT (2MSL):**
- Ensure final ACK reaches server
- Prevent old packets from interfering with new connections

**💡 实战问题:**
- 大量TIME_WAIT → 调整`tcp_tw_reuse`, `tcp_max_tw_buckets`
- CLOSE_WAIT堆积 → 应用未正确关闭socket

---

### Q6: HTTP和HTTPS的区别？HTTPS加密过程？
### Difference between HTTP and HTTPS? HTTPS encryption?

**🇨🇳 中文回答：**

#### HTTP vs HTTPS 对比

| 特性 | HTTP | HTTPS |
|------|------|-------|
| **端口** | 80 | 443 |
| **安全性** | 明文传输 | SSL/TLS加密 |
| **证书** | 不需要 | 需要CA证书 |
| **性能** | 快 | 稍慢（加密开销） |

#### HTTPS加密流程（SSL/TLS握手）

```
Client                                    Server
  |------ ClientHello ------------------→ |
  | (支持的加密套件、随机数Random1)        |
  |←----- ServerHello -------------------|
  | (选择的加密套件、Random2、证书)        |
  |------ 验证证书 -------------------------|
  | (检查CA签名、域名、有效期)               |
  |------ PreMaster Secret -----------→  |
  | (用服务器公钥加密)                      |
  |  生成会话密钥 = f(Random1, Random2, PreMaster) |
  [开始用对称加密传输数据]
```

**为什么结合非对称和对称加密？**

| 加密方式 | 优点 | 缺点 | 用途 |
|---------|------|------|------|
| 非对称加密（RSA）| 安全 | 慢（1000倍）| 握手阶段 |
| 对称加密（AES）| 快 | 需密钥交换 | 数据传输 |

**🇺🇸 English Answer:**

#### HTTPS Encryption Process

1️⃣ **ClientHello** - Send cipher suites, Random1
2️⃣ **ServerHello** - Send chosen cipher, Random2, certificate  
3️⃣ **Certificate Verification** - Validate CA signature, domain
4️⃣ **Key Exchange** - Encrypt PreMaster with server's public key
5️⃣ **Session Key** - Both compute: `Key = PRF(PreMaster, R1, R2)`
6️⃣ **Data Transfer** - Use AES-256 symmetric encryption

**Why hybrid encryption?**
- Asymmetric (RSA): Secure but slow → used in handshake
- Symmetric (AES): Fast → used for data transmission

---

### Q7: GET和POST的区别？
### Difference between GET and POST?

**🇨🇳 中文回答：**

#### 表面区别

| 特性 | GET | POST |
|------|-----|------|
| **参数位置** | URL查询字符串 | 请求体 |
| **缓存** | 可缓存 | 不缓存 |
| **长度限制** | URL限制（2-8KB） | 无限制 |
| **幂等性** | 幂等 | 非幂等 |

#### ❌ 常见误区

**误区1："GET限制2KB"** - HTTP协议本身无限制，限制来自浏览器

**误区2："POST更安全"** - 两者在HTTP下都是明文，真正安全靠HTTPS

#### ✅ 本质区别（RFC 7231）

**GET - 获取资源（Read）**
```http
GET /users/123 HTTP/1.1
# 幂等、安全、可缓存
# 不应有副作用
```

**POST - 创建资源（Create）**
```http
POST /users HTTP/1.1
Content-Type: application/json

{"name": "Alice"}
# 非幂等（多次调用创建多个资源）
# 有副作用
```

**RESTful API最佳实践：**
```http
GET    /users      # 查询所有
GET    /users/123  # 查询单个
POST   /users      # 创建
PUT    /users/123  # 更新（幂等）
DELETE /users/123  # 删除
```

**🇺🇸 English Answer:**

#### Essential Difference

**Semantic:**
- **GET**: Retrieve resource (idempotent, safe, cacheable)
- **POST**: Create resource (non-idempotent, has side effects)

**Common Misconceptions:**
- "GET has 2KB limit" → HTTP has no limit, browsers do
- "POST is more secure" → Both plaintext in HTTP, use HTTPS for security

---

<a name="dsa"></a>
## 🧮 数据结构与算法 DSA

### Q8: 常见排序算法的时间复杂度和稳定性？
### Time complexity and stability of sorting algorithms?

**🇨🇳 中文回答：**

| 算法 | 平均时间 | 最坏时间 | 空间 | 稳定性 | 场景 |
|------|---------|---------|------|--------|------|
| **快速排序** | O(n log n) | O(n²) | O(log n) | ❌ | ⭐通用排序 |
| **归并排序** | O(n log n) | O(n log n) | O(n) | ✅ | 稳定排序 |
| **堆排序** | O(n log n) | O(n log n) | O(1) | ❌ | Top K |
| **插入排序** | O(n²) | O(n²) | O(1) | ✅ | 小数据/近乎有序 |

#### 🔥 快速排序（最常用）

```java
public void quickSort(int[] arr, int left, int right) {
    if (left >= right) return;
    
    int pivot = partition(arr, left, right);
    quickSort(arr, left, pivot - 1);
    quickSort(arr, pivot + 1, right);
}

private int partition(int[] arr, int left, int right) {
    int pivot = arr[right];
    int i = left - 1;
    
    for (int j = left; j < right; j++) {
        if (arr[j] <= pivot) {
            i++;
            swap(arr, i, j);
        }
    }
    swap(arr, i + 1, right);
    return i + 1;
}
```

**优化：三数取中法 + 小数组用插入排序**

---

### Q9: HashMap的底层实现原理？
### How does HashMap work internally?

**🇨🇳 中文回答：**

#### 核心数据结构（JDK 1.8+）

```
数组 + 链表 + 红黑树

数组索引 0: Node → Node → Node
         1: TreeNode (红黑树，链表长度>8时转树)
         2: null
```

#### PUT操作流程

```java
public V put(K key, V value) {
    // 1️⃣ 计算哈希值
    int hash = hash(key);  // 高16位异或低16位
    
    // 2️⃣ 计算桶索引
    int i = (n - 1) & hash;  // 等价于 hash % n
    
    // 3️⃣ 处理冲突
    if (桶为空) {
        直接插入;
    } else if (key相同) {
        覆盖value;
    } else if (是红黑树) {
        树插入 O(log n);
    } else {
        链表插入;
        if (链表长度 >= 8) {
            转红黑树;
        }
    }
    
    // 4️⃣ 检查扩容
    if (++size > threshold) {
        resize();  // 容量扩大2倍
    }
}
```

#### 关键参数

```java
初始容量: 16
负载因子: 0.75
扩容阈值 = 16 * 0.75 = 12

链表转树阈值: 8
树转链表阈值: 6
```

**为什么负载因子是0.75？**
- 泊松分布下，桶内元素≤8的概率99.99%
- 平衡时间和空间

#### 扩容优化（JDK 1.8）

```
原容量16扩到32:
原索引5的元素 → 新索引只有两种：
- 仍是5 (hash & oldCap == 0)
- 或是5+16=21 (hash & oldCap != 0)

无需重新计算hash！
```

#### 线程安全问题

**❌ HashMap不是线程安全的**

**问题1: 并发扩容 → 环形链表（JDK 1.7）→ CPU 100%**

**问题2: 数据丢失 → 两个线程同时put到同一桶**

**解决方案：**
```java
// 1. ConcurrentHashMap（推荐）
Map<K,V> map = new ConcurrentHashMap<>();

// 2. Collections.synchronizedMap
Map<K,V> map = Collections.synchronizedMap(new HashMap<>());
```

**🇺🇸 English Answer:**

#### Internal Structure (JDK 1.8+)

```
Array + LinkedList + Red-Black Tree
- Empty bucket → insert directly
- Hash collision → append to LinkedList
- LinkedList length ≥ 8 → convert to Red-Black Tree
```

#### PUT Operation

1️⃣ Hash calculation: `(h = key.hashCode()) ^ (h >>> 16)`
2️⃣ Bucket index: `(n - 1) & hash`
3️⃣ Collision handling: LinkedList or Tree insertion
4️⃣ Resize check: If size > threshold (capacity * 0.75)

#### Thread Safety

**Not thread-safe!**
- Use `ConcurrentHashMap` for concurrent access
- Or `Collections.synchronizedMap(new HashMap<>())`

---

<a name="database"></a>
## 🗄️ 数据库 Database

### Q10: 数据库索引原理？B+树为什么适合？
### How do indexes work? Why B+ Tree?

**🇨🇳 中文回答：**

#### 为什么选择B+树？

| 数据结构 | 查找 | 为什么不用？ |
|---------|------|-------------|
| **哈希表** | O(1) | ❌ 无法范围查询 |
| **二叉树** | O(log n) | ❌ 可能退化O(n) |
| **红黑树** | O(log n) | ❌ 树太高（IO多） |
| **B+树** | O(log n) | ✅ 完美适配磁盘 |

#### B+树特性

```
                [10, 20, 30]         ← 非叶子节点（只存索引）
               /    |    |    \
      [5,7]  [12,15] [25] [35,40]   ← 非叶子节点
       / \    / \    |    / \
[1-4][5-8][9-12][13-24][25-34][35-40] ← 叶子节点（存数据）
  ↔    ↔    ↔     ↔     ↔      ↔     ← 双向链表
```

**核心特性：**
1️⃣ **所有数据在叶子节点** - 非叶子节点只存索引
2️⃣ **叶子节点有序链表** - 支持范围查询
3️⃣ **非叶子节点存更多key** - 树更矮，IO更少

#### MySQL InnoDB实例

**三层B+树容量：**
```
页大小: 16KB
非叶子节点: 16KB / 14B ≈ 1170个key
叶子节点: 16KB / 1KB ≈ 16条记录

总记录数 = 1170 × 1170 × 16 ≈ 2000万条
只需3次磁盘IO！
```

#### 聚簇索引 vs 非聚簇索引

**聚簇索引（主键）：**
```
叶子节点直接存储完整行数据
```

**非聚簇索引（辅助）：**
```
叶子节点存储索引列 + 主键
需要"回表"查询完整数据
```

**覆盖索引优化：**
```sql
CREATE INDEX idx_age_name ON users(age, name);
SELECT name FROM users WHERE age = 25;  -- 无需回表！
```

#### 索引失效场景

```sql
-- ❌ 1. 使用函数
WHERE YEAR(birthday) = 1990

-- ❌ 2. 隐式类型转换
WHERE phone = 13800138000  -- phone是VARCHAR

-- ❌ 3. 前缀通配符
WHERE name LIKE '%Alice'

-- ❌ 4. 违反最左前缀（联合索引）
INDEX(a, b, c)
WHERE b = 2 AND c = 3  -- 失效！
```

**🇺🇸 English Answer:**

#### Why B+ Tree?

**Advantages:**
- All data in leaf nodes (non-leaf only stores keys)
- Leaf nodes form linked list (enables range queries)
- Shorter tree → fewer disk I/O

**3-Level B+ Tree Capacity:**
```
20 million records with only 3 disk I/O!
```

**Clustered vs Non-Clustered:**
- Clustered (primary key): Leaf stores full row data
- Non-clustered: Leaf stores index column + primary key, requires table lookup

**Index Invalidation:**
- Using functions on indexed column
- Type mismatch (implicit conversion)
- Prefix wildcard (`LIKE '%x'`)
- Violating leftmost prefix rule

---

<a name="java"></a>
## ☕ Java核心 Java Fundamentals

### Q11: JVM内存结构？垃圾回收机制？
### JVM memory structure? Garbage collection?

**🇨🇳 中文回答：**

#### JVM内存结构

```
┌──────────────────────────────────┐
│       Method Area (方法区)        │ ← 类元数据、常量池
├──────────────────────────────────┤
│         Heap (堆)                │ ← 对象实例
│  ┌────────────┬────────────────┐ │
│  │  Young Gen │   Old Gen      │ │
│  │ Eden|S0|S1 │                │ │
│  └────────────┴────────────────┘ │
├──────────────────────────────────┤
│       Stack (栈)                 │ ← 局部变量、方法调用
├──────────────────────────────────┤
│    Program Counter (PC寄存器)    │ ← 当前指令地址
├──────────────────────────────────┤
│   Native Method Stack (本地栈)   │ ← JNI调用
└──────────────────────────────────┘
```

#### 堆内存分代

```
Young Generation (新生代):
  Eden: 80%    ← 新对象分配
  S0: 10%      ← Survivor区
  S1: 10%

Old Generation (老年代): 存活时间长的对象

Minor GC: 清理Young Gen (频繁)
Major GC: 清理Old Gen (耗时)
Full GC: 清理整个堆 (STW)
```

#### 垃圾回收算法

**1️⃣ 标记-清除 Mark-Sweep**
```
标记存活对象 → 清除未标记对象
❌ 产生内存碎片
```

**2️⃣ 标记-复制 Mark-Copy**
```
Eden满了 → 复制存活对象到Survivor
✅ 无碎片
❌ 浪费50%空间
📍 用于Young Gen
```

**3️⃣ 标记-整理 Mark-Compact**
```
标记 → 移动存活对象到一端 → 清除边界外内存
✅ 无碎片
❌ 移动对象耗时
📍 用于Old Gen
```

#### GC Roots

```java
可作为GC Roots的对象：
1. 虚拟机栈中的引用（局部变量）
2. 方法区中的静态变量
3. 方法区中的常量
4. JNI引用

不可达对象 → 垃圾回收
```

**🇺🇸 English Answer:**

#### JVM Memory Structure

- **Heap**: Object instances (Young + Old Gen)
- **Method Area**: Class metadata, constant pool
- **Stack**: Local variables, method frames
- **PC Register**: Current instruction address

#### Garbage Collection

**Minor GC**: Clean Young Gen (frequent, fast)
**Major GC**: Clean Old Gen (slow)
**Full GC**: Clean entire heap (Stop-The-World)

**GC Algorithms:**
- **Mark-Copy**: For Young Gen (no fragmentation, wastes space)
- **Mark-Compact**: For Old Gen (no fragmentation, slow)

**GC Roots:**
- Stack local variables
- Static variables
- Constants
- JNI references

---

<a name="spring"></a>
## 🍃 Spring框架 Spring Framework

### Q12: Spring IOC和AOP原理？
### Spring IOC and AOP principles?

**🇨🇳 中文回答：**

#### IOC (Inversion of Control) 控制反转

**定义：** 对象的创建和依赖注入由Spring容器管理，而不是程序主动创建。

```java
// ❌ 传统方式：程序主动创建依赖
class UserService {
    private UserDao userDao = new UserDaoImpl();  // 紧耦合
}

// ✅ IOC方式：Spring容器注入依赖
@Service
class UserService {
    @Autowired
    private UserDao userDao;  // 松耦合
}
```

**Bean生命周期：**
```
1. 实例化 Bean
2. 设置属性值 (依赖注入)
3. 调用 BeanNameAware.setBeanName()
4. 调用 BeanFactoryAware.setBeanFactory()
5. 调用 BeanPostProcessor.postProcessBeforeInitialization()
6. 调用 InitializingBean.afterPropertiesSet()
7. 调用 init-method
8. 调用 BeanPostProcessor.postProcessAfterInitialization()
9. Bean可用
10. 容器关闭 → DisposableBean.destroy()
11. 调用 destroy-method
```

#### AOP (Aspect-Oriented Programming) 面向切面编程

**定义：** 在不修改源代码的情况下，通过动态代理给程序动态添加功能。

**核心概念：**
```
Aspect (切面): @Aspect注解的类
Join Point (连接点): 方法执行点
Pointcut (切入点): 匹配Join Point的表达式
Advice (通知): Before, After, Around等
```

**示例：**
```java
@Aspect
@Component
public class LogAspect {
    
    @Pointcut("execution(* com.example.service.*.*(..))")
    public void serviceMethods() {}
    
    @Before("serviceMethods()")
    public void logBefore(JoinPoint jp) {
        System.out.println("方法执行前: " + jp.getSignature());
    }
    
    @Around("serviceMethods()")
    public Object logAround(ProceedingJoinPoint pjp) throws Throwable {
        long start = System.currentTimeMillis();
        Object result = pjp.proceed();  // 执行目标方法
        long time = System.currentTimeMillis() - start;
        System.out.println("执行耗时: " + time + "ms");
        return result;
    }
}
```

**AOP实现原理：**

**1️⃣ JDK动态代理（接口）**
```java
// UserService实现了接口
UserService proxy = (UserService) Proxy.newProxyInstance(
    classLoader,
    new Class[]{UserService.class},
    new InvocationHandler() {
        public Object invoke(Object proxy, Method method, Object[] args) {
            // Before通知
            Object result = method.invoke(target, args);  // 执行目标方法
            // After通知
            return result;
        }
    }
);
```

**2️⃣ CGLIB动态代理（类）**
```java
// UserService没有实现接口
Enhancer enhancer = new Enhancer();
enhancer.setSuperclass(UserService.class);
enhancer.setCallback(new MethodInterceptor() {
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) {
        // Before通知
        Object result = proxy.invokeSuper(obj, args);  // 执行父类方法
        // After通知
        return result;
    }
});
UserService proxy = (UserService) enhancer.create();
```

**🇺🇸 English Answer:**

#### IOC (Inversion of Control)

**Definition:** Spring container manages object creation and dependency injection.

**Bean Lifecycle:**
1. Instantiation
2. Dependency Injection
3. Aware interfaces
4. BeanPostProcessor (before)
5. InitializingBean / init-method
6. BeanPostProcessor (after)
7. Bean ready
8. Destroy

#### AOP (Aspect-Oriented Programming)

**Definition:** Add functionality dynamically via proxy without modifying source code.

**Implementation:**
- **JDK Proxy**: For interfaces (uses `Proxy.newProxyInstance`)
- **CGLIB**: For classes (subclass + method interception)

**Use Cases:**
- Logging
- Transaction management (@Transactional)
- Security
- Performance monitoring

---

<a name="distributed"></a>
## 🌍 分布式系统 Distributed Systems

### Q13: CAP定理和BASE理论？
### CAP theorem and BASE theory?

**🇨🇳 中文回答：**

#### CAP定理

```
C - Consistency   (一致性)
A - Availability  (可用性)
P - Partition Tolerance (分区容错性)

❌ 三者不可兼得，最多满足两个
```

**详解：**

**C (一致性)：** 所有节点在同一时间看到相同的数据
```
写入node1 → 立即读node2 → 能读到最新值
```

**A (可用性)：** 每个请求都能得到响应（成功或失败）
```
即使部分节点故障，系统仍能响应
```

**P (分区容错)：** 网络分区时系统仍能继续工作
```
Node1 ↔ ❌ ↔ Node2  (网络分区)
系统仍能提供服务
```

**取舍：**
```
CP系统: 一致性 + 分区容错
  → 牺牲可用性
  → 例：Zookeeper, HBase

AP系统: 可用性 + 分区容错
  → 牺牲一致性（最终一致性）
  → 例：Cassandra, DynamoDB

CA系统: 一致性 + 可用性
  → 不支持分区（单机系统）
  → 例：传统关系数据库
```

#### BASE理论

```
BA - Basically Available  (基本可用)
S  - Soft State          (软状态)
E  - Eventually Consistent (最终一致性)
```

**Basically Available：**
```
允许损失部分可用性
例：响应时间增加、部分功能降级
```

**Soft State：**
```
允许中间状态存在
例：数据同步延迟
```

**Eventually Consistent：**
```
经过一段时间后，所有副本最终一致
例：DNS更新、微博点赞数
```

**🇺🇸 English Answer:**

#### CAP Theorem

**Cannot achieve all three:**
- **C (Consistency)**: All nodes see same data
- **A (Availability)**: Every request gets response
- **P (Partition Tolerance)**: System works despite network partition

**Trade-offs:**
- **CP**: Sacrifice availability (Zookeeper, HBase)
- **AP**: Sacrifice consistency (Cassandra, eventual consistency)
- **CA**: No partition tolerance (single-node DB)

#### BASE Theory

- **BA (Basically Available)**: Partial availability OK
- **S (Soft State)**: Intermediate states allowed
- **E (Eventually Consistent)**: Eventual consistency after delay

**Example:** DNS updates, social media like counts

---

<a name="redis"></a>
## 🔴 Redis缓存 Redis Cache

### Q14: Redis数据类型和应用场景？
### Redis data types and use cases?

**🇨🇳 中文回答：**

#### 五大基本数据类型

**1️⃣ String (字符串)**
```redis
SET key value
GET key
INCR counter  # 原子自增

应用场景：
- 缓存对象（JSON）
- 计数器（点赞数、访问量）
- 分布式锁
```

**2️⃣ List (列表)**
```redis
LPUSH queue task1
RPOP queue  # 队列

应用场景：
- 消息队列
- 最新文章列表
- 粉丝列表
```

**3️⃣ Hash (哈希)**
```redis
HSET user:1 name "Alice"
HGET user:1 name

应用场景：
- 对象存储（用户信息）
- 购物车
```

**4️⃣ Set (集合)**
```redis
SADD tags:1 java python
SINTER tags:1 tags:2  # 交集

应用场景：
- 标签系统
- 共同好友
- 去重
```

**5️⃣ Sorted Set (有序集合)**
```redis
ZADD leaderboard 100 "player1"
ZRANGE leaderboard 0 9  # Top 10

应用场景：
- 排行榜
- 延迟队列（按时间排序）
```

#### 缓存穿透、击穿、雪崩

**缓存穿透：** 查询不存在的数据
```
解决方案：
1. 布隆过滤器
2. 缓存空值（短TTL）
```

**缓存击穿：** 热点key过期，大量请求打到DB
```
解决方案：
1. 热点数据永不过期
2. 互斥锁（只有一个线程查DB）
```

**缓存雪崩：** 大量key同时过期
```
解决方案：
1. 过期时间加随机值
2. Redis集群
3. 限流降级
```

**🇺🇸 English Answer:**

#### Redis Data Types

| Type | Use Case |
|------|----------|
| **String** | Cache, counter, distributed lock |
| **List** | Message queue, timeline |
| **Hash** | Object storage (user profile) |
| **Set** | Tags, de-duplication |
| **Sorted Set** | Leaderboard, delayed queue |

#### Cache Issues

**Cache Penetration:** Query non-existent data
- Solution: Bloom filter, cache null values

**Cache Breakdown:** Hot key expires, DB overwhelmed
- Solution: Never expire hot keys, mutex lock

**Cache Avalanche:** Mass key expiration
- Solution: Random TTL, cluster, rate limiting

---

## 📝 总结 Summary

**🎯 背诵重点 Key Points:**

1. **操作系统**: 进程/线程区别、虚拟内存、死锁四条件
2. **网络**: TCP三次握手/四次挥手、HTTPS加密、GET/POST区别
3. **数据结构**: 排序算法、HashMap原理
4. **数据库**: B+树索引、事务ACID、隔离级别
5. **Java**: JVM内存、GC机制、volatile/synchronized
6. **Spring**: IOC/AOP、动态代理
7. **分布式**: CAP定理、Redis缓存问题

**💡 面试技巧 Interview Tips:**
- 先回答核心概念，再展开细节
- 结合实际项目经验
- 主动对比不同方案的优劣
- 遇到不会的诚实表达，展示学习能力

**🔥 高频考点 Hot Topics:**
- HashMap底层实现（必考）
- MySQL索引失效场景（必考）
- Redis缓存三大问题（必考）
- Spring AOP实现原理（高频）
- 分布式事务解决方案（高频）

---

**📚 持续更新中... Continuously Updated...**

祝你面试成功！Good luck with your interviews! 💪
