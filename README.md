介绍
====

这个一个演示项目，目的是演示如何组织Go语言项目结构。

刚开始用Go语言开发项目的时候，大家应该都会有同样的困扰，如何组织功能模块的代码才能避免出现递归引用呢？甚至项目已经进行到一半了，随着功能交叉点的增多，才暴露出递归引用的问题。

其实问题的关键点在于合理的使用interface，下面我先介绍一下这个demo项目的结构。

这个demo项目假想成具有一定规模的服务端项目，其中包含一个以上的为不同目的开发的服务器程序，以及程序间公用的代码库。

这里用`server1`和`server2`代表两个服务器程序，比如游戏项目经常会有游戏服务端和游戏网关等多个进程。`library`目录下则是公共的代码。

library目录的结构这里就不需介绍了，各种开源的Go框架或Go语言自身提供的库就是典型的库结构，没有复杂的业务逻辑交叉，不是这个demo要演示的重点。

这里我们用server1来给大家做演示，server2的存在意义是告诉大家不同的程序之间是可以有共同的代码组织规范的，所以server2里面就不放具体代码了，只是象征性的存在。

Go语言默认的编译规则要求能独立运行的程序必须由一个main包的main函数作为入口，这里我们分别将server1和server2的main放在其根目录中的main.go。

在server1的子一级，我们创建一个叫module的目录用来放置功能模块的代码，module目录里又分别为各个功能独立创建一个子目录，比如player模块，item模块等等。

举一个最常见的需求，物品模块需要有购买操作，购买时需要扣除对应物品价格的铜钱。

为了满足这个需求，直接让item模块去引用player模块，在项目初期这么做是没有问题的。但是随着项目越来越复杂，player模块可能反过来需要引用到item模块的另外一个操作，这时候为了避免递归引用就得想各种办法的。

不对项目进行大的重构就只能保持item模块引用player模块，为了让player模块里的操作需求能得到满足，需要在player模块里定义一个接口，要求接口实现所需的操作，然后让item模块实现这个操作并把接口实现注册到玩家模块里，这样就可以保持item模块引用player模块了。

但是这样的办法总归是马后炮的fix，用多了就会导致很多零散的接口声明和实现以及注册。

最好的办法是一开始就不要让模块之间互相直接引用，demo中我在`module.go`里定义了两个接口，分别是`PlayerModule`和`ItemModule`，其中只有`PlayerModule`有一个`DecreaseCoins`方法，原因是在演示中，我们暂时没有出现需要提供给外部使用的Item模块的方法。

`module.go`中除了定义接口，还声明了两个全局变量，分包是`Player`和`Item`，但是没有初始化。反过来，我们在item模块和player模块的`init`函数中，初始化`module.Item`和`module.Player`。

这样的结构，导致了module模块永远都是被引用的，而不会主动引用到功能模块，它只知道功能模块应该提供哪些公共方法，但这些方法具体实现在哪里它是不知道的。而各个功能模块不会再直接的互相引用，而是统一的引用module模块，当item模块需要使用player模块的扣铜钱方法时，用`module.Player.DecreaseCoins()`的形式来调用。

最终，所有的公共接口被放在module里管理，不会再出现递归引用，我们还顺便得到了一个可以方便单元测试的项目结构，当要单元测试某个模块的时候，我们可以mock它所依赖的外部方法，可以用来提供测试数据或者记录测试过程。

