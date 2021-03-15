---
title: backtrader
date: 2020-09-05 20:31:54
tags: 投资
category: 投资
---

# Backtrader

## 引言

前基于Python的量化回测框架有很多，开源框架有zipline、vnpy、pyalgotrader和backtrader等，而量化平台有Quantopian（国外）、聚宽、万矿、优矿、米筐、掘金等，这些量化框架或平台各有优劣。就个人而言，比较偏好用backtrader，因为它功能十分完善，有完整的使用文档，安装相对简单（直接pip安装即可）。优点是运行速度快，支持pandas的矢量运算；支持参数自动寻优运算，内置了talib股票分析技术指标库；支持多品种、多策略、多周期的回测和交易；支持pyflio、empyrica分析模块库、alphalens多因子分析模块库等；扩展灵活，可以集成TensorFlow、PyTorch和Keras等机器学习、神经网络分析模块。而不足之处在于，backtrader学习起来相对复杂，编程过程中使用了大量的元编程（类class），如果Python编程基础不扎实（尤其是类的操作），学起来会感到吃力。本文作为backtrader的入门系列之一，对其运行框架进行简要介绍，并以实际案例展示量化回测的过程。

## Backtrader介绍

如果将backtrader包分解为核心组件，主要包括以下组成部分：

1. 数据加载（Data Feed）：将交易策略的数据加载到回测框架中。
2. 交易策略（Strategy）：该模块是编程过程中最复杂的部分，需要设计交易决策，得出买入/卖出信号。
3. 回测框架设置（ Cerebro）：需要设置（i）初始资金（ii）佣金（iii）数据馈送（iv）交易策略交易头寸大小。
4. 运行回测：运行Cerebro回测并打印出所有已执行的交易。
5. 评估性能（Analyzers）:以图形和风险收益等指标对交易策略的回测结果进行评价。

“Lines”是backtrader回测的数据，由一系列的点组成，通常包括以下类别的数据：Open（开盘价）, High（最高价）, Low（最低价）, Close（收盘价）, Volume（成交量）, OpenInterest（无的话设置为0）。Data Feeds（数据加载）、Indicators（技术指标）和Strategies（策略）都会生成 Lines。价格数据中的所有”Open” (开盘价)按时间组成一条 Line。所以，一组含有以上6个类别的价格数据，共有6条 Lines。如果算上“DateTime”（时间，可以看作是一组数据的主键），一共有7条 Lines。当访问一条 Line 的数据时，会默认指向下标为 0 的数据。最后一个数据通过下标 -1 来访问，在-1之后是索引0，用于访问当前时刻。因此，在回测过程中，无需知道已经处理了多少条/分钟/天/月，”0”一直指向当前值，下标 -1 来访问最后一个值。

![](批注 2020-09-06 181654.png)

## Backtrader环境搭建

- 安装python环境 (anaconda)
- pip install backtrader[plotting]
- 新建jupyterProject文件夹，在其路径栏输入 jupyter lab,按enter键,等待启用jupyter

## 回测应用实例

量化回测说白了是使用历史数据去验证交易策略的性能，因此回测的第一步是搭建交易策略，这也是backtrader要设置的最重要和复杂的部分，策略设定好后，其余部分的代码编写是手到擒来。

### 构建策略（Strategy）

交易策略类代码包含重要的参数和用于执行策略的功能，要定义的参数或函数名如下：

（1）params-全局参数，可选：更改交易策略中变量/参数的值，可用于参数调优。

（2）log：日志，可选：记录策略的执行日志，可以打印出该函数提供的日期时间和txt变量。

（3） __init__：用于初始化交易策略的类实例的代码。

（4）notify_order，可选：跟踪交易指令（order）的状态。order具有提交，接受，买入/卖出执行和价格，已取消/拒绝等状态。

（5）notify_trade，可选：跟踪交易的状态，任何已平仓的交易都将报告毛利和净利润。

（6）next，必选：制定交易策略的函数，策略模块最核心的部分。

下面以一个简单的单均线策略为例，展示backtrader的使用过程，即当收盘价上涨突破20日均线买入（做多），当收盘价下跌跌穿20日均线卖出（做空）。为简单起见，不报告交易回测的日志，因此log、notify_order和notify_trade函数省略不写。

	class my_strategy1(bt.Strategy):
	    #全局设定交易策略的参数
	    params=(
	        ('maperiod',20),
	           )
	
	    def __init__(self):
	        #指定价格序列
	        self.dataclose=self.datas[0].close
	        # 初始化交易指令、买卖价格和手续费
	        self.order = None
	        self.buyprice = None
	        self.buycomm = None
	
	        #添加移动均线指标，内置了talib模块
	        self.sma = bt.indicators.SimpleMovingAverage(
	                      self.datas[0], period=self.params.maperiod)
	    def next(self):
	        if self.order: # 检查是否有指令等待执行, 
	            return
	        # 检查是否持仓   
	        if not self.position: # 没有持仓
	            #执行买入条件判断：收盘价格上涨突破20日均线
	            if self.dataclose[0] > self.sma[0]:
	                #执行买入
	                self.order = self.buy(size=500)         
	        else:
	            #执行卖出条件判断：收盘价格跌破20日均线
	            if self.dataclose[0] < self.sma[0]:
	                #执行卖出
	                self.order = self.sell(size=500)

### 数据加载（Data Feeds）

策略设计好后，第二步是数据加载，backtrader提供了很多数据接口，包括quandl（美股）、yahoo、pandas格式数据等，我们主要分析A股数据。

	#先引入后面可能用到的包（package）
	import pandas as pd  
	from datetime import datetime
	import backtrader as bt
	import matplotlib.pyplot as plt
	%matplotlib inline   
	
	#正常显示画图时出现的中文和负号
	from pylab import mpl
	mpl.rcParams['font.sans-serif']=['SimHei']使用tushare获取浦发银行（代码：600000）数据。
	#使用tushare旧版接口获取数据
	import tushare as ts 
	def get_data(code,start='2010-01-01',end='2020-03-31'):
	    df=ts.get_k_data(code,autype='qfq',start=start,end=end)
	    df.index=pd.to_datetime(df.date)
	    df['openinterest']=0
	    df=df[['open','high','low','close','volume','openinterest']]
	    return df
	dataframe=get_data('600000')
	
	#回测期间
	start=datetime(2010, 3, 31)
	end=datetime(2020, 3, 31)
	# 加载数据
	data = bt.feeds.PandasData(dataname=dataframe,fromdate=start,todate=end)

### 回测设置（Cerebro）

回测设置主要包括几项：回测系统初始化，数据加载到回测系统，添加交易策略， broker设置（如交易资金和交易佣金），头寸规模设置作为策略一部分的交易规模等，最后显示执行交易策略时积累的总资金和净收益。

# 初始化cerebro回测系统设置                           
cerebro = bt.Cerebro()  
#将数据传入回测系统
cerebro.adddata(data) 
# 将交易策略加载到回测系统中
cerebro.addstrategy(my_strategy1) 
# 设置初始资本为10,000
startcash = 10000
cerebro.broker.setcash(startcash) 
# 设置交易手续费为 0.2%
cerebro.broker.setcommission(commission=0.002) 

### 执行回测

输出回测结果。

	print(f'净收益: {round(pnl,2)}')
	
	d1=start.strftime('%Y%m%d')
	d2=end.strftime('%Y%m%d')
	print(f'初始资金: {startcash}\n回测期间：{d1}:{d2}')
	#运行回测系统
	cerebro.run()
	#获取回测结束后的总资金
	portvalue = cerebro.broker.getvalue()
	pnl = portvalue - startcash
	#打印结果
	print(f'总资金: {round(portvalue,2)}')结果如下：
	初始资金: 10000
	回测期间：20100331:20200331
	总资金: 12065.36
	净收益: 2065.36

### 可视化

对上述结果进行可视化，使用内置的matplotlib画图。至此，简单的单均线回测就完成了。下面图形展示了浦发银行在回测期间的价格走势、买卖点和交易总资金的变化等。

	# 画图
    cerebro.plot()

## 回测实例

	from __future__ import (absolute_import, division, print_function,
	                        unicode_literals)
	
	import datetime  # For datetime objects
	import os.path  # To manage paths
	import sys  # To find out the script name (in argv[0])
	
	# Import the backtrader platform
	import backtrader as bt
	
	# Create a Stratey
	
	class TestStrategy(bt.Strategy):
	
	    params = (
	        ('exitbars', 5),
	        ('maperiod', 24),
	        ('printlog', False),
	    )
	
	    def log(self, txt, dt=None, doprint=False):
	        ''' Logging function for this strategy'''
	        if self.params.printlog or doprint:
	            dt = dt or self.datas[0].datetime.date(0)
	            print('[%s] %s' % (dt.isoformat(), txt))
	
	    def __init__(self):
	       # Keep a reference to the "close" line in the data[0] dataseries
	        self.dataclose = self.datas[0].close
	        # To keep track of pending orders and buy price/commission
	        self.order = None
	        self.buyprice = None
	        self.buycomm = None
	        # Add a MovingAverageSimple indicator
	        self.sma = bt.indicators.SimpleMovingAverage(
	            self.datas[0], period=self.params.maperiod)
	
	        # Indicators for the plotting show
	        bt.indicators.ExponentialMovingAverage(self.datas[0], period=25)
	        bt.indicators.WeightedMovingAverage(
	            self.datas[0], period=25).subplot = True
	        bt.indicators.StochasticSlow(self.datas[0])
	        bt.indicators.MACDHisto(self.datas[0])
	        rsi = bt.indicators.RSI(self.datas[0])
	        bt.indicators.SmoothedMovingAverage(rsi, period=10)
	        bt.indicators.ATR(self.datas[0]).plot = False
	
	    def notify_order(self, order):
	        if order.status in [order.Submitted, order.Accepted]:
	            # Buy/Sell order submitted/accepted to/by broker - Nothing to do
	            return
	
	        # Check if an order has been completed
	        # Attention: broker could reject order if not enough cash
	        if order.status in [order.Completed]:
	            if order.isbuy():
	                self.log(
	                    'BUY EXECUTED, Price: %.2f, Cost: %.2f, Comm %.2f' %
	                    (order.executed.price,
	                     order.executed.value,
	                     order.executed.comm))
	
	                self.buyprice = order.executed.price
	                self.buycomm = order.executed.comm
	            else:  # Sell
	                self.log('SELL EXECUTED, Price: %.2f, Cost: %.2f, Comm %.2f' %
	                         (order.executed.price,
	                          order.executed.value,
	                          order.executed.comm))
	
	            self.bar_executed = len(self)
	
	        elif order.status in [order.Canceled, order.Margin, order.Rejected]:
	            self.log('Order Canceled/Margin/Rejected')
	
	        self.order = None
	
	    def notify_trade(self, trade):
	        if not trade.isclosed:
	            return
	
	        self.log('OPERATION PROFIT： GROSS %.2f, NET %.2f' %
	                 (trade.pnl, trade.pnlcomm))
	
	    def next(self):
	        # Simply log the closing price of the series from the reference
	        self.log('Close: %.2f' % self.dataclose[0])
	
	        # Check if an order is pending ... if yes, we cannot send a 2nd one
	        if self.order:
	            return
	
	       # Check if we are in the market
	        if not self.position:
	            # Not yet ... we MIGHT BUY if ...
	            if self.dataclose[0] > self.sma[0]:
	                # current close less than previous close
	                if self.dataclose[-1] < self.dataclose[-2]:
	                    # previous close less than the previous close
	
	                    # BUY, BUY, BUY!!! (with default parameters)
	                    self.log('BUY CREATE: %.2f' % self.dataclose[0])
	
	                    # Keep track of the created order to avoid a 2nd order
	                    self.order = self.buy()
	        else:
	            # Already in the market ... we might sell
	            if self.dataclose[0] < self.sma[0]:
	                # SELL, SELL, SELL!!! (with all possible default parameters)
	                self.log('SELL CREATE: %.2f' % self.dataclose[0])
	
	                # Keep track of the created order to avoid a 2nd order
	                self.order = self.sell()
	
	    def stop(self):
	        self.log('(MA Period %2d) Ending Value %.2f' %
	                 (self.params.maperiod, self.broker.getvalue()), doprint=True)
	
	
	if __name__ == '__main__':
	    cerebro = bt.Cerebro()
	
	    # Add a strategy
	    cerebro.addstrategy(TestStrategy)
	    # cerebro.optstrategy(TestStrategy, maperiod=range(10, 31))
	
	    # 初始化数据的路径
	    modpath = os.path.dirname(os.path.abspath(sys.argv[0]))
	    datapath = os.path.join(modpath, '.\\/datas\\/orcl-1995-2014.txt')
	    # Create a Data Feed,reverse 代表是否反转数据
	    data = bt.feeds.YahooFinanceCSVData(
	        dataname=datapath,
	        # Do not pass values before this date
	        fromdate=datetime.datetime(2000, 1, 1),
	        # Do not pass values after this date
	        todate=datetime.datetime(2000, 12, 31),
	        reverse=False)
	
	    # Add the Data Feed to Cerebro
	    cerebro.adddata(data)
	    # 改变账户初始金额
	    cerebro.broker.set_cash(100000.0)
	
	    # Set the commission - 0.1% ... divide by 100 to remove the % 交易佣金设置
	    cerebro.broker.setcommission(commission=0.001)
	    # 设置每笔交易交易的股票数量
	    cerebro.addsizer(bt.sizers.FixedSize, stake=10)
	
	    print('Starting Portfolio Value: %.2f' % cerebro.broker.getvalue())
	
	    cerebro.run()
	
	    print('Final Portfolio Value: %.2f' % cerebro.broker.getvalue())
	    # 画图
	    cerebro.plot()

先来回顾一下交易策略模块（Strategy）的构成。交易策略类代码包含参数或函数名如下：

（1）params-全局参数，可选：更改交易策略中变量/参数的值，可用于参数调优。

（2）log：日志，可选：记录策略的执行日志，可以打印出该函数提供的日期时间和txt变量。

（3） init：用于初始化交易策略的类实例的代码。

（4）notify_order，可选：跟踪交易指令（order）的状态。order具有提交，接受，买入/卖出执行和价格，已取消/拒绝等状态。

（5）notify_trade，可选：跟踪交易的状态，任何已平仓的交易都将报告毛利和净利润。

（6）next，必选：制定交易策略的函数，策略模块最核心的部分。

下面仍然以简单均线策略为例，重点介绍参数寻优和交易日志报告。

实现代码如下：

	#先引入后面可能用到的包（package）
	import pandas as pd  
	import numpy as np
	import tushare as ts 
	import matplotlib.pyplot as plt
	%matplotlib inline   
	#正常显示画图时出现的中文和负号
	from pylab import mpl
	mpl.rcParams['font.sans-serif']=['SimHei']
	mpl.rcParams['axes.unicode_minus']=False

params是全局参数，maperiod是MA均值的长度，默认15天，printlog为打印交易日志，默认不输出结果，策略模块的核心在next（）函数。

	from datetime import datetime
	import backtrader as bt
	class MyStrategy(bt.Strategy):
	    params=(('maperiod',15),
	            ('printlog',False),)
	    def __init__(self):
	        #指定价格序列
	        self.dataclose=self.datas[0].close
	        # 初始化交易指令、买卖价格和手续费
	        self.order = None
	        self.buyprice = None
	        self.buycomm = None
	        #添加移动均线指标
	        self.sma = bt.indicators.SimpleMovingAverage(
	                      self.datas[0], period=self.params.maperiod)
	    #策略核心，根据条件执行买卖交易指令（必选）
	    def next(self):
	        # 记录收盘价
	        #self.log(f'收盘价, {dataclose[0]}')
	        if self.order: # 检查是否有指令等待执行, 
	            return
	        # 检查是否持仓   
	        if not self.position: # 没有持仓
	            #执行买入条件判断：收盘价格上涨突破15日均线
	            if self.dataclose[0] > self.sma[0]:
	                self.log('BUY CREATE, %.2f' % self.dataclose[0])
	                #执行买入
	                self.order = self.buy()         
	        else:
	            #执行卖出条件判断：收盘价格跌破15日均线
	            if self.dataclose[0] < self.sma[0]:
	                self.log('SELL CREATE, %.2f' % self.dataclose[0])
	                #执行卖出
	                self.order = self.sell()
	    #交易记录日志（可省略，默认不输出结果）
	    def log(self, txt, dt=None,doprint=False):
	        if self.params.printlog or doprint:
	            dt = dt or self.datas[0].datetime.date(0)
	            print(f'{dt.isoformat()},{txt}')
	    #记录交易执行情况（可省略，默认不输出结果）
	    def notify_order(self, order):
	        # 如果order为submitted/accepted,返回空
	        if order.status in [order.Submitted, order.Accepted]:
	            return
	        # 如果order为buy/sell executed,报告价格结果
	        if order.status in [order.Completed]: 
	            if order.isbuy():
	                self.log(f'买入:\n价格:{order.executed.price},\
	                成本:{order.executed.value},\
	                手续费:{order.executed.comm}')
	                self.buyprice = order.executed.price
	                self.buycomm = order.executed.comm
	            else:
	                self.log(f'卖出:\n价格：{order.executed.price},\
	                成本: {order.executed.value},\
	                手续费{order.executed.comm}')
	            self.bar_executed = len(self) 
	        # 如果指令取消/交易失败, 报告结果
	        elif order.status in [order.Canceled, order.Margin, order.Rejected]:
	            self.log('交易失败')
	        self.order = None
	    #记录交易收益情况（可省略，默认不输出结果）
	    def notify_trade(self,trade):
	        if not trade.isclosed:
	            return
	        self.log(f'策略收益：\n毛收益 {trade.pnl:.2f}, 净收益 {trade.pnlcomm:.2f}')
	    #回测结束后输出结果（可省略，默认输出结果）
	    def stop(self):
	        self.log('(MA均线： %2d日) 期末总资金 %.2f' %
	                 (self.params.maperiod, self.broker.getvalue()), doprint=True)
 

下面定义一个主函数，用于对某股票指数（个股）在指定期间进行回测，使用tushare的旧接口获取数据，包含开盘价、最高价、最低价、收盘价和成交量。这里主要以3到30日均线为例进行参数寻优，考察以多少日均线与价格的交叉作为买卖信号能获得最大的收益。

	def main(code,start,end='',startcash=10000,qts=500,com=0.001):
	    #创建主控制器
	    cerebro = bt.Cerebro()      
	    #导入策略参数寻优
	    cerebro.optstrategy(MyStrategy,maperiod=range(3, 31))    
	    #获取数据
	    df=ts.get_k_data(code,autype='qfq',start=start,end=end)
	    df.index=pd.to_datetime(df.date)
	    df=df[['open','high','low','close','volume']]
	    #将数据加载至回测系统
	    data = bt.feeds.PandasData(dataname=df)    
	    cerebro.adddata(data)
	    #broker设置资金、手续费
	    cerebro.broker.setcash(startcash)           
	    cerebro.broker.setcommission(commission=com)    
	    #设置买入设置，策略，数量
	    cerebro.addsizer(bt.sizers.FixedSize, stake=qts)   
	    print('期初总资金: %.2f' %                    
	    cerebro.broker.getvalue())    
	    cerebro.run(maxcpus=1)    
	    print('期末总资金: %.2f' % cerebro.broker.getvalue())
 

再定义一个画图函数，对相应股票（指数）在某期间的价格走势和累计收益进行可视化。

	def plot_stock(code,title,start,end):
	    dd=ts.get_k_data(code,autype='qfq',start=start,end=end)
	    dd.index=pd.to_datetime(dd.date)
	    dd.close.plot(figsize=(14,6),color='r')
	    plt.title(title+'价格走势\n'+start+':'+end,size=15)
	    plt.annotate(f'期间累计涨幅:{(dd.close[-1]/dd.close[0]-1)*100:.2f}%', xy=(dd.index[-150],dd.close.mean()), 
	             xytext=(dd.index[-500],dd.close.min()), bbox = dict(boxstyle = 'round,pad=0.5',
	            fc = 'yellow', alpha = 0.5),
	             arrowprops=dict(facecolor='green', shrink=0.05),fontsize=12)
	    plt.show()

以上证综指为例，回测期间为2010-01-01至2020-03-30，期间累计收益率为-15.31%，惨不忍睹。

	面分别对3-30日均线进行回测，这里假设指数可以交易，初始资金为100万元，每次交易100股，注意如果指数收盘价乘以100超过可用资金，会出现交易失败的情况，换句话说在整个交易过程中，是交易固定数量的标的，因此仓位的大小跟股价有直接关系。

	main('sh','2010-01-01','',1000000,100)

![](批注 2020-09-06 183530.png)

	plot_stock('sh','上证综指','2010-01-01','2020-03-30')

![](批注 2020-09-06 183419.png)

## Analyzers模块

Analyzers模块涵盖了评价一个量化策略的完整指标，如常见的夏普比率、年化收益率、最大回撤、Calmar比率等等。Analyzers模块原生代码能获取的评价指标如下图所示，其中TradeAnalyzer和PeriodStats又包含了不少指标。由于采用元编程，Analyzers的扩展性较强，可以根据需要添加自己的分析指标，如获取回测期间每一时刻对应的总资金。

![](批注 2020-09-06 183823.png)

### 策略模块编写

（1）params-全局参数，可选：更改交易策略中变量/参数的值，可用于参数调优。

（2）log：日志，可选：记录策略的执行日志，可以打印出该函数提供的日期时间和txt变量。

（3） init：用于初始化交易策略的类实例的代码。

（4）notify_order，可选：跟踪交易指令（order）的状态。order具有提交，接受，买入/卖出执行和价格，已取消/拒绝等状态。

（5）notify_trade，可选：跟踪交易的状态，任何已平仓的交易都将报告毛利和净利润。

（6）next，必选：制定交易策略的函数，策略模块最核心的部分。

（7）其他，包括start()、nextsstart()、stop()、prenext()、notify_fund()、notify_store()和notify_cashvalue。

面以技术分析指标RSI（不了解的请自行百度）的择时策略为例，当RSI<30时买入，RSI>70时卖出。为了简便起见，策略模块中只包含最核心的交易信号。

	import pandas as pd
	import backtrader as bt
	from datetime import datetime
	class MyStrategy(bt.Strategy):
	    params=(('short',30),
	            ('long',70),)
	    def __init__(self):
	        self.rsi = bt.indicators.RSI_SMA(
	                   self.data.close, period=21)
	    def next(self):
	        if not self.position:
	            if self.rsi < self.params.short:
	                self.buy()
	        else:
	            if self.rsi > self.params.long:
	                self.sell()

### 回测设置

回测系统设置与之前一样，主要是数据加载、交易本金、手续费、交易数量的设置，此处以tushare的旧接口获取股票002537的交易数据进行量化回测。

	from __future__ import (absolute_import, division, print_function,  
	                        unicode_literals) 
	import tushare as ts
	#以股票002537为例
	df=ts.get_k_data('002537',start='2010-01-01')
	df.index=pd.to_datetime(df.date)
	#df['openinterest'] = 0
	df=df[['open','high','low','close','volume']]
	data = bt.feeds.PandasData(dataname=df,                               
	                            fromdate=datetime(2013, 1, 1),                               
	                            todate=datetime(2020, 4, 17) )
	# 初始化cerebro回测系统设置                           
	cerebro = bt.Cerebro()  
	# 加载数据
	cerebro.adddata(data) 
	# 将交易策略加载到回测系统中
	cerebro.addstrategy(MyStrategy) 
	# 设置初始资本为100,000
	cerebro.broker.setcash(100000.0) 
	#每次固定交易数量
	cerebro.addsizer(bt.sizers.FixedSize, stake=1000) 
	#手续费
	cerebro.broker.setcommission(commission=0.001) 

### 运行回测

这里重点是Analyzers模块的调用与结果输出，调用模块是cerebro.addanalyzer()，再从模块中获取分析指标，如夏普比率是bt.analyzers.SharpeRatio，然后是给该指标重命名方便之后调用，即 _name='SharpeRatio'。要获取分析指标，需要先执行回测系统，cerebro.run()，并将回测结果赋值给变量results，分析指标存储在results[0]里 (strat变量代替)，通过strat.analyzers.SharpeRatio.get_analysis()即可获取相应数据，其他指标操作方法类似。

	print('初始资金: %.2f' % cerebro.broker.getvalue())
	cerebro.addanalyzer(bt.analyzers.SharpeRatio, _name = 'SharpeRatio')
	cerebro.addanalyzer(bt.analyzers.DrawDown, _name='DW')
	results = cerebro.run()
	strat = results[0]
	print('最终资金: %.2f' % cerebro.broker.getvalue())
	print('夏普比率:', strat.analyzers.SharpeRatio.get_analysis())
	print('回撤指标:', strat.analyzers.DW.get_analysis())

	输出结果：
	初始资金: 100000.00
	最终资金: 110215.33
	夏普比率: OrderedDict([('sharperatio', 0.094)])
	回撤指标: AutoOrderedDict([('len', 280), ('drawdown', 1.01), ('moneydown', 1126.60), ('max', AutoOrderedDict([('len', 280), ('drawdown', 3.61), ('moneydown', 4016.60)]))])

### 回测结果可视化

下面输出回测图表，一张大图上包含了三张图：

（1）资金变动图：可以看到在实施交易策略的数据期内，资金的盈利/损失。

（2）交易收益/亏损。蓝色（红色）点表示获利（亏损）交易以及获利（亏损）多少。

（3）价格图表。绿色和红色箭头分别表示交易策略的进入点和退出点。黑线是交易标的随时间变化的价格， 条形图表示每个条形图期间资金的交易量。

![](批注 2020-09-06 184322.png)

### Analyzers模块指标可视化



## 其他

__init__

任何类在生成的时候都是先调用这一初始化构造函数。也就是说，在实例生成的时候，这个函数将被调用。

1. Birth: start

    start方法在cerebro告诉strategy，是时候开始行动了，也就是说，通知策略激活的时候被调用。

2. Childhood: prenext

    有些技术指标，比如我们提到的MA，存在一个窗口，也就是说，需要n天的数据才能产生指标，那么在没有产生之前呢？这个prenext方法就会被自动调用。
3. Adulthood: next
这个方法是最核心的，就是每次移动到下一的时间点，策略将会调用这个方法，所以，策略的核心往往都是写在这个方法里的。

4. Death: stop

    策略的生命周期结束，cerebro把这一策略退出。

## 策略当中的回调函数

Strategy 类就像真实世界的交易员一样，当交易执行的时候，他会得到一些消息，譬如order是否执行，一笔trader赚了多少钱，等等。这些消息都将在Strategy类中通过回调函数被得以知晓。这些回调函数如下：

notify_order(order)：下的单子，order的任何状态变化都将引起这一方法的调用

notify_trade(trade)：任何一笔交易头寸的改变都将调用这一方法

notify_cashvalue(cash, value)：任何现金和资产组合的变化都将调用这一方法 
notify_store(msg, *args, **kwargs)：可以结合cerebro类进行自定义方法的调用

那么问题接踵而至，这里我们只关注前2种方法中监测对象的可变化方式。

trade指的是一笔头寸，trade是open的状态指当前时刻，这一标的的头寸从0变到某一非零值。trade是closed则刚好相反。

    trade大概有如下常用属性
	ref: 唯一id
	size (int): trade的当前头寸
	price (float): trade资产的当前价格
	value (float): trade的当前价值
	commission (float): trade的累计手续费
	pnl (float): trade的当前pnl
	pnlcomm (float): trade的当前pnl减去手续费
	isclosed (bool): 当前时刻trade头寸是否归零
	isopen (bool): 新的交易更新了trade
	justopened (bool): 新开头寸
	dtopen (float): trade open的datetime
	dtclose (float): trade close的datetime

	Orders

    order是strategy发出的指令，让cerebro去执行。
    strategy自身有buy, sell and close方法来生成order，cancel方法来取消一笔order。下单的方式有很多，后续会介绍，这里主要讲回调函数中，咱们可以获得哪些信息。
	order.status可以返回order的当前状态
	
	order.isbuy可以获得这笔order是否是buy
	
	order.executed.price
	order.executed.value
	order.executed.comm

分别可以获得执行order的价格，总价，和手续费
	
	class TestStrategy(bt.Strategy):
	    params = (
	        ('maperiod', 15),
	    )
	 
	    def log(self, txt, dt=None):
	        ''' Logging function fot this strategy'''
	        dt = dt or self.datas[0].datetime.date(0)
	        print('%s, %s' % (dt.isoformat(), txt))
	 
	    def __init__(self):
	        # Keep a reference to the "close" line in the data[0] dataseries
	        self.dataclose = self.datas[0].close
	 
	        # To keep track of pending orders and buy price/commission
	        self.order = None
	        self.buyprice = None
	        self.buycomm = None
	 
	        # Add a MovingAverageSimple indicator
	        self.sma = bt.indicators.SimpleMovingAverage(
	            self.datas[0], period=self.params.maperiod)
	    def start(self):
	        print("the world call me!")
	 
	    def prenext(self):
	        print("not mature")
	 
	    def notify_order(self, order):
	        if order.status in [order.Submitted, order.Accepted]:
	            # Buy/Sell order submitted/accepted to/by broker - Nothing to do
	            return
	 
	        # Check if an order has been completed
	        # Attention: broker could reject order if not enougth cash
	        if order.status in [order.Completed]:
	            if order.isbuy():
	                self.log(
	                    'BUY EXECUTED, Price: %.2f, Cost: %.2f, Comm %.2f' %
	                    (order.executed.price,
	                     order.executed.value,
	                     order.executed.comm))
	 
	                self.buyprice = order.executed.price
	                self.buycomm = order.executed.comm
	            else:  # Sell
	                self.log('SELL EXECUTED, Price: %.2f, Cost: %.2f, Comm %.2f' %
	                         (order.executed.price,
	                          order.executed.value,
	                          order.executed.comm))
	 
	            self.bar_executed = len(self)
	 
	        elif order.status in [order.Canceled, order.Margin, order.Rejected]:
	            self.log('Order Canceled/Margin/Rejected')
	 
	        self.order = None

可以看到打印出来的结果中，有start和prenext，最后当然也有death

## Backtrader的indicator

    def __init__(self):
        # Keep a reference to the "close" line in the data[0] dataseries
        self.dataclose = self.datas[0].close
 
        # To keep track of pending orders and buy price/commission
        self.order = None
        self.buyprice = None
        self.buycomm = None
 
        # Add a MovingAverageSimple indicator
        self.sma = bt.indicators.SimpleMovingAverage(
            self.datas[0], period=self.params.maperiod)

 这里的最后，我们使用了一个backtrader内置的indicator，后续我们将尝试自己编写一个indicator。

## 数据的获取

datafeed，也就是cerebro的本源，数据

    dataframe = pd.read_csv('dfqc.csv', index_col=0, parse_dates=True)
    dataframe['openinterest'] = 0
    data = bt.feeds.PandasData(dataname=dataframe,
                            fromdate = datetime.datetime(2015, 1, 1),
                            todate = datetime.datetime(2016, 12, 31)
                            )
    # Add the Data Feed to Cerebro
    cerebro.adddata(data)

	2014-03-13 00:00:00.005,1.425,1.434,1.449,1.418,457767208.0
	2014-03-14 00:00:00.005,1.429,1.422,1.436,1.416,196209439.0
	2014-03-17 00:00:00.005,1.433,1.434,1.437,1.422,250946201.0
	2014-03-18 00:00:00.005,1.434,1.425,1.437,1.424,245516577.0
	2014-03-19 00:00:00.005,1.423,1.419,1.423,1.406,331866195.0
	2014-03-20 00:00:00.005,1.412,1.408,1.434,1.407,379443759.0
	2014-03-21 00:00:00.005,1.406,1.463,1.468,1.403,825467935.0

	dataframe = pd.read_csv('dfqc.csv', index_col=0, parse_dates=True)

把csv读入pandas的参数，index_col=0表示第一列时间数据是作为pandas 的index的，parse_dates=Ture是自动把数据中的符合日期的格式变成datetime类型。为什么要这样呢？其实读入后的pandas长怎么样都是由backtrader规定的


pandas的要求的结构，我们就知道，不仅仅有self.datas[0].close,还会有self.datas[0].open。也确实如此。只是我们通常拿close作为一个价格基准

	self.datas[0].close

返回的是一个lines。lines是backtrader一个很重要的概念，可以理解为时间序列流，这类数据，后面可以跟index，也就是说，可以有

	self.datas[0].close[0]
	self.datas[0].close[-1]

这里的index是有意义的，0代表当前时刻，-1代表前一时刻，1代表后一时刻，以此类推

所以在next中使用self.dataclose[0],self.dataclose[-1]

## 安装TA-lib

下载TA_Lib-0.4.19-cp37-cp37m-win_amd64

	pip install TA_Lib-0.4.19-cp37-cp37m-win_amd64.whl