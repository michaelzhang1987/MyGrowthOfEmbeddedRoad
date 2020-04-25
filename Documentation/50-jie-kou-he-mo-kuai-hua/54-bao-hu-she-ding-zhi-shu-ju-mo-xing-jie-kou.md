5.4 保护设定值数据模型接口
===============

![](https://csdnimg.cn/release/phoenix/template/new_img/original.png)  

[小马儿2019](https://me.csdn.net/zhangmalong) 2020-04-24 12:25:05 ![](https://csdnimg.cn/release/phoenix/template/new_img/articleRead.png) 10 ![](https://csdnimg.cn/release/phoenix/template/new_img/collect.png) ![](https://csdnimg.cn/release/phoenix/template/new_img/tobarCollectionActive.png) 收藏 

最后发布:2020-04-24 12:25:05首发:2020-04-24 12:25:05

文章标签： [嵌入式](https://so.csdn.net/so/search/s.do?q=嵌入式&t=blog) [c语言](https://so.csdn.net/so/search/s.do?q=c语言&t=blog) [程序设计](https://so.csdn.net/so/search/s.do?q=程序设计&t=blog)

版权声明：本文为博主原创文章，遵循 [CC 4.0 BY-SA](http://creativecommons.org/licenses/by-sa/4.0/) 版权协议，转载请附上原文出处链接和本声明。

本文链接：[https://blog.csdn.net/zhangmalong/article/details/105678635](https://blog.csdn.net/zhangmalong/article/details/105678635)

我新换一部智能手机后，一般会忍不住将所有设置菜单功能体验一遍，虽然设置层层叠叠，但能将手机设置成自己喜欢的样子还是颇有成就感的。

类同智能手机，微机保护等工业产品随着各种功能的增加，各种各样的设置也变得繁杂起来。微机保护产品有很多类型的设定值，如遥测越限值，如设备通讯状态等，但其中有一类设定值比较特殊，就是保护功能相关设定值，我们团队习惯称之为保护设定值。

保护设定值用于配置保护功能，如过流保护有两个最基本的设定值：过流设定值ISet和时间设定值t，如下图所示。

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020042212020060.png)  
当电流Ia大于设定值ISet，且保持时间超过t时，过流保护触发，然后驱动出口继电器跳开电力断路器，切除故障。

相比于其他设定值，保护设定值有一些特殊性：

1.  保护设定值必须整体修改，如过流保护将ISet和t两个定值分开逐个修改，就有可能导致保护设备误动作。保护功能是强实时模块，是通过中断触发，而应用模块内定值是一个个依次写入的，如在多个写入之间触发保护逻辑，就有可能误动作。
2.  定值修改过程中需要暂时闭锁保护功能，相当于一段时间内电力设备在无防护的状态下运行，因此必须将该过程压缩到小于设备带故障运行时间（一般以ms计）。但不管是通过液晶修改设定值，还是通过维护软件修改设定值，其过程都比较长。
3.  随着微机保护集成的功能越来越多，设定值数量也在持续增加，不仅给用户带来了使用负担，规约传输时也经常要分多帧传输。
4.  设定值对安全性、完整性等要求很高。

为了保证保护设定值能在短时间内整体写入，必须增加中间缓冲区，且将所有的读写操作都基于缓冲区，最后阶段，在闭锁保护功能的前提下将整个缓冲区整体写入。

假设有一规约对象，需要完成上述功能，大概需要如下几部分代码逻辑：

1.  初始化时刻开辟缓冲区，缓冲区大小可以通过读取设定值个数apiSettingCount()获取。

    DWORD dwSize = sizeof(FLOAT) * apiSettingCount() + 4;		/* +4,存储CRC校验码 */
    FLOAT *pSettingBuf = (FLOAT*)hwMalloc(dwSize, HW_MALLOC_NORMAL);
    

2.  远传设定值不需要用到缓冲区，可以直接基于序号读取。

    DWORD dwCount = apiSettingCount();
    for (i = 0; i < dwCount; i++)
    {
    	*pReadBuf++ = apiSettingGet(i);		/* pReadBuf指向规约远传帧内存空间 */
    }
    

3.  写操作比较麻烦，可能接受到的定值可能不完整（大部分情况下用户仅修改部分设定值），也可能因为改写定值较多需要分多帧传输，而且还需要在定值传输完毕后，将整个缓冲区写入。代码示例如下：

    /* 如果接受是第一帧，首先用默认值填充缓冲区 */
    if (bFirst)
    {
    	DWORD dwCount = apiSettingCount();
    	FLOAT *pBuf = pSettingBuf;
    	for (i = 0; i < dwCount; i++)
    			*pBuf++ = apiSettingGet(i);
    }
    
    /* 将接受到的定值写入缓冲区相应位置 */
    for (;;)
    	...
    
    /* 如果是最后一帧，需要执行定值投入操作 */
    if (bEnd)
    {
    	/* 闭锁保护 */
    	...
    
    	/* 将缓冲区中的定值整体写入 */
    	apiSettingLaunch(pSettingBuf);
    
    	/* 解锁保护闭锁 */
    	...
    }
    

如果是液晶设定值修改界面，基本流程同规约对象，也需要构建缓冲区，唯独读操作存在差异。规约对象的读和写是完全分离的，而通过液晶界面修改定值时，读和写操作都必须基于缓冲区，不然会导致刚写的定值读不出来。

现在来细细分析上述代码示例，我们很容易发现一些不舒服的地方，如：

1.  每个规约和液晶对象都需要分配一块缓冲区，考虑定值和规约数量都比较多，对于内存是稀缺资源的工业嵌入式设备是奢侈的行为。
2.  各应用模块重复一些列关于缓冲区的操作，代码冗余。
3.  同步问题，规约修改缓冲区过程中，液晶可能在同时修改定值。
4.  缓冲区的管理缺失，如果通讯意外中断，原有缓冲区中的脏数据如何处理。

很多问题在别处可能已经有了答案。为了进一步提炼优化保护设定值的接口，我们引入了数据库中的“事务”概念，将对缓冲区的操作全部内化到接口函数内部。此时，关键的几个接口函数如下：

    /*
     *  Description: 启动定值读写事务
     *  Input: 
     *    DWORD dwTimeout: 超时,在超时时间内如没有读写操作,事务自动撤销。
     *  Return: 成功返回TRUE,否则返回FALSE
     *  Others: 为了防止系统锁死，超时有最大限制，不允许无穷等待。为了防止事务被取消，需增加读写操作。
     */
    BOOL apiSettingStart(DWORD dwTimeout);
    
    /*
     *  Description: 读取单个定值
     *  Input: 
     *    DWORD dwIndex: 定值索引
     *  Return: 成功返回定值,否则返回0.0f
     *  Others: 如包含在事务内时,读取事务内定值,否则读取真实定值。
     */
    FLOAT apiSettingGet(DWORD dwIndex);
    
    /*
     *  Description: 改写单个定值
     *  Input: 
     *    DWORD dwIndex: 定值索引
     *    FLOAT fSetting: 改写定值
     *  Return: 成功返回TRUE,否则返回FALSE
     *  Others: 必须包含在事务内,否则失败。写定值内部会进行边界判断，失败返回FALSE。
     */
    BOOL apiSettingSet(DWORD dwIndex, FLOAT fSetting);
    
    /* 定值事务提交 */
    BOOL apiSettingCommit(void);
    
    /* 定值事务撤销 */
    void apiSettingCancel(void);
    

借助于事务概念接口，各应用程序立马简洁了很多，而且事务接口额外提供了一种互斥策略，假如规约正在修改定值，液晶界面想去修改定值，事务请求会直接失败返回。

在微机保护装置内部，很多简单模块的接口提炼过程都非常类似保护设定值，先分析汇总各应用模块相关功能流程，然后“切”出最佳边界。当然需求可能会持续变化，接口也会持续成长，然后慢慢收敛并稳定下来。

——————————————

[返回目录](https://blog.csdn.net/zhangmalong/article/details/103197670)