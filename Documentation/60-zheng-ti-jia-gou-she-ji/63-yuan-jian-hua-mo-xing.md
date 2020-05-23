6.3 元件化模型
=========

![](https://csdnimg.cn/release/phoenix/template/new_img/original.png)  

[小马儿2019](https://me.csdn.net/zhangmalong) 2020-05-22 17:27:35 ![](https://csdnimg.cn/release/phoenix/template/new_img/articleRead.png) 122 ![](https://csdnimg.cn/release/phoenix/template/new_img/collect.png) ![](https://csdnimg.cn/release/phoenix/template/new_img/tobarCollectionActive.png) 收藏 

最后发布:2020-05-22 17:27:35首发:2020-05-22 17:27:35

文章标签： [c语言](https://so.csdn.net/so/search/s.do?q=c语言&t=blog) [程序设计](https://so.csdn.net/so/search/s.do?q=程序设计&t=blog) [嵌入式](https://so.csdn.net/so/search/s.do?q=嵌入式&t=blog)

版权声明：本文为博主原创文章，遵循 [CC 4.0 BY-SA](http://creativecommons.org/licenses/by-sa/4.0/) 版权协议，转载请附上原文出处链接和本声明。

本文链接：[https://blog.csdn.net/zhangmalong/article/details/106287576](https://blog.csdn.net/zhangmalong/article/details/106287576)

通过严格化层次模型，软件各层之间进行了隔离，但同一层内的各软件模块之间还易存在耦合。回顾我们的微机保护产品架构图，会发现其它层还好，但api抽象层各模块之间容易出现耦合。

如何缓解这一问题呢，在前一章信号复归例程中已经给出了答案：**脚本和元件化模型**。

因为需求迭代，信号复归模块和按键模块存在强耦合，为了解耦，对信号复归和按键模块进行了元件化改造并提炼接口，然后由脚本模块实现这些接口之间的耦合，这种策略被我们团队称之为元件化模型。信号复归和按键模块抽象过程如下图所示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200522172146188.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW5nbWFsb25n,size_16,color_FFFFFF,t_70)  
其中：信号复归模块增加了两个接口：signalRstIn和signalRstOut，按键模块增加了一个接口：signalRstKey。关联按键模块和信号复归模块脚本如下：

    signalRstIn = time(signalRstKey, 1000, 0);		//按键持续1000ms后才置位signalRstIn
    

脚本和元件化模型，有两个关键点：**各软件模块元件化改造和脚本执行环境**。

元件化相对比较好理解，主要是指对一个软件模块进行接口抽象。如过流保护中的过流模块，其工作示意图如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020052217223131.png)  
正常情况下，过流后应该触发出口动作，但微机保护一般有多个硬件出口，触发哪个出口呢？碰到这类需求，就可以为过流保护提炼一个接口：OC1_A。然后增加如下脚本即可：

    OUT1 = OC1_A + OC1_B + OC1_C;
    

该处，该脚本有一个额外的要求，从过流判断到保护设备出口整个过程都属于强实时模块，因此需要上述脚本能够快速执行（ms级别）。

不同于整体架构设计，在真实产品具体实施过程中，元件化是一个持续迭代过程，一般是由需求推动，然后才能有效识别出那些软件模块属于可变模块，然后才能将其慢慢的元件化。

元件化提供了灵活度，但也会带来一定程度的效率损失，切忌过度元件化。我们团队的一些产品，开始整体设计时就犯过这样的问题，提炼了过多的接口，不仅拖累了研发进度，也影响了程序执行效率，事后分析，超过一半的接口在工程现场从未用过。

◇◇◇

不同于元件化比较依赖行业经验，脚本执行环境是一个纯IT模块，对非计算机专业人员会显得稍微有点复杂，幸运的是这几乎是计算机行业中最为成熟的软件模块了，国内几乎所有计算机本科生都会要求写一个简单的编译系统，对大多数应用可能已经足够了。

随着计算机行业的快速发展，也出现了很多脚本执行环境中间件提供商，如我当初在跨国企业工作时，发现他们就是引入的外部工具，功能相当的强大，支持完整的IEC1131-3工业标准化编程语言标准，运行截图如下所示：  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200522172304513.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW5nbWFsb25n,size_16,color_FFFFFF,t_70)  
我们自己的团队花不起钱，只好采取自研的策略，先后经历的几个阶段：

1.  刚开始仅提供了基础的表达式解析，采用最简单的堆栈解析策略，可用于完成一些简单的非强实时模块的脚本解析。
2.  随着保护模块元件化的推进，需要以高效的方式解析（ms级别）。为了实现这一特点，我们借鉴sel保护的设计理念，仅支持布尔表达式，借助布尔表达式的特性（1or…=1，0and…=0），以大幅度提升解析效率，现场常见脚本配置都可以在1ms内解析执行完毕。
3.  随着现场应用场景的增加，布尔表达式的弊端开始隐现，一些稍微复杂的逻辑用纯布尔表达式难以编写（如电力系统备自投设备），有时候需要编写一些复杂闭环控制逻辑（如PID）。为了应对这样的需求，我们对原有布尔表达式进行了语义扩展，支持if-then-else，while等语句。
4.  随着脚本使用的越来越多，整体架构设计也在适应迭代中，脚本开始嵌入到产品的方方面面，有的脚本用于保护强实时逻辑，有的脚本用于系统参数设置，维护软件等外围支撑软件也日趋人性化。

即使经过了无数次迭代，满足了很多现场的变态需求，我们团队依然维持着一个相对比较简单的脚本系统，主要功能包括：

1.  常见的布尔表达式和逻辑表达式，其中布尔表达式部分支持快速解析。
2.  支持三种变量定义：布尔、32位整数、单精度浮点数。
3.  if-then-else语句。
4.  while语句。
5.  for语句。
6.  break语句。
7.  内部函数：类型转换、各类延时函数、pid控制等。
8.  变量观察窗口，便于现场调试。
9.  仿真执行环境，用于脚本写入前验证。

简单脚本的实现有很多优秀书籍，我个人主要参考了《编译原理及实践》和《编译原理(龙书)》这两本书，我们团队脚本的BNF语法描述如下，借此可快速写出自顶向下的递归下降语法分析程序，有兴趣的朋友可以参考：

1.  program->declaration_list
2.  declaration\_list->declaration\_list declaration | declaration
3.  declaration->var\_declaration | fun\_declaration
4.  var\_declaration->type\_specifier ID{, …}`;` | type_specifier ID `[` INT `]`{, …}`;`
5.  type_specifier->`int` | `chaår` | `float`
6.  fun\_declaration->fun\_specifier ID `(` param_list `)`  
    `{` loal\_declarations statement\_list `}`
7.  fun_specifier->`void` | `int` | `char` | `float`
8.  param_list->`void` | empty | param{`,`param}
9.  param->type\_specifier ID | type\_specifier ID `[` `]`
10.  local\_declarations->local\_declarations var\_declaration | var\_declaration | empty
11.  statement\_list->statement\_list statement | statement
12.  statement-> if\_stmt | while\_stmt | for\_stmt | break\_stmt  
    | continue\_stmt | return\_stmt | expression\_stmt | compound\_stmt | call’;’
13.  if_stmt->`if` `(` expression `)` compound_stmt  
    | `if` `(` expression `)` compound_stmt `else` compound_stmt
14.  while_stmt->`while` `(` expression `)` compound_stmt
15.  for_stmt->`for` `(` expression_stmt expression `;` var `=` expression `)` compound_stmt
16.  break_stmt->`break` `;`
17.  continue_stmt->`continue` `;`
18.  return_stmt->`return` `;` | `return` expression `;`
19.  compound_stmt->`{` statement_list `}` | statement
20.  expression_stmt-> var `=` expression`;`
21.  expression->expression rel\_op add\_expression | add_expression  
    ->add\_expression{rel\_op add_expression}
22.  rel_op-> `<<` | `>>`  
    | `&` | `^` | `|` | `&&` | `||`  
    | `<=` | `<` | `>` | `>=` | `==` | `!=`
23.  add\_expression->add\_expression add\_op mul\_expression | mul_expression  
    ->mul\_expression{add\_op mul_expression}
24.  add_op-> `+` | `-`
25.  mul\_expression-> mul\_expression mul_op factor | factor  
    ->factor{ mul_op factor }
26.  mul_op-> `*` | `/` | `%`
27.  factor->`(` expression `)` | var | call | single_op expression | INT | FLOAT | STRING;
28.  single_op->`-` | `!` | `~`
29.  var->ID | ID `[` expression `]`
30.  call->ID `(` args `)`
31.  args-> empty | expression {`,` expression}

◇◇◇

大学时，我迷恋过一段时间的3dmax，后来竟然惊奇的发现可以用脚本控制绘制各种复杂的立体图形。然后，我发现几乎所有的优秀软件都可以通过脚本控制或扩展，如office、cad、mud等，甚至自己整日用的source insight也可以用脚本扩展，被我鼓捣出很多好用的功能。

冥冥之中，我有一种强烈的直觉：**元件化和脚本，几乎是软件架构世界的通行证**。可能正因为这个原因吧，在微机保护产品的架构设计中，我尤其会关注了其他厂家是如何将脚本嵌入到产品中的，其中最典型的就是sel保护，非常有特点。  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200522172341349.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW5nbWFsb25n,size_16,color_FFFFFF,t_70)  
走上了这条路后，虽然经历了诸多波折，但收获是巨大的，最开始的体现就是现场特殊工程版本数量开始大幅度减少，很多特殊现场需求仅需要指导工程人员编写几行脚本就ok了。

然后，在我们刻意的将脚本和元件化设计理念引入架构设计后，保护设备种类也开始大幅度减少，最后甚至做到了一款综合保护装置适用于半个用户供电现场的情况。后期随着持续迭代，甚至开始影响到全流程，甚至给服务、生产、运维、测试等环节都带来了深刻的影响。

目前，不管是引入还是自研，简单脚本系统的门槛已经大幅降低了。根据我的过往经验，元件化和脚本是工业产品中最重要的设计理念之一，因此我强烈建议大家引入。无论如何，不能因脚本系统的一点点复杂度而阻挠整个系统架构更进一步，因小失大。

——————————————

[返回目录](https://blog.csdn.net/zhangmalong/article/details/103197670)
