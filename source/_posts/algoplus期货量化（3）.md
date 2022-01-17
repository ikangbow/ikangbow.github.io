---
title: algoplus期货量化（3）
date: 2022-01-17 23:40:19
tags: algoplus
category: algoplus
---

## 使用AlgoPlus接收期货实时行情

关于CTP


CTP是Comprehensive Transaction Platform的简称。CTP有MdApi和TraderApi两个独立的开放接口。

MdApi负责行情相关操作（订阅、接收）。

![](234200.png)

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