---
title: algoplus期货量化（2）
date: 2022-01-16 11:12:47
tags: algoplus
category: algoplus
---

## 买卖撤查请求

买卖报单、撤单、查询概括了所有的交易业务。这些业务都是由交易者主动发起的，并且提供必要的信息。交易者只需要传递相应参数，AlgoPlus就可以按照CTP标准组织信息并发起请求。


	b"ExchangeID": b"SHFE", b"Direction": 1, b"Volume": 1

	exchange_id：b"SHFE"
	instrument_id：b"rb2001"
	order_vol：1

    买开仓
	buy_open(exchange_id, instrument_id, order_price, order_vol)
	
	卖平仓
	sell_close(exchange_id, instrument_id, order_price, order_vol, is_today)
	
	卖开仓
	sell_open(exchange_id, instrument_id, order_price, order_vol)
	
	买平仓
	buy_close(exchange_id, instrument_id, order_price, order_vol, is_today)
	
	撤单
	req_order_action(exchange_id, instrument_id, order_ref, order_sysid)
	
	查成交
	req_qry_trade()
	
	查持仓
	req_qry_investor_position()
	
	查资金账户
	req_qry_trading_account()
	
	查合约
	req_qry_instrument()


## 买卖撤查通知

交易者发起的买卖报单、撤单请求后，期货公司柜台会推送一条响应信息，AlgoPlus的回调函数以此作为参数被调用。

	OnRspOrderInsert(pInputOrder, pRspInfo, nRequestID, bIsLast)
	
	OnRspOrderAction(pInputOrderAction, pRspInfo, nRequestID, bIsLast)
	
	pRspInfo是一个python字典，内容如下

	{
	'ErrorID': 0, # 错误代码
	'ErrorMsg': "", # 错误信息
	}

买卖报单、撤单请求到达交易所被执行的过程中，交易所实时推送订单状态变化信息，AlgoPlus的回调函数OnRtnOrder(pOrder)以此作为参数被调用。pOrder是一个python字典，内容如下

	{
	'BrokerID': "", # 经纪公司代码
	'InvestorID': "", # 投资者代码
	'InstrumentID': "", # 合约代码
	'OrderRef': "", # 报单引用
	'UserID': "", # 用户代码
	'OrderPriceType': "", # 报单价格条件
	'Direction': "", # 买卖方向
	'CombOffsetFlag': "", # 组合开平标志
	'CombHedgeFlag': "", # 组合投机套保标志
	'LimitPrice': 0.0, # 价格
	'VolumeTotalOriginal': 0, # 数量
	'TimeCondition': "", # 有效期类型
	'GTDDate': "", # GTD日期
	'VolumeCondition': "", # 成交量类型
	'MinVolume': 0, # 最小成交量
	'ContingentCondition': "", # 触发条件
	'StopPrice': 0.0, # 止损价
	'ForceCloseReason': "", # 强平原因
	'IsAutoSuspend': 0, # 自动挂起标志
	'BusinessUnit': "", # 业务单元
	'RequestID': 0, # 请求编号
	'OrderLocalID': "", # 本地报单编号
	'ExchangeID': "", # 交易所代码
	'ParticipantID': "", # 会员代码
	'ClientID': "", # 客户代码
	'ExchangeInstID': "", # 合约在交易所的代码
	'TraderID': "", # 交易所交易员代码
	'InstallID': 0, # 安装编号
	'OrderSubmitStatus': "", # 报单提交状态
	'NotifySequence': 0, # 报单提示序号
	'TradingDay': "", # 交易日
	'SettlementID': 0, # 结算编号
	'OrderSysID': "", # 报单编号
	'OrderSource': "", # 报单来源
	'OrderStatus': "", # 报单状态
	'OrderType': "", # 报单类型
	'VolumeTraded': 0, # 今成交数量
	'VolumeTotal': 0, # 剩余数量
	'InsertDate': "", # 报单日期
	'InsertTime': "", # 委托时间
	'ActiveTime': "", # 激活时间
	'SuspendTime': "", # 挂起时间
	'UpdateTime': "", # 最后修改时间
	'CancelTime': "", # 撤销时间
	'ActiveTraderID': "", # 最后修改交易所交易员代码
	'ClearingPartID': "", # 结算会员编号
	'SequenceNo': 0, # 序号
	'FrontID': 0, # 前置编号
	'SessionID': 0, # 会话编号
	'UserProductInfo': "", # 用户端产品信息
	'StatusMsg': "", # 状态信息
	'UserForceClose': 0, # 用户强平标志
	'ActiveUserID': "", # 操作用户代码
	'BrokerOrderSeq': 0, # 经纪公司报单编号
	'RelativeOrderSysID': "", # 相关报单
	'ZCETotalTradedVolume': 0, # 郑商所成交数量
	'IsSwapOrder': 0, # 互换单标志
	'BranchID': "", # 营业部编号
	'InvestUnitID': "", # 投资单元代码
	'AccountID': "", # 资金账号
	'CurrencyID': "", # 币种代码
	'IPAddress': "", # IP地址
	'MacAddress': "", # Mac地址
	}


OrderStatus取值及含义

	全部成交
	OrderStatus_AllTraded = b'0'
	#部分成交还在队列中
	OrderStatus_PartTradedQueueing = b'1'
	#部分成交不在队列中
	OrderStatus_PartTradedNotQueueing = b'2'
	#未成交还在队列中
	OrderStatus_NoTradeQueueing = b'3'
	#未成交不在队列中
	OrderStatus_NoTradeNotQueueing = b'4'
	#撤单
	OrderStatus_Canceled = b'5'
	#未知
	OrderStatus_Unknown = b'a'
	#尚未触发
	OrderStatus_NotTouched = b'b'
	#已触发
	OrderStatus_Touched = b'c'


OrderSubmitStatus取值及含义

	#已经提交
	OrderSubmitStatus_InsertSubmitted = b'0'
	#撤单已经提交
	OrderSubmitStatus_CancelSubmitted = b'1'
	#修改已经提交
	OrderSubmitStatus_ModifySubmitted = b'2'
	#已经接受
	OrderSubmitStatus_Accepted = b'3'
	#报单已经被拒绝
	OrderSubmitStatus_InsertRejected = b'4'
	#撤单已经被拒绝
	OrderSubmitStatus_CancelRejected = b'5'
	#改单已经被拒绝
	OrderSubmitStatus_ModifyRejected = b'6'

当订单有成交发生时，交易所还会推送一条成交信息，AlgoPlus的回调函数OnRtnTrade(pTrade)以此作为参数被调用。除了成交价格之外，pTrade中的其他信息在pOrder中都有。

行情数据通知

创建行情接口实例时，AlgoPlus会根据交易者传递的合约列表参数自动订阅合约。在盘中，AlgoPlus的回调函数OnRtnDepthMarketData(pDepthMarketData)以实时行情数据为参数被调用。pDepthMarketData是一个python字典，内容如下：

	{
	'TradingDay': b'20200113', # 交易日
	'InstrumentID': b'rb2005', # 合约代码
	'ExchangeID': b'', # 交易所代码
	'ExchangeInstID': b'', # 合约在交易所的代码
	'LastPrice': 3559.0, # 最新价
	'PreSettlementPrice': 3568.0, # 上次结算价
	'PreClosePrice': 3571.0, # 昨收盘
	'PreOpenInterest': 1357418.0, # 昨持仓量
	'OpenPrice': 3565.0, # 今开盘
	'HighestPrice': 3567.0, # 最高价
	'LowestPrice': 3544.0, # 最低价
	'Volume': 347796, # 数量
	'Turnover': 12361542250.0, # 成交金额
	'OpenInterest': 1345077.0, # 持仓量
	'ClosePrice': 1.7976931348623157e+308, # 今收盘
	'SettlementPrice': 1.7976931348623157e+308, # 本次结算价
	'UpperLimitPrice': 3782.0, # 涨停板价
	'LowerLimitPrice': 3353.0, # 跌停板价
	'PreDelta': 0.0, # 昨虚实度
	'CurrDelta': 1.7976931348623157e+308, # 今虚实度
	'UpdateTime': b'23:00:01', # 最后修改时间
	'UpdateMillisec': 0, # 最后修改毫秒
	'BidPrice1': 3559.0, # 申买价一
	'BidVolume1': 158, # 申买量一
	'AskPrice1': 3560.0, # 申卖价一
	'AskVolume1': 18, # 申卖量一
	'BidPrice2': 1.7976931348623157e+308, # 申买价二
	'BidVolume2': 0, # 申买量二
	'AskPrice2': 1.7976931348623157e+308, # 申卖价二
	'AskVolume2': 0, # 申卖量二
	'BidPrice3': 1.7976931348623157e+308, # 申买价三
	'BidVolume3': 0, # 申买量三
	'AskPrice3': 1.7976931348623157e+308, # 申卖价三
	'AskVolume3': 0, # 申卖量三
	'BidPrice4': 1.7976931348623157e+308, # 申买价四
	'BidVolume4': 0, # 申买量四
	'AskPrice4': 1.7976931348623157e+308, # 申卖价四
	'AskVolume4': 0, # 申卖量四
	'BidPrice5': 1.7976931348623157e+308, # 申买价五
	'BidVolume5': 0, # 申买量五
	'AskPrice5': 1.7976931348623157e+308, # 申卖价五
	'AskVolume5': 0, # 申卖量五
	'AveragePrice': 35542.50839572623, # 当日均价
	'ActionDay': b'20200110' # 业务日期
	}

## 盈损管理

examples/profit_loss_manager.py是一个相对复杂的例子，启动后可以监控账户的所有的成交，包括从快期或者其他终端软件报的单，当达到止盈止损条件时，就会自动平仓。启动前需要设置好止盈止损参数：

    pl_parameter = {
	    'StrategyID': 9,
	    # 盈损参数，'0'代表止盈, '1'代表止损，绝对价差
	    'ProfitLossParameter': {
	        b'rb2010': {'0': [2], '1': [2]},
	        b'ni2007': {'0': [20], '1': [20]},
	    },
	}

## 技术分析体系

![](114604.png)