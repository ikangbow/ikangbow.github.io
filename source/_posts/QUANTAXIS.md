---
title: QUANTAXIS
date: 2020-08-20 21:47:00
tags: 投资
category: 投资
---

# 环境准备

## 安装cmder

1. 官网下载地址
	
	`http://cmder.net/`

	下载好解压包可直接使用

2. 环境变量配置

	在系统属性里面配置环境变量，将Cmder.exe所在文件路径添加至path里

![](1.webp)

win+R,输入cmder,确定，即可运行cmder

3. 配置右键快捷启动

	    // 设置任意地方鼠标右键启动Cmder
		Cmder.exe /REGISTER ALL

![](2.webp)

4. 快捷键

    Tab       自动路径补全
	Ctrl+T    建立新页签
	Ctrl+W    关闭页签
	Ctrl+Tab  切换页签
	Alt+F4    关闭所有页签
	Alt+Shift+1 开启cmd.exe
	Alt+Shift+2 开启powershell.exe
	Alt+Shift+3 开启powershell.exe (系统管理员权限)
	Ctrl+1      快速切换到第1个页签
	Ctrl+n      快速切换到第n个页签( n值无上限)
	Alt + enter 切换到全屏状态
	Ctr+r       历史命令搜索
	Tab         自动路径补全
	Ctrl+T      建立新页签
	Ctrl+W      关闭页签
	Ctrl+Tab    切换页签
	Alt+F4      关闭所有页签
	Alt+Shift+1 开启cmd.exe
	Alt+Shift+2 开启powershell.exe
	Alt+Shift+3 开启powershell.exe (系统管理员权限)
	Ctrl+1      快速切换到第1个页签
	Ctrl+n      快速切换到第n个页签( n值无上限)
	Alt + enter 切换到全屏状态
	Ctr+r       历史命令搜索
	Win+Alt+P   开启工具选项视窗

5. 中文乱码问题

将下面的4行命令添加到cmder/config/aliases文件末尾。

    l=ls --show-control-chars 
	la=ls -aF --show-control-chars 
	ll=ls -alF --show-control-chars 
	ls=ls --show-control-chars -F

## 安装docker桌面版

### 下载

    https://www.docker.com/

下载Docker Desktop

### 安装可能遇到的问题

    Installation failed:one pre-requisite is not fullfilled

提示我们系统版本低，解决办法，伪装成专业版系统。用管理员权限开启运行[cmd]命令开启命令行，输入如下指令

    REG ADD "HKEY_LOCAL_MACHINE\software\Microsoft\Windows NT\CurrentVersion" /v EditionId /T REG_EXPAND_SZ /d Professional /F

再次安装docker可以成功

## 下载QUANTAXIS的docker-compose.yaml文件

    `https://github.com/QUANTAXIS/QUANTAXIS`

如果你是股票方向的 ==>  选择 qa-service 下的docker-compose.yaml

如果你是期货方向的 ==> 选择 qa-service-future 下的docker-compose.yaml

你可以理解 docker的构成类似搭积木的模式,  你需要这个功能的积木, 就选择他放在你的docker-compose.yaml里面

期货方向的yaml 比股票多一个  QACTPBEE的docker-container  [这是用于分发期货的tick行情所需的 股票则无需此积木]

可通过git拉取全部代码到本地，从本地拷贝出需要的dockerfile文件

## docker部署quantaxis

1. 选取一个空间较大的盘，最好不放c盘，新建quantaxis文件夹，将docker-compose.yaml拷贝到quantaxis文件夹
2. cd到quantaxis文件夹，运行如下命令

		docker volume create --name=qamg
		docker volume create --name=qacode
		docker-compose up -d

意思在后台启动这个docker环境，如果需要控制台打印输出，则把-d去掉

3. 在无报错的情况下，打开浏览器输入localhost:81即可看到

![](3.jpg)

在上方随意点击栏目，都可进入登陆界面，默认密码是quantaxis

4. docker做了什么

帮你直接开启你需要的服务

    
- 27017 mongodb
- 8888 jupyter
- 8010 quantaxis_webserver
- 81 quantaxis_community 社区版界面
- 61208 系统监控
- 15672 qa-eventmq

5. 日志查看

    	docker logs cron容器名

6. 其他命令

		docker ps
		
		docker stats
		
		docker-compose top （必须到dockerfile文件夹目录下）
		
		docker-compose ps （必须到dockerfile文件夹目录下）

		docker stop $(docker ps -a -q)停止容器

		docker rm $(docker ps -a -q)删除容器

		docker-compose pull 更新（必须到dockerfile文件夹目录下）
		docker-compose up -d 重启服务（必须到dockerfile文件夹目录下）
		docker-compose stop 停止服务（必须到dockerfile文件夹目录下）

		docker run  --rm -v qamg:/data/db \ -v $(pwd):/backup alpine \tar zcvf /backup/dbbackup.tar /data/db  备份数据库到当前目录下
		docker run  --rm -v qamg:/data/db \
		-v $(pwd):/backup alpine \
		sh -c "cd /data/db \
		&& rm -rf diagnostic.data \
		&& rm -rf journal \
		&& rm -rf configdb \
		&& cd / \
		&& tar xvf /backup/dbbackup.tar" 还原当前目录下的dbbackup.tar到mongod数据库

		
## 保存数据

1. 先进入到jupyter的登陆页登陆，找到terminal

![](4.jpg)

2. 在点开的terminal界面中，输入quantaxis 回车, 进入quantaxis cli的命令行界面


![](5.jpg)

3. 在命令行界面 输入 save  按回车, 你可以看到许多命令行选项

![](6.jpg)

4. 参考

		save all  (股票/指数 的日线数据 | 权息数据 | 板块数据)  
		save x   (股票/指数的 日线/分钟线数据  | 权息数据| 板块数据)
		
		save future_min_all  (期货的全部合约的分钟线数据)
		save future_min  (q期货主连的分钟线数据)
		
		save future_day_all (期货全部合约的日线数据)
		save future_day (期货主连的日线数据)
		
		save index_day    (指数数据  此处也要存, 因为在做回测的时候, 需要沪深300作为标的对照物)

5. 存完数据后可以打开notebook,做一个回测

		import QUANTAXIS as QA
		import numpy as np
		import pandas as pd
		import datetime
		st1=datetime.datetime.now()
		# define the MACD strategy
		def MACD_JCSC(dataframe, SHORT=12, LONG=26, M=9):
		    """
		    1.DIF向上突破DEA，买入信号参考。
		    2.DIF向下跌破DEA，卖出信号参考。
		    """
		    CLOSE = dataframe.close
		    DIFF = QA.EMA(CLOSE, SHORT) - QA.EMA(CLOSE, LONG)
		    DEA = QA.EMA(DIFF, M)
		    MACD = 2*(DIFF-DEA)
		
		    CROSS_JC = QA.CROSS(DIFF, DEA)
		    CROSS_SC = QA.CROSS(DEA, DIFF)
		    ZERO = 0
		    return pd.DataFrame({'DIFF': DIFF, 'DEA': DEA, 'MACD': MACD, 'CROSS_JC': CROSS_JC, 'CROSS_SC': CROSS_SC, 'ZERO': ZERO})
		
		
		# create account
		user = QA.QA_User(username='quantaxis', password='quantaxis')
		portfolio = user.new_portfolio('qatestportfolio')
		
		
		Account = portfolio.new_account(account_cookie='macd_stock', init_cash=1000000)
		Broker = QA.QA_BacktestBroker()
		
		QA.QA_SU_save_strategy('MACD_JCSC','Indicator',Account.account_cookie)
		# get data from mongodb
		QA.QA_SU_save_strategy('MACD_JCSC', 'Indicator',
		                       Account.account_cookie, if_save=True)
		data = QA.QA_fetch_stock_day_adv(
		    ['000001', '000002', '000004', '600000'], '2017-09-01', '2018-05-20')
		data = data.to_qfq()
		
		# add indicator
		ind = data.add_func(MACD_JCSC)
		# ind.xs('000001',level=1)['2018-01'].plot()
		
		data_forbacktest=data.select_time('2018-01-01','2018-05-01')
		
		
		for items in data_forbacktest.panel_gen:
		    for item in items.security_gen:
		        ###################
		        daily_ind=ind.loc[item.index]
		
		        if daily_ind.CROSS_JC.iloc[0]>0:
		            order=Account.send_order(
		                code=item.code[0], 
		                time=item.date[0], 
		                amount=1000, 
		                towards=QA.ORDER_DIRECTION.BUY, 
		                price=0, 
		                order_model=QA.ORDER_MODEL.CLOSE, 
		                amount_model=QA.AMOUNT_MODEL.BY_AMOUNT
		                )
		            #print(item.to_json()[0])
		            Broker.receive_order(QA.QA_Event(order=order,market_data=item))
		            trade_mes=Broker.query_orders(Account.account_cookie,'filled')
		            res=trade_mes.loc[order.account_cookie,order.realorder_id]
		            order.trade(res.trade_id,res.trade_price,res.trade_amount,res.trade_time)
		        elif daily_ind.CROSS_SC.iloc[0]>0:
		            #print(item.code)
		            if Account.sell_available.get(item.code[0], 0)>0:
		                order=Account.send_order(
		                    code=item.code[0], 
		                    time=item.date[0], 
		                    amount=Account.sell_available.get(item.code[0], 0), 
		                    towards=QA.ORDER_DIRECTION.SELL, 
		                    price=0, 
		                    order_model=QA.ORDER_MODEL.MARKET, 
		                    amount_model=QA.AMOUNT_MODEL.BY_AMOUNT
		                    )
		                #print
		                Broker.receive_order(QA.QA_Event(order=order,market_data=item))
		                trade_mes=Broker.query_orders(Account.account_cookie,'filled')
		                res=trade_mes.loc[order.account_cookie,order.realorder_id]
		                order.trade(res.trade_id,res.trade_price,res.trade_amount,res.trade_time)
		    Account.settle()
		
		print('TIME -- {}'.format(datetime.datetime.now()-st1))
		print(Account.history)
		print(Account.history_table)
		print(Account.daily_hold)
		
		# create Risk analysis
		Risk = QA.QA_Risk(Account)
		
		Account.save()
		Risk.save()

6. 推荐的做法

使用最新的QAStrategy来做回测/模拟

首先 打开terminal (上面有讲), 输入

	pip install qastrategy

然后 新建一个notebook, 输入

	from QAStrategy import QAStrategyCTABase
	import QUANTAXIS as QA
	import pprint
	
	
	class CCI(QAStrategyCTABase):
	
	    def on_bar(self, bar):
	
	        res = self.cci()
	
	        print(res.iloc[-1])
	
	        if res.CCI[-1] < -100:
	
	            print('LONG')
	
	            if self.positions.volume_long == 0:
	                self.send_order('BUY', 'OPEN', price=bar['close'], volume=1)
	
	            if self.positions.volume_short > 0:
	                self.send_order('SELL', 'CLOSE', price=bar['close'], volume=1)
	
	        elif res.CCI[-1] > 100:
	            print('SHORT')
	            if self.positions.volume_short == 0:
	                self.send_order('SELL', 'OPEN', price=bar['close'], volume=1)
	            if self.positions.volume_long > 0:
	                self.send_order('BUY', 'CLOSE', price=bar['close'], volume=1)
	
	    def cci(self,):
	        return QA.QA_indicator_CCI(self.market_data, 61)
	
	    def risk_check(self):
	        pass
	        # pprint.pprint(self.qifiacc.message)

然后 你可以自由指定回测/模拟

首先实例化这个类

	strategy =CCI(code='RB2005', frequence='1min',strategy_id='a3916de0-bd28-4b9c-bea1-94d91f1744ac', start=‘2020-01-01‘, end=‘2020-02-07’)

如果你需要测试这个策略

	strategy.debug()

如果你需要做回测

	strategy.run_backtest()

如果你需要让他直接挂模拟

在挂模拟的时候, 你需要注意一些东西

挂模拟的标的需要和真实标的一致
挂模拟的时候, 你的行情必须是有推送的, 并且申请了你所需要的的分钟线级别的数据

(如何申请行情数据? 你可以看这里 )
https://github.com/yutiansut/QUANTAXIS_RealtimeCollector

	# 期货订阅请求
	  curl -X POST "http://127.0.0.1:8011?action=new_handler&market_type=future_cn&code=au1911"
	  ```bash
	  # 股票订阅请求
	  curl -X POST "http://127.0.0.1:8011?action=new_handler&market_type=stock_cn&code=000001"
	  # 二次采样请求
	  curl -X POST "http://127.0.0.1:8011?action=new_resampler&market_type=future_cn&code=au1911&frequence=2min"
	  
	  对于小白可能难以理解curl是个啥, 此处给出一个简单易懂的代码
	  
	  import requests
	  requests.post("http://127.0.0.1:8011?action=new_handler&market_type=future_cn&code={}".format("rb2001")
	  以此类推其他的请求都可以这么做
	  ```
	像 螺纹2001 合约, 你需要改成 rb2001  注意此处是小写
	
	strategy =CCI(code='rb2001', frequence='1min',strategy_id='a3916de0-bd28-4b9c-bea1-94d91f1744ac')
	strategy.debug_sim()

做完了这些操作以后, 你可以点击 回测 你就可以看到类似这样的结果

![](7.jpg)

7. 更多参考

	http://www.yutiansut.com:3000/topic/5dc5da7dc466af76e9e3bc5d
	
	如何修改期货的实盘行情地址   http://www.yutiansut.com:3000/topic/5dfade9efe01257b44740e70
	
	实时如何申请行情  http://www.yutiansut.com:3000/topic/5dd1be9b0c8e672840f3fea7
	
	如何接入你的实盘期货账户  http://www.yutiansut.com:3000/topic/5dc865e8c466af76e9e3bdd1
	
	如何实现模拟盘/实盘的跟单  http://www.yutiansut.com:3000/topic/5ddb5ba8fe01257b4474080a
	
	docker 如何访问外部数据库  https://github.com/QUANTAXIS/QUANTAXIS/issues/1346   http://www.yutiansut.com:3000/topic/5e4531c96d3b182e88b4ebb4
	
	docker小白用户的推荐 http://www.yutiansut.com:3000/topic/5e4cb13f6d3b182e88b4ef64


