3.3 编码细节观
=========

原创 [小马儿2019](https://me.csdn.net/zhangmalong) 最后发布于2020-02-21 10:57:52 阅读数 107 已收藏

发布于2020-02-21 10:57:52

[](http://creativecommons.org/licenses/by-sa/4.0/)版权声明：本文为博主原创文章，遵循 [CC 4.0 BY-SA](http://creativecommons.org/licenses/by-sa/4.0/) 版权协议，转载请附上原文出处链接和本声明。

本文链接：[https://blog.csdn.net/zhangmalong/article/details/104424040](https://blog.csdn.net/zhangmalong/article/details/104424040)

通过文件全局观，我们将代码审核的着力点从设计文档细化到了具体函数。面对详细的一行行代码，有哪些策略可以提升代码审核效率呢？

我们人类的大脑喜欢结构化思维，但讨厌流水账。很不巧，现实中的代码经常是灭绝老太的裹脚布——又长又臭。程序主要功能夹杂在各种繁琐的检测代码中，层层叠叠的分支判断和缩进，冗长的句子，经常让我们代码审核时举步维艰。

破解这一困局的最佳策略，就是要让程序结构化起来。我们团队在多年的实践和反复迭代中，探索出了三个易学易用效果好的形式化手段：**节和空行、缩进和换行、空格，统称为代码细节观**。

3.3.1 节和空行
----------

代码分节这个概念前面已经多次提及过了，大家试着回忆一下例程1中的代码分节。

    int main()
    {
        /* 输入内外菱形高度,并进行合法判断 */
    
        /* 循环输出菱形 */
        for (……)
        {
            /* 输出前导空格 */
    
            /* 输出左边星号 */
    
            /* 输出中间空格 */
    
            /* 输出右边星号 */
    
            /* 输出换行 */
            printf("\n");
        }
        return 0;
    }
    

已经过去一段时间了，大家现在可以再品味一次例程1完整代码，是否阅读的时候，即使不读代码，仅阅读节注释都可以快速理解程序大概呢？

真实产品代码，服从二八定律，绝大部分是普通流程代码，仅有20%左右的代码是复杂和易错的。通过代码分节，不仅可有效提升代码审核的速度，而且便于刻意锻炼将精力集中于复杂易错的代码段，也会提升代码审核的效能。

多年的反复实践，我们团队逐渐形成了一个观点：**节是代码思考、构建和阅读的最小逻辑单位**。

◇◇◇

代码分节看似容易，但做起来并不容易。如何让新人快速养成代码分节的习惯呢？经过多年实践，我们团队迭代出一个形式化的策略：**不管复杂还是简单，节代码必须增加节注释，且必须以空行分割**。通过这样的策略，虽然大家代码分节的水平还需要持续提升，但至少表面上像模像样了。

在C语言中，{的位置有两种典型用法，如下图所示：

    /* 典型用法一 */
    if (b) {
    	……
    }
    
    /* 典型用法二 */
    if (b)
    {
    	……
    }
    

我们团队不去约定必须采取哪种策略，如果你将if语句内的所有内容都当做一节，第一种策略较好，如果在if语句内还需要分为多节时，第二种策略的更加，因为{那一行可以认为是空行。示意如下：

    /* 节注释 */
    if (b) {
    	……
    }
    
    /* 节注释 */
    if (b)
    {
    	/* 节注释 */
    	……
    
    	/* 节注释 */
    	……
    }
    

前面关于节注释的风格约定，不仅仅是对函数内代码的约定，实际上是对所有代码的约定。最典型的用法，每个函数就是一个节的起始。如下示意：

    /* 函数注释 */
    void a(void)
    {
    	……
    }
    
    /* 函数注释 */
    void b(void)
    {
    	……
    }
    

为了让约定简单，有些地方为了保持形式化的统一，可能需要增加一些废话了，如下示意：

    /* 内部函数提前声明 */
    static void _msgRecv(void *pOwner, const TXFMsg *pMsg);
    static void _msgTime(void *pOwner, const TXFMsg *pMsg);
    static void _msgDOCmd(void *pOwner, const TXFMsg *pMsg);
    static BOOL _frameCheck(TBaseModbusSample *me, DWORD dwLen);
    static void _txdFrameHead(TBaseModbusSample *me);
    

通过这些形式化的训练，大部分人能快速掌握代码分节技能，进而能持续提升程序结构化能力。

3.3.2 缩进和换行
-----------

c语言是结构化语言，大家习惯通过缩进来体现这种结构化。网络上关于缩进是使用空格还是tab键，有大量的口水仗，我们项目组懒得参与争执，只要项目组内统一即可。我们的项目组统一约定：**缩进使用tab键，且限定4个空格宽**（绝大部分编辑器都支持该设置）。

过多的缩进会增加代码阅读审核负担，是逻辑混乱的起始信号。很多新手写程序时，每增加一个判断条件，会自然而然的新增加一层if…else嵌套，这类代码都是阅读不友好的代码。

考虑程序中可能有两层for循环且夹杂if语句，或者用if…else分节且每节内有for循环，或者switch…case语句中有if语句，综合权衡，我们团队约定**最大不允许超过5次缩进**。

实际的代码有很多可以减少缩进的技巧，如：

1.  很多的层叠判断语句是设计层面的问题，尝试用函数指针或其他技巧替代；
2.  增加子程序；
3.  大量的嵌套是因为各种参数合法判断造成的，可以将这部分代码提炼成单独的一节，如下示例：

    int example(int a)
    {
        /* 参数合法判断 */
        if (a <= 100)
            return -1;
    
        /* 正常处理流程，减少一层缩进 */
        ……
    }
    

4.  循环中的条件判断可以通过continue和break减少缩进层次，如下代码示例：

    int example(int a)
    {
        /* 循环处理 */
        while (a <= 100)
        {
            /* 不合法判断 */
            if (...)
                continue;
           if (...)
                break;
    
            /* 合法处理，减少一层缩进 */
            ……
        }
    }
    

上面这些示例程序，将不合法判断单独提炼成一个节，不仅层层叠叠的缩进减少了，代码逻辑也清晰了，是一常用编程技巧。

额外强调一点，在misra-c编程规范中，推荐不使用continue语句，主要是担心continue语句给程序流程带来混乱。我们团队习惯从工程和实用角度约定编程规范，不迷信权威，不教条，否定了这一条限制。

◇◇◇

与缩进关联的就是换行了，缩进少，一行代码会显的比较短，但工程代码中会使用较长的变量名，也会导致很多代码显得很长。

很多编程规范中经常限制每行最多80个字符，但目前的显示器大多是宽屏显示器，我们团队内部放宽了这一限制条件，习惯约定为120个字符。因为调试时一般会有右侧占用屏幕空间，为了调试方便，建议大家将一行代码控制在100个字符内。

为了便于阅读，过长的语句，我们不建议采用换行策略，而是将遗憾代码改成多行代码。如下所示：

    dwEvent = ((BYTE)me->pPara->comm.dwOverallIndex << 16) + 
    ((BYTE)me->dwCurDevIndex << 8) + me->dwSelectDOIndex;
    

修改为：

    dwEvent = ((BYTE)me->pPara->comm.dwOverallIndex << 16);
    dwEvent += ((BYTE)me->dwCurDevIndex << 8);
    dwEvent += me->dwSelectDOIndex;
    

3.3.3 空格
--------

我在介绍编程规范时，更新了一个观念：程序不是给编译器读的，而是给人读的。这里，在请大家思考一个问题：代码是让CPU执行，还是让人脑执行的呢？

最初听到这样的提问，我感觉不是提问者犯傻，就是脑筋急转弯。但在前辈苦头婆心的劝说下，在走了很多弯路后，我终于养成了**程序编译之前先在脑袋中执行三遍**的习惯。我习惯第一遍侧重代码细节，第二遍侧重程序流程，第三遍侧重整体和对外关联。这种看似慢、实则快的编码技巧，帮助我节约了很多时间，而且也大幅度提升了代码质量，然后我有点庆幸，也有点大汗淋漓的后怕。

自己掌握并不等于团队掌握，在新人培养时，我发现这项策略很难推进。一些人表面上答应，背后骂你好low的观点。你去审核代码，也不知道他是因能力不佳，还是态度不端正导致的代码混乱。

同空格之于节一般，我再次引入了形式化的策略，通过约定空格的用法，逼迫大家在潜移默化中锻炼编程习惯。

大家如果读过一些优秀的商业代码，会发现这些代码中经常使用空格凸显关键字和关键运算符。类似，我们也详细约定了什么情况下应该增加空格。

大多数新人写第一遍程序是记不住去加空格的，为了通过审核，可能需要多次重复检查，顺便也多次执行了（在大脑中）自己的代码（仅空格不是足够的，后续注释和名称约定也有类似的价值）。

我们项目组约定必须增加空格的情形如下，注意加空格本身并非目的，大家可以各自定义自己的规范，该处的规范仅用于形式化：

1.  逗号后面加空格。

    int a, b, c;
    f(a, b, c);
    

2.  比较操作符, 赋值操作符"="、 “+=”，算术操作符"+"、"%"，逻辑操作符"&&"、"&"，位域操作符"<<"、"^"等双目操作符的前后加空格。

    if (currentTime >= MAX_TIME_VALUE)
    a = b + c;
    a *= 2;
    a = b ^ 2;
    

3.  “!”、"~"、"++"、"–"、"&"（地址运算符）等单目操作符前后不加空格。

    *p = 'a';             /* 内容操作"*"与内容之间 */
    flag = !isEmpty;     /* 非操作"!"与内容之间 */
    p = &mem;            /* 地址操作"&" 与内容之间 */
    i++;                  /* "++","--"与内容之间 */
    

4.  “->”、"."前后不加空格。

    p->id = pid;     /* "->"指针前后不加空格 */
    

5.  if、for、while、switch等与后面的括号间应加空格，使if等关键字更为突出、明显。

    if (a >= b && c > d)
    while (b)
    

6.  长语句中，局部可以不加空格，以保证程序整体结构清晰。

    getMessage(buf， mem， 1+n+m*q);
    

◇◇◇

空行、空格、缩进、换行，一些列形式化的要求，不仅仅让代码看起来更舒服，也通过这种形式化的手段，去持续优化提升大家的程序技能。

[返回目录](https://blog.csdn.net/zhangmalong/article/details/103197670)