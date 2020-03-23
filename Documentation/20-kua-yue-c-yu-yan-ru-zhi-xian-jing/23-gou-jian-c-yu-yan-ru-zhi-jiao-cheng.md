2.3 构建C语言入职教程
=============

原创 [小马儿2019](https://me.csdn.net/zhangmalong) 最后发布于2020-01-03 09:00:23 阅读数 202 收藏

发布于2020-01-03 09:00:23

[](http://creativecommons.org/licenses/by-sa/4.0/)版权声明：本文为博主原创文章，遵循 [CC 4.0 BY-SA](http://creativecommons.org/licenses/by-sa/4.0/) 版权协议，转载请附上原文出处链接和本声明。

本文链接：[https://blog.csdn.net/zhangmalong/article/details/103814489](https://blog.csdn.net/zhangmalong/article/details/103814489)

2.3.1 职场需求和学校教育的脱节
------------------

在工控嵌入式领域，C语言能力是一个几乎要求所有工程师都必须掌握的基础技能，不管你是软件工程师、硬件工程师还是测试工程师。假设你从事嵌入式软件工作，则还会进一步要求你达到相当的深度，工作方可应付自如。

本书侧重于嵌入式软件和架构设计，如果一名新人想在这方面有所建树，嵌入式C语言技能就是必须迈过去的第一道坎。上一节我们找到了快速入门的钥匙，那如何打开“C语言入职”这把锁呢？

◇◇◇

工科类院校，大多数专业都要求学习C语言，而且后续课程也会提供多次加强练习的机会，按理说大家毕业时应该已经具备一定素养的C语言编程能力。然而，理想很丰满，现实很骨感。可能大部分优秀学生都被头部企业抢走了吧，我的多年招聘生涯中，发现大部分同学和预期都相差甚远，甚至相当比例同学写不出只有几十行代码的小程序。

和一些学生交流，发现目前的大学C语言课程还是沿袭着多年前的模式，侧重于讲语法，喜欢扣细节，考试也是侧重语法细节，练习反而做得很少。想写好文章，仅认字是不够的，同理，想写出好程序，仅学语法也是远远不够的。

我个人比较讨厌这种教学模式，尤其讨厌谭浩强类的教程，整天研究p=(i++)+(i++)+(i++)类表达式有什么意思。年少轻狂时，我去某知名企业面试，教官给我出了一道C语言题：int n=3,5; 请问n是多少。我当时直接脱口而出，您不应该问这样的问题，产品中也不应该出现这样的代码。然后，我就被悲催的轰出去了。想在回想起来都有点尴尬，不过大家应该能体会到我多么讨厌这类东东了。

除了不重视实践和练习，大学教育还有一个特点，也不太重视各类编程工具的使用训练。在大学阶段很多程序都比较短小，敲进去执行就可以了，但工作中面对铺天盖地的代码，没有过硬的调试分析能力，工作效率就会大打折扣，甚至举步维艰。

职场教育和学校教育还存在一个很大的不同，职场环境下再也没有人手把手的教你了，顶多整体指点一下。职场更多是靠自学，然后借助工作迭代提升，而这种学习模式对刚入职的新人也需要一段适应期。

另外，职场中也很难给你提供大块的学习时间。C语言入门说难不难，说简单也不简单，即使是最小必要知识，也涉及到开发环境、数据模型和运算符、控制流、函数、指针、结构、库函数、基础程序、单循环体、常用数据结构等诸多概念。如果新人基础比较薄弱，可能需要两三个月的时间才能熟练掌握，但企业能给新人这么长的时间吗？又如何解决呢？

学校教育和职场需求存在脱节，面对这一系列困局，我们的C语言入职课程该如何构建呢？

2.3.2 兴趣是如何产生的
--------------

我们经常能听到类似这样的观点：“某人程序写的好，是因为他对编程有兴趣。”这句话的言外之意就是：“我的程序写的不好，是因为我没兴趣，这是天份决定的，我无可奈何，也无能为力。”

好完美的借口，难道真是这样子吗？

我记得在大学第一学期学习Pascal语言时，可能惯性吧，一开始整天抱着书在纸上写程序。那段时间我对程序没有任何感觉，总感觉有一层窗户纸似的，朦朦胧胧。后来，我在图书馆里淘到一本Pascal书籍，是关于程序竞赛题目的讲解书籍，但不同于传统的编程书籍，其中每个例程都会由普通到巧妙的有几个版本。

我清楚的记得，这本书的第一个例程是一个关于菱形输出的例程。我买了几个小时的上机时间，花了一个下午将各个版本的例程依次敲入电脑中，那一刻，我好似有了一种明悟，好像突然开始理解程序是怎么一回事。后来，这本书被我循环借了整整四年，青蛙跳、八皇后、数字魔方，一个个有趣的难题，一段段精巧的程序，让我乐在其中。与此同时，我也慢慢成为了同学们眼中的编程爱好者。

因此，所谓兴趣爱好，仅仅是一些列正反馈的产物而已。大部分工控产品程序复杂度有限，应该还不到拼智商的程度，你仅仅是没有写过好、更好、更更好、更更更好的程序而已。

我们是否可以在C语言入职培训时，顺便提升一下大家的兴趣呢？

2.3.3 构建C语言入职教程
---------------

为了应对这些困局，经过数年和无数新人的反复试错迭代，我打磨出一套行之有效的C语言入职训练课程。这套课程采用基于项目的模式，并巧妙的将新人需要快速掌握的各种技能融入其中，故意让新人去采坑并克服。因此，部门内部经常将这套课程戏称为“跨越C语言入职陷阱”。

除了部分头部优秀企业，大多数企业招聘到的都是普通学生，我们也不例外。如果能碰到可以熟练写出单循环判断程序，我们就要笑开花了。单循环判断程序结构是指仅需要一层循环，但循环体中的内容会依赖循环子而执行的程序，如“检测一个整数中0的个数”等例程。而这，也是这套入职培训教材的起点。

当然，如果一些学生暂时达不到这一要求，我们会给一个机会。推荐他快速阅读《C程序设计语言》第二版这本书中的指定章节，大概有100页左右内容，一般一个星期可以读完。此时如果能达到上述要求，说明这个新人的快速学习和承压能力都不错，企业也是非常乐意录用的。《C程序设计语言》第二版这本书是我推荐我们团队所有小伙伴细细阅读的一本书，不仅仅是因为这本书很经典，有很多漂亮的小例程，更关键的是薄，适合职场学习。除此之外，其他c语言书籍大都被我们仅仅当作参考书了。

◇◇◇

为了让新人快速适应企业的学习模式，整个培训教材采取了基于项目的模式。提出一个需求，让新人去实施，审核后指出问题所在，持续迭代，持续改进。做完后，会指导新人进行总结反思，而此时总结的内容恰恰就是需要新人掌握的技能点。整个培训教程，由三个小例程组成的。

我曾经细细分析过自己所负责产品的代码，发现需要复杂数据结构和算法的程序占比非常小，绝大部分代码都是简单流程类代码，但其中一类程序结构出现的非常多，那就是单循环判断程序结构。因此，我们的第一个例程主要用于锻炼新人这方面的能力。当然，还会融入很多的职场技能，如：调试技能、代码分节、工作闭环、权衡艺术、边界判断、总结反思等。

一般情况下，第一个例程大多数新人都需要折腾六七次左右，才能写出完全符合要求的程序，但可喜的是，大多数人在最后总结时都能清晰感受到自己的进步，内心会油然而生一种成就感。这就是我刻意进行的兴趣培养，也希望能在大家心中埋下一颗种子，帮助他们以后走的更远更高。

谈及C语言，就不能不谈及指针。很遗憾的是，我们招聘的新人，大多都喜欢刻意回避指针。问及原因，一般都是道听途说很难很难，因此，大家就故意视而不见，躲的远远的。没法子，工作中还是需要指针的，尤其是随着程序复用的提升，指针越发的满天飞了，因此第二个例程侧重于克服大家的指针痛点。

第二个例程中，我会先给大家系统的讲解指针的概念，然后会让大家写一个指针列表的例程。这个例程的名字叫“学生信息管理系统”，是否有点似曾相识的感觉，因为该例程恰恰是很多学校的C语言作业之一。实践中，一些新人会感觉这个例程比较简单，没想到这才刚刚开始，我们随即进行的一系列批判行为，才会让大家真正深刻认识到具体工作中的的链表是怎么样子的。

程序模块并非简单堆砌在一起的，需要有机的组织起来才能形成产品，背后就需要架构的支撑。第三个例程是一个简单的架构入门例程，当然也是嵌入式系统中一种常见的架构策略。经过这样的铺垫，新人在后续接触真实产品时，就会多一点点亲切感，或许就能走的更顺一些。

◇◇◇

各大企业为了抢到最优秀的毕业生，招聘工作往往会提前，一般大四上学期就开始了。形势所逼，我们被迫也需要在这个时间点进行校招。这样会出现了一个问题，新人就职和招聘之间有很长一段时间，进而会导致很多变数。

企业招聘是有人员额度限制的，如大部分学生都会骑着马找马，碰到薪水更高的工作会立即违约，我的团队就经常碰到新人大半违约，但招聘季又过了的尴尬局面。

针对这种情况，我们会在录取通过后立即着手进行入职培训。这样做有多方面的好处，不仅可以充分利用在校学生的大把空余时间，一入职就可以上岗；而且可以提前甄别，过滤不合格学生；甚至在某种程度上会提前建立企业和学生之间的情感纽带，以降低违约率。

总之，经过一段时间的磨合，我们会明显的感觉到新人会分为三六九等，或直接违约、或重点挽留，都能进一步节约后续用人成本。现在，让我们一起蹚一蹚这套C语言入职培训教程吧！

[返回目录](https://blog.csdn.net/zhangmalong/article/details/103197670)  