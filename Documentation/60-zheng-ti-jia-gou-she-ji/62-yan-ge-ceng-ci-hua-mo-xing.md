6.2 严格层次化模型
===========

![](https://csdnimg.cn/release/phoenix/template/new_img/original.png)  

[小马儿2019](https://me.csdn.net/zhangmalong) 2020-05-15 20:09:14 ![](https://csdnimg.cn/release/phoenix/template/new_img/articleRead.png) 983  ![](https://csdnimg.cn/release/phoenix/template/new_img/collect.png) ![](https://csdnimg.cn/release/phoenix/template/new_img/tobarCollectionActive.png) 收藏   ![](https://csdnimg.cn/release/phoenix/template/new_img/planImg.png) 原力计划

最后发布:2020-05-15 20:09:14首发:2020-05-15 20:09:14

文章标签： [c语言](https://so.csdn.net/so/search/s.do?q=c语言&t=blog) [嵌入式](https://so.csdn.net/so/search/s.do?q=嵌入式&t=blog) [程序设计](https://so.csdn.net/so/search/s.do?q=程序设计&t=blog)

版权声明：本文为博主原创文章，遵循 [CC 4.0 BY-SA](http://creativecommons.org/licenses/by-sa/4.0/) 版权协议，转载请附上原文出处链接和本声明。

本文链接：[https://blog.csdn.net/zhangmalong/article/details/106148707](https://blog.csdn.net/zhangmalong/article/details/106148707)

前文提及，直面真实可卖的产品，才会品味到真正的架构设计。真实可卖的产品有一个很重要的特点：有很多约束条件，其中最主要的一项约束条件就是团队人员约束。

作为项目经理，我做产品时肯定希望自己手下人才济济，能人辈出，但现实经常打脸，能给我一两名稍微有点经验的就不错了，其他大多都是新招聘的，很多甚至都没写过几行代码，(꒦_꒦) 。一个公司不可能将优秀的人员都集中到一个项目组，不仅不经济，而且会带来诸多管理问题。现实情况约束下，大多数项目组构成都是一个牛人带着一群新手。

团队能力约束仅仅是一方面，还有专业水平的约束。真实可卖的产品一般都跨越多个专业方向，如本书描述的两款微机保护产品，不仅需要计算机程序员，而且需要电力专业工程师。电力工程师需要编写保护模块的程序，但我们不能指望他们也拥有丰富的编程经验。

我认为，真实世界，**人员专业方向和能力水平是产品构建时最基本约束条件**。

◇◇◇

职场人员的成长需要时间，不可能将所有人员培养成功了在做产品，破解人员约束条件的最佳策略就是**代码分层**。

在前面的一些示例代码中，我多次描述过互斥锁机制。在嵌入式系统中，互斥锁是一个很重要的技术难点，如果不认真对待，最终产品经常会出现一些莫名其妙的问题，隐晦且难以复现。嵌入式系统中的互斥锁种类繁多，如中断和主程序的互斥、各任务之间的互斥、各分布式系统之间的互斥等，这导致互斥锁代码不仅不容易编写，而且不容易调试，让新人头疼。

面对互斥锁，我们团队的新人大概会经历如下三个阶段。  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200515200224439.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW5nbWFsb25n,size_16,color_FFFFFF,t_70)

第一阶段：无知。新人一开始根本意识不到竟然还需要考虑互斥锁，无知者无畏，直到有一天，产品测试时被各种诡异现象持续打脸，然后才如梦初醒，迈入下一阶段。

第二阶段：害怕。此时新人已经知道需要认真对待互斥锁机制了，但会感觉几乎到处都存在互斥，举步维艰，不知该如何下手。

第三阶段：熟练。掌握了常见的互斥机制，熟悉何种情况下该使用哪种策略最佳，渐入掌控随心境界。

经过多年努力，我们团队的知识库已总结出各种情况下的最佳互斥锁策略。但即使如此，新人迈过这三个阶段，依然需要很长的时间。

为了应对这个约束条件，我们团队采取的策略是：**在下一层隐藏锁机制**，如下图所示：  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200515200254414.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW5nbWFsb25n,size_16,color_FFFFFF,t_70)

将互斥锁都被隐藏封装在一个个的模块api接口函数内部，此时，功能模块就可以安全忽略互斥锁了。采用这样的策略后，我做产品是会发现工作就很好安排了，挑选有经验的工程师负责底层模块，细节多复杂度高但代码量少，老人愿意干；新人负责功能模块，代码量虽大但基本是流程代码，比较简单，新人能hold住，也干劲十足。

◇◇◇

做真实产品时，能力约束相对还比较容易克服，不管如何大家还是同一专业方向，可以进行高效的交流，专业约束更让人头疼。

我曾参与的某项目，因为涉及到复杂的算法，团队成员中有一些电力领域专家，他们级别都蛮高的。专家虽然专业领域水平很高，但不一定写程序水平也很高，但因为领域专家与生俱来的傲气，给团队沟通交流带来了诸多困难。

我们以微机保护为例，假设保护算法部分需要领域专家来完成，过流保护指电流大于设定值且保持一段时间后动作，领域专家希望有如下一段简单直观的代码来实现上述功能：

    if (Ia >= ISet)
    {
    	if (tCur >= t)		/* 其中tCur为当前状态保持时间 */
    	{
    		/* 触发保护动作 */
    		…
    	}
    }
    

然而遗憾的是，真实产品都是多专业结合的产物，上述Ia、Iset等变量不仅领域专家要用，维护软件等其他模块也需要，一系列单独变量的模式适合领域专家，但并不适合维护软件。双方的冲突如下图示意：  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200515200328265.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW5nbWFsb25n,size_16,color_FFFFFF,t_70)  
我们不可能让所有的专家都成为程序高手，此时最好的策略也是分层，基于专业的分层。为了解决这一冲突，我们团队引入了一个配置软件的概念，统一管理微机保护数据模型，并自动分别生成各应用方向需要的数据结构，如下图所示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020051520041975.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW5nbWFsb25n,size_16,color_FFFFFF,t_70)  
借助配置软件，专业人员面对的是一个个单独的变量，便于编写单纯简单的领域代码，同时，维护软件程序员面对的是变量参数表，便于编写统一代码。代码示意如下：

    /* "过流I段(速断)",针对微机保护专业人员变量 */
    FLOAT  SET_fOC1;                                        /* I段过流(速断)定值 */
    DWORD  SET_dwOC1_Delay;                                 /* I段过流(速断)时延 */
    DWORD  SET_dwOC1_Type;                                  /* I段过流(速断)动作特性 */
    ……
    
    /* 设定值属性描述表，用于平台模块*/
    const TAPISettingProp g_apiSettingProp[] = {
    	{
    		700ul, 44ul, 5ul, 32772ul, 0ul, 0ul,
    		(FLOAT)0.250f, (FLOAT)100.000f, (FLOAT)30.000f,
    		&SET_fOC1, (TAPICoeffProp*)&g_apiCoefProp[1]
    	},
    	{
    		701ul, 702ul, 2ul, 4ul, 0ul, 0ul,
    		(FLOAT)0.000f, (FLOAT)400.000f, (FLOAT)0.000f,
    		&SET_dwOC1_Delay, (TAPICoeffProp*)&g_apiCoefProp[22]
    	},
    	{
    		703ul, 0ul, 2ul, 8ul, 22ul, 4ul,
    		(FLOAT)0.000f, (FLOAT)3.000f, (FLOAT)0.000f,
    		&SET_dwOC1_Type, (TAPICoeffProp*)&g_apiCoefProp[19]
    	},
    	……
    };
    

实际上，沿着这条路径持续完善下去，产品研发会逐渐分化成两个层次：平台化研发和领域产品研发。平台化研发部分侧重于基础功能，如芯片驱动、如通用数据模型、如维护软件等；领域产品借助平台提供的基础条件，进行各行业产品快速研发，如电力行业、石油行业、仪表计量领域等。

基于这种结构，平台化研发时很多模块会自然而然的考虑尽量复用，因此我们团队内部经常称自己的研发架构为高复用平台化架构（还不是最终名称），如下图所示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200515200451997.png)  
◇◇◇

在“接口和模块化”章节中，我们提到软件模块本身具备强耦合特性，即使一开始接口抽象的比较好，如果不加以控制，用不了多久就会迭代成一团乱麻。实际上不仅软件模块如此，层次化接口也如此，虽然一开始层次划分的比较好，但很容易在迭代中变形了。

为了解决这一问题，我们团队提出了一个概念：严格层次化模型。简单来说，就是仅允许上层直接调用下层，应用直接调用平台，但不允许反方向调用。普通层次化模块同严格层次化模型对比如下图所示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200515200511787.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW5nbWFsb25n,size_16,color_FFFFFF,t_70)  
为了实现这种严格要求的层次化模型，需要使用一些简单的程序技巧，主要有：

1.  回调函数机制。回调函数机制是最简单的策略，碰到需要下层调用上层情况时，需要优化为上层注册回调函数，然后下层在必要时候调用该回调函数。采用回调函数策略，被回调函数在底层任务空间执行，如果产品基于多任务系统，会带来较多的同步互斥问题，因此回调函数机制常用于前后台系统。
2.  消息调用机制。这种策略类似于回调函数，差别仅仅在于注册的是消息ID，下层不在是调用回调函数，而是发送消息函数。相比回调函数机制，消息函数机制有一个优点，应用模块的所有消息函数都是在当前任务环境空间执行，此时可以安全的忽略同步互斥问题。因为消息机制的这个优点，在嵌入式设备架构中，我们团队主要使用这种策略。
3.  虚函数机制。这是适用于C++语言的常见策略，C++类库中经常采取这种策略，应用模块主要通过类继承和重载虚函数的方式构建，虚函数的调用经常会下沉到基类或类库中。这种策略我们常用于外围软件中，如维护软件和配置软件等。
4.  Qt信号和槽机制。我们团队的外围软件大多基于Qt构建，基于Qt的程序有一种很优秀的弱耦合机制：“信号和槽”，其类似于消息，但有更多的优秀特性。在构建基于Qt的程序时，如能合理利用该特性，可以做到平台和应用层充分解耦。

除了这些常见的程序技巧外，还有很多优秀的策略，我平时接触学习各种优秀类库时，经常会有叹为观止的感觉。借助知识库工具，平时多积累，会慢慢产生一种潜移默化的作用，内心甚至会开始强烈抵触一团乱麻的程序。

基于这种严格化的要求，我们团队产品架构中的“平台化”特征越发强烈，为了便于交流，我们团队内部经常称这种架构为：**高复用严格平台化架构**。

本书以两款微机保护产品为抓手，按照高复用严格平台化策略，整个产品的层次架构图如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200515200537376.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW5nbWFsb25n,size_16,color_FFFFFF,t_70)  
说明如下：

1.  最底层为硬件抽象层。增加硬件抽象层，最大的好处就是硬件移植方便，不会因为换一个芯片而修改一大堆代码了。
2.  第二层是基础平台层，这部分是我们团队多年抽象凝练的一些高复用软件模块，或者引入的各种程序架构。这些模块同各类设备的具体数据模型没关系，可以横跨多类设备复用。有平台的味道，但又比较基础通用，我们团队习惯称之为基础平台层。
3.  第三层是api层。这一层侧重于各类数据模型的抽象提炼，因为这一层所有的接口函数都包含了api前缀，因此我们团队习惯性称之为api层。提炼数据模型时，习惯性分为上下两层：基础数据模型和高级数据模型，高级数据模型会引用基础数据模型，两类数据模型之间严格分层。
4.  最上层是高级应用层，既包含规约、界面等平台通用模块，也包含过流、差动等专业应用模块。各高级应用模块是基于且仅基于底层平台程序的，也就是说各高级应用模块之间要严格分割。这一层的很多代码是依赖于具体产品的，很多不需要复用化考虑，平台结合特定功能模块，最终组成了各种各样的产品。
5.  在上述分层体系中，不仅各层之间都是严格分层的，除了基础数据模型层，大部分模块也都是严格抽象的。

从普通的分层概念，到高复用严格平台化，您品到了一点点架构的味道吗！

——————————————

[返回目录](https://blog.csdn.net/zhangmalong/article/details/103197670)