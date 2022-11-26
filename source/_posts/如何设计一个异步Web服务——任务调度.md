---
title: '如何设计一个异步Web服务——任务调度'
date: 2014-12-01 17:03:00
cover: false
---
接上一篇《[如何设计一个异步Web服务&mdash;&mdash;接口部分](http://www.cnblogs.com/silenttiger/p/4076333.html "如何设计一个异步Web服务&mdash;&mdash;接口部分")》

Application已经将任务信息发到了Service服务器中，接下来，Service服务器改如何对自身的资源进行合理分配以满足Application对功能、性能、用户体验等各方面的需求呢？

可以从如下几个方向入手去考虑：

1. 当task提交到Service后，我们希望Service能够尽可能快的完成这个task并返回结果。
2. 当大量task同时提交Service后，我们希望Service不要因为需要同时处理大量task导致性能下降，甚至失去响应。
3. 当有多个task被提交到Service时，我们不希望某一个task占用了所有计算资源，导致其他task长时间处于等待状态。

根据上面的要求，我们会产生如下的设计要求：

1. 根据第一点要求，为了能够尽快完成一个task，我们可以使用多线程（或多进程）技术，将一个task拆分为多个子task然后并行处理，充分利用多核CPU的计算资源。
2. 根据第二点要求，我们需要为Service实现一个任务队列，以免大量并发请求导致Service计算资源被耗尽。同时，大量的并发也会导致CPU为进行资源调度浪费许多计算资源。
3. 根据第三点要求，虽然Service会使用任务队列对任务进行排队处理，但我们仍然希望有少量的task是并行进行的。
4. 另外，一个task被拆分为许多子task后，如果为每个子task创建一个单独的线程去处理，会导致CPU将大量时间消耗在线程的创建、销毁过程中。所以，应该使用线程池（或进程池）技术。

下面，我们就根据上述的这些要求开始设计。

首先，我们需要一个http服务器作为接收Application请求的接口。然后，我们创建一个QueenAnt（蚁后）类来负责任务和资源的调度，同时还需要若干个WorkerAnt（工蚁）类来处理各个具体的task。

![](011656008114793.png)

注意，这里的QueenAnt类是静态的，或者也可以用单例模式创建。前面提到我们需要使用线程池（或进程池）技术，所以，在QueenAnt类被实例化以后首先就需要把这个线程池创建出来，并创建若干线程放入这个池中。其中，每个线程都会实例化一个WorkerAnt类来等待QueenAnt发过来的task。这个地方还有一个问题，那就是我们的线程池中到底创建几个线程最优？这个问题我留到后面说明。

![](011656017331878.png)

![](011656025453762.png)

当这些准备好了以后，Service就可以等待Application的请求了。

当Application向Service发出addTask的请求时，http服务器会将这个请求通知给QueenAnt，并返回QueenAnt返回的taskId。

![](011656035145833.png)

QueenAnt在收到task请求后，除了返回taskId，还需要对这个task的相关信息进行初始化，比如设置task的状态信息，将task添加到任务队列等等。

![](011656043896933.png)

![](011656054985962.png)

等这些结束以后，QueenAnt就开始针对已经收到的task进行任务调度和资源分配了。我们定义一个allocateResource方法来处理相关的逻辑。该方法将会指定threadPool中的哪个具体线程会来处理这个task。这之后，我们就可以把task相关的数据发给这个指定的thread进行处理了。而当有task完成时，处理该task的线程中的WorkerAnt就会发送相关信息给QueenAnt，调用QueenAnt的taskEndCallback方法，让QueenAnt重新分配资源。

![](011656063737061.png)

![](011656076861859.png)

当WorkerAnt完成某一个task之后，他需要将这个task的相关信息返回给QueenAnt。同时标记自己为空闲状态，以便QueenAnt再进行资源分配。

QueenAnt在收到WorkerAnt关于task完成的消息后，他也需要更新于这个task的相关状态信息，并在此根据threadPool和taskQueue的具体情况重新进行资源分配。

![](011656088584401.png)

![](011656110301772.png)

到这里，我们就通过上图描述的逻辑，满足了设计要求中的第二和第四点要求。那第一和第三点要求呢，就得通过allocateResource这个方法去实现了。

下面我们详细讲一下allocateResource这个方法的内部逻辑。

这里先声明一下后文的描述方法，我们把Application发过来的一个任务叫做"task"，而把由这个任务拆分出来的许许多多的小任务叫做"子task"。

可能有人会产生疑惑，根据设计要求中的第一点，我们应该把task拆分为子task。可上面的设计中，我们放入taskQueue的却是Application传过来的task，是不是差一个拆分的步骤呢？

其实并不是这样，这样的设计是因为开头的考虑方向中的第三点和设计要求中第三点，都要求一个task不可以占用所有的计算资源。这样说可能不太好理解，我们来举个例子：

首先，Application向Service提交了task01，该task共20个子task，需要Service满负荷运行5分钟才能完成。

到第3分钟的时候，Application又向Service提交了task02，该task共4个子task，需要Service满负荷运行1分钟即可完成。

我们来分析一下这个场景。如果我们在将task01加入taskQueue之前，就将其拆分为许多的子task。并把threadPool中的资源依次分给这些子task。那么到第3分钟加入task02的各个子task的时候，由于task01的子task没有完成，task02只好处于等待状态。而且需要等task01的几乎所有子task都完成以后，才能进入处理中的状态，这一等就是10分钟。这显然违背了我们考虑方向中的第三点和设计要求中第三点。

那么，怎样设计这个allocateResource的逻辑才能既满足设计要求中的第一点，又能满足第三点了？我的思路是这样的。

首先，我们给task加上两个属性threadRequirement和runningThread。threadRequirement表示，为了完成这个task，如果给其每个子task分配一个线程，那么一共需要多少个线程，随着子task的完成，这个数值会越来越小，最后变为0即表示这个task已经全部完成。runningThread表示，当前有几个线程正在处理这个task的子task。

然后，allocateResource这个方法有两个地方会调用，一是当Service收到新的task请求的时候。二是当某个子task完成，QueenAnt中的taskEndCallback被调用的时候。

allocateResource在给task分配资源的时候，应遵守以下几个准则：

1. taskQueue中处于等待状态的task应该尽可能的少。
2. 同时进行的task的数量不得超过threadPool中线程的总数。
3. 每个task都应该至少有一个线程在处理其子task。
4. 在满足以上条件的情况下，threadRequirement最小的task分配到所有剩余的空闲线程资源。

这样说可能有些抽象。我们还是来举个上面那个例子，假设threadPool中共4个线程，task01的threadRequirement为20，task02的threadRequirement为2。过程如下：

1. QueenAnt收到task01的请求后开始调用allocateResource方法。且当前threadPool中有空闲的线程资源。
2. 根据准则1，我们看到taskQueue中当前有一个task，就是task01。
3. 当前没有正在进行的task数量没有达到threadPool中的线程总数4，满足准则2。于是将task01从taskQueue中取出准备为其分配线程资源。
4. 为满足准则3，我们将threadPool中的一个空闲线程thread01分配给task01的一个子task：childTask01。
5. 为满足准则4，我们将threadPool中剩下的所有空闲线程都分配给task01，这样以来，task01的4个子task，childTask01~childTask04同时接受处理。
6. 一分钟后，childTask01~childTask04相继结束，taskEndCallback被触发，allocateResource再次被调用。重复上面的步骤3和步骤4。childTask05~childTask08开始接受处理。
7. 又一分钟后，childTask05~childTask08相继结束，taskEndCallback被触发，allocateResource再次被调用。重复上面的步骤3和步骤4。childTask09~childTask12开始接受处理。
8. 一秒钟后，QueenAnt收到task02的请求后开始调用allocateResource方法。但当前threadPool中没有空闲的线程资源，所以方法退出，task02停留在taskQueue中等待。
9. 大概59秒以后，task01的一个childTask09结束，taskEndCallback被触发，allocateResource再次被调用。
10. 根据准则2，当前正在进行的task只有task01，远没达到threadPool中线程的总数4，所以我们可以将task02从taskQueue中取出准备为其分配线程资源。
11. 根据准则3，每个task至少分配一个线程资源，而当前task02的runningThread为0。所以我们把刚才处理task01中childTask09的线程thread01分配给task02。就这样，task02也开始运行起来了。
12. 接下来，threadPool中的thread02和thread03也相继完成了task01的childTask10和childTask11，并触发taskEndCallback调用allocateResource。
13. 此时，我们根据准则4，会把刚刚释放出来的thread02和thread03两个线程资源都分配给task02。这时，task01只有一个线程资源thread04在处理，而其他三个线程资源都被用来处理task02了。
14. 再接下来，thread04处理完task01的childTask12后，根据准则3又会被分配给task01处理childTask13。

大概的逻辑就是上面这样了。步骤看起来虽然略显复杂，但其实只有掌握了前面说的4个准则，allocateResource的逻辑还是很好实现：

![](011656139051527.png)

至此，关于Service任务调度和资源分配的设计也结束了。

下面，我们来说一下前面遗留的一个问题：线程池中到底创建几个线程最优？

为什么最后要特别来谈谈这个问题，是因为市面上有一种叫做超线程的CPU虚拟化的技术。比如Intel公司的酷睿i3系列CPU，明明是两个物理核心，在Windows的任务管理中，或在Linux系统的top命令下，显示的却是4个核心，因为CPU在硬件层面将两个物理核心模拟为4个逻辑核心了。根据我们上面的设计，自然是希望threadPool中的线程数量越多越好，可是也不能太多。因为多个线程同时争用一个CPU核心的资源是没有必要的。所以，如果是4核的CPU，我们一般会起4个线程放入threadPool。可是在这种使用了超线程技术的CPU平台上，如果你把线程数目配置为与CPU逻辑核心数目一致却是没有必要的。我在i3平台上实测数据如下：

|||||||
<colgroup><col /><col /><col /><col /><col /><col /></colgroup>|---|---|---|---|---|---|
|线程数|总耗时(s)|CPU 0 使用率|CPU 1 使用率|CPU 2 使用率|CPU 3 使用率|
|1|22.4|1|0|98|0|
|2|12.6|2|98|0|97|
|3|11.2|78|64|97|4|
|4|10.5|98|99|100|98|

我想上面的数据已经很好地说明了问题，虽然是4个逻辑核心，虽然你可以让4个线程同时运行，但其实在CPU物理层面，同时运行的指令最多就两个。也就是4个线程中每两个线程去争用一个物理核心的运算资源。

其结果就是，性能上的微小进步却带来了CPU使用率的大幅飙升，反而使得用来作为接口的httpServer响应时间变长。
