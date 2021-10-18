---
title: TBQUANT
date: 2021-05-22 10:55:31
tags: 投资
category: 投资
---
## 输出
 	PlotString("BarStatus",Text(BarStatus));


## 定义参数

整个公式中只能出现一个 Params 宣告，并且要放到公式的开始部分，在变量定义之前

    Params
 	Bool bTest(False); //定义布尔型参数 bTest，默认值为 False;
 	Numberic Length(10); //定义数值型参数 Length，默认值为 10；
 	NumericSeries Price(0); //定义数值型序列参数 Price，默认值为 0；
 	NumericRef output(0); //定义数值型引用参数 output，默认值为 0；
 	String strTmp("Hello"); //定义字符串参数 strTmp,默认值为 Hello

## 定义变量

整个公式中只能出现一个 Vars 声明，并且要放到公式的开始部分，在参数定义之后，正文之前。

	Vars
	NumericSeries MyVal1(0); //定义数值型序列变量 MyVal1，默认值为 0；
	Numeric MyVal2(0); //定义数值型变量 MyVal2，默认值为 0；
	Bool MyVal3(False); //定义布尔型变量 MyVal3，默认值为 False;
	String MyVal4("Test"); //定义字符串变量 MyVal4，默认值为 Test。

## 用户函数编写示例

	Average 函数脚本如下：
	Params
	NumericSeries Price(1);
	Numeric Length(10);
	Vars
	Numeric AvgValue; 
	Begin
	AvgValue = Summation(Price, Length) / Length;
	Return AvgValue;
	End
上面的例子，返回值只有一个，即最后求得的平均值，在函数中直接使用 Return 语句返回就可以
了。

	Params
	NumericSeries Price(1);
	Numeric Length(10);
	NumericRef HighestBar; //设置引用型的变量
	Vars
	Numeric MyVal;TradeBlazer 公式开发指南 TradeBlazer 用户函数
	- 69 -
	Numeric MyBar;
	Numeric i; 
	Begin
	MyVal = Price;
	MyBar = 0;
	For i = 1 to Length – 1
	{
	If ( Price[i] > MyVal)
	{
	MyVal = Price[i];
	MyBar = i; //记录最大值 Bar 与当前 Bar 的偏移量
	}
	}
	HighestBar = MyBar; //通过将偏移量赋值给引用型变量，它可将该值传递回去
	Return MyVal;
	End

	函数参数声明类型 可传入的变量类型
	Numeric== Numeric，NumericRef，NumericSeries
	NumericRe==f Numeric，NumericRef
	NumericSeries== Numeric，NumericRef，NumericSeries
	Bool== Bool，BoolRef，BoolSeriesTradeBlazer
	BoolRef== Bool，BoolRef
	BoolSeries== Bool，BoolRef，BoolSeries
	String== String，StringRef，StringSeries
	StringRef== String，StringRef
	StringSeries== String，StringRef，StringSeries 

## 输出
	PlotNumeric-----在当前 Bar 输出一个数值；
	语法：PlotNumeric(String Name,Numeric Number,Numeric Locator=0,Integer Color=-1,Integer 
	BarsBack=0)
	 Name 输出值的名称的字符串，可以为中、英文、数字或者其它字符；
	 Number 输出的数值；
	 Locator 输出值的定位点，默认时输出单点，否则输出连接两个值线段，用法请看例 2；
	 Color 输出值的显示颜色，默认表示使用属性设置框中的颜色；
	 BarsBack 从当前 Bar 向前回溯的 Bar 数，默认值为当前 Bar。
	例 1：PlotNumeric ("RSI",RSIValue);
	输出 RSI 的值。
	例 2：PlotNumeric ("OpenToClose",Open,Close);
	输出开盘价与收盘价的连线。（需要在公式属性的输出线形选择柱状图）
	例 3：PlotNumeric ("AvgValue",average(close,5),0,Blue);
	输出一条蓝色的以收盘价计算的五日平均线。
	注意：当后面的参数都使用默认值的情况下，可省略不写，如例 1。但如果后面还有其它参数要指明，
	而只是中间某一个或者多个参数需要默认值的话，则中间参数不可省略，需将默认值一一填写，如例


	PlotBool-----在当前 Bar 输出一个布尔值。
	语法：PlotBool(String Name,Bool bPlot,Numeric Locator=0,Integer Color=-1,Integer BarsBack=0)
	Name 输出值的名称，不区分大小写；
	bPlot 输出的布尔值；
	Locator 输出值的定位点；
	Color 输出值的显示颜色，默认表示使用属性设置框中的颜色
	BarsBack 从当前 Bar 向前回溯的 Bar 数，默认值为当前 Bar 

	示例 ：PlotBool ("con",con,High); 
	在 Bar 的最高价位置输出条件 con 的布尔值。
	PlotString-----在当前 Bar 输出一个字符串。
	语法：PlotString(String Name,String str,Numeric Locator=0,Integer Color=-1,Integer BarsBack=0)
	Name 输出值的名称，不区分大小写；
	str 输出的字符串；
	Locator 输出值的定位点；
	Color 输出值的显示颜色，默认表示使用属性设置框中的颜色；
	BarsBack 从当前 Bar 向前回溯的 Bar 数，默认值为当前 Bar。
	示例 ：
	PlotString ("Bear Market","Bear Bear",High,Blue); 
	在 Bar 的最高价位置输出一个蓝色的字符串 Bear Bear


## 公式正文


参数与变量的声明之后，在公式的正文部分，对所声明的变量进行赋值计算，必要的话还可以将所需的
条件语句编写进公式里。最后则是将计算结果以数值、布尔或字符串的形式在图表上输出。
现在我们来试着建立一个技术分析类的公式应用。首先，需要确立我们的指标里需要什么信息，以及需
要通过何种计算从而得到我们想要的结果。比如，现在我们需要得到两条移动平均线，那么首先可以确
定，要输出的是两个数值型的变量的值，然后我们就可以在公式中进行定义与赋值了。
有如下代码：

	Params //参数的声明
	numeric length1(10);
	numeric length2(20);
	Vars //变量的声明
	numeric ma1;
	numeric ma2;
	Begin
	ma1 = averageFC(close,length1);
	ma2 = averageFC(close,length2); // 给变量的赋值
	PlotNumeric("ma1",ma1,0,cyan);
	PlotNumeric("ma2",ma2,0,Magenta); //输出计算出来的平均值
	End

以上公式先声明参数与变量，然后在 Begin 与 End 之间对公式的主体进行了赋值与输出。

## 交易指令函数

交易开拓者提供 Buy，SellShort， BuyToCover，Sell，A_SendOrder 这五个函数来做下单的动作
指令。

Buy，SellShort， BuyToCover，Sell 这四个函数可以针对历史数据在图表上做出讯号标识，同时在实
时行情中也可以及时地发出下单委托的动作。因为可以对历史数据的图表上做出讯号的标识记录，交易
设置的性能测试也需要由这几个函数参与计算的公式应用方可实现。

Buy ---- 对当前合约发出买入开仓的指令，如果图表讯号显示当前持有空仓，则会先平掉空仓，再开多
仓；

SellShort ---- 对当前合约发出卖出开仓的指令，如果图表讯号显示当前持有多仓，则会先平掉多仓，再
开空仓；

BuyToCover ---- 对当前合约发出平空仓的指令，当图表讯号显示有空头持仓时，方可执行此指令；

Sell ---- 对当前合约发出平多仓的指令，当图表讯号显示有多头持仓时，方可执行此指令；

	if(condition1 && marketposition!=1) //条件满足且没有持多仓情况下开多
	{
	 buy();
	}else if(condition2 && marketposition!=-1) //条件满足且没有持空仓时开空
	{
	sellshort();
	}

## 调试语句输出 

	If(close>open)
	 Commentary("收阳="+Text(close)); // 在收阳的 K 线上显示收盘价