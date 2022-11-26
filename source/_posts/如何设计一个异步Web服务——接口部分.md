---
title: '如何设计一个异步Web服务——接口部分'
date: 2014-11-05 14:40:00
cover: false
---
需求比较简单，提供一个异步Web服务供使用者调用。比如说，某应用程序需要批量地给图片加lomo效果。由于加lomo效果这个操作非常消耗CPU资源，所以我们需要把这个加lomo效果的程序逻辑放到一台单独的服务器上去运行，以免影响应用本身所在服务器的性能。

这篇先讲讲服务的接口部分，侧重于理清应用和服务之间的调用关系，有时间的话，后面再写一篇关于服务内部任务分派资源调度的随笔。

根据这个需求，我们可以很快设计出一套流程：

![](051427057206299.png)

Application通过向service的addTask接口post任务相关的信息创建一个任务，同时，service将此任务的任务id，即taskId返回给Application，以便Application接下来可以用这个taskId通过getStatus接口查询此任务的进行状态。

问题一：

Application作为一个独立的应用，他肯定需要有自己的任务管理逻辑。也就是说，Application在向Service发出创建任务的请求前，肯定需要先往自己的DB中写入一条对应这个任务的相关数据，以便后续跟踪每个任务的状态和数据。那么application在收到response时，如何将这个response的taskId和自己之前写入DB的那条数据对应起来呢？

如果是上面的那种接口设计，应该是无法实现这个要求的。所以我们需要给addTask接口加一个用来标记任务唯一性的参数customId。如下图：

![](051427197527645.png)

问题二：

根据上面的设计，Application在创建task之后，会不断调用Service的getStatus接口，来获取该任务的最新状态，比如任务是在等待中、进行时还是已完成。但是，这显然不是一个非常高效的方式。如果任务进行的时间比较长，Application则会在这段时间内多次调用getStatus接口以及时获取任务状态信息。这样做不但加重了Application的负担，更加重了Service的负担。所以，我们可以把这个地方的依赖关系反过来，让Service主动将任务的状态信息通知给Application。相应的，Service和Application的接口也要做相应变化：

![](051427334244036.png)

如上图，我们将原来由Application调用Service的getStatus接口的过程，改为了Service调用Application的setStatus接口。这样就实现了，一旦task状态发生变化，Service都能实时通知Application，从而减少了两个服务间无用的请求。

问题三：

假如，Application调用Service的addTask接口后，服务器因为种种原因无法响应外部请求了。那么接下来，当task结束以后，Service就没法将这个重要的状态信息反馈给Application了。而且，由于Service不可能永远不停地尝试去调用Application的setStatus接口，直到成功。那么，当Application重新启动以后如何获取之前那个任务的状态就成了一个问题。所以，基于这样的原因，之前的getStatus接口其实还是有存在的必要的。

![](051427528617463.png)

有了getStatus这个接口以后，如若Application在调用addTask以后出现当机等现象而错过了Service推送的task完成的消息。那么Application在重新启动以后，还可以自己通过Service的getStatus接口获取这个任务的最新状态。

至此，API部分设计完毕。

---

关于接口的部分，目前我所能想到的问题大概就这么多，不知道有没有什么遗漏或不妥。希望大家不吝赐教，在下感激万分。

下篇：《[如何设计一个异步Web服务&mdash;&mdash;任务调度](http://www.cnblogs.com/silenttiger/p/4135461.html "如何设计一个异步Web服务&mdash;&mdash;任务调度")》
