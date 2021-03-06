6.1 产品化思维，迈入架构世界
================

![](https://csdnimg.cn/release/phoenix/template/new_img/original.png)  

[小马儿2019](https://me.csdn.net/zhangmalong) 2020-05-08 12:07:28 ![](https://csdnimg.cn/release/phoenix/template/new_img/articleRead.png) 89 ![](https://csdnimg.cn/release/phoenix/template/new_img/collect.png) ![](https://csdnimg.cn/release/phoenix/template/new_img/tobarCollectionActive.png) 收藏 

最后发布:2020-05-08 12:07:28首发:2020-05-08 12:07:28

文章标签： [c语言](https://so.csdn.net/so/search/s.do?q=c语言&t=blog) [程序设计](https://so.csdn.net/so/search/s.do?q=程序设计&t=blog) [嵌入式](https://so.csdn.net/so/search/s.do?q=嵌入式&t=blog)

版权声明：本文为博主原创文章，遵循 [CC 4.0 BY-SA](http://creativecommons.org/licenses/by-sa/4.0/) 版权协议，转载请附上原文出处链接和本声明。

本文链接：[https://blog.csdn.net/zhangmalong/article/details/105992562](https://blog.csdn.net/zhangmalong/article/details/105992562)

同很多同事聊天时，我发现大家都有这样的困惑：工作多年，除了刚开始成长感觉比较明显外，后续很长时间都感觉不到任何提升，也不知道该如何用力。

上一章介绍的接口抽象能力，是我们在工作中自然而然衍生出来的能力，因为大多数程序员每天工作面对的就是一个个的程序模块，差别仅仅在于接口抽象的好与坏。

从上一章我们发现，一些优秀的接口抽象，已经需要其他模块的配合。更进一步，一款可卖的真正产品，是由很多个模块组合起来的。遗憾的是，并非一堆抽象优秀的模块就可以堆砌出优秀的产品，需要超越模块级别，需要从更高一层次的产品角度去思考，才能组合出优秀的产品。在我的眼中，这种将模块组织成产品的诸多策略就是产品架构，而只有产品化思维，才会带领我们迈入架构设计的精彩世界。

从入职到架构师，我们在逐级台阶往上爬，入职——团队——抽象——架构——复用——质量——标准化，为了让自己更进一步，我们必须迈入架构的世界。在真实职场中，大部分程序员的工作仅仅是基于原有产品的修修补补，这也是大家容易有瓶颈期的感觉。在此，我真诚的期望，这本书能起到抛砖引玉的作用，能帮助一些朋友迈入架构的世界。

◇◇◇

先前我尝试着提炼自己的工作经验时，采取的是先整体在细节的策略，因此一开始就和大家交流的是产品架构理念。我经常会眉飞色舞的讲起跨国公司的优秀架构设计、某类库的优秀组织模式……，但大多数情况下，大家也就听一个热闹，这让我很苦恼。

一次同一朋友交流时，他道出了其背后的原因：你讲的太虚，虽然所提及的设计理念都很好，但与我何干，对我的具体工作有什么帮助呢？

一句话醍醐灌顶，我试着分析其背后原因，估计有多方面的原因：

1.  我的表达方式欠佳。
2.  现实中，大多人对优秀程序的理解依然停留在程序技巧层次，走到产品架构这一级的相对较少。
3.  没有足够的接口抽象和复杂模块构建经历，贸然接触架构设计都有点虚的感觉。
4.  架构设计不是简单的知识，而是技能。知识可以通过简单的学习获取，而技能必须经过一定量的练习才能习得。更麻烦的是，各类工业产品千差万别，架构设计需要同产品特性契合，它山之石仅可参考，没法照搬。架构的这些特性也增加了大家的学习难度。

正是因为诸多原因，我后来重构时，采取从简单到复杂的策略，并巧妙的契合一个人的成长经历。大家发现没有，这也正是本书的组织架构。

◇◇◇

在我的职场生涯中，有一个架构师对我的帮助极大。之前，我接触到的产品经理大多是被赶鸭子上架，很少有从整体到细节且覆盖整个生命周期的架构分析。幸运的是，在我参与跨国公司项目研发中，亲身经历了一个架构师的谆谆教导，也得以近距离观察思考一个真正架构师的日常。看到就是一种力量，很多东西仅是一层窗户纸，这段经历对我的帮助很大。

刚参与项目组时，我被分配负责一个很小的模块。按照以往工作惯例，师傅带着你了解一下接口后就会让你直接开始工作了。但此次不同，架构师对我进行了完整产品架构培训，甚至会讲到一些为了车间调试采取的设计理念，为了现场工程人员的安全采取的策略等等之类，颇让人意外。

该架构师有一句至理名言：**架构是团队所有成员的共识**。刚开始我总是将其当做一句戏言，但后来，随着我的工作范围越来越宽，到最后直面产品和团队时，我才真正的理解了这段话的真谛。

直面真实可卖的产品，需要考虑很多因素，如团队能力、车间生产、工程调试、维护升级，甚至还有如何推诿扯皮、如何快速将责任甩给其他团队等等诸多现实因素。在这些约束条件下，带领团队构建产品的能力，才是真正的架构设计。如果仅关注单独的一些模块，思维容易受限，可能细节上很优秀的设计在全局不仅无益而且有害。简言以概之：不谋全局者，不足以谋一偶。

架构是全局，模块是细节，很多时候整体和局部是冲突的。为了全局考虑，某些特定模块需要增加一些复杂的功能，可能会加大单一模块的工作量，但对整个产品是有益的。维护系统的能力就是架构的能力，也是不容易做到的。一个公司每个部门都想得到更多的资金，而不是去优化整个公司利益，最终反而会伤害整个公司。

架构是团队所有人的共识，话挺漂亮，但共识这个词有点虚无缥缈，容易让人无从下手，如何破呢？我发现该架构师的做法让人眼前一亮。

以前，我做产品时文档很少，领导经常批评我们，人家大公司的文档都是一堆一堆的，然后我只好无奈的开始补各种自己从来不看的文档。可能是因为各种补文档的经历比较痛苦吧，因此当架构培训时，我们发现整个架构设计文档仅仅是几十页图纸，我被深深的震撼了。

何为共识，共识是大家可以坐在一起反复交流的，厚厚的文档阅读都困难，如何交流！架构师的一番话让我有一种“暮然回首，那人却在阑珊处”的豁然开朗的感觉。实际上，我在跨国公司参与项目这段经历中接触的各类文档，除了那种由程序生成的条目类文档外，大部分文档都比较简洁，也经常采取图形的策略。

我一直珍藏着一份当初面对的第一份文档，也算是一次难得的人生经历吧！这幅图花花绿绿的，大家不需要关注其内容，但不先大略妨感受一下。  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200508120346572.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW5nbWFsb25n,size_16,color_FFFFFF,t_70)

◇◇◇

借助架构设计，能让我们进入产品的世界，此时我们可以冠一个产品经理的称呼了。但是，做出产品并不等于做出好产品，好产品的架构又是怎样的呢？

单纯的说好架构是很虚的，根本让人无从下手，都是些好听的废话。幸运的是，在跨国公司这段经历，让我找到一个难得的可实施的抓点。

当初做架构培训时，该架构师不仅会和我们讲目前的架构设计，而且会给我们讲解当前这样的架构设计是如何一点点依据需求迭代出来的。以他的话，架构设计是活的，是会随着产品需求迭代成长的，而绝大部分优秀的设计理念都是在迭代过程中演化生长出来的。

还记得我们上一章信号模块中按键迭代的例子，虽然需求在迭代，但我们希望信号复归和按键模块依然能保持可复用的特性，顺带着，一系列优秀的设计理念被带了出来。

至此，大家应该会发现，我们的那个可实施的抓点，就是**复用**。我所在的跨国公司，所有基于可复用平台研发的应用产品，对复用性的最低门槛要求是70%，注意该处的可复用可不是拷贝粘贴级别的复用，是代码一行不修改或者干脆是二进制级别的复用。而正是这一抓手，帮助该跨国公司内部长出了一套蔚为壮观的基于平台的高复用产品架构。

**可复用的架构设计，是我们这一章的着力点**。

◇◇◇

直面产品，让我迈入了架构的世界；一段难得的经历，让我眼中的架构世界进行了重构。从此，架构设计在我眼中不再是虚无缥缈的概念了，而是成为可以努力去搏一搏的机会了。那一刻，成为一名优秀架构师成为我的人生目标。

借助基于图形的交流和高复用这个抓手，我在慢慢尝试和攀爬着，让人惊奇的是经过多年迭代后，我们团队内部竟然也长出了一套适合自己的优秀产品架构。在给别人讲解这套架构时，大家会好奇的问，你们哪有空整这些东东啊，看到架构设计一点点长大，内心颇有一点欣慰的感觉。

架构设计不同于其他，别人的东西你是拿不走的，但我相信大家只要握住“高复用”这个抓手，日常工作中多通过图形交流加强共识，你也会长成别人羡慕的样子。

架构师的成长是比较漫长的，培养一个适合自己公司产品的架构师更是一件不容易的事，优秀的架构师是一个公司最宝贵的财富。

——————————————

[返回目录](https://blog.csdn.net/zhangmalong/article/details/103197670)