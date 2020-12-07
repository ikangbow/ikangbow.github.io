---
title: arctic
date: 2020-12-07 21:16:24
tags: arctic
category: arctic
---

# 北极介绍

北极是位于MongoDB之上的时间序列/数据框架数据库。Arctic支持将许多数据类型序列号以存储在mongo文档模型中。

## 为什么使用北极

- 序列化许多数据类型，例如Pandas DataFrames,Numpy数组,Python对象等,SO不必手动处理不同的数据类型。
- 默认情况下客户端使用LZ4压缩，以节省大量网络/磁盘空间。
- 允许对队形的不同阶段进行版本控制并快照状态（某种程度上类似于git）,并允许自由的试验,然后紧还原快照。【仅限VersionStore】
- 是否为您进行分块（将数据拆分为较小的部分）
- 添加了可以在Mongo的auth上构建的Users and Per User Libraries的概念。
- 拥有不同类型的商店，每种都有其自身的优势。Versionstore允许您对版本和快照内容进行版本控制，TickStore用于存储和高效检索流数据，ChunkStore允许您对数据块进行分块并有效地检索范围。
- 限制对Mongo的数据访问，从而防止对未索引/未分片的集合进行临时查询

## 基本操作

Arctic提供了一个包装器,用于处理与Mongo的连接。该Arctic实际连接到北极。

    conn = Arctic('127.0.0.1')

仅此连接句柄可以执行许多操作。最基本的是list_libraries和initiailize_library。

北极将数据分为不同的库。它们可能是不同的用户，不同的市场，不同的地理区域等。库名称是字符串，完全由用户定义。

    conn.list_libraries()
    []

在这种情况下，系统上没有库，因此可以对其进行初始化。

    >>> conn.initialize_library('library_name')
	>>> conn.list_libraries()
    [u'library_name']

initialize_library有一个名为arg的可选参数，lib_type默认为VersionStore（稍后会在Arctic存储引擎类型中提供更多信息）。

库初始化后，可以像这样访问它：

    >>> lib = conn['library_name']

使用此库的句柄，我们可以开始从Arctic存储和检索数据。

（注意，大多数存储引擎的支持相同的基本方法（read，write，等），但每个人都有自己一套独特的方法为好）

write最基本的形式是北极symbol和数据。的symbol是，用于存储/检索数据的用户定义键。在data多数情况下是一个熊猫数据帧，虽然有些存储引擎支持其他类型的（所有支持dataframes，和一些支持类型的字典，并与pickle对象）。

read如您所料，会symbol读取数据。不同的存储引擎具有不同的参数，这些参数使您可以对数据进行子集设置（稍后会详细介绍）。

    >>> data = pd.DataFrame(.....)
	>>> lib.write('symbolname', data)
	>>> df = lib.read('symbolname')
	>>> df
	                data
	date
	2016-01-01       1
	2016-01-02       2
	2016-01-03       3

其他基本方法：



- library.list_symbols()
   符合您的期望-列出给定库中的所有符号 ['US_EQUITIES', 'EUR_EQUITIES', ...]
- arctic.get_quota(library_name)， arctic.set_quota(library_name, quota_in_bytes)
- 北极内部设置库的配额，这样它们就不会占用太多空间。您可以使用这两种方法检查和设置配额。请注意，这些操作针对 Arctic对象，而非库。

## 北极存储引擎

Arctic的设计具有很高的可扩展性，目前支持多种不同的用例。要了解Arctic的功能，必须了解其使用的存储模型。北极目前支持三个存储引擎

- TickStore
- 版本库
- 块存储

每一个都有各种功能，旨在支持特定和通用的用例。

# 快速开始

## 安装北极

    pip install git+https://github.com/manahl/arctic.git

## 运行一个mongo

    mongod --dbpath <path/to/db_directory>

## 使用VersionStore

	from arctic import Arctic
	import quandl
	
	# Connect to Local MONGODB
	store = Arctic('localhost')
	
	# Create the library - defaults to VersionStore
	store.initialize_library('NASDAQ')
	
	# Access the library
	library = store['NASDAQ']
	
	# Load some data - maybe from Quandl
	aapl = quandl.get("WIKI/AAPL", authtoken="your token here")
	
	# Store the data in the library
	library.write('AAPL', aapl, metadata={'source': 'Quandl'})
	
	# Reading the data
	item = library.read('AAPL')
	aapl = item.data
	metadata = item.metadata

# 存储引擎版本库

VersionStore在MongoDB中序列化并存储Pandas对象，numpy数组以及其他python类型。修改versioneda时，对象是并且创建了新版本symbol。

## 使用VersionStore读写数据

    from arctic import Arctic

	a = Arctic(‘localhost’)
	a.initialize_library('vstore')
	lib = a[‘vstore’]

此时，您有一个空的VersionStore库。您不需要指定存储类型，因为VersionStore是Arctic中的默认库类型。您可以通过几种方式向其中写入数据。最基本的就是使用该write方法。写采用以下参数：

	symbol, data, metadata=None, prune_previous_version=True, **kwargs

symbol是用于在北极存储/检索数据的名称。data是要存储在MongoDB中的数据。metadata是可选的用户定义元数据。它一定是一个dict。prune_previous_versions将删除/删除数据的先前版本（前提是快照中未包含它们）。kwargs被传递给各个写处理程序。有针对不同数据类型的写处理程序。

write用于写入和替换数据。如果test使用一个数据集写入符号，然后使用另一个数据集再次写入符号，则原始数据将被替换为新版本的数据。

	>>> from pandas import DataFrame, MultiIndex
	>>> from datetime import datetime as dt
	
	
	>>> df = DataFrame(data={'data': [1, 2, 3]},
	                   index=MultiIndex.from_tuples([(dt(2016, 1, 1), 1),
	                                                 (dt(2016, 1, 2), 1),
	                                                 (dt(2016, 1, 3), 1)],
	                                                names=['date', 'id']))
	>>> lib.write('test', df)
	VersionedItem(symbol=test,library=arctic.vstore,data=<class 'NoneType'>,version=1,metadata=None,host=127.0.0.1)
	
	>>> lib.read('test').data
	               data
	date       id      
	2016-01-01 1      1
	2016-01-02 1      2
	2016-01-03 1      3
	
	
	>>> df = DataFrame(data={'data': [100, 200, 300]},
	                   index=MultiIndex.from_tuples([(dt(2016, 1, 1), 1),
	                                                 (dt(2016, 1, 2), 1),
	                                                 (dt(2016, 1, 3), 1)],
	                                               names=['date', 'id']))
	
	>>> lib.write('test', df)
	VersionedItem(symbol=test,library=arctic.vstore,data=<class 'NoneType'>,version=2,metadata=None,host=127.0.0.1)
	
	>>> lib.read('test').data
	               data
	date       id      
	2016-01-01 1    100
	2016-01-02 1    200
	2016-01-03 1    300

write返回一个VersionedItem对象。VersionedItem包含以下成员：

符号
图书馆
版本-写入数据的版本号
元数据-元数据（如果存在），或无
数据（对于写入，它为None；对于读取，它包含从数据库读取的数据）。
主办
您还应注意，VersionStorewrite有效覆盖了已写入的数据。在上面的示例中，第二个示例用新数据帧write替换了符号test中的数据。原始数据（版本1）仍然可用，但必须以其版本号引用才能检索它。该read方法采用以下参数：

	symbol, as_of=None, date_range=None, from_version=None, allow_secondary=None, **kwargs

as_of允许您检索特定时间点的数据。您可以通过多种方式定义该时间点。

- 快照的名称（字符串）
- 版本号（整数）
- 日期时间（datetime.datetime）

date_range使您可以通过Arctic DateRange对象对数据进行子集化。DateRanges允许您指定日期范围（“ 2016-01-01”，“ 2016-09-30”），开始日期和结束日期以及开放式范围（无，“ 2016-09-30”）。范围可以在任一端打开。allow_secondary使您可以覆盖默认行为，以允许或禁止从mongo集群的辅助成员读取。

	>>> lib.read('test').data
	               data
	date       id      
	2016-01-01 1    100
	2016-01-02 1    200
	2016-01-03 1    300
	
	>>> lib.read('test', as_of=1).data
	
	date       id  data    
	2016-01-01 1      1
	2016-01-02 1      2
	2016-01-03 1      3
	
	
	>>> from arctic.date import DateRange
	>>> lib.read('test', date_range=DateRange('2016-01-01', '2016-01-01')).data
	
	date       id  data    
	2016-01-01 1    100

DateRange仅适用于pandas DataFrame，并且该数据帧必须具有datetime索引。

写入数据的另一种方法是使用append方法。append接受以下参数：

    symbol, data, metadata=None, prune_previous_version=True, upsert=True, **kwargs

upsert是唯一的新论据。upsert表示如果该符号不存在，它将创建它。如果upsert出现False错误，因为将没有现有数据要追加。

	>>> lib.append('new', df, upsert=False)
	~/arctic/arctic/store/version_store.py in append(self, symbol, data, metadata, prune_previous_version, upsert, **kwargs)
	    505             return self.write(symbol=symbol, data=data, prune_previous_version=prune_previous_version, metadata=metadata)
	    506 
	--> 507         assert previous_version is not None
	    508         dirty_append = False
	    509 
	
	AssertionError: 


>>> lib.append('new', df, upsert=True)
VersionedItem(symbol=new,library=arctic.vstore,data=<class 'NoneType'>,version=1,metadata=None,host=127.0.0.1)

## 实用方法

- 删除
- has_symbol
- list_versions
- read_metadata
- write_metadata
- restore_version

delete符合您的预期-从库中删除符号。它只有一个参数symbol。has_symbol并且list_symbols将返回有关库符号的当前状态信息。他们的签名是：

    list_symbols(self, all_symbols=False, snapshot=None, regex=None, **kwargs)

	def has_symbol(self, symbol, as_of=None)

for list_symbols，all_symbols如果设置为，则将true从所有快照返回所有符号，即使该符号已在当前版本中删除（但已保存在快照中）也是如此。snapshot允许您在指定的下方列出符号snapshot。regex允许您提供正则表达式以进一步限制查询返回的符号列表。北极使用MongoDB的$regex功能。Mongo支持PERL语法正则表达式；更多信息在这里

has_symbol返回True或False基于符号是否存在。您可以version通过将此检查限制为特定检查as_of。

	>>> lib.delete('new')
	
	>>> lib.list_symbols()
	['test']
	
	>>> lib.has_symbol('new')
	False
	
	>>> lib.write('test2', df)
	
	>>> lib.list_symbols(regex=".*2")
	['test2']

read_metadata并write_metadata允许您直接为给定符号读取/设置用户定义的元数据。

	>>> lib.read_metadata('test2')
	VersionedItem(symbol=test2,library=arctic.vstore,data=<class 'NoneType'>,version=1,metadata=None,host=127.0.0.1)
	
	>>> lib.read_metadata('test2').metadata
	
	>>> lib.write_metadata('test2', {'meta': 'data'})
	VersionedItem(symbol=test2,library=arctic.vstore,data=<class 'NoneType'>,version=2,metadata={'meta': 'data'},host=127.0.0.1)
	
	>>> lib.read_metadata('test2').metadata
	{'meta': 'data'}

restore_version使您可以将最新版本设置为旧版本。您可以list_versions用来查看有关版本当前状态的信息。

	>>> lib.list_versions('test') 
	[{'symbol': 'test',
	  'version': 3,
	  'deleted': False,
	  'date': datetime.datetime(2018, 8, 16, 17, 59, 47, tzinfo=tzfile('/usr/share/zoneinfo/America/New_York')),
	  'snapshots': []},
	 {'symbol': 'test',
	  'version': 2,
	  'deleted': False,
	  'date': datetime.datetime(2018, 8, 15, 18, 0, 33, tzinfo=tzfile('/usr/share/zoneinfo/America/New_York')),
	  'snapshots': []}]
	
	
	>>> lib.restore_version('test', 2)
	VersionedItem(symbol=test,library=arctic.vstore,data=<class 'NoneType'>,version=4,metadata=None,host=127.0.0.1)
	
	>>> lib.read('test').data
	               data
	date       id      
	2016-01-01 1    100
	2016-01-02 1    200
	2016-01-03 1    300
	
	>>> lib.list_versions('test') 
	[{'symbol': 'test',
	  'version': 4,
	  'deleted': False,
	  'date': datetime.datetime(2018, 8, 16, 18, 54, 10, tzinfo=tzfile('/usr/share/zoneinfo/America/New_York')),
	  'snapshots': []},
	 {'symbol': 'test',
	  'version': 3,
	  'deleted': False,
	  'date': datetime.datetime(2018, 8, 16, 17, 59, 47, tzinfo=tzfile('/usr/share/zoneinfo/America/New_York')),
	  'snapshots': []},
	 {'symbol': 'test',
	  'version': 2,
	  'deleted': False,
	  'date': datetime.datetime(2018, 8, 15, 18, 0, 33, tzinfo=tzfile('/usr/share/zoneinfo/America/New_York')),
	  'snapshots': []}]

使用restore_version不会删除最新版本，它只是使用用户提供的版本引用的数据创建了一个新版本。

## 快照

VersionStore允许您创建数据快照并为其分配名称。快照中仍包含属于快照的一部分的数据。快照方法是：

- 快照
- delete_snapshot
- list_snapshots

snapshot允许您创建快照。它的签名很简单

	snap_name, metadata=None, skip_symbols=None, versions=None

snap_name是要创建的快照的名称。metadata允许您向快照提供用户定义的元数据。skip_symbols允许您从快照中排除符号。versions允许您指定要包含在快照中的特定版本。

delete_snapshot和list_snapshot功能分别类似于delete和list_versions。list_snapshots返回snapshot映射到metadata快照的名称的字典。

	>>> lib.list_symbols()
	['test', 'test2']
	
	>>> lib.snapshot('backup')
	
	>>> lib.list_snapshots()
	{'backup': None}
	
	>>> lib.list_symbols(snapshot='backup')
	['test', 'test2']
	
	>>> lib.delete('test')
	
	>>> lib.delete('test2')
	
	>>> lib.list_symbols()
	[]
	
	>>> lib.list_symbols(snapshot='backup')
	['test', 'test2']
	
	>>> lib.read('test')
	~/arctic/arctic/store/version_store.py in _read_metadata(self, symbol, as_of, read_preference)
	    455         metadata = _version.get('metadata', None)
	    456         if metadata is not None and metadata.get('deleted', False) is True:
	--> 457             raise NoDataFoundException("No data found for %s in library %s" % (symbol, self._arctic_lib.get_name()))
	    458 
	    459         return _version
	
	NoDataFoundException: No data found for test in library arctic.vstore
	
	
	>>> lib.read('test', as_of='backup')
	VersionedItem(symbol=test,library=arctic.vstore,data=<class 'pandas.core.frame.DataFrame'>,version=4,metadata=None,host=127.0.0.1)
	
	>>> lib.read('test', as_of='backup').data 
	               data
	date       id      
	2016-01-01 1    100
	2016-01-02 1    200
	2016-01-03 1    300


# API参考

附加

	append(symbol, item)
	    Appends data from item to symbol's data in the database.
	
	    Is not idempotent
	
	    Parameters
	    ----------
	    symbol: str
	        the symbol for the given item in the DB
	    item: DataFrame or Series
	        the data to append

用法示例：

	>>> df = DataFrame(data={'data': [100, 200, 300]},
	                  index=MultiIndex.from_tuples([(dt(2016, 1, 1), 1),
	                                                (dt(2016, 1, 2), 1),
	                                                (dt(2016, 1, 3), 1)],
	                                               names=['date', 'id']))


	>>> lib.write('test', df, chunk_size='M')
	>>> lib.read('test')
	               data
	date       id
	2016-01-01 1    100
	2016-01-02 1    200
	2016-01-03 1    300
	
	>>> lib.append('test', df)
	>>> lib.append('test', df)
	>>> lib.read('test')
	               data
	date       id
	2016-01-01 1    100
	           1    100
	           1    100
	2016-01-02 1    200
	           1    200
	           1    200
	2016-01-03 1    300
	           1    300
	           1    300

删除

	delete(symbol, chunk_range=None)
	    Delete all chunks for a symbol, or optionally, chunks within a range
	
	    Parameters
	    ----------
	    symbol : str
	        symbol name for the item
	    chunk_range: range object
	        a date range to delete

用法示例：

	>>> lib.read('test')
	               data
	date       id
	2016-01-01 1    100
	2016-01-03 1    300
	
	>>> lib.delete('test', chunk_range=pd.date_range('2016-01-01', '2016-01-01'))
	>>> lib.read('test')
	               data
	date       id
	2016-01-03 1    300

get_chunk_ranges

	get_chunk_ranges(symbol, chunk_range=None, reverse=False)
	    Returns a generator of (Start, End) tuples for each chunk in the symbol
	
	    Parameters
	    ----------
	    symbol: str
	        the symbol for the given item in the DB
	    chunk_range: None, or a range object
	        allows you to subset the chunks by range
	    reverse: boolean
	
	    Returns
	    -------
	    generator

用法示例：

	>>> list(lib.get_chunk_ranges('new_name'))
	[('2016-01-01', '2016-01-01'), ('2016-01-03', '2016-01-03')]

获取信息

	get_info(symbol)
	    Returns information about the symbol, in a dictionary
	
	    Parameters
	    ----------
	    symbol: str
	        the symbol for the given item in the DB
	
	    Returns
	    -------
	    dictionary

迭代器

	iterator(symbol, chunk_range=None):
	    Returns a generator that accesses each chunk in ascending order
	
	    Parameters
	    ----------
	    symbol: str
	        the symbol for the given item in the DB
	    chunk_range: None, or a range object
	        allows you to subset the chunks by range
	
	    Returns
	    -------
	    generator

list_symbols

	list_symbols()
	    Returns all symbols in the library
	
	    Returns
	    -------
	    list of str

读

	read(symbol, chunk_range=None, filter_data=True, **kwargs)
	    Reads data for a given symbol from the database.
	
	    Parameters
	    ----------
	    symbol: str
	        the symbol to retrieve
	    chunk_range: object
	        corresponding range object for the specified chunker (for
	        DateChunker it is a DateRange object or a DatetimeIndex,
	        as returned by pandas.date_range
	    filter_data: boolean
	        perform chunk level filtering on the data (see filter in _chunker)
	        only applicable when chunk_range is specified
	    kwargs: ?
	        values passed to the serializer. Varies by serializer
	
	    Returns
	    -------
	    DataFrame or Series

用法示例：


	>>> dr = pd.date_range(start='2010-01-01', periods=1000, freq='D')
	>>> df = DataFrame(data={'data': np.random.randint(0, 100, size=1000),
	                         'date': dr
	                        })
	
	>>> lib.write('symbol_name', df, chunk_size='M')
	>>> lib.read('symbol_name', chunk_range=pd.date_range('2012-09-01', '2016-01-01'))
	    data       date
	0     61 2012-09-01
	1     69 2012-09-02
	2     96 2012-09-03
	3     23 2012-09-04
	4     66 2012-09-05
	5     54 2012-09-06
	6     21 2012-09-07
	7     92 2012-09-08
	8     95 2012-09-09
	9     24 2012-09-10
	10    87 2012-09-11
	11    33 2012-09-12
	12    59 2012-09-13
	13    54 2012-09-14
	14    48 2012-09-15
	15    67 2012-09-16
	16    73 2012-09-17
	17    72 2012-09-18
	18     6 2012-09-19
	19    24 2012-09-20
	20     8 2012-09-21
	21    50 2012-09-22
	22    40 2012-09-23
	23    45 2012-09-24
	24     8 2012-09-25
	25    73 2012-09-26

改名

	rename(from_symbol, to_symbol)
	    Rename a symbol
	
	    Parameters
	    ----------
	    from_symbol: str
	        the existing symbol that will be renamed
	    to_symbol: str
	        the new symbol name

reverse_iterator

	reverse_iterator(symbol, chunk_range=None):
	    Returns a generator that accesses each chunk in descending order
	
	    Parameters
	    ----------
	    symbol: str
	        the symbol for the given item in the DB
	    chunk_range: None, or a range object
	        allows you to subset the chunks by range
	
	    Returns
	    -------
	    generator

更新

	update(symbol, item, chunk_range=None, upsert=False, **kwargs)
	    Overwrites data in DB with data in item for the given symbol.
	
	    Is idempotent
	
	    Parameters
	    ----------
	    symbol: str
	        the symbol for the given item in the DB
	    item: DataFrame or Series
	        the data to update
	    chunk_range: None, or a range object
	        If a range is specified, it will clear/delete the data within the
	        range and overwrite it with the data in item. This allows the user
	        to update with data that might only be a subset of the
	        original data.
	    upsert: bool
	        if True, will write the data even if the symbol does not exist.
	    kwargs:
	        optional keyword args passed to write during an upsert. Includes:
	        chunk_size
	        chunker

写

	write(symbol, item, chunker=DateChunker(), **kwargs)
	    Writes data from item to symbol in the database
	
	    Parameters
	    ----------
	    symbol: str
	        the symbol that will be used to reference the written data
	    item: Dataframe or Series
	        the data to write the database
	    chunker: Object of type Chunker
	        A chunker that chunks the data in item
	    kwargs:
	        optional keyword args that are passed to the chunker. Includes:
	        chunk_size:
	            used by chunker to break data into discrete chunks.
	            see specific chunkers for more information about this param.

用法示例：

	>>> dr = pd.date_range(start='2010-01-01', periods=1000, freq='D')
	>>> df = DataFrame(data={'data': np.random.randint(0, 100, size=1000),
	                         'date': dr
	                        })

	>>> lib.write('symbol_name', df, chunk_size='M')





