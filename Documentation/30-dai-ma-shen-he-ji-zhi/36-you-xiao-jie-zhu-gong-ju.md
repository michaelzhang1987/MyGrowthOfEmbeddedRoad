3.6 有效借助工具
==========

原创 [小马儿2019](https://me.csdn.net/zhangmalong) 最后发布于2020-03-06 09:42:24 阅读数 48 收藏

发布于2020-03-06 09:42:24

[](http://creativecommons.org/licenses/by-sa/4.0/)版权声明：本文为博主原创文章，遵循 [CC 4.0 BY-SA](http://creativecommons.org/licenses/by-sa/4.0/) 版权协议，转载请附上原文出处链接和本声明。

本文链接：[https://blog.csdn.net/zhangmalong/article/details/104689446](https://blog.csdn.net/zhangmalong/article/details/104689446)

我们先来回顾前四条规范中的一些约定，如下：

1.  约定.h和.c文件的整体风格；
2.  所有的正式注释仅允许使用“/\* */”风格，其中两边各增加一空格，用于凸显注释内容；
3.  调试时大块注释使用//@风格；
4.  函数使用特定的注释风格，包含完整和简化两种；
5.  ……

有没有感觉这些约定会让你很为难。如其中第二条，注释风格为“/\* 注释内容 */”，就有点找虐的感觉。一次两次无所谓，多几次就该骂娘了，特别在注释是中文且用source insight编辑器时，“/”不是输成了“、”，就是在切换输入法时引入各种乱码。

**不舒服的约定一定是无效的约定**，如何克服呢？实际上，我们经常使用的软件都可以进行自定义功能扩展，如在source insight中，编写如下的一条代码，并将其关联到一个快捷键，就可以快速的完成“/* */”输入。

    //插入"/*  */"注释
    macro placeRemark()
    {
    	//获取当前文档状态
    	hFileBuf = GetCurrentBuf()
    	if (hFileBuf == 0) {
    		msg("没有打开的文件")
    		stop
    	}
    
    	//输出C89注释
    	SetBufSelText(hFileBuf, "/*  */")
    	
    	//光标定位
    	nSel = GetWndSel(GetCurrentWnd())
    	nSel.ichFirst = nSel.ichFirst - 3
    	nSel.ichLim = nSel.ichLim - 3
    	SetWndSel(GetCurrentWnd(), nSel)
    }
    

如果是微软的MSVC，可以如下实现，然后继续关联快捷键：

    '插入/*  */注释
    sub placeRemark()
    	ActiveDocument.Selection.Text = "/*  */"
    	ActiveDocument.Selection.CharLeft dsMove, 3
    End Sub
    

目前，大多数开发工具都支持用户自定义扩展功能，如大家能充分挖掘，甚至会发现各种彩蛋！

◇◇◇

在文件全局观一节中，我们约定.h和.c文件都有统一的文件头描述，.h文件额外还有防止重复包含宏和extern “C”{约定，如何通过用户自定义扩展功能自动生成呢？

在source insight中，实现代码如下：

    //增加文件描述
    macro AddFileHeadDescription()
    {
    	//获取当前文档
    	hFileBuf = GetCurrentBuf()
    	if (hFileBuf == 0) {
    		msg("没有打开的文件")
    		stop
    	}
    
    	//跳转到文件头,增加描述信息
    	szPathName = GetBufName(hFileBuf)
    	nIndex = strLen(szPathName) - 1
    	while (nIndex > 0) {
    		if(szPathName[nIndex] == CharFromAscii(92)) break
    		nIndex = nIndex - 1
    	}
    	szFileName = strmid(szPathName, nIndex + 1, strlen(szPathName))
    	CurTime = GetSysTime(true);
    	szCurTime = CurTime.year # "-" # CurTime.month # "-" # CurTime.day
    	SetBufIns(hFileBuf, 0, 0);
    	InsBufLine(hFileBuf, 0, "/*************************************")
    	InsBufLine(hFileBuf, 1, "* ")
    	InsBufLine(hFileBuf, 2, "*   Copyright (C), 2000-2020, anotherWay. Co., Ltd.")
    	InsBufLine(hFileBuf, 3, "* ")
    	InsBufLine(hFileBuf, 4, "*   文件名称: " # szFileName)
    	InsBufLine(hFileBuf, 5, "*   版    本: 1.0")
    	InsBufLine(hFileBuf, 6, "*   生成日期: " # szCurTime)
    	InsBufLine(hFileBuf, 7, "*   作    者: 小马儿")
    	InsBufLine(hFileBuf, 8, "*   所属模块: <...>")
    	InsBufLine(hFileBuf, 9, "*   功    能: <...>")
    	InsBufLine(hFileBuf, 10, "* ")
    	InsBufLine(hFileBuf, 11, "*************************************/")
    	InsBufLine(hFileBuf, 12, "")
    	
    	//如果为.h文件,增加防止重复编译宏
    	if (szFileName[strlen(szFileName) - 1] == "h" || szFileName[strlen(szFileName) - 1] == "H") 
    	{
    		InsBufLine(hFileBuf, 13, "#ifndef __" # strleft(szFileName, strlen(szFileName) - 2) # "_H")
    		InsBufLine(hFileBuf, 14, "#define __" # strleft(szFileName, strlen(szFileName) - 2) # "_H")
    		InsBufLine(hFileBuf, 15, "")
    		InsBufLine(hFileBuf, 16, "#ifdef __cplusplus")
    		InsBufLine(hFileBuf, 17, "extern " # CharFromAscii(34) # "C" # CharFromAscii(34) # "{")
    		InsBufLine(hFileBuf, 18, "#endif")
    		InsBufLine(hFileBuf, 19, "")
    	}
    	
    	//如果为.h文件,增加防止重复编译宏
    	nLineCount = GetBufLineCount(hFileBuf)
    	SetBufIns(hFileBuf, nLineCount - 1, 0);
    	if (szFileName[strlen(szFileName) - 1] == "h" || szFileName[strlen(szFileName) - 1] == "H") 
    	{
    		InsBufLine(hFileBuf, nLineCount + 0, "")
    		InsBufLine(hFileBuf, nLineCount + 1, "/**********************/")
    		InsBufLine(hFileBuf, nLineCount + 2, "#ifdef __cplusplus")
    		InsBufLine(hFileBuf, nLineCount + 3, "}")
    		InsBufLine(hFileBuf, nLineCount + 4, "#endif")
    		InsBufLine(hFileBuf, nLineCount + 5, "")
    		InsBufLine(hFileBuf, nLineCount + 6, "#endif /* __" # strLeft(szFileName, strLen(szFileName) - 2) # "_H */")
    		InsBufLine(hFileBuf, nLineCount + 7, "")
    		stop
    	}
    }
    

在编程细节观一节中，我们约定调试过程中的临时大块使用//@风格注释。单纯看这条约定，会感觉比较诡异，为何用//@而非//呢。如果能结合工具，就比较容易理解了。

调试时经常需要大段注释代码，因为原代码中夹杂着很多/\* \* /注释，没法使用/\* \* /风格，只能使用//风格。困难的是如果代码段很大，使用//风格也是一件头疼的事情，因此最好的策略就是增加自定义扩展功能。而使用//@风格，便于扩展功能取消注释时定位，也可防止意外的取消原有注释。

伪代码示意如下：

    //注释选择文本，或取消当前光标处所在注释块
    macro TogCommentSel()
    {
    	//获取当前文档状态
    	…
    
    	//获取当前选择文本
    	…
    
    	//未选择代码时
    	if ()
    	{
    		//从当前行开始向后取消//@注释
    		…
    
    		//从当前行开始向前取消//@注释
    		…
    
    		//结束
    		stop
    	}
    
    	//有选择代码时
    	else 
    	{
    		//如果当前行有//@风格注释，则执行取消注释操作
    		if () {
    			先前取消注释
    			向后取消注释
    			stop
    		}
    
    		//否则注释当前选择代码
    		…
    	}
    }
    

同理，借助各种自定义功能，不仅让我们的约定成为最舒服的操作姿态，而且还能挖掘出很多彩蛋，如：

1.  生成约定风格的函数注释，且支持依据.h文件和.c文件分别生成完整或简化注释风格；
2.  生成代码分段符；
3.  在当前光标处放置时间；
4.  在当前光标处放置常用匈牙利命名法定义及前缀，这样可以省去好多大小混写输入；
5.  将//风格的注释修改为/\* */风格，且支持大块注释修改；
6.  选择一个函数的全部内容；
7.  查找一个结构定义中的未对齐空穴，并增加相应变量补齐；
8.  IAR软件自带MISRA-C规范，不妨试着用起来；
9.  ……

因为未增加额外约定，内部交流时我们喜欢将“有效借助工具”这条规范称之为半条规范，合并起来统称“四条半”规范。虽然只有半条，但实际上没有这些扩展功能支撑，前面的一些约定也很难实施。当然，切忌过犹不及，我们始终未实现“选择一块代码，并按照规范自动插入空格”的扩展功能，就是为了防止颠覆规范背后的意义。

◇◇◇

与自定义功能相关联的，就是快捷键了。在多年的实践活动中，我们发现，如果团队内部能统一快捷键，非常有利于团队互相交流学习。

我在跨国企业的工作经历中发现，有两种很高效的带人策略：**让别人看着你写程序，或者盯着别人写程序，这都属于现地现物策略的一环**。实践中，很多新人会被我盯的毛骨悚然，在他们的电脑上演示时，经常因快捷键不一致而引发各种尴尬。因此，慢慢的，我们团队开始统一快捷键。

很多快捷键可以保存，如source insight软件，不仅快捷键，甚至各种显示风格设定、自定义功能等都可以保存到配置文件中。这种特性便于团队进一步统一编辑器。

不仅仅是快捷键值得统一，我们团队发现在研发活动中，如果能统一开发环境设置、项目路径等，也非常有助于团队代码审核。

[返回目录](https://blog.csdn.net/zhangmalong/article/details/103197670)