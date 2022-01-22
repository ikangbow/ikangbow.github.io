---
title: algoplus期货量化（2）
date: 2022-01-16 11:12:47
tags: algoplus
category: algoplus
---

记录学习使用AlgoPlus接收期货实时行情，原文参考[https://zhuanlan.zhihu.com/p/86082225](https://zhuanlan.zhihu.com/p/86082225)

## 使用AlgoPlus接收期货实时行情

关于CTP

CTP是Comprehensive Transaction Platform的简称。CTP有MdApi和TraderApi两个独立的开放接口。

MdApi负责行情相关操作（订阅、接收）。

![](234220.png)

TraderApi负责交易相关的操作（买、卖、撤、查）。

![](234517.png)

MdApi与TraderApi方法的执行过程都是异步的，每一个请求都对应一个或多个负责接收执行结果的回调函数。例如，通过ReqOrderInsert方法向交易所发出买开仓指令，对应的回调方法OnRtnOrder可以实时接收交易所服务器发回来的执行通知。

## AlgoPlus创建行情接口

MdApi是行情接口，使用时只需要传递账户参数创建一个实例就可以了。示例：

    from AlgoPlus.CTP.MdApi import MdApi
 
	class TickEngine(MdApi):
	    # 深度行情通知
	    def OnRtnDepthMarketData(self, pDepthMarketData):
	        print(pDepthMarketData)
	        # print(f"{pDepthMarketData.InstrumentID}当前最新价：{pDepthMarketData.LastPrice}")
	 
	if __name__ == '__main__':
	    from account_info import my_future_account_info_dict
	    future_account = my_future_account_info_dict['SimNow']
	    tick_engine = TickEngine(future_account.server_dict['MDServer']
	                             , future_account.broker_id
	                             , future_account.investor_id
	                             , future_account.password
	                             , future_account.app_id
	                             , future_account.auth_code
	                             , future_account.instrument_id_list
	                             , None
	                             , future_account.md_page_dir)
	    tick_engine.Join()


1、从AlgoPlus.CTP.MdApi文件中导入MdApi类。MdApi已对工作流程的前六步进行了封装。

2、TickEngine是MdApi的子类。TickEngine类主要实现收到行情的数据处理算法，示例只将收到的行情打印出来。

3、创建行情接口实例前，需要导入账户信息。示例的账户信息存放在同一个目录下的account_info.py文件中。

4、交易时间运行以上代码就可以将接收到的实时期货行情打印出来。

5、回调函数OnRtnDepthMarketData接收到的pDepthMarketData行情是DepthMarketDataField结构体的实例，在AlgoPlus.CTP.ApiStruct中被定义。以调用属性的方式可以获取行情任意字段的数值，例如pDepthMarketData.LastPrice表示最新价。DepthMarketDataField包括以下字段：

    class DepthMarketDataField(BaseField):
    """深度行情"""
    _fields_ = [
        ('TradingDay', c_char * 9)  # ///交易日
        , ('InstrumentID', c_char * 31)  # 合约代码
        , ('ExchangeID', c_char * 9)  # 交易所代码
        , ('ExchangeInstID', c_char * 31)  # 合约在交易所的代码
        , ('LastPrice', c_double)  # 最新价
        , ('PreSettlementPrice', c_double)  # 上次结算价
        , ('PreClosePrice', c_double)  # 昨收盘
        , ('PreOpenInterest', c_double)  # 昨持仓量
        , ('OpenPrice', c_double)  # 今开盘
        , ('HighestPrice', c_double)  # 最高价
        , ('LowestPrice', c_double)  # 最低价
        , ('Volume', c_int)  # 数量
        , ('Turnover', c_double)  # 成交金额
        , ('OpenInterest', c_double)  # 持仓量
        , ('ClosePrice', c_double)  # 今收盘
        , ('SettlementPrice', c_double)  # 本次结算价
        , ('UpperLimitPrice', c_double)  # 涨停板价
        , ('LowerLimitPrice', c_double)  # 跌停板价
        , ('PreDelta', c_double)  # 昨虚实度
        , ('CurrDelta', c_double)  # 今虚实度
        , ('UpdateTime', c_char * 9)  # 最后修改时间
        , ('UpdateMillisec', c_int)  # 最后修改毫秒
        , ('BidPrice1', c_double)  # 申买价一
        , ('BidVolume1', c_int)  # 申买量一
        , ('AskPrice1', c_double)  # 申卖价一
        , ('AskVolume1', c_int)  # 申卖量一
        , ('BidPrice2', c_double)  # 申买价二
        , ('BidVolume2', c_int)  # 申买量二
        , ('AskPrice2', c_double)  # 申卖价二
        , ('AskVolume2', c_int)  # 申卖量二
        , ('BidPrice3', c_double)  # 申买价三
        , ('BidVolume3', c_int)  # 申买量三
        , ('AskPrice3', c_double)  # 申卖价三
        , ('AskVolume3', c_int)  # 申卖量三
        , ('BidPrice4', c_double)  # 申买价四
        , ('BidVolume4', c_int)  # 申买量四
        , ('AskPrice4', c_double)  # 申卖价四
        , ('AskVolume4', c_int)  # 申卖量四
        , ('BidPrice5', c_double)  # 申买价五
        , ('BidVolume5', c_int)  # 申买量五
        , ('AskPrice5', c_double)  # 申卖价五
        , ('AskVolume5', c_int)  # 申卖量五
        , ('AveragePrice', c_double)  # 当日均价
        , ('ActionDay', c_char * 9)  # 业务日期
    ]

说明：

1、队列是实现行情进程与策略进程之间共享数据的最简单有效的方案，也是AlgoPlus默认使用的方案。

2、每个策略对应一个队列，将这些队列的列表赋值给参数md_queue_list。

3、在OnRtnDepthMarketData中，将收到的行情放入所有队列。

## 策略接收行情

	import time
	from datetime import datetime, timedelta
	from multiprocessing import Process, Queue
	from AlgoPlus.CTP.TraderApi import TraderApi
	from AlgoPlus.CTP.ApiStruct import *
	from tick_engine import TickEngine
	
	
	class TraderEngine(TraderApi):
	    def __init__(self, td_server, broker_id, investor_id, password, app_id, auth_code, md_queue=None
	                 , page_dir='', private_resume_type=2, public_resume_type=2):
	        self.order_ref = 0  # 报单引用
	        self.order_time = None  # 报单时间
	        self.order_status = b""  # 订单状态
	        self.Join()
	
	    # 撤单
	    def req_order_action(self, exchange_id, instrument_id, order_ref, order_sysid=''):
	        input_order_action_field = InputOrderActionField(
	            BrokerID=self.broker_id,
	            InvestorID=self.investor_id,
	            UserID=self.investor_id,
	            ExchangeID=exchange_id,
	            ActionFlag="0",
	            InstrumentID=instrument_id,
	            FrontID=self.front_id,
	            SessionID=self.session_id,
	            OrderSysID=order_sysid,
	            OrderRef=str(order_ref),
	        )
	        l_retVal = self.ReqOrderAction(input_order_action_field)
	
	    # 报单
	    def req_order_insert(self, exchange_id, instrument_id, order_price, order_vol, order_ref, direction, offset_flag):
	        input_order_field = InputOrderField(
	            BrokerID=self.broker_id,
	            InvestorID=self.investor_id,
	            ExchangeID=exchange_id,
	            InstrumentID=instrument_id,
	            UserID=self.investor_id,
	            OrderPriceType="2",
	            Direction=direction,
	            CombOffsetFlag=offset_flag,
	            CombHedgeFlag="1",
	            LimitPrice=order_price,
	            VolumeTotalOriginal=order_vol,
	            TimeCondition="3",
	            VolumeCondition="1",
	            MinVolume=1,
	            ContingentCondition="1",
	            StopPrice=0,
	            ForceCloseReason="0",
	            IsAutoSuspend=0,
	            OrderRef=str(order_ref),
	        )
	        l_retVal = self.ReqOrderInsert(input_order_field)
	
	    # 买开仓
	    def buy_open(self, exchange_ID, instrument_id, order_price, order_vol, order_ref):
	        self.req_order_insert(exchange_ID, instrument_id, order_price, order_vol, order_ref, '0', '0')
	
	    # 卖开仓
	    def sell_open(self, exchange_ID, instrument_id, order_price, order_vol, order_ref):
	        self.req_order_insert(exchange_ID, instrument_id, order_price, order_vol, order_ref, '1', '0')
	
	    # 买平仓
	    def buy_close(self, exchange_ID, instrument_id, order_price, order_vol, order_ref):
	        if exchange_ID == "SHFE" or exchange_ID == "INE":
	            self.req_order_insert(exchange_ID, instrument_id, order_price, order_vol, order_ref, '0', '3')
	        else:
	            self.req_order_insert(exchange_ID, instrument_id, order_price, order_vol, order_ref, '0', '1')
	
	    # 卖平仓
	    def sell_close(self, exchange_ID, instrument_id, order_price, order_vol, order_ref):
	        if exchange_ID == "SHFE" or exchange_ID == "INE":
	            self.req_order_insert(exchange_ID, instrument_id, order_price, order_vol, order_ref, '1', '3')
	        else:
	            self.req_order_insert(exchange_ID, instrument_id, order_price, order_vol, order_ref, '1', '1')
	
	    # 报单通知
	    def OnRtnOrder(self, pOrder):
	        self.order_status = pOrder.OrderStatus
	        if pOrder.OrderStatus == b"a":
	            status_msg = "未知状态！"
	        elif pOrder.OrderStatus == b"0":
	            if pOrder.Direction == b"0":
	                if pOrder.CombOffsetFlag == b"0":
	                    status_msg = "买开仓已全部成交！"
	                else:
	                    status_msg = "买平仓已全部成交！"
	            else:
	                if pOrder.CombOffsetFlag == b"0":
	                    status_msg = "卖开仓已全部成交！"
	                else:
	                    status_msg = "卖平仓已全部成交！"
	        elif pOrder.OrderStatus == b"1":
	            status_msg = "部分成交！"
	        elif pOrder.OrderStatus == b"3":
	            status_msg = "未成交！"
	        elif pOrder.OrderStatus == b"5":
	            status_msg = "已撤！"
	        else:
	            status_msg = "其他！"
	
	        self._write_log(f"{status_msg}=>{pOrder}")
	
	    def Join(self):
	        while True:
	            if self.status == 0:
	                last_md = None
	                # 如果队列非空，从队列中取数据
	                while not self.md_queue.empty():
	                    last_md = self.md_queue.get(block=False)
	
	                if last_md:
	                    # ############################################################################# #
	                    if self.order_ref == 0:
	                        # 涨停买开仓
	                        self.order_ref += 1
	                        self.buy_open(test_exchange_id, test_instrument_id, last_md.BidPrice1, test_vol, self.order_ref)
	                        self.order_time = datetime.now()
	                        self._write_log(f"=>买开仓请求！")
	
	                    if self.order_ref == 1 and self.order_status == b"3" and datetime.now() - self.order_time > timedelta(seconds=3):
	                        self.order_status = b""
	                        self.req_order_action(test_exchange_id, test_instrument_id, self.order_ref)
	                        self._write_log(f"=>发出撤单请求！")
	
	                    if self.order_ref == 1 and self.order_status == b"0":
	                        self.order_ref += 1
	                        self.order_status = b""
	                        self.sell_close(test_exchange_id, test_instrument_id, last_md.BidPrice1, test_vol, self.order_ref)
	                        self.order_time = datetime.now()
	                        self._write_log(f"=>买开仓已全部成交，发出卖平仓请求！")
	
	                    # ############################################################################# #
	                    if self.order_ref == 1:
	                        if self.order_status == b"5":
	                            print("老爷，买开仓单超过3秒未成交，已撤销，这里的测试工作已经按照您的吩咐全部完成！")
	                            break
	                        elif datetime.now() - self.order_time > timedelta(seconds=3):
	                            print("买开仓执行等待中！")
	                    elif self.order_ref == 2:
	                        if self.order_status == b"0":
	                            print("老爷，卖平仓单已成交，这里的测试工作已经按照您的吩咐全部完成！")
	                            break
	                        elif datetime.now() - self.order_time > timedelta(seconds=3):
	                            print("卖平仓执行等待中！")
	            else:
	                time.sleep(1)
	
	
	说明：
	
	1、直接在TraderApi的子类中编写策略是最简单的方案。但是，该方案不适合单账户策略比较多的情况，因为CTP支持同时在线的终端个数有限。如果策略比较多，则创建有限个TraderApi，在独立的Strategy类与MdApi和TraderApi之间实现共享数据。
	
	2、在Join方法中实现了策略逻辑：登录成功之后，先以排队价发开仓委托，如果挂单超过3秒未成交，则撤单并退出策略。如果开仓全部成交，则以对手价发平仓委托，等待全部成交后退出策略。
	
	3、Join方法中的策略每次执行时，从队列中取出所有数据，以最后一笔行情的盘口价格作为委托价。

	4、OnRtnOrder收到订单状态通知时更新本地订单状态、持仓手数，在策略中根据状态的变化进行后续操作。
	
	## 多进程
	
	# 请在这里填写需要测试的合约数据
	# 警告：该例子只支持上期所品种平今仓测试
	test_exchange_id = 'SHFE'  # 交易所
	test_instrument_id = 'ag1912'  # 合约代码
	test_vol = 1  # 报单手数
	
	share_queue = Queue(maxsize=100)  # 队列
	
	if __name__ == "__main__":
	    import sys
	
	    sys.path.append("..")
	    from account_info import my_future_account_info_dict
	
	    future_account = my_future_account_info_dict['SimNow']
	
	    # 行情进程
	    md_process = Process(target=TickEngine, args=(future_account.server_dict['MDServer']
	                                                  , future_account.broker_id
	                                                  , future_account.investor_id
	                                                  , future_account.password
	                                                  , future_account.app_id
	                                                  , future_account.auth_code
	                                                  , future_account.instrument_id_list
	                                                  , [share_queue]
	                                                  , future_account.md_page_dir)
	                         )
	
	    # 交易进程
	    trader_process = Process(target=TraderEngine, args=(future_account.server_dict['TDServer']
	                                                        , future_account.broker_id
	                                                        , future_account.investor_id
	                                                        , future_account.password
	                                                        , future_account.app_id
	                                                        , future_account.auth_code
	                                                        , share_queue
	                                                        , future_account.td_page_dir)
	                             )
	
	    md_process.start()
	    trader_process.start()
	
	    md_process.join()
	    trader_process.join()
	
	
	说明：
	
	1、前几节的例子中需要手动设置涨跌停价作为报单价，这里我们以实时行情的盘口价作为报单价，所以不再需要设置涨跌停价参数。
	
	2、share_queue是一个队列，在多进程中，队列数据可以实现共享。
	
	3、md_process和trader_process分别是行情进程和交易进程。这两个进程通过队列share_queue共享数据。
	
	4、所有进程的join方法须在start方法之后最后调用。

	5、这段代码放置在策略代码最后，执行即可看到执行结果。

	6、参考这个例子可以很方便的扩展一对多、多对多的进程间数据共享模式。
	
	
	AlgoPlus的设计在登录时通过查询获取初始持仓、可用资金，成交发生时自动增减持仓数量，当平仓报单时自动增加冻结数量。当报撤单、出入金时自动增减可用资金。
	
	
	## 智能交易指令
	
	买卖智能开平指令
	
	只关注买卖，不关注平仓还是开仓，优先平仓，无持仓的情况下再开仓。
	
	除了buyOpen、sellClose、sellOpen、buyClose、closeLong、closeShort这些指定了开平方向的指令，其他都是智能开平指令。
