5.1 模块化，知易行难
============

原创 [小马儿2019](https://me.csdn.net/zhangmalong) 最后发布于2020-04-02 23:09:47 阅读数 13 收藏

发布于2020-04-02 23:09:47

文章标签： [c语言](https://so.csdn.net/so/search/s.do?q=c语言&t=blog) [程序设计](https://so.csdn.net/so/search/s.do?q=程序设计&t=blog) [嵌入式](https://so.csdn.net/so/search/s.do?q=嵌入式&t=blog)

[](http://creativecommons.org/licenses/by-sa/4.0/)版权声明：本文为博主原创文章，遵循 [CC 4.0 BY-SA](http://creativecommons.org/licenses/by-sa/4.0/) 版权协议，转载请附上原文出处链接和本声明。

本文链接：[https://blog.csdn.net/zhangmalong/article/details/105281958](https://blog.csdn.net/zhangmalong/article/details/105281958)

从入职到架构师，我们在逐级台阶慢慢往上爬，入职——团队——抽象——复用——架构——标准化，现在我们又迈上了一个新的台阶：模块化和抽象层。

学习软件编程时，老师会告诉我们软件模块接口要清晰，要追求低耦合强内聚，这样，整个程序才能具备易扩展易维护的特点。

可惜理想很丰满，现实很骨感。在真实产品中，我们会发现软件模块初期还能相对清晰，但几次迭代下来，模块之间就会冒出很多相互调用，用不了多久，模块化就名存实亡了。

此时，经常会出现一种现象：我们增加或修改一些功能时，需要修改多处代码，即使小心翼翼，也容易犯错。我早期工作经历，就经常会面对刚发布的程序就被反馈一堆问题的尴尬。这种情况进一步恶化，会导致某些产品只能由特定老人维护。新人上不了手，老人脱不开身，是这种困局最生动的写照。

为何会出现这种现象呢？思考其背后的原因，我认为最主要的原因有两点：

1.  程序本身有强耦合特性，如果不持续通过外力纠偏，随着需求增加，会自动长成一团。
2.  真实工作中，任务往往是紧急的，需要快速解决。此时，程序员喜欢使用hack大法，怎么快怎么来，或者怎样容易怎么来。但hack大法很多情况下是不充分的，程序在修修补补中就凌乱了。

如何克服这两点呢，我们团队多年迭代出两条关键的策略：

1.  持续积累模块化技巧。手里有粮，心中不慌，大画家随意的几根线条都是精品，同理，胸中有丰富的模块化策略，就自然的不乐意使用hack大法了。
2.  将模块接口作为审核标的物，借助团队和流程的力量，不仅可减少hack操作，而且可提升模块抽象层质量。

◇◇◇
---

模块化的关键是对接口的抽象，而公共子程序因为抽象容易，调用清晰，是模块化的最佳起点。在我们团队中，公共子程序抽象主要策略如下：

1.  三次原则（rule of three）法则。这句话包含两方面的意思，其一是不要看到个功能函数就想将其提炼成通用函数。其二是一个通用功能被三次复用，就要开始考虑是否因将其移入公共复用库。各项目差异很大，公共子程序库是长出来的。
2.  C语言没有命名空间，通过前缀区分是惯用手法。前缀要由数据字典全局管控，且不适宜分的过细。我们团队内部，所有的公共子程序都使用前缀hlp，代表辅助的意思。
3.  通用子程序在生成的过程中容易成簇，该过程中尤其需要注意版本问题。如crc校验码计算，因为有很多种参数模型，而且有不同的初始值，可能需要构建多种接口。

因为多个模块需要使用，最初我们选择了三种crc校验方式，作为团队内部通用算法，接口函数如下：

    /*
     *  Description: 计算整个缓冲区校验码，8位
     *  Input: 
     *    const void *pBuf: 缓冲区地址
     *    DWORD dwLen: 缓冲区长度
     *  Return: 返回校验码
     */
    DWORD hlpCalcCRC8(const void *pBuf, DWORD dwLen);
    
    /* 计算整个缓冲区校验码，16位 */
    DWORD hlpCalcCRC16(const void *pBuf, DWORD dwLen);
    
    /* 计算整个缓冲区校验码，32位 */
    DWORD hlpCalcCRC32(const void *pBuf, DWORD dwLen);
    

后来，我们发现一些项目中对crc-16进行计算时，需要不同的初值，因此增加了如下接口函数：

    /*
     *  Description: 计算整个缓冲区校验码，16位，带初值
     *  Input: 
     *    DWORD dwInit: 初值
     *    const void *pBuf: 缓冲区地址
     *    DWORD dwLen: 缓冲区长度
     *  Return: 返回校验码
     */
    DWORD hlpCalcCRC16Ex(DWORD dwInit, const void *pBuf, DWORD dwLen);
    

再后来，发现modbus规约使用的算法同我们的默认算法不一致，只好继续增加接口函数：

    /*
     *  Description: 计算整个缓冲区校验码，16位，用于modbus规约
     *  Input: 
     *    DWORD dwInit: 初值
     *    const void *pBuf: 缓冲区地址
     *    DWORD dwLen: 缓冲区长度
     *  Return: 返回校验码
     */
    DWORD hlpCalcCRC16Modbus(DWORD dwInit, const void *pBuf, DWORD dwLen);
    

慢慢的，我们团队内部长出了一簇crc校验公共子程序。一开始长的快，后来就慢慢定型了，终于长成一套通用功能子程序，给后续的项目代码编写带来很大帮助。

◇◇◇
---

在真实产品中，相当比例的模块接口类似于公共子程序，仅需要薄薄的一层抽象，而且都是简单的单向调用。如硬件抽象层中，虽然eeprom型号、大小、速度等差异颇大，但我们很容易构建如下抽象接口：

    /*
     *  Description: 读取EEPROM内容
     *  Input: 
     *    DWORD dwOffset: 偏移位置
     *    BYTE *pBuf: 数据地址
     *    DWORD dwLen: 读取长度
     *  Return: 成功返回TRUE,否则返回FALSE;
     */
    BOOL hwReadEEPROM(DWORD dwOffset, BYTE *pBuf, DWORD dwLen);
    
    /*
     *  Description: 改写EEPROM内容
     *  Input: 
     *    DWORD dwOffset: 偏移位置
     *    const BYTE *pBuf: 数据地址
     *    DWORD dwLen: 改写长度
     *  Return: 成功返回TRUE,否则返回FALSE;
     */
    BOOL hwWriteEEPROM(DWORD dwOffset, const BYTE *pBuf, DWORD dwLen);
◇◇◇
---

产品复杂后，程序一般具备分层特性，如果下层程序接口抽象的比较好，有助于上层模块的复用。如遥测数据模块接口不清晰，应用层规约代码不可能复用。本章后续会通过几个例程帮助大家理解如何构建这类模块的接口层。

当然，还有相当多的模块，同诸多模块存在耦合，如信号复归模块，其触发的源头可能有：装置前面板按键、远方通讯规约、机柜开入信号灯。触发信号复归后，需要息屏、关闭自保持led、清除动作报告状态等。这些因素会导致信号复归功能同很多模块之间强耦合。本章最后一节，我们一起来探讨如何给信号复归模块解耦。

虽然模块化的关键在于构建优秀的接口，但接口是持续迭代中的，不可能一开始就非常完美。因此，不妨先起步，然后再慢慢演化，持续迭代。

——————————————

[返回目录](https://blog.csdn.net/zhangmalong/article/details/103197670)