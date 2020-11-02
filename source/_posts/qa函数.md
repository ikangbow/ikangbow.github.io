---
title: qa函数
date: 2020-08-29 11:08:26
tags: 投资
category: 投资
---
## 数据

## 下单

### QA_Account()
QA_Account() 是quantaxis的核心类, 其作用是一个可以使用规则兼容各种市场的账户类
1.3.0以后, QA_Account需要由组合来进行创建(推荐)

	调用方式
    import QUANTAXIS as QA
	user = QA.QA_User(username ='quantaxis', password = 'quantaxis')
	portfolio=user.new_portfolio('x1')
	account = *portfolio.new_account*(account_cookie='test')

	QA_AccountPRO?
	Init signature:
	QA_AccountPRO(
	    user_cookie:str,
	    portfolio_cookie:str,
	    account_cookie=None,
	    strategy_name=None,
	    market_type='stock_cn',
	    frequence='day',
	    broker='backtest',
	    init_hold={},
	    init_cash=1000000,
	    commission_coeff=0.00025,
	    tax_coeff=0.001,
	    margin_level={},
	    allow_t0=False,
	    allow_sellopen=False,
	    allow_margin=False,
	    running_environment='backtest',
	    auto_reload=False,
	    generated='direct',
	    start=None,
	    end=None,
	)
	Docstring:     
	JOB是worker 需要接受QA_EVENT 需要完善RUN方法
	👻QA_Broker 继承这个类
	👻QA_Account 继承这个类
	👻QA_OrderHandler 继承这个类
	这些类都要实现run方法，在其它线程🌀中允许自己的业务代码
	File:           /usr/local/lib/python3.6/site-packages/QUANTAXIS/QAARP/QAAccountPro.py
	Type:           type
	Subclasses:     
	accpro = *portfolio.new_accountpro*('pro2',market_type=QA.MARKET_TYPE.STOCK_CN)
	accpro.positions
	{}
	*accpro.receive_deal*?
	Signature:
	accpro.receive_deal(
	    code,
	    trade_id:str,
	    order_id:str,
	    realorder_id:str,
	    trade_price,
	    trade_amount,
	    trade_towards,
	    trade_time,
	    message=None,
	)
	Docstring: <no docstring>
	File:      /usr/local/lib/python3.6/site-packages/QUANTAXIS/QAARP/QAAccountPro.py
	Type:      method
	`accpro.receive_deal`('000001','001','001','0001',trade_price=12,trade_amount=10000,trade_towards=QA.ORDER_DIRECTION.BUY,trade_time='2020-08-18')
	*accpro.positions*
	{'000001': < QAPOSITION 000001 amount 10000/0 >}
	*accpro.history_table*


	*accpro.send_order?*
	Signature:
	accpro.send_order(
	    code=None,
	    amount=None,
	    time=None,
	    towards=None,
	    price=None,
	    money=None,
	    order_model='LIMIT',
	    amount_model='by_amount',
	    order_id=None,
	    position_id=None,
	    *args,
	    **kwargs,
	)
	Docstring: <no docstring>
	File:      /usr/local/lib/python3.6/site-packages/QUANTAXIS/QAARP/QAAccountPro.py
	Type:      method

	基于期货市场的账户初始化
	future_account = portfolio.new_account(account_cookie ='future',allow_t0=True,allow_margin=True,allow_sellopen=True, running_environment=QA.MARKET_TYPE.FUTURE_CN)

	account的其他属性可以.出来，可以自己试着看
	*accpro.cash*
	[1000000, 879970.0, 779945.0]
	*accpro.cash_available*
	779945.0
	*accpro.get_history*
	*accpro.get_position*('000001')

	pos2 = accpro.get_position('000002')

	*pos2.message*
	{'code': '000002',
	 'instrument_id': '000002',
	 'user_id': 'pro2',
	 'portfolio_cookie': 'x1',
	 'username': 'quantaxis',
	 'position_id': '3e2cd89e-7143-4e0e-b344-40571e84eda5',
	 'account_cookie': 'pro2',
	 'frozen': {},
	 'name': None,
	 'spms_id': None,
	 'oms_id': None,
	 'market_type': 'stock_cn',
	 'exchange_id': None,
	 'moneypreset': 100000,
	 'moneypresetLeft': 0.0,
	 'lastupdatetime': '',
	 'volume_long_today': 5000,
	 'volume_long_his': 0,
	 'volume_long': 5000,
	 'volume_short_today': 0,
	 'volume_short_his': 0,
	 'volume_short': 0,
	 'volume_long_frozen_today': 0,
	 'volume_long_frozen_his': 0,
	 'volume_long_frozen': 0,
	 'volume_short_frozen_today': 0,
	 'volume_short_frozen_his': 0,
	 'volume_short_frozen': 0,
	 'margin_long': 100000.0,
	 'margin_short': 0,
	 'margin': 100000.0,
	 'position_price_long': 20.0,
	 'position_cost_long': 100000.0,
	 'position_price_short': 0,
	 'position_cost_short': 0.0,
	 'open_price_long': 20.0,
	 'open_cost_long': 100000.0,
	 'open_price_short': 0,
	 'open_cost_short': 0.0,
	 'trades': [],
	 'orders': {},
	 'last_price': 20,
	 'float_profit_long': 0.0,
	 'float_profit_short': 0.0,
	 'float_profit': 0.0,
	 'position_profit_long': 0.0,
	 'position_profit_short': 0.0,
	 'position_profit': 0.0}

	*pos2.volume_long*
	5000
	*pos2.volume_long_today*
	5000
	*pos2.volume_long_his*
	0
	*pos2.last_price*
	20
	*pos2.on_price_change*(21)
	if pos2.float_profit_long > 2000:
	    print('sell')
	sell

### QIFIAccount

	from QIFIAccount import QIFI_Account
	aifiacc = QIFI_Account(username='x2',password='x2',)
	aifiacc.initial()
	Create new Account
	aifiacc?
	Type:           QIFI_Account
	String form:    <QIFIAccount.QAQIFIAccount.QIFI_Account object at 0x7f6e3b2dfc18>
	File:           /usr/local/lib/python3.6/site-packages/QIFIAccount/QAQIFIAccount.py
	Docstring:      <no docstring>
	Init docstring:
	Initial
	QIFI Account是一个基于 DIFF/ QIFI/ QAAccount后的一个实盘适用的Account基类
	
	
	1. 兼容多持仓组合
	2. 动态计算权益
	
	使用 model = SIM/ REAL来切换


	sr = aifiacc.send_order('003',10,12,ORDER_DIRECTION.BUY)
	aifiacc.send_order?
	Signature:
	aifiacc.send_order(
	    code:str,
	    amount:float,
	    price:float,
	    towards:int,
	    order_id:str='',
	)
	Docstring: <no docstring>
	File:      /usr/local/lib/python3.6/site-packages/QIFIAccount/QAQIFIAccount.py
	Type:      method
	sr = aifiacc.send_order('004',10.0,12.0,QA.ORDER_DIRECTION.BUY)
	{'volume_long': 0, 'volume_short': 0, 'volume_long_frozen': 0, 'volume_short_frozen': 0}
	{'volume_long': 0, 'volume_short': 0}
	order check success
	下单成功 b3f155c7-ab2b-4ac1-94b7-08b6af152738
	td = aifiacc.make_deal(sr)
	全部成交 b3f155c7-ab2b-4ac1-94b7-08b6af152738
	update trade