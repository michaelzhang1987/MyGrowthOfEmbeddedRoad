5.2 AI数据接口
==========

原创 [小马儿2019](https://me.csdn.net/zhangmalong) 最后发布于2020-04-10 12:58:25 阅读数 5 收藏

发布于2020-04-10 12:58:25

文章标签： [程序设计](https://so.csdn.net/so/search/s.do?q=程序设计&t=blog) [c语言](https://so.csdn.net/so/search/s.do?q=c语言&t=blog) [嵌入式](https://so.csdn.net/so/search/s.do?q=嵌入式&t=blog)

[](http://creativecommons.org/licenses/by-sa/4.0/)版权声明：本文为博主原创文章，遵循 [CC 4.0 BY-SA](http://creativecommons.org/licenses/by-sa/4.0/) 版权协议，转载请附上原文出处链接和本声明。

本文链接：[https://blog.csdn.net/zhangmalong/article/details/105431091](https://blog.csdn.net/zhangmalong/article/details/105431091)

本小节，我们尝试从一个相对简单的模块出发，尝试提炼接口抽象函数，而在基本数据模型中，模拟量输入AI模块就是最基础的数据模型。

欲构建AI模块抽象接口，需先进行需求分析。AI是基础数据模型，不仅是各种高级数据模型的基础，也是各种应用模块的基础，如规约、液晶等。汇总后AI的需求主要包括：

1.  初始化。
2.  AI值，包括读写操作。
3.  属性，包括个数、名称、描述、单位、精度等属性值。
4.  取代操作。
5.  遥测越限等高级应用。

◇◇◇

除了没有状态量的简单模块，大部分模块都需要一个初始化接口函数，示例如下：

    /* AI模块初始化 */
    BOOL apiAIInit(void);
    

不同于其他接口，apiAIInit接口函数一般不是由应用层直接调用的，大概率是被api层统一的初始化接口函数调用的，这表明apiAIInit接口函数是一层内部函数。

还记得我们在文件全局观中的约定吗？一个模块对应3个文件，因此AI模块对应ai.c，ai.h和aiPrivate.h三个文件，而apiAIInit接口函数就应该位于aiPrivate.h文件中。

◇◇◇

AI模块最基本的操作是读取单个AI值，此时有两种命名风格：apiGetAI和apiAIGet，哪一种风格好呢？apiGetAI阅读起来会更加自然一些，但apiAIGet命名规范隐含面向对象思维，且程序输入更友好，也便于记忆。因此，我们团队习惯使用“名词+动词”命名风格。

读取AI值，如果仅从字面翻译，接口函数应该命名为apiAIRead(…)。但如果全部遵从最佳文学用词，可能仅基本数据模型就会有很多组动词。这种命名习惯会增加大家的编码和记忆难度，我们团队内部反复协商后，最后大家一致同意采取统一的中性动词对：get/set。

一些类库命名接口函数时，为了减少编码键入工作量，喜欢将最常使用的接口函数压缩到最短，如将读AI压缩为apiAI(…)。这种策略一则需要决定哪个函数最常用，二则增加接口函数记忆负担，额外考虑目前接口函数编码一般都是借助自动输入工具，我们团队不允许使用这种命名策略。

AI接口函数，对外提供接口，对内功能封装，其中很重要的一点就是对互斥锁的封装。AI读写存在互斥，但很多初级程序员难以驾驭互斥信号量，因此最佳策略就是将互斥锁操作封装在AI接口函数内部。这样，仅需要少量高级程序员编写AI模块，而大量的应用功能可以由初级程序员完成，提高人力资源利用率。

目前，基于cortex-m核的单片机都已经是32位的，其中M4核的单片机还支持单精度浮点数运算，这些不仅为微机保护设备研发带来了诸多便利，也将计算能力提升到了32位，也即常规的单精度浮点数操作可以单指令内完成，单个读写之间不存在互斥了。顺势而为，我们将AI的数据类型约定为单精度浮点数。汇总前面的所有分析，读取单个AI值的接口函数如下：

    /*
     *  Description: 读取单个AI值
     *  Input: 
     *    DWORD dwIndex: AI值索引
     *  Return: 返回AI值，异常时返回0.0f
     */
    FLOAT apiAIGet(DWORD dwIndex);
    

规约或液晶应用一般都是批量读取AI值，示例代码如下：

    void lcdDemo(void)
    {
    	DWORD i;
    	DWORD dwCount;
    
    	/* 读取所有AI值 */
    	dwCount = apiAICount();
    	for (i = 0; i < dwCount(); i++)
    	{
    		... = apiAIGet(i);
    	}
    }
    

上面这段例程，在读取AI过程中，可能存在写入AI操作，因此会导致AI值不同步。但因为AI值都是瞬时量，即使存在不同步现象，对大部分应用模块是没有影响的，因此是没有问题的。

另一些应用场合，期望批量读取AI值，此时可以构建如下接口函数：

    /*
     *  Description: 批量读取AI值
     *  Input: 
     *    DWORD dwStartIndex: 起始AI值索引
     *    DWORD dwCount: 读取个数
     *  Output: 
     *    FLOAT *pAIBuf: 存储AI缓冲区,由应用层保证缓冲区空间足够
     *  Return: 成功返回TRUE,否则返回FALSE
     */	
    BOOL apiAIGetMulti(DWORD dwStartIndex, DWORD dwCount, FLOAT *pAIBuf);
    

此时，批量接口程序不仅可以简化部分应用层功能，而且可以内建互斥机制，保证在整个读取过程中没有写入操作。

◇◇◇

AI值的属性包括个数、名称、描述、单位、精度等，主要用于液晶或维护软件等应用模块，因为是常量，因此只有读操作。

AI个数信息接口比较简单，如下：

    /* 读取AI个数 */
    DWORD apiAICount(void);
    

读取属性值的接口函数经常有两种形式，如下：

    1.	TAIProp* apiAIProp(…);
    2.	BOOL apiAIProp(TAIProp* …);
    

第一种类型接口特点如下：

1.  可以将属性结构放置与flash空间并直接返回，以节约珍贵的堆栈资源。
2.  大概率没有拷贝操作，执行效率高。
3.  失败时返回NULL，仅支持通过NULL进行合法判断，无法容纳其他异常信息。

第二种类型接口特点如下：

1.  需要应用层额外开辟堆栈空间，且因为存在拷贝操作，执行效率比较低。
2.  实现灵活，可增加各种额外逻辑。如flash中是压缩格式的属性结构，拷贝到ram中后可执行解压缩操作。
3.  返回值可容纳其他异常信息。

两种策略各有优缺点，在我们团队中，如可能会尽量优选第一种策略，主要考虑如下几点：

1.  嵌入式系统中，相对flash，ram资源更加紧张，分配给堆栈的空间也比较有限，不希望在堆栈中有过多的结构定义。
2.  执行效率比较高。
3.  有配置工具的支持，可以提前生成嵌入式程序恰好使用的数据结构，不仅降低了属性结构二次加工的必要性，而且极大的提升了程序执行效率。该特性会在配置软件相关章节进一步阐述。
4.  碰到少许需要进行属性结构重构的情况，可以增加中间static变量或全局变量来实现，此时需要谨慎多任务互斥。

综述，读取AI属性接口函数如下：

    /*
     *  Description: 读取AI属性
     *  Input: 
     *    DWORD dwIndex: AI索引
     *  Return: 成功返回AI属性,否则返回NULL
     */
    TAIProp* apiAIProp(DWORD dwIndex);
    

◇◇◇

在工程应用中，经常需要AI值取代操作，如在规约对点测试时，希望逐一修改AI值并测试通讯远传是否成功。

取代操作在提升工程友好性的同时，也会带来一些问题，很多系统问题就是因为取代后忘记返回而导致的。因此，在设计取代接口函数时，需要从内部规避。

构建取代接口函数时，有两种策略：

1.  单一写操作，写入时立即进入取代状态，超时后自动返回。
2.  构建额外的取代进入和取代返回接口函数。

这两种策略各有优缺点，我们团队习惯采用第二种策略，主要考虑如下几点：

1.  api层很多类似的模块也采用start/commit/cancel类接口，保持命名风格统一。
2.  单独的start接口，不仅可以统一超时时间设定，而且可以增加额外功能。
3.  cancel函数可以快速退出取代状态，而单一写函数只能等超时了。

此时取代相关接口函数定义如下：

    /* 执行取代操作后,AI值赋值类型 */
    #define API_AI_REPLACE_ZERO			0		/* 全0.0f值 */
    #define API_AI_REPLACE_CURRECT		1		/* 当前值,类似冻结效果 */
    #define API_AI_REPLACE_BUF			2		/* 上一次取代后缓存值 */
    
    /*
     *  Description: 进入AI取代状态
     *  Input: 
     *    DWORD dwType: 执行取代操作后,AI值赋值类型
     *    DWORD dwTimeout: 超时,在该时间内没有取代操作自动退出取代状态
     *  Return: 成功返回TRUE,否则返回FALSE
     */
    BOOL apiAIReplaceStart(DWORD dwType, DWORD dwTimeout);
    
    /* 退出取代状态 */
    void apiAIReplaceCancel(void);
    

注：取代的英文术语是substitute for，很多文献中经常缩写为sub，但为了同减（常缩写为sub）区分，我们团队在数据字典中约定取代的英文使用replace一词。

◇◇◇

不知大家有没有意识到，我们前面定义的取代操作是针对所有AI值的。也即，假如目前进入了取代状态，所有AI值都是取代值。

基于AI有一个高级功能叫遥测越限，可以将AI值转化为报警事件，如电压超过120V报警等。此时，如果我们采用类同取代的策略，将遥测越限内化到AI模块内部是否合适呢？

此时，首先每个AI值需要增加八个属性：越上限、越上上限、越下限、越下下限，以及另外四个使能选项。一般情况下，AI值较多，但只需要对很少的AI值进行遥测越限观察。采用这样的策略后，会导致记录大量不必要的属性和越限信息，浪费资源。因此，遥测越限不应该内嵌在AI模块内部。

类似的情况还有很多。一次，我们的设备需要增加一103规约，传输时需要知道每个AI值的满度值，我们顺手就在AI属性中增加了一个满度值属性，后来才发现这是噩梦的开始。另一些规约需要传输归一值怎么办，再一些需要判断AI值边界怎么办，……

经历了诸多的迭代反复，我们终于明白一点：该是规约的属性，单独构建，应该尽量保持AI属性的简洁。

是否将某功能内嵌到AI内部，还是单独提炼，这属于AI模块边界定位的问题。没有统一的答案，只能具体情况具体分析，一般大多数AI值都需要的功能建议内嵌，复杂的或仅涉及少量AI值的功能单独提炼。如遥测越限虽然单独提炼，有可能AI值合法判断就是内嵌，此时每个AI值需要增加范围属性了。

◇◇◇

至此，我们已经完成了AI接口的所有提炼抽象，完整代码示例如下。要注意，这个版本并非是最终版本，后期结合配置软件后，写操作还会进一步的迭代优化。

---------aiPrivate.h------------

    /* AI模块初始化 */
    BOOL apiAIInit(void);
    

---------ai.h------------

    /*
     *  Description: 读取单个AI值
     *  Input: 
     *    DWORD dwIndex: AI值索引
     *  Return: 返回AI值，异常时返回0.0f
     */
    FLOAT apiAIGet(DWORD dwIndex);
    
    /*
     *  Description: 改写单个AI值
     *  Input: 
     *    DWORD dwIndex: AI值索引
     *    FLOAT fAI: AI值
     *  Return: 成功返回TRUE,否则返回FALSE
     */
    BOOL apiAISet(DWORD dwIndex, FLOAT fAI);
    
    /*
     *  Description: 批量读取AI值
     *  Input: 
     *    DWORD dwStartIndex: 起始AI值索引
     *    DWORD dwCount: 读取个数
     *  Output: 
     *    FLOAT *pAIBuf: 存储AI缓冲区,由应用层保证缓冲区空间足够
     *  Return: 成功返回TRUE,否则返回FALSE
     */
    BOOL apiAIGetMulti(DWORD dwStartIndex, DWORD dwCount, FLOAT *pAIBuf);
    
    /*
     *  Description: 批量改写AI值
     *  Input: 
     *    DWORD dwStartIndex: 起始AI值索引
     *    DWORD dwCount: 写入个数
     *    const FLOAT *pAIBuf: 写入AI值
     *  Return: 成功返回TRUE,否则返回FALSE
     */
    BOOL apiAISetMulti(DWORD dwStartIndex, DWORD dwCount, const FLOAT *pAIBuf);
    
    /* 读取AI个数 */
    DWORD apiAICount(void);
    
    /*
     *  Description: 读取AI属性值
     *  Input: 
     *    DWORD dwIndex: AI索引
     *  Return: 成功返回AI属性,否则返回NULL
     */
    TAIProp* apiAIProp(DWORD dwIndex);
    
    /* 执行取代操作后,AI值赋值类型 */
    #define API_AI_REPLACE_ZERO			0		/* 全0.0f值 */
    #define API_AI_REPLACE_CURRECT		1		/* 当前值,类似冻结效果 */
    #define API_AI_REPLACE_BUF			2		/* 上一次取代后缓存值 */
    
    /*
     *  Description: 进入AI取代状态
     *  Input: 
     *    DWORD dwType: 执行取代操作后,AI值赋值类型
     *    DWORD dwTimeout: 超时,在该时间内没有取代操作自动退出取代状态
     *  Return: 成功返回TRUE,否则返回FALSE
     */
    BOOL apiAIReplaceStart(DWORD dwType, DWORD dwTimeout);
    
    /* 退出取代状态 */
    void apiAIReplaceCancel(void);
——————————————

[返回目录](https://blog.csdn.net/zhangmalong/article/details/103197670)