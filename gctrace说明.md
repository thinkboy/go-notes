本文主要阐述gctrace打出信息的意义，看完官方的文档解释还是有些疑问，描述的偏简单，这里多啰嗦几句。以下所有信息来自于go v1.9.2版本

首先需要打开go的gctrace功能，可通过环境变量`GODEBUG="gctrace=1"`。

启动命令如下:

```
$ GODEBUG gctrace=1 ./main
```

gctrace信息实例如下:

```
gc 84 @1385.961s 2%: 0.20+4138+0.29 ms clock, 6.4+33879/40532/68952+9.3 ms cpu, 14799->15033->7927 MB, 15626 MB goal, 40 P
gc 85 @1480.087s 2%: 0.24+3538+0.50 ms clock, 7.8+26004/34463/60357+16 ms cpu, 15229->15424->7997 MB, 15855 MB goal, 40 P
scvg9: inuse: 11264, idle: 5904, sys: 17168, released: 0, consumed: 17168 (MB)
gc 86 @1578.615s 2%: 0.32+3306+0.39 ms clock, 10+51638/32350/29866+12 ms cpu, 15495->15644->7850 MB, 15994 MB goal, 40 P
scvg10: inuse: 14311, idle: 2857, sys: 17169, released: 0, consumed: 17169 (MB)
gc 87 @1670.874s 2%: 0.79+3487+0.25 ms clock, 25+35794/33965/43450+8.0 ms cpu, 15307->15447->8134 MB, 15700 MB goal, 40 P
gc 88 @1770.877s 2%: 0.78+3089+0.73 ms clock, 25+78896/30001/138+23 ms cpu, 15872->16026->8069 MB, 16269 MB goal, 40 P
scvg11: inuse: 11927, idle: 5244, sys: 17171, released: 0, consumed: 17171 (MB)
gc 89 @1870.706s 2%: 0.91+4589+0.54 ms clock, 29+66817/45106/53413+17 ms cpu, 15622->15752->7973 MB, 16139 MB goal, 40 P
scvg12: inuse: 14763, idle: 2409, sys: 17172, released: 0, consumed: 17172 (MB)
gc 90 @1968.025s 2%: 0.29+3293+0.77 ms clock, 9.5+64199/32319/20392+24 ms cpu, 15533->15679->7852 MB, 15947 MB goal, 40 P
gc 91 @2062.342s 2%: 0.86+3342+0.41 ms clock, 27+25711/32617/56874+13 ms cpu, 15287->15390->7823 MB, 15705 MB goal, 40 P
scvg13: 16 MB released
scvg13: inuse: 12113, idle: 5060, sys: 17174, released: 16, consumed: 17157 (MB)
gc 92 @2158.075s 2%: 0.82+3243+0.47 ms clock, 26+52762/31577/28539+15 ms cpu, 15257->15407->7861 MB, 15646 MB goal, 40 P
scvg14: 2 MB released
scvg14: inuse: 15374, idle: 1799, sys: 17174, released: 4, consumed: 17169 (MB)
gc 93 @2254.810s 2%: 0.73+3210+0.29 ms clock, 23+44683/31320/35113+9.5 ms cpu, 15327->15475->7867 MB, 15722 MB goal, 40 P
gc 94 @2349.917s 2%: 0.83+4463+1.9 ms clock, 26+102056/43573/11557+62 ms cpu, 15349->15486->7937 MB, 15734 MB goal, 40 P
```

go官方解释如下：

```
gctrace: setting gctrace=1 causes the garbage collector to emit a single line to standard
	error at each collection, summarizing the amount of memory collected and the
	length of the pause. Setting gctrace=2 emits the same summary but also
	repeats each collection. The format of this line is subject to change.
	Currently, it is:
		gc # @#s #%: #+#+# ms clock, #+#/#/#+# ms cpu, #->#-># MB, # MB goal, # P
	where the fields are as follows:
		gc #        the GC number, incremented at each GC
		@#s         time in seconds since program start
		#%          percentage of time spent in GC since program start
		#+...+#     wall-clock/CPU times for the phases of the GC
		#->#-># MB  heap size at GC start, at GC end, and live heap
		# MB goal   goal heap size
		# P         number of processors used
	The phases are stop-the-world (STW) sweep termination, concurrent
	mark and scan, and STW mark termination. The CPU times
	for mark/scan are broken down in to assist time (GC performed in
	line with allocation), background GC time, and idle GC time.
	If the line ends with "(forced)", this GC was forced by a
	runtime.GC() call.

	Setting gctrace to any value > 0 also causes the garbage collector
	to emit a summary when memory is released back to the system.
	This process of returning memory to the system is called scavenging.
	The format of this summary is subject to change.
	Currently it is:
		scvg#: # MB released  printed only if non-zero
		scvg#: inuse: # idle: # sys: # released: # consumed: # (MB)
	where the fields are as follows:
		scvg#        the scavenge cycle number, incremented at each scavenge
		inuse: #     MB used or partially used spans
		idle: #      MB spans pending scavenging
		sys: #       MB mapped from the system
		released: #  MB released to the system
		consumed: #  MB allocated from the system

	memprofilerate: setting memprofilerate=X will update the value of runtime.MemProfileRate.
	When set to 0 memory profiling is disabled.  Refer to the description of
	MemProfileRate for the default value.
```

###拿个例子做说明

```
gc 84 @1385.961s 2%: 0.20+4138+0.29 ms clock, 6.4+33879/40532/68952+9.3 ms cpu, 14799->15033->7927 MB, 15626 MB goal, 40 P
```
信息输出的代码位置：`runtime/mgc.go``func gcMarkTermination(nextTriggerRatio float64)`函数里。

`84` 第84次进行gc操作，进程启动时的第一次gc为1

`@1385.961s` 相对进程启动后进行了1385.961秒的时间

`2%` (TODO 还没理解，待补充 )

`0.20+4138+0.29 ms clock` 分别为STW(stop-the-word)的并发标记、扫描、清理三个阶段的耗时，单位为ms。可以看到扫描阶段用4138ms，这个时间阶段是不会STW的，进入该阶段前会先start-the-word。因此真正stop-the-word的只有并发标记跟清理两个阶段，也就是0.20+0.29=0.49ms。

`6.4+33879/40532/68952+9.3 ms cpu` 
