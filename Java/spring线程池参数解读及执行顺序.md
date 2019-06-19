## ThreadPoolExcutor主要配置参数
1. corePoolSize: 核心线程数
2. maxPoolSize：线程池内最大线程数
3. waitingQueue：等待队列
4. RejectedHandler：拒绝策略
5. keepAlive:空闲线程存活时间
6. TimeUnit：时间单位

## 执行步骤
1. 有请求过来，线程数 < corePoolSize,创建新线程，处理任务。
2. 当线程数>corePoolSize，waitingQueue没满，新任务放入等待队列，等有空闲的线程时，分配给queue中任务。
3. 当线程数>corePoolSize，waitingQueue满了，但线程数<maxPoolSize,创建新线程，处理任务。
4. 当线程数==corePoolSize，waitingQueue满了，直接拒绝任务。

## 线程池大小配置
依据任务特点配置，采取不同策略。原则：
1. 以不让CPU空闲为前提。
2. 以减少线程切换为前提，因为线程的切换性能损耗大。

#### CPU密集型CPU密集型：任务以CPU运算为主。 
	策略：corePoolSize=CPU数+1。
#### IO密集型：任务总时间有大量时间耗在io上。
	策略：corePoolSize=CPU数*N + 1。
#### 混合型：以上两种混合在一起。
    把任务分拆，多阶段，用不同的线程池，适配前两种策略。
	
#### 线程池线程数计算公式：
	核心线程数=(IO等待时间/cpu执行时间+1)*CPU核数

**可以参考：PoolSizeCalculator**




