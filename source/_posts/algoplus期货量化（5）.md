---
title: algoplus期货量化（5）
date: 2022-01-19 23:39:13
tags: algoplus
category: algoplus
---

用AlgoPlus构建自己的交易盈亏风控系统

## 止盈止损方法

常用的止盈止损方案有：固定止损、固定止盈、跟踪止损、阶梯止损、保本、时间止损。

## 撤单次数数据结构

    # {"InstrumentID": 0}
	self.order_action_num_dict = {}

	撤单次数是一个以合约名为键值的字典。收到撤单通知后，对撤单次数计算增加。

	收到撤单通知时增加撤单次数计数

	def OnRtnOrder(self, pOrder):
    """
    当收到订单状态变化时，可以在本方法中获得通知。不适宜在回调函数里做比较耗时的操作。
    :param pOrder: AlgoPlus.CTP.ApiStruct中OrderField的实例。
    :return:
    """
    if pOrder.OrderStatus == b"5":
        if pOrder.InstrumentID in self.action_num_dict.keys():
            self.action_num_dict[pOrder.InstrumentID] += 1
        else:
            self.action_num_dict[pOrder.InstrumentID] = 1

## 止盈止损参数数据结构

	# {"InstrumentID": {"Type": []}}
	self.pl_parameter_dict = {}

	止损参数是一个以合约名为键值的字典，根据不同的止损止盈类型存储价差/时间差等参数。其中，止损类型参数取值：
	
	Type止损逻辑b”0″固定止盈b”1″固定止损
	
	价差/时间差等参数以列表形式存储，有些止损逻辑可能需要多个参数。

## 持仓数据结构

    # {"InstrumentID": {"LongVolume": 0, "LongPositionList": [], "ShortVolume": 0, "ShortPositionList": []}}
	self.local_position_dict = {}

    账户持仓以合约名为键值存入字典中，LongVolume统计合约总多头持仓，ShortVolume统计合约总空头持仓，LongPositionList、ShortPositionList以OrderRef为单位存储成交明细。

	成交明细是在AlgoPlus.CTP.ApiStruct中TradeField基础上附加IsLock、AnchorTime、StopProfitDict、StopLossDict、MaxProfitPrice字段。

	rtn_trade["IsLock"] = False # 平仓状态
	rtn_trade["AnchorTime"] = timer() # 成交发生时间
	rtn_trade["StopProfitDict"] = {} # 止盈触发价格，持仓期间实时更新
	rtn_trade["StopLossDict"] = {} # 止损触发价格，持仓期间实时更新

	成交通知

	收到成交通知时放入一个列表中，等待后续处理，避免在此设计复杂的耗时操作。

    def OnRtnTrade(self, pTrade):
    """
    当报单成交时，可以在本方法中获得通知。不适宜在回调函数里做比较耗时的操作。
    :param pTrade: AlgoPlus.CTP.ApiStruct中的TradeField实例。
    :return:
    """
    self.local_rtn_trade_list.append(pTrade.to_dict_raw())

## 处理成交通知

    根据开买卖开平字段将成交成交信息放入持仓数据结构中。


	def process_rtn_trade(self):
	    """
	    从上次订单ID位置开始处理订单数据。
	    :return:
	    """
	    last_rtn_trade_id = len(self.local_rtn_trade_list)
	    for rtn_trade in self.local_rtn_trade_list[self.last_rtn_trade_id:last_rtn_trade_id]:
	        if rtn_trade["InstrumentID"] not in self.instrument_id_registered:
	            self.instrument_id_registered.append(rtn_trade["InstrumentID"])
	
	        rtn_trade["IsLock"] = False
	        rtn_trade["AnchorTime"] = timer()
	        rtn_trade["StopProfitDict"] = {}
	        rtn_trade["StopLossDict"] = {}
	        if rtn_trade["InstrumentID"] not in self.local_position_dict.keys():
	            self.local_position_dict[rtn_trade["InstrumentID"]] = {"LongVolume": 0, "LongPositionList": [], "ShortVolume": 0, "ShortPositionList": []}
	        local_position_info = self.local_position_dict[rtn_trade["InstrumentID"]]
	
	        # 开仓
	        if rtn_trade["OffsetFlag"] == b'0':
	            self.update_stop_price(rtn_trade)
	            if rtn_trade["Direction"] == b'0':
	                local_position_info["LongVolume"] += rtn_trade["Volume"]
	                local_position_info["LongPositionList"].append(rtn_trade)
	            elif rtn_trade["Direction"] == b'1':
	                local_position_info["ShortVolume"] += rtn_trade["Volume"]
	                local_position_info["ShortPositionList"].append(rtn_trade)
	        elif rtn_trade["Direction"] == b'0':
	            local_position_info["ShortVolume"] = max(local_position_info["ShortVolume"] - rtn_trade["Volume"], 0)
	
	        elif rtn_trade["Direction"] == b'1':
	            local_position_info["LongVolume"] = max(local_position_info["LongVolume"] - rtn_trade["Volume"], 0)
	
	    self.last_rtn_trade_id = last_rtn_trade_id
## 实时监控当前行情价格是否触及止盈止损价

遍历持仓数据结构中的所有成交明细，判断最新行情是否触发某个止盈止损阈值，如果触发则录入平仓报单。


	def check_position(self):
	    """
	    检查所有持仓是否触发持仓阈值。
	    """
	    try:
	        for instrument_id, position_info in self.local_position_dict.items():
	            for long_position in position_info["LongPositionList"]:
	                if not long_position["IsLock"]:
	                    trigger = False
	                    order_price = None
	                    for stop_profit in long_position["StopProfitDict"].values():
	                        if self.md_dict[instrument_id]["LastPrice"] &gt; stop_profit:
	                            trigger = True
	                            order_price = self.get_stop_profit_price(instrument_id, long_position["Direction"])
	                            break
	
	                    if not trigger:
	                        for stop_loss in long_position["StopLossDict"].values():
	                            if self.md_dict[instrument_id]["LastPrice"] &lt; stop_loss:
	                                trigger = True
	                                order_price = self.get_stop_loss_price(instrument_id, long_position["Direction"])
	                                break
	
	                    if trigger and order_price:
	                        self.order_ref += 1
	                        self.sell_close(long_position["ExchangeID"], instrument_id, order_price, long_position["Volume"], self.order_ref)
	                        long_position["IsLock"] = True
	
	            for short_position in position_info["ShortPositionList"]:
	                if not short_position["IsLock"]:
	                    trigger = False
	                    order_price = None
	                    for stop_profit in short_position["StopProfitDict"].values():
	                        if self.md_dict[instrument_id]["LastPrice"] &lt; stop_profit:
	                            trigger = True
	                            order_price = self.get_stop_profit_price(instrument_id, short_position["Direction"])
	                            break
	
	                    if not trigger:
	                        for stop_loss in short_position["StopLossDict"].values():
	                            if self.md_dict[instrument_id]["LastPrice"] &gt; stop_loss:
	                                trigger = True
	                                order_price = self.get_stop_loss_price(instrument_id, short_position["Direction"])
	                                break
	
	                    if trigger and order_price:
	                        self.order_ref += 1
	                        self.buy_close(short_position["ExchangeID"], instrument_id, order_price, short_position["Volume"], self.order_ref)
	                        short_position["IsLock"] = True
	    except Exception as err:
	        self._write_log(err)

## 止盈止损逻辑

根据止盈止损（固定止盈、固定止损、跟踪止损、阶梯止损、保本止损）逻辑计算出触发价格，存入成交明细字典中。这里给出了固定止盈和固定止损的例子


	def update_stop_price(self, position_info):
	    """
	    获取止盈止损阈值。止损类型参考https://7jia.com/1002.html
	    :param position_info: 持仓信息
	    :return:
	    """
	    for instrument_id, pl_dict in self.pl_parameter_dict.items():
	        if isinstance(pl_dict, dict):
	            for pl_type, delta in pl_dict.items():
	                # 固定止盈
	                sgn = 1 if position_info["Direction"] == b'0' else -1
	                if pl_type == b"0":
	                    position_info["StopProfitDict"][b"0"] = position_info["Price"] + delta[0] * sgn
	                # 固定止损
	                elif pl_type == b"1":
	                    position_info["StopLossDict"][b"1"] = position_info["Price"] - delta[0] * sgn

1. 行情，目前可获取CTP实时推送的TICK行情，并可合成1minK线
2. 历史数据+实时行情组装策略计算所需要的数据，由实时行情驱动策略，产生交易信号
3. 根据产生的交易信号触发交易
4. 风控模块，即止盈止损，首先需要获取交易账户所有持仓及持仓成本，需要本地维护一个字典，用来记录持仓信息
5. 通知模块，成交通知，目前采用钉钉进行交易信息推送
