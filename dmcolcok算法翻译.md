# <center>  mColcok算法中文  </center>

1. 简述     

mColcok算法是一种时间标签调度算法,客户端(虚拟机)根据业务类型被分配了IO资源，隶属该客户端(虚拟机)的每个请求会被分配 R(预留资源时间标签),L(资源限制时间标签),P(权重时间标签)。
mColcok算法依据标签的大小完成调度

2.mColcok 算法      

  mClock uses two main ideas: multiple real-time clocks and dynamic clock selection.   
  mColcok算法采用多实时时钟和动态时钟选择器完成时间标签的分配和重置。       
***
Each VM IO request is assigned three tags, one for each clock: a reservation tag R, a limit tag L, and a proportional share tag P for weight-based allocation.   
基于每个时钟，每个虚拟机IO请求都会被分配三个时间标签：预留资源时间标签,资源限制时间标签,权重时间标签。    
***
Different clocks are used to keep track of each of the three controls, and tags based on one of the clocks are dynamically chosen to do the constraint-based or weight-based scheduling.  
不同的时钟确保不同的约束,每个时钟标签都是被条件约束或者权重调度器动态选择的。  
***

  The scheduler has three main components: (i) Tag Assignment (ii) Tag Adjustment and (iii) Request Scheduling.   
  调度器主要有一下三个组件: 标签分配,标签调整,请求调度。  
***
We will explain each of these in more detail below.


* Tag Assignment(标签分配):   
  ***  
  This routine assigns R, L and P tags to a request r from VM vi arriving at time t.   
  虚拟机中一个请求到达，L,R,P标签就会被分配。  
  ***
All the tags are assigned using the same underlying principle, which we illustrate here using the reservation tag.    
所有类型的标签分配遵循的原则是一致的，因此将使用预留资源时间标签分配说明。   
  ***
The R tag assigned to this request is the higher of the arrival time or the previous R tag + 1/ri . That is:     
一个请求分配的预留资源时间标签的取值是当前时间和前标签+1/ri中的最大值，具体的公式如下：  
  ***
This gives us two key properties: first, the R tags of a continuously backlogged VM are spaced 1/ri apart.     
公式包含两个关键参数：第一个是标签的步进参数1/ri    
  ***
In an interval of length T,a backlogged VM will have about T*ri requests with R tag values in that interval.    
在一个长度为T的时间间隔离，一个虚拟机将只能收到 T*ri的请求(预留资源时间标签决定了单位时间内虚拟机能收到的请求数，即: 单位时间/步进单位)。    
  ***
Second, if the current time is larger than this value due to vi becoming active after a period of inactivity,the request is assigned an R tag equal to the current time.    
第二个是当前时间，如果当前时间大于标签的值，由于虚拟机之前处于非活动期间，这个标签的值将设置为当前时间     
  ***
Thus idle VMs do not gain any idle credit for future service.    
因此空闲的虚拟机不会被调度    
  ***

  Similarly, the L tag is set to the maximum of the current time and (Lr−1i+ 1/li). The L tags of a backlogged VM are spaced out by 1/li .     
  同样地，资源限制的时间标签也是同样的分配规则.标签的增长步长是1/li     
  ***
Hence, if the L tag of the first pending request of a VM is less than the current time, it has received less than its upper limit at this time.   
因此，如果一个请求的资源限制的时间标签小于当前的时间，表示当前的资源占用低于请求的上限。   
  ***
A limit tag higher than the current time would indicate that the VM has received its limit and should not be scheduled.   
一个限制资源时间标签的值大于当前时间表明这个虚拟机的资源使用率已经超过限制，这个请求将不会被调度。  
  ***

  The proportional share tag Pr is also the larger of the arrival time of the request and (P i r−1 + 1/w i ) and subsequent backlogged requests are spaced by 1/wi .   
  权重时间标签的分配也是一样，标签的增长步长是1/wi  

* Tag Adjustment(标签调整):   
  ***
  Tag adjustment is used to calibrate the proportional share tags against real time.This is required whenever an idle VM becomes active again.  
  标签的调整主要是为了及时地校准权重时间标签的值, 每当非活动的虚拟机被激活是执行调整活动。  
  ***
In virtual time based schedulers [10, 15] this synchronization is done using global virtual time.  
在一个基于虚拟时间调度器中[10, 15]，调整权重时间标签是通过更新全局虚拟时间完成的。   
  ***
The initial P tag value of a freshly active VM is set to the current time, but the spacing of P tags after that is determined by the relative weights of the VMs.  
虚拟机被创建时，它的权重时间标签被初始化为当前时间，但之后的权重时间标签间隔的值由权重时间标签的步长决定。  
  ***
After the VM has been active for some time, the P tag values become unrelated to real time.   
在虚拟机处于活动的一段时间内，权重时间标签的更新和系统时间无关(主要依赖权重步长进行更新).  
  ***
This can lead to starvation when a new VM becomes active, since the existing P tags are unrelated to the P tag of the new VM.  
这样做可能会导致一些虚拟机被饿死，一个新的虚拟机被创建权重时间标签设置为当前时间，这个值可能一直大于已存在的权重时间标签。   
  ***
Hence existing P tags are adjusted so that the smallest P tag matches the time of arrival of the new VM, while maintaining their relative spacing.   
据此，需要对已存在的权重时间标签进行调整来保持最小的权重时间标签和新的权重时间标签的关系。  
  ***
In the implementation, when a VM is activated,we assign it an offset equal to the difference between the effective value of the smallest existing P tag
and the current time. During scheduling, the offset is added to the P tag to obtain the effective P tag value.  
一个虚拟机被激活时，分配一个偏移量,偏移量等于最小的权重时间标签和当前时间的差值，在调度期间，这个偏移量被加给新分配的权重时间标签。  
  ***
The relative ordering of existing P tags is not altered by this transformation; however, it ensures that the newly activated VMs compete fairly with existing VMs.  
此操作不会影响权重时间标签的顺序，同时确保了权重之间的公平性。  

- Request Scheduling(请求调度):  
  ***
  mClock needs to check three different tags to make its scheduling decision instead of a single tag in previous algorithms.    
  不同于传统单个标签算法，mColcok算法通过三个不同的标签决定调度策略   
  ***
As noted earlier,the scheduler alternates between constraint-based and weight-based phases.  
正如之前表述，mColcok算法存在两个调度过程：基于约束条件和基于权重   
  ***
First, the scheduler checks if there are any eligible VMs with R tags no more than the current time.    
首先，调度器检查满足不超过当前时间的所有预留资源时间标签    
  ***
If so, the request with smallest R tag is dispatched for service. This is defined as the constraint-based phase. This phase ends (and the weight-based
phase begins) at a scheduling instant when all the R tags exceed the current time.
如果存在,最小标签的请求被调度，这个过程被称为基于约束条件调度阶段。这个阶段处理完所有小于当前时间的资源预留时间标签。(此阶段结束进入基于权重调度阶段)
  ***
  During a weight-based phase, all VMs have received their reservations guaranteed up to the current time. 
  在基于权重调度节点， 所有的虚拟机的预留资源已经得到保证  
  ***
The scheduler therefore allocates server capacity to achieve proportional service. It chooses the request with smallest P tag, but only from VMs which have not reached
their limit (whose L tag is smaller than the current time).  
 因此，调度器开始为满足权重的服务分配资源。调度器选择值最小但是最大资源限制时间标签未超的权重时间标签进行调度。  
  ***
Whenever a request from VM vi is scheduled in a weight-based phase, the R tags of the outstanding requests of vi are decreased by 1/ri. 
在基于权重调度阶段一个请求的预留资源标签都是按照步长增长。
  ***
This maintains the condition that R tags are always spaced apart by 1/ri,so that reserved service is not affected by the service provided in the weight-based phase.   
这保持了R标签总是间隔1/ri的条件，使得保留的服务不受基于权重阶段提供的服务的影响。


3.存储系统问题
            
- Burst Handling(突发处理)  
  ***
Storage workloads are known to be bursty, and requests from the same VM often have a high spatial locality.   
存储业务很可能出现突发访问，一个虚拟机的请求经常需要很大的存储空间。
  ***
We help bursty workloads that were idle to gain a limited preference in scheduling when the system next has spare capacity.   
我们解决突发业务问题是，调度器会在系统下次分配空间时给予一定的优先级给突发业务。
  ***
This is similar to some of the ideas proposed in BVT [14] and SMART [28]. However, we do it in a manner so that reservations are not impacted.   
mColcok算法采用相同的思想，但是不影响预留资源的分配。
  ***
To accomplish this, we allow VMs to gain idle credits. In particular, when an idle VM becomes active, we compare the previous P tag with current time t and allow it to lag behind t by a bounded amount based on a VM-specific burst parameter.   

  ***
Instead of setting the P tag to the current time, we set it equal to t − σi ∗ (1/wi). Hence the actual assignment looks like:   
       P i r = max{P i r−1 + 1/wi , t − σi /wi }
  ***
The parameter σi can be specified per VM and determines the maximum amount of credit that can be gained by becoming idle.   
  ***
Note that adjusting only the P tag has the nice property that it does not affect the reservations of other VMs; however if there is spare capacity in the system, it will be preferentially given to the VM that was idle.   
  ***
This is because the R and L tags have strict priority over the P tags, so adjusting P tags cannot affect the constraint-based phase of the scheduler.





