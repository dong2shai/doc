#  <center> CFQ (Complete Fairness Queueing) 完全公平IO调度算法</center>

1.简述   
--
The main aim of CFQ scheduler is to provide a fair allocation of the disk I/O bandwidth for all the processes which requests an I/O operation.    
对于操作IO的请求，CFQ调度器提供了一种公平的磁盘带宽分配方法
***
CFQ maintains the per process queue for the processes which request I/O operation(synchronous requests).   
对于同步IO请求，CFQ调度器会维持一个处理队列来实时处理理同步请求。
***
In case of asynchronous requests, all the requests from all the processes are batched together according to their process's I/O priority.
在异步IO的场景下，所有的异步请求将会根据IO的优先级批量处理。
***

2.CFQ调度器参数   
--
- #####slice_idle空闲时间片   
This specifies how long CFQ should idle for next request on certain cfq queues (for sequential workloads) and service trees (for random workloads) before queue is expired and CFQ selects next queue to dispatch from.  
在调度队列过期或者CFQ选择下一个调度队列调度之前，处理下一个请求时，CFQ将要休息的时间片段。
  ***   
By default slice_idle is a non-zero value. That means by default we idle on queues/service trees.    
slice_idle的默认值是个非0的，这意味这调度器默认有空闲时刻在处理调度队列或者服务树
  ***   
This can be very helpful on highly seeky media like single spindle SATA/SAS disks where we can cut down on overall number of seeks and see improved throughput.   
slice_idle非0对单轴SATA/SAS具有高波动的磁盘十分有帮助的，可以降低寻道总数，提高磁盘吞吐。
  ***   
Setting slice_idle to 0 will remove all the idling on queues/service tree level and one should see an overall improved throughput on faster storage devices like multiple SATA/SAS disks in hardware RAID configuration.  
将slice_idle=0， 调度器的等待时间片将被禁止，对于快速设备如多轴SATA/SAS或者硬件RAID可以提高磁盘的吞吐。
  ***   
The down side is that isolation provided from WRITES also goes down and notion of IO priority becomes weaker. So depending on storage and workload, it might be useful to set slice_idle=0. 
不利的一面是，WRITES提供的隔离也会下降，IO优先级的概念也会变得更弱。因此，根据存储和工作负载的不同，将slice_idle设置为0可能很有用
  ***   
In general I think for SATA/SAS disks and software RAID of SATA/SAS disks keeping slice_idle enabled should be useful.  
一般来说，我认为对于SATA/SAS磁盘和SATA/SAS的软件RAID，保持slice_idle启用应该很有用
  ***   
For any configurations where there are multiple spindles behind single LUN (Host based hardware RAID controller or for storage arrays), setting slice_idle=0 might end up in better throughput and acceptable latencies.   
对于单个LUN（基于主机的硬件RAID控制器或存储阵列）后面有多个磁盘轴的任何配置，设置slice_idle=0可能最终会获得更好的吞吐量和可接受的延迟。
  ***   

- #####back_seek_max反向最大寻道数   
This specifies, given in Kbytes, the maximum "distance" for backward seeking.   
这以千字节为单位指定向后寻道的最大“距离”。
  ***   
The distance is the amount of space from the current head location to the sectors that are backward in terms of distance.   
距离是指从当前磁头位置到距离向后的扇区的空间量。
  ***   
This parameter allows the scheduler to anticipate requests in the "backward" direction and consider them as being the "next" if they are within this
distance from the current head location.
此参数允许调度程序预测“向后”方向的请求，并将它们视为“下一个”请求（如果它们在该范围内）与当前磁头位置的距离。
  ***   
- #####back_seek_penalty
This parameter is used to compute the cost of backward seeking.   
此参数用于计算反向寻道的成本。
  ***   
If the backward distance of request is just 1/back_seek_penalty from a "front" request, then the seeking cost of two requests is considered equivalent.
如果一个向后寻道的请求的距离是一个向前寻道请求的1/back_seek_penalty,则认为这两个请求寻道的代价一致
  ***   
So scheduler will not bias toward one or the other request (otherwise scheduler will bias toward front request). Default value of back_seek_penalty is 2.
所以调度器不会偏向任何一个请求（否则调度器会偏向向前寻道的请求）。back_seek_penalty默认值2
  ***   
- #####fifo_expire_async 异步请求超时时间    
This parameter is used to set the timeout of asynchronous requests. Default value of this is 248ms.   
异步请求的超时时间，默认值248ms。
  ***   
- #####fifo_expire_sync 同步请求超时时间    
This parameter is used to set the timeout of synchronous requests. Default value of this is 124ms. 
同步请求超时时间，默认值124ms
  ***   
In case to favor synchronous requests over asynchronous one, this value should be decreased relative to fifo_expire_async.  
在同步请求大于异步请求的场景下，这个值应该比fifo_expire_async小
  ***   
- #####group_idle 组层次的空闲时间片段   
This parameter forces idling at the CFQ group level instead of CFQ queue level.   
CFQ组级别的调度器的空闲时间片，主要用来代替CFQ队列级别的slice_idle
This was introduced after a bottleneck was observed in higher end storage due to idle on sequential queue and allow dispatch from a single queue.    
这是在使用高端存储时，由于顺序队列闲置造成瓶颈后引入的，并允许从单个队列进行调度。
  ***   
The idea with this parameter is that it can be run with slice_idle=0 and group_idle=8, so that idling does not happen on individual queues in the group but happens overall on the group and thus still keeps the IO controller working.   
使用该参数的目的是，可以在 slice_idle=0 和 group_idle=8 的情况下运行，这样闲置就不会发生在组中的单个队列上，而是发生在组的整体上，从而仍能保持 IO 控制器正常工作。
  ***   
Not idling on individual queues in the group will dispatch requests from multiple queues in the group at the same time and achieve higher throughput on higher end storage. Default value for this parameter is 8ms. 
组内单个队列不空闲，就能同时调度组内多个队列的请求，在高端存储上实现更高的吞吐量, 默认值8ms。
  ***   
- #####low_latency 最低时延开启开关    
This parameter is used to enable/disable the low latency mode of the CFQ scheduler.    
用来开启CFQ调度器的最低时延保证功能。
  ***   
If enabled, CFQ tries to recompute the slice time for each process based on the target_latency set for the system. This favors fairness over throughput.
如果开启，根据target_latency参数，CFQ调度器重新计算每个进程的时间片，这将提高调度的公平性，但吞吐可能会受到影响。
  ***   
Disabling low latency (setting it to 0) ignores target latency, allowing each process in the system to get a full time slice. By default low latency mode is enabled  
禁止low_latency(low_latency=0),target_latency参数将被忽略，每个进程获得完整的时间片。默认是开启的。
  ***   
- #####target_latency 目标时延     
This parameter is used to calculate the time slice for a process if cfq's latency mode is enabled. It will ensure that sync requests have an estimated latency.   
如果参数被开启，调度器会为每个进程重新计算时间片，此参数确保同步请求有一个估计的时延。
  ***   
But if sequential workload is higher(e.g. sequential read), then to meet the latency constraints, throughput may decrease because of less
time for each process to issue I/O request before the cfq queue is switched.  
但是，在高负载任务是顺序操作的场景下，为了满足时延的要求，吞吐量可能会降低。
  ***   
Though this can be overcome by disabling the latency_mode, it may increase the read latency for some applications. 
虽然这种情况可以通过禁止时延模式来解决.对于某些应用，这也可能造成读时延的增加。
  ***   
This parameter allows for changing target_latency through the sysfs interface which can provide the balanced throughput and read latency. Default value for target_latency is 300ms.   
这个参数可以通过sys文件系统被修改来平衡吞吐量和读时延的问题。默认值300ms
  ***   
- #####slice_async异步时间片    
This parameter is same as of slice_sync but for asynchronous queue. The default value is 40ms.
异步队列的时间片，默认40ms
  ***   
- #####slice_async_rq时间片内异步请求数量    
This parameter is used to limit the dispatching of asynchronous request to device request queue in queue's slice time.  
这个参数用于限制在时间片内发送到设备队列的异步请求。
  ***   
The maximum number of request that are allowed to be dispatched also depends upon the io priority. Default value for this is 2.
允许分派的最大请求数也取决于IO的优先级，默认2
  ***   
- #####slice_sync同步时间片   
When a queue is selected for execution, the queues IO requests are only executed for a certain amount of time(time_slice) before switching to another
queue.  
当一个队列被选中执行的时候，这个队列中的IO请求只能按照一定时间被执行。
  ***   
This parameter is used to calculate the time slice of synchronous queue.
这个参数就是用来计算同步请求被执行时间的。
  ***   
time_slice is computed using the below equation:
计算公式如下：  
  ***   
```
time_slice = slice_sync + (slice_sync/5 * (4 - prio)).     
```   
  To increase the time_slice of synchronous queue, increase the value of slice_sync.Default value is 100ms.  
为了增加同步请求的时间片，可以调整这个参数，默认值100ms
 ***   
- #####quantum数量  
This specifies the number of request dispatched to the device queue.  
这个参数为发送到设备队列的请求数
  ***   
In a queue's time slice, a request will not be dispatched if the number of request in the device exceeds this parameter. This parameter is used for synchronous request. 
在一个队列的调度时间片中，如果发送到大于这个参数，之后的请求将不会被发送。这个参数用于同步请求。
  ***   
In case of storage with several disk, this setting can limit the parallel processing of request.   
如果一个存储设备拥有多个磁盘，这个设置将会限制请求处理的并发度。
  ***   
Therefore, increasing the value can improve the performance although this can cause the latency of some I/O to increase due to more number of requests.   
因此，增加该值可以提高性能，但由于请求数量增多，可能会导致某些 I/O 的延迟增加。
  ***   

3.CFQ Group scheduling
---   
CFQ supports blkio cgroup and has "blkio." prefixed files in each blkio cgroup directory. It is weight-based and there are four knobs for configuration - weight[_device] and leaf_weight[_device].   
CFQ支持blkio的cgroup功能，同时会在blikio cgroup的目录下创建前缀"blkio."的文件。CFQ是基于权重的调度器，存在四种权重指标可配置。 
  ***   
Internal cgroup nodes (the ones with children) can also have tasks in them, so the former two configure how much proportion the cgroup as a whole is entitled to at its parent's level while the latter two configure how much proportion the tasks in the cgroup have compared to its direct children.   
cgroup 内部节点（有子节点的节点）中也可以有任务，因此前两者决定了整个 cgroup 在其父节点级别上所占的比例，而后两者决定了 cgroup 中的任务与其直接子节点相比所占的比例。
  ***   
Another way to think about it is assuming that each internal node has an implicit leaf child node which hosts all the tasks whose weight is configured by leaf_weight[_device].   
另一种思考方式是，假设每个内部节点都有一个隐含的叶子节点，它承载着所有任务，这些任务的权重由 leaf_weight[_device]配置。
  ***   
Let's assume a blkio hierarchy composed of five cgroups - root, A, B, AA and AB - with the following weights where the names represent the hierarchy.  
假设 blkio 层次结构由根、A、B、AA 和 AB 五个 c 组组成，其权重如下，其中名称代表层次结构。
  ***   
```

        weight leaf_weight    
 root :  125    125   
 A    :  500    750   
 B    :  250    500  
 AA   :  500    500  
 AB   : 1000    500  
```
root never has a parent making its weight is meaningless. For backward compatibility, weight is always kept in sync with leaf_weight.   
如果根节点没有父节点，那么它的权重就没有意义。为了向后兼容，权重始终与 leaf_weight 保持同步。
  ***   
B, AAand AB have no child and thus its tasks have no children cgroup to compete with. 

  ***   
They always get 100% of what the cgroup won at the parent level. Considering only the weights which matter, the hierarchy looks like the following.
  ***   
```
          root
       /    |   \
      A     B    leaf
     500   250   125
   /  |  \
  AA  AB  leaf
 500 1000 750
```

If all cgroups have active IOs and competing with each other, disk time will be distributed like the following.
  ***   

Distribution below root. The total active weight at this level is A:500 + B:250 + C:125 = 875.
  ***   
```
 root-leaf :   125 /  875      =~ 14%
 A         :   500 /  875      =~ 57%
 B(-leaf)  :   250 /  875      =~ 28%
```
A has children and further distributes its 57% among the children and the implicit leaf node. The total active weight at this level is 
  ***   
```
AA:500 + AB:1000 + A-leaf:750 = 2250.

 A-leaf    : ( 750 / 2250) * A =~ 19%
 AA(-leaf) : ( 500 / 2250) * A =~ 12%
 AB(-leaf) : (1000 / 2250) * A =~ 25%
```
4.CFQ IOPS Mode for group scheduling  
  ***   
Basic CFQ design is to provide priority based time slices. Higher priority process gets bigger time slice and lower priority process gets smaller time
slice.    
  ***   
Measuring time becomes harder if storage is fast and supports NCQ and it would be better to dispatch multiple requests from multiple cfq queues in request queue at a time. In such scenario, it is not possible to measure time consumed by single queue accurately.  
  ***   

What is possible though is to measure number of requests dispatched from a single queue and also allow dispatch from multiple cfq queue at the same time.
  ***   
This effectively becomes the fairness in terms of IOPS (IO operations per second).
  ***   

If one sets slice_idle=0 and if storage supports NCQ, CFQ internally switches to IOPS mode and starts providing fairness in terms of number of requests
dispatched. Note that this mode switching takes effect only for group scheduling. For non-cgroup users nothing should change.
  ***   

5.CFQ IO scheduler Idling Theory  
  ***   
Idling on a queue is primarily about waiting for the next request to come on same queue after completion of a request.   
  ***   
In this process CFQ will not dispatch requests from other cfq queues even if requests are pending there.
  ***   

The rationale behind idling is that it can cut down on number of seeks on rotational media.   
  ***   
For example, if a process is doing dependent sequential reads (next read will come on only after completion of previous one), then not dispatching request from other queue should help as we did not move the disk head and kept on dispatching sequential IO from one queue.
  ***   
CFQ has following service trees and various queues are put on these trees.  
```
	sync-idle	sync-noidle	async
```
All cfq queues doing synchronous sequential IO go on to sync-idle tree. On this tree we idle on each queue individually.
  ***   

All synchronous non-sequential queues go on sync-noidle tree. Also any synchronous write request which is not marked with REQ_IDLE goes on this service tree. 
  ***   

On this tree we do not idle on individual queues instead idle on the whole group of queues or the tree. So if there are 4 queues waiting for IO to dispatch we will idle only once last queue has dispatched the IO and there is no more IO on this service tree.
  ***   

All async writes go on async service tree. There is no idling on async queues.
  ***   

CFQ has some optimizations for SSDs and if it detects a non-rotational media which can support higher queue depth (multiple requests at in flight at a time), then it cuts down on idling of individual queues and all the queues move to sync-noidle tree and only tree idle remains.   
  ***   

This tree idling provides isolation with buffered write queues on async tree.
  ***   

6.FAQ
  ***   

Q1. Why to idle at all on queues not marked with REQ_IDLE.

A1. We only do tree idle (all queues on sync-noidle tree) on queues not marked with REQ_IDLE. 
  ***   
This helps in providing isolation with all the sync-idle queues. 
  ***   
Otherwise in presence of many sequential readers, other synchronous IO might not get fair share of disk.
  ***   
For example, if there are 10 sequential readers doing IO and they get 100ms each. 
  ***   
If a !REQ_IDLE request comes in, it will be scheduled roughly after 1 second. If after completion of !REQ_IDLE request we do not idle, and after a couple of milli seconds a another !REQ_IDLE request comes in, again it will be scheduled after 1second.
  ***   
Repeat it and notice how a workload can lose its disk share and suffer due to multiple sequential readers.
  ***   
fsync can generate dependent IO where bunch of data is written in the context of fsync, and later some journaling data is written.   
  ***   
Journaling data comes in only after fsync has finished its IO (atleast for ext4 that seemed to be the case). 
  ***   
Now if one decides not to idle on fsync thread due to !REQ_IDLE, then next journaling write will not get scheduled for another second. 
  ***   
A process doing small fsync, will suffer badly in presence of multiple sequential readers.
  ***   
Hence doing tree idling on threads using !REQ_IDLE flag on requests provides isolation from multiple sequential readers and at the same time we do not idle on individual threads.
  ***        


Q2. When to specify REQ_IDLE  
  ***        
A2. I would think whenever one is doing synchronous write and expecting more writes to be dispatched from same context soon, should be able to specify REQ_IDLE on writes and that probably should work well for most of the cases.
  ***        
