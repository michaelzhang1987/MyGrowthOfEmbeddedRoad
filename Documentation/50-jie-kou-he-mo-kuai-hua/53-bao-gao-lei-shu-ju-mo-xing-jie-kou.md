5.3 报告类数据模型接口
=============

![](https://csdnimg.cn/release/phoenix/template/new_img/original.png)  

[小马儿2019](https://me.csdn.net/zhangmalong) 2020-04-17 13:09:24 ![](https://csdnimg.cn/release/phoenix/template/new_img/articleRead.png) 28 ![](https://csdnimg.cn/release/phoenix/template/new_img/collect.png) ![](https://csdnimg.cn/release/phoenix/template/new_img/tobarCollectionActive.png) 收藏 

最后发布:2020-04-17 13:09:24首发:2020-04-17 13:09:24

文章标签： [嵌入式](https://so.csdn.net/so/search/s.do?q=嵌入式&t=blog) [c语言](https://so.csdn.net/so/search/s.do?q=c语言&t=blog) [程序设计](https://so.csdn.net/so/search/s.do?q=程序设计&t=blog)

版权声明：本文为博主原创文章，遵循 [CC 4.0 BY-SA](http://creativecommons.org/licenses/by-sa/4.0/) 版权协议，转载请附上原文出处链接和本声明。

本文链接：[https://blog.csdn.net/zhangmalong/article/details/105578239](https://blog.csdn.net/zhangmalong/article/details/105578239)

在继保的各种数据模型中，有一类数据模型其组织方式非常类似，包含遥信变位、动作报告、操作事项、遥测越限等，都是多条条目的方式组织。我们团队习惯将这类数据模型称之为报告类数据模型。

在嵌入式应用领域，这类信息一般存储在flash或eeprom等非易失性存储芯片中，各条目信息组织结构一致，并在产品构建初期就约定好存储条数、空间和位置等信息。

报告类数据模型，一般采取环形缓冲区数据结构，如果缓冲区满且有新报告时，会覆盖最旧的条目。环形缓冲区是最基本的数据结构，有两个指针用于读写，并通过浪费一个存储空间来区分缓冲区满和空。

针对报告类数据模型，如何构建抽象接口层呢？

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200417130653806.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW5nbWFsb25n,size_16,color_FFFFFF,t_70)  
类同AI，为了提炼报告类模型接口，需要先分析汇总各类应用需求。报告数据模型接口抽象的关键是访问接口，需求汇总如下：

1.  远传类规约，需要检测是否有新报告，如有新报告时读取并远传，并执行确认操作以防止重复读取。
2.  远传类规约，运行后需从最旧的报告开始上传。
3.  远传类规约，在长期未运行且报告被覆盖后，恢复运行时从最旧有效报告开始上传。
4.  液晶界面或维护软件类应用，需要查询目前有多少条报告，并支持正序或逆序逐条读取报告。
5.  各应用任务运行独立，隐藏互斥信号量等内部数据结构。

为了实现这些繁杂的报告访问功能，最常见的策略是为每个应用任务构建一个单独的访问指针，如下图示意：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200417130715298.png)  
但这种接口模式存在诸多问题：

1.  访问指针是位于应用任务空间呢，还是位于报告数据模型空间？如位于应用任务空间，当缓冲区覆盖后报告模块如何去调整访问指针呢（需求4）。如位于报告模型空间，各应用该用哪一个呢（需求5）。
2.  将读写指针暴露给应用任务，也就将同步互斥等相关工作暴露给了应用任务（需求5）。
3.  ……

为了满足报告需求，我们需要额外增加一层抽象层，并引入“句柄”概念。在计算机领域，句柄可以理解为一个控制锚点，只要拥有了句柄，就可以执行一系列操作，而不需要去关心句柄背后是什么。

各应用任务对报告进行访问时，首先需申请控制句柄，接口函数如下：

    /*
     *  Description: 注册报告访问句柄
     *  Input: 
     *    DWORD dwMsgID: 接受新报告消息ID, -1表示不接受消息
     *  Return: 成功返回报告访问句柄,否则返回NULL
     *  Others: 
     */
    HANDLE apiReportRegister(DWORD dwMsgID);
    

报告是非常重要的信息，远传类规约需要将报告信息不重复不遗漏且及时的传送出去。此时，远传类规约获取报告一般有两种模式：查询或消息。消息模式指每发现一个新报告时，应用任务会自动收到一条消息，然后在消息函数中完成相应处理。为了能接受到消息，需要在开始注册时传递消息ID。

如果不接受消息，就需要应用任务主动定时查询，接口函数如下：

    /*
     *  Description: 检测是否有新报告
     *  Input: 
     *    DWORD dwBrowse: 报告访问句柄
     *  Return: 有新报告返回TRUE,否则返回FALSE
     */
    BOOL apiReportHaveNew(DWORD dwBrowse);
    

调用该接口函数时，需要传递注册函数返回的控制句柄，实际上在报告接口中，所有的访问类接口函数，都需要传递控制句柄。

如果检测到有新报告，基本就是读取操作了，接口函数如下：

    /*
     *  Description: 读取报告
     *  Input: 
     *    DWORD dwBrowse: 报告访问句柄
     *  Output: 
     *    TAPIReport *pReport: 读取报告，由应用层负责内存空间
     *  Return: 成功返回TRUE,否则返回FALSE
     */
    BOOL apiReportGet(DWORD dwBrowse, TAPIReport *pReport);
    

不同于属性类数据结构，为了节约eeprom或flash资源，报告数据存储经常使用压缩方式。为了应用模块使用更方便，一般使用上述接口模式。

报告类数据模型不仅要保证尽快将报告不遗漏的传送出去，而且也不能重复传送，因此，每次应用任务确认传输成功后，需要一个确认操作，确认操作实际上是调整访问指针，接口函数如下：

    /* 报告指针调整方式 */
    #define API_REPORT_ADJUST_FORWARD	0	/* 正向调整 */
    #define API_REPORT_ADJUST_REVERSE	1	/* 反向调整 */
    
    /*
     *  Description: 调整报告指针
     *  Input: 
     *    DWORD dwBrowse: 报告访问句柄
     *    DWORD dwType: 报告指针调整方式
     *  Return: 成功返回TRUE,否则返回FALSE
     */
    BOOL apiReportAdjust(DWORD dwBrowse, DWORD dwType);
    

调整报告指针有两个方向，正向调整或反向调整，报告上传确认为正向调整，反向调整用于报告遍历。

至此，远传类规约需要的接口已经构建完毕，但还有一类访问需求，如通过液晶界面查询报告，期望能以列表方式从新到旧（或从旧到新）的显示所有报告。这类操作相当于是一个遍历操作，首先需要将访问指针复位到最新（或最旧）的位置，然后遍历读取。复位操作接口如下：

    /* 报告指针复位方式 */
    #define API_REPORT_RESET_OLD	0	/* 复位到最旧一条报告 */
    #define API_REPORT_RESET_NEW	1	/* 复位到最新一条报告 */
    
    /*
     *  Description: 复位报告指针
     *  Input: 
     *    DWORD dwBrowse: 报告访问句柄
     *    DWORD dwType: 报告指针复位方式
     *  Return: 成功返回TRUE,否则返回FALSE
     */
    BOOL apiReportReset(DWORD dwBrowse, DWORD dwType);
    

先进行指针复位操作，然后就可以通过apiReportAdjust()和apiReportGet()两个接口函数遍历读取所有报告信息了。

除了上述访问类接口函数，报告模块还需要一些功能，如获取报告个数信息，清除所有报告，写报告等诸多接口。报告接口函数的难点在于访问接口，限于篇幅，这些函数就不一一赘述了。

◇◇◇

对应用层，句柄是一个抽象概念，但在报告模块内部，句柄到底是什么呢？实际上，句柄就是访问指针的抽象化。具体实现可以有很多种策略，如每次注册时new一个访问指针对象，并用链表穿起来，由报告模块统一管理。或者在报告模块内部预先开辟一足够大访问指针对象数组，每次注册时返回数组索引。当然也可以在句柄中增加安全验证机制，以提升程序鲁棒性。

针对报告类数据模型，且考虑应用模块数量有限且相对固定，我们团队内部喜欢将句柄抽象为“检测flag+预分配数组下标”模式，此时，每次注册过程相当于返回一数组下标。

通过这些接口函数，不仅隐藏了各种同步互斥操作，而且可以有效的避免各种指针紊乱情况。如某备用远传规约长期处于停止状态，报告缓冲区已经被覆盖了多次，假设现在备用远传规约开始进入工作状态，此时它的访问指针就会出现紊乱。为了避免该异常，最佳策略就是让写指针赶着访问指针往前走。

写报告伪代码示例如下：

    void apiReportSet(...)
    {
    	/* 获取读写互斥锁 */
    	...
    
    	/* 保存报告 */
    	...
    
    	/* 写指针追赶读指针前行，覆盖最旧一条记录 */
    	...
    
    	/* 写指针追赶所有有效访问指针前行 */
    	for (...)
    	{
    		...
    	}
    
    	/* 释放读写互斥锁 */
    	...
    
    	/* 给注册的应用任务发送消息 */
    	for (...)
    	{
    		...
    	}
    }
    

——————————————

[返回目录](https://blog.csdn.net/zhangmalong/article/details/103197670)