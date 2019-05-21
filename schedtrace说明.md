# schedtrace说明

```
SCHED 97151512ms: gomaxprocs=40 idleprocs=40 threads=82 spinningthreads=0 idlethreads=66 runqueue=0 [0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0]
```

`97151512ms`: 从进程启动到当前时间的毫秒数

`gomaxprocs=40`: 允许最大并行执行的线程数，也就是P的数量

`idleprocs=40`: 当前空闲的P数量

`threads=82`: 当前创建的线程(Pthread)的数量

`spinningthreads=0`: 正在启动M的个数,只有请求量正在增长的时候该参数会有值

`idlethreads=66`: 当前空闲的线程数量，也就是空闲M数量

`runqueue=0`: 全局队列中待运行的G的数量，如果很多说明调度很繁忙

`[0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0]`: 各个P里面的待运行的G数量