# 进程管理和调度 ——《深入Linux内核架构》学习笔记

# CFS调度器

## 原理

### vruntime

* 就绪队列使用红黑树管理，总是选择树中最左边的节点投入运行

* 树节点排列的依据是调度实体(schedule entity)中的 **vruntime** 成员，值越小越靠左

* 进程在得到运行时，其调度实体中的 **vruntime** 成员会增长，但增长幅度受**权重**影响，而权重则受**优先级**影响，vruntime的增长计算公式为：
  $$
  vruntime\ +=\ delta\_exec * \frac{NICE\_0\_LOAD}{weight}
  $$
  $$
  vruntime\ +=\ (delta\_exec * \frac{NICE\_0\_LOAD * 2^{32}}{weight}) / 2^{32}
  $$

  $$
  vruntime\ +=\ (delta\_exec * NICE\_0\_LOAD * \frac{2^{32}}{weight})/2^{32}
  $$

  $$
  vruntime\ +=\ (delta\_exec * NICE\_0\_LOAD * inv\_weight)/2^{32}
  $$

  通过以上公式转换，出发运算就变成了乘法和移位运算，inv_weight = 2^32 /  weight ， 因为weight已经是常数表，故inv_weight也是一个常数表。进程的load_weight结构体中同时存储了  weight和inv_weight。

  *进程在运行时，内核会不定期调用 **update_curr** 函数来更新vruntime值，所以 delta_exec 就是两次调用 **update_curr** 函数之间积累的真实运行时间差，**NICE_0_LOAD**代表的是nice值为0的权重(1024)，而**load_weight**则代表当前运行进程的权重值。由上述公式可知，当进程的nice值比nice0小时，因为其权重会比1024大，所以vruntime增长的值比实际运行的时间差要小。那么它在红黑树中向右移动的速度就会比nice值大的进程要慢，下次投入运行的机会就会比较大。*

* CFS调度器使用的优先级是**100~139**，这是内核内部使用的优先级，值越小，优先级越高，权重也就越高。对应的nice值为**-20~19**。

* 如果进程进入睡眠，则vruntime保持不变。那么睡眠进程醒来后，在红黑树中的位置会更靠左，其投入运行的机会也会更大。
* 

### 延迟跟踪

* **sysctl_sched_latency**: 保证每个可运行的进程都应该至少运行一次的时间间隔。默认为20ms。它可以通过**/proc/sys/kernel/sched_latency_ns**设置

* **sched_nr_latency**: 控制一个延迟周期内处理的最大活动进程数目，如果活动进程的数目超出该上限，则延迟周期也成比例的线性扩展。该值可以通过**sysctl_sched_min_granularity**间接控制，后者可通过**/proc/sys/kernel/sched_min_granularity**设置，默认是4ms。

* **sysctl_sched_min_granularity**: 进程运行的最少时间粒度

* 当队列里的可运行进程数目超出sched_nr_latency时，延迟周期相信线性扩展的公式为：
  $$
  sysctl\_sched\_latency = sysctl\_sched\_latency * \frac{nr\_running}{sched\_nr\_latency}
  $$
  实际上这就是 **__sched_period()**函数的实现

* 每次sysctl_sched_latency/sysctl_sched_min_granularity之一改变时，都会重新计算sched_nr_latency。

* **将延迟周期按进程权重分配到运行队列里的各个进程，这实际上就是队列里每个进程能占用CPU的时间（内核使用“理想运行时间”来描述），因为一旦某个进程占用CPU超过了这个时间，这就会导致在延迟周期内，有某一个进程得不到CPU。**在*check_preempt_tick()*函数中实际就这样对比的，将进程真实运行的时间与上述分配到的时间做对比，若发现真实运行的时间大于该时间，则请求重新调度。