---
title: mongo
date: 2020-05-23 08:51:04
tags: mongo
category: mongo
---

# 什么是MongoDB ?

MongoDB 是由C++语言编写的，是一个基于分布式文件存储的开源数据库系统。
在高负载的情况下，添加更多的节点，可以保证服务器性能。
MongoDB 旨在为WEB应用提供可扩展的高性能数据存储解决方案。
MongoDB 将数据存储为一个文档，数据结构由键值(key=>value)对组成。MongoDB 文档类似于 JSON 对象。字段值可以包含其他文档，数组及文档数组。

NoSQL，指的是非关系型的数据库。NoSQL有时也称作Not Only SQL的缩写，是对不同于传统的关系型数据库的数据库管理系统的统称。
NoSQL用于超大规模数据的存储。（例如谷歌或Facebook每天为他们的用户收集万亿比特的数据）。这些类型的数据存储不需要固定的模式，无需多余操作就可以横向扩展。

# 主要特点

- MongoDB 是一个面向文档存储的数据库，操作起来比较简单和容易。

- 你可以在MongoDB记录中设置任何属性的索引 (如：FirstName=”Sameer”,Address=”8 Gandhi Road”)来实现更快的排序。

- 你可以通过本地或者网络创建数据镜像，这使得MongoDB有更强的扩展性。

- 如果负载的增加（需要更多的存储空间和更强的处理能力） ，它可以分布在计算机网络中的其他节点上这就是所谓的分片。

- Mongo支持丰富的查询表达式。查询指令使用JSON形式的标记，可轻易查询文档中内嵌的对象及数组。

- MongoDb 使用update()命令可以实现替换完成的文档（数据）或者一些指定的数据字段 。

- Mongodb中的Map/reduce主要是用来对数据进行批量处理和聚合操作。

- Map和Reduce。Map函数调用emit(key,value)遍历集合中所有的记录，将key与value传给Reduce函数进行处理。

- Map函数和Reduce函数是使用Javascript编写的，并可以通过db.runCommand或mapreduce命令来执行MapReduce操作。

- GridFS是MongoDB中的一个内置功能，可以用于存放大量小文件。

- MongoDB允许在服务端执行脚本，可以用Javascript编写某个函数，直接在服务端执行，也可以把函数的定义存储在服务端，下次直接调用即可。

- MongoDB支持各种编程语言:RUBY，PYTHON，JAVA，C++，PHP，C#等多种语言。

- MongoDB安装简单。

  # 安装

1. 下载mongodb-linux-x86_64-3.6.3.tar.gz

2. 解压tar -zxvf mongodb-linux-x86_64-3.0.6.tgz

3. 在与bin同级的目录下建立data文件夹

4. mongo.config文件内容

   ```
   port=27017
   logpath=/airthink/mongodb-linux-x86_64-3.6.3/mongod.log
   pidfilepath=/airthink/mongodb-linux-x86_64-3.6.3/mongod.pid
   logappend=true
   fork=true
   maxConns=3000
   dbpath=/airthink/mongodb-linux-x86_64-3.6.3/data
   auth=true
   ```

5. 启动./mongodb-linux-x86_64-3.6.3/bin/mongo

   ```
   var schema = db.system.version.findOne({_id:"authSchema"});//驱动版本5--SCRAM-SHA-1需要修改成：3--Mongodb-CR
   schema. currentVersion = 3;
   db.system.version.save(schema);
   
   db.createUser({user:"admin",pwd:"mypass",roles:["readWriteAnyDatabase", "userAdminAnyDatabase", "dbAdminAnyDatabase"]});
   
   windows下的启动命令到bin目录下  start mongod
   ```

   # MongoDB用户角色配置

## 基本知识介绍

MongoDB基本的角色

1. 数据库用户角色：read、readWrite;
2. 数据库管理角色：dbAdmin、dbOwner、userAdmin；
3. 集群管理角色：clusterAdmin、clusterManager、clusterMonitor、hostManager；
4. 备份恢复角色：backup、restore；
5. 所有数据库角色：readAnyDatabase、readWriteAnyDatabase、userAdminAnyDatabase、dbAdminAnyDatabase
6. 超级用户角色：root (这里还有几个角色间接或直接提供了系统超级用户的访问（dbOwner 、userAdmin、userAdminAnyDatabase,其中MongoDB默认是没有开启用户认证的，也就是说游客也拥有超级管理员的权限。userAdminAnyDatabase：有分配角色和用户的权限，但没有查写的权限)

## 连接到MongoDB服务器

```
建库操作开始

./mongodb-linux-x86_64-3.6.3/bin/mongo   进入mongo shell

use admin(使用)

db.createUser({user:"root",pwd:"password",roles:["root"]})//创建root用户

db.createUser({user:"admin",pwd:"admin",roles:[{role:"userAdminAnyDatabase",db:"admin"}]})（创建admin用户）

db.auth('admin', 'admin');（授权）

修改mongod.conf文件

security:
authorization: enabled//启用授权

重启MongoDB服务器

service mongod restart

创建数据库读写权限用户

use admin
db.auth("admin","password");
use ballmatch
db.createUser({
    user: "baidu",
    pwd: "password",
    roles: [{role: "readWrite",db: "baidu"}]
})
```

## Java程序连接MongoDB

```
MongoCredential credential = MongoCredential.createCredential("username", "dbName", "password".toCharArray());
ServerAddress serverAddress = new ServerAddress("192.168.10.242", 27017);
MongoClient mongoClient = new MongoClient(serverAddress, Arrays.asList(credential));
DB db = mongoClient.getDB("dbName");
returndb;

//方式二
String sURI = String.format("mongodb://%s:%s@%s:%d/%s", "username", "password", "192.168.10.242", 27017, "dbName"); 
MongoClientURI uri = new MongoClientURI(sURI); 
MongoClient mongoClient = new MongoClient(uri); 
DB db = mongoClient.getDB("dbName");
```

## 命令介绍

```
修改用户密码
db.updateUser( "admin",{pwd:"password"});

密码认证
db.auth("admin","password");

MongoDB连接信息查询
db.serverStatus().connections;

关闭MongoDB服务
use admin;
db.shutdownServer();

删除用户
删除用户(需要root权限，会将所有数据库中的football用户删除)
db.system.users.remove({user:"baidu"});

删除用户(权限要求没有那么高，只删除本数据中的football用户)
db.dropUser("baidu");
```

# mongoTemplate使用示例

## 对匹配到的数据进行更新(inc)

```
Query query = new Query();
query.addCriteria(Criteria.where("accountName").is(username));
if (null != password){
        query.addCriteria(Criteria.where("accountPwd").is(password));
}
Account account = this.mongoTemplate.findOne(query, Account.class);
if (account != null) {
    Update update = new Update();
    update.inc("loginTimes", 1);
    this.mongoTemplate.updateFirst(query, update, Account.class);//查找并更新
}


mongoTemplate.updateFirst(Query.query(Criteria.where("_id").is(favor.getTypeId())),new Update().inc("favorTimes", 1), News.class);

mongoTemplate.updateFirst(Query.query(Criteria.where("_id").is(favor.getTypeId())),new Update().inc("favorTimes", -1), News.class);
```

## 查询一条记录和保存（findOne/save）

```
Account account = mongoTemplate.findOne(Query.query(Criteria.where("accountName").is(accountName)),Account.class);

account.setAccountPwd(passwd);

mongoTemplate.save(account);
```

## 多条件匹配（并的关系）

```
Criteria criteria = new Criteria();
criteria.andOperator(Criteria.where("phone").is(phone), Criteria.where("status").is(0));
List<VerifyCode> codes = mongoTemplate.find(Query.query(criteria), VerifyCode.class);

Criteria criteria = new Criteria();
criteria.andOperator(Criteria.where("entityID").is(entityID), Criteria.where("accountType").is(accountType), Criteria.where("weUnionId").ne("").ne(null));
Account Account = mongoTemplate.findOne(Query.query(criteria), Account.class);

Boolean isFavor = mongoTemplate.exists(Query.query(Criteria.where("typeId").is(favor.getTypeId()).and("type").is(favor.getType()).and("userId").is(favor.getUserId())), Favor.class);
```

## 多条件匹配（或的关系）

```
Criteria criteria = new Criteria();
criteria.orOperator(Criteria.where("status").is(1), Criteria.where("status").is(0));
List<VerifyCode> codes = mongoTemplate.find(Query.query(criteria), VerifyCode.class);
```

## 根据id查找并删除

```
Account account = mongoTemplate.findById(id, Account.class); 
mongoTemplate.remove(account);
```

## 计数

```
long count = mongoTemplate.count(Query.query(Criteria.where("accountName").is(accountName)), Account.class);
```

## 模糊匹配并排序

```
Criteria criteria = new Criteria();
criteria.andOperator(Criteria.where("province").regex(province));//正则
mongoTemplate.find(Query.query(criteria).with(new Sort(Direction.ASC, "order")), Bbatch.class);
```

## distinct查询

```
 DBObject dbObject = new BasicDBObject("schoolId", bschool.get_id());
List<String> provinces = mongoTemplate.getCollection(
mongoTemplate.getCollectionName(BschoolScoreline.class)) .distinct("province", dbObject);
bschool.setProvinces(provinces);
```

## 查找并所有排序

```
mongoTemplate.find(new Query().with(new Sort(Direction.ASC, "order","createTime")), Bschool.class);
```

## in查询

```
List<MpropValue> mpropValueList = mongoTemplate.find(Query.query(Criteria.where("propId").in(mprop1.get_id())), MpropValue.class);
```

## 设置唯一字段

```
//used 是1 将其他所有是1的更新为0
    if(address.getUsed()==1){
    Query q = new Query(Criteria.where("userId").is(user.get_id()));//查询
    Update u = new Update().set("used",0);
    BulkOperations ops = mongoTemplate.bulkOps(BulkOperations.BulkMode.UNORDERED, "address");
    ops.updateMulti(q,u);//全部更新
    ops.execute();
}
```

## 插入多条数据

```
public ReturnStatus upsert(List<BspecialLink> specialLinks) {
    ReturnStatus status;
    mongoTemplate.insertAll(specialLinks);
    status = new ReturnStatus(true);
    return status;
}
```

## 获取数据库所有集合名称

```
Set<String> sets = mongoTemplage.getDb().getCollectionNames();
```

## 查询当天数据

    private static Date getStartTime() {
	    Calendar todayStart = Calendar.getInstance();
	    todayStart.set(Calendar.HOUR_OF_DAY, 0);
	    todayStart.set(Calendar.MINUTE, 0);
	    todayStart.set(Calendar.SECOND, 0);
	    todayStart.set(Calendar.MILLISECOND, 0);
	    return todayStart.getTime();
	}
	
	private static Date getEndTime() {
	    Calendar todayEnd = Calendar.getInstance();
	    todayEnd.set(Calendar.HOUR_OF_DAY, 23);
	    todayEnd.set(Calendar.MINUTE, 59);
	    todayEnd.set(Calendar.SECOND, 59);
	    todayEnd.set(Calendar.MILLISECOND, 999);
	    return todayEnd.getTime();
	}
	
	Criteria criteria = new Criteria();
	criteria.andOperator(
	        Criteria.where("type").is(type),
	        Criteria.where("typeId").is(typeId),
	        Criteria.where("userId").is(user.get_id()),
	        Criteria.where("createTime").gte(getStartTime())
	                .lte(getEndTime()));
	return mongoTemplate.exists(Query.query(criteria), Zan.class);
# 其他

1. 删除字段

   ```
   db.yourcollection.update({},{$unset:{"需要删除的字段":""}},false,true)
   ```

2. 将匹配到的值全部更新

   ```
   db.getCollection("bschool").update({"state":0},{$set:{"state":"1"}},{'multi':true}) 
   ```

   db.getCollection(“pdfChapterItem”).update({}, {$rename : {“remark” : “abbr”}}, false, true)//更新所有字段名

3. 建立索引

   ```
   db.bschoolScorelineDetail.ensureIndex({"schoolId":1,"type":1,"batch":1,"province":1},{"name"："schoolScorelineDetail"})
   ```

   mongo中建立索引用ensureIndex({“字段1”:1,”字段2”:”-1”},{“name”:”索引名”}) 1和-1代表正序和倒序

4. mongo数据库备份操作(在mongo的安装目录的bin下执行如下命令)

   ```
   不带验证的数据库备份：
   
   mongodump -d mrmf -o /tools/test   mrmf数据库名   /tools/test要备份到的地方
   
   不带验证的数据库恢复
   
   mongorestore --db mrmf /tools/test/mrmf   mrmf数据库名   /tools/test要恢复的数据
   
   带验证的数据库备份：
   
   mongodump -d mrmf -u admin -p "ie8*kskIkd123" --authenticationDatabase=admin -o /tools/test  
   
   带验证的数据库恢复：
   
   mongorestore --db mrmf -u admin -p "ie8*kskIkd123" --authenticationDatabase=admin /tools/test/red
   ```

5. 开启慢日志

   ```
   在客户端调用db.setProfilingLevel(级别) 命令来实时配置。可以通过db.getProfilingLevel()命令来获取当前的Profile级别。
   
   > db.setProfilingLevel(2); 
   > {"was" : 0 , "ok" : 1} 
   > db.getProfilingLevel()
   上面斜体的级别可以取0，1，2 三个值，他们表示的意义如下：
   0 – 不开启
   1 – 记录慢命令 (默认为>100ms)
   2 – 记录所有命令
   Profile 记录在级别1时会记录慢命令，那么这个慢的定义是什么?上面我们说到其默认为100ms，当然有默认就有设置，其设置方法和级别一样有两种，一种是通过添加–slowms启动参数配置。第二种是调用db.setProfilingLevel时加上第二个参数：
   db.setProfilingLevel( level , slowms ) 
   db.setProfilingLevel( 1 , 10 );
   Mongo Profile 记录是直接存在系统db里的，记录位置 system.profile ，所以，我们只要查询这个Collection的记录就可以获取到我们的 Profile 记录了。
   
   Profile 信息内容详解：
   
   ts-该命令在何时执行.
   
   millis Time-该命令执行耗时，以毫秒记.
   
   info-本命令的详细信息.
   
   query-表明这是一个query查询操作.
   
   ntoreturn-本次查询客户端要求返回的记录数.比如, findOne()命令执行时 ntoreturn 为 1.有limit(n) 条件时ntoreturn为n.
   
   query-具体的查询条件(如x>3).
   
   nscanned-本次查询扫描的记录数.
   
   reslen-返回结果集的大小.
   
   nreturned-本次查询实际返回的结果集.
   
   update-表明这是一个update更新操作.
   
   upsert-表明update的upsert参数为true.此参数的功能是如果update的记录不存在，则用update的条件insert一条记录.
   ```

　　moved-表明本次update是否移动了硬盘上的数据，如果新记录比原记录短，通常不会移动当前记录，如果新记录比原记录长，那么可能会移动记录到其它位置，这时候会导致相关索引的更新.磁盘操作更多，加上索引更新，会使得这样的操作比较慢.

　　insert-这是一个insert插入操作.

　　getmore-这是一个getmore 操作，getmore通常发生在结果集比较大的查询时，第一个query返回了部分结果，后续的结果是通过getmore来获取的。

```
#下面是一个超过200ms的查询语句
{ 
"op" : "query",                     #操作类型，有insert、query、update、remove、getmore、command  
"ns" : "F10data3.f10_2_8_3_jgcc", 
"query" : {                         #具体的查询语句 包括过滤条件，limit行数  排序字段
    filter" : {
        "jzrq" : {
            "$gte" : ISODate("2017-03-31T16:00:00.000+0000"), 
            "$lte" : ISODate("2017-06-30T15:59:59.000+0000")
        }, 
        "jglxfldm" : 10.0
    }, 
    "ntoreturn" : 200.0,    
    "sort" : {                      #如果有排序  则显示排序的字段 这里是 RsId
        "RsId" : 1.0
    }
}, 
"keysExamined" : 0.0,               #索引扫描数量 这里是全表扫描，没有用索引 所以是 0
"docsExamined" : 69608.0,           #浏览的文档数 这里是全表扫描 所以是整个collection中的全部文档数
"numYield" : 546.0,                 #该操作为了使其他操作完成而放弃的次数。通常来说，当他们需要访问
                                    还没有完全读入内存中的数据时，操作将放弃。这使得在MongoDB为了
                                    放弃操作进行数据读取的同时，还有数据在内存中的其他操作可以完成。
"locks" : {                         #锁信息，R：全局读锁；W：全局写锁；r：特定数据库的读锁；w：特定数据库的写锁
    "Global" : {
        "acquireCount" : {
            "r" : NumberLong(1094)  #该操作获取一个全局级锁花费的时间。
        }
    }, 
    "Database" : {
        "acquireCount" : {
            "r" : NumberLong(547)  
        }
    }, 
    "Collection" : {
        "acquireCount" : {
            "r" : NumberLong(547)
        }
    }
}, 
"nreturned" : 200.0,                #返回的文档数量
"responseLength" : 57695.0,         #返回字节长度，如果这个数字很大，考虑值返回所需字段
"millis" : 264.0,                   #消耗的时间（毫秒）
"planSummary" : "COLLSCAN, COLLSCAN", #执行概览 从这里看来 是全表扫描 
"execStats" : {                       #详细的执行计划 这里先略过 后续可以用 explain来具体分析
}, 
"ts" : ISODate("2017-08-24T02:32:49.768+0000"),  #命令执行的时间
"client" : "10.3.131.96",                        #访问的ip或者主机
"allUsers" : [
], 
"user" : ""
}
```

MongoDB 查询优化

1. 　如果发现 millis 值比较大，那么就需要作优化。
2. 　如果docsExamined数很大，或者接近记录总数（文档数），那么可能没有用到索引查询，而是全表扫描。
3. 　如果keysExamined数为0，也可能是没用索引。
4. 　结合 planSummary 中的显示，上例中是 “COLLSCAN, COLLSCAN” 确认是全表扫描
5. 　如果 keysExamined 值高于 nreturned 的值，说明数据库为了找到目标文档扫描了很多文档。这时可以考虑创建索引来提高效率。
6. 　索引的键值选择可以根据 query 中的输出参考，上例中 filter:包含了 jzrq和jglxfldm 并且按照RsId排序，所以 我们的索引索引可以这么建: db.f10_2_8_3_jgcc.ensureindex({jzrq:1,jglxfldm:1,RsId:1})

# 聚合操作

```
示例一
Criteria criteria = new Criteria();
criteria.andOperator(Criteria.where("userId").is(uid), Criteria.where("type").is(2));//查询条件
Aggregation aggregation = Aggregation.newAggregation(Aggregation.match(criteria),Aggregation.group("content").count().as("count"),Aggregation.sort(Sort.Direction.ASC,"count"));
AggregationResults<SearchKeyWords> ar = mongoTemplate.aggregate(aggregation,
Searched.class, SearchKeyWords.class);//Searched为要查询的类，SearchKeyWords为查询结果的vo对象
List<SearchKeyWords> list = ar.getMappedResults();

public class SearchKeyWords {

    public String _id;//_id: "中华"  重复的字段
    public int count;//count: 57   重复次数
    public int getCount() {
        return count;
    }
    public void setCount(int count) {
        this.count = count;
    }
}
```

![](aggregation.png)

```
示例二
//分组group、排序sort统计每个分类下商品个数
Criteria criteria = new Criteria();
Aggregation aggregation = Aggregation.newAggregation(Aggregation.match(criteria),
        Aggregation.group("categoryId").count().as("count"),Aggregation.sort(Sort.Direction.ASC,"count"));

AggregationResults<BasicDBObject> ar = mongoTemplate.aggregate(aggregation,
        Goods.class, BasicDBObject.class);
for (BasicDBObject basicDBObject : ar) {
    logger.info(JsonUtils.toJson(basicDBObject));
}
//输出结果
2019-05-06 11:13:11 [com.badou.service.basic.goods.GoodsService]-[INFO] {"_id":"6430379335205566832","count":1}//分类id和当前分类下商品个数
2019-05-06 11:13:11 [com.badou.service.basic.goods.GoodsService]-[INFO] {"_id":"8732390723817262538","count":2}
2019-05-06 11:13:11 [com.badou.service.basic.goods.GoodsService]-[INFO] {"_id":"4479460853249390323","count":2}
2019-05-06 11:13:11 [com.badou.service.basic.goods.GoodsService]-[INFO] {"_id":"6571438787886532072","count":16}


示例三
//两个集合关联查询 Aggregation.lookup
//若 A集合关联B集合
//参数： from B集合名
//localField A集合存外键的字段
//foreignField B集合与A集合关联的字段
//as 重新组合成C集合名称
语法
db.collection.aggregate([{
   $lookup:
     {
       from: <collection to join>,
       localField: <field from the input documents>,
       foreignField: <field from the documents of the "from" collection>,
       as: <output array field>
     }
}])

Criteria criteria = new Criteria();
criteria.andOperator(Criteria.where("_id").is("6118446118773213048"));
Aggregation aggregation = Aggregation.newAggregation(
        Aggregation.match(criteria),
        Aggregation.lookup("goodsCategory", "categoryId", "_id", "goodsCategory"),
        Aggregation.unwind("goodsCategory"),
        Aggregation.project("title","price").and("goodsCategory.name").as("categoryName").and("goodsCategory.order").as("categoryOrder")
        );

AggregationResults<BasicDBObject> ar = mongoTemplate.aggregate(
        aggregation, Goods.class, BasicDBObject.class);
for (BasicDBObject basicDBObject : ar) {
    logger.info(JsonUtils.toJson(basicDBObject));
}
//输出结果
2019-05-06 21:17:26 [com.badou.service.basic.goods.GoodsService]-[INFO] {"_id":"6118446118773213048","title":"小学生必备国学常识&小学生必读国学经典","price":"10.0","categoryName":"小学生","categoryOrder":"1"}

示例四(未曾验证)
//全文搜索查询
//设置索引：db.logInfo.ensureIndex(msg:"text")
Query query = TextQuery.query(new TextCriteria().matching("coffee").matching("-cake"));
//-cake表示不匹配不包含cake与notMatching类似
//Query query = TextQuery.searching((new TextCriteria().matching("coffee").notMatching("-cake"))
List<LogInfoCol> lfs = mongoTemplate.find(query,LogInfoCol.class);
if(lfs!=null){
    for(LogInfoCol logInfoCol:lfs){
        logger.info(JSON.toString(logInfoCol));
    }
}

示例五(未曾验证)
//得到范围内数据：以Circle构造函数，第一第二个参数为坐标，第三个参数为距离半径查找符合条件的
Circle circle = new Circle(4.2341111,63.00001,0.01);
List<PositionCol> positionCols = mongoTemplate.find(new Query(Criteria.where("location").within(circle)),PositionCol.class);
if(positionCols!=null){
    for(PositionCol positionCol:positionCols){
        logger.info(JSON.toJSONString(positionCol))
    }
}

示例六(未曾验证)
//查找坐标周围10KM内的所有商店
Point location = new Point(116.425253,39.925338);
NearQuery query = NearQuery.near(location).maxDistance(new Distance(10,Metrice.KILOMETERS));
GeoResults<PositionCol> positions = mongoTemplate.geoNear(query,PositionCol.class);
if(positionCols!=null){
    for(GeoResults<PositionCol>geoResult :positions){
        logger.info(geoResult.getContent().getBuissName()+"-"+geoResult.getDistance().getValue());
    }
}
示例七
mongo支持的查询参数实例
fpi.getParams().put("state|integer", "1");           相当于is       一对一
fpi.getParams().put("ne:state|integer", "1");           相当于not is       一对一
fpi.getParams().put("all:keywords", keyword);        keywords为list 多对一
fpi.getParams().put("in:_id|array", "0,1");          包含    一对多
fpi.getParams().put("nin:_id|array", "0,1");         排除id  一对多
fpi.getParams().put("gte:serialNum", start);        
fpi.getParams().put("lte:serialNum", end); 
//日期查询
Date currentDate = DateUtil.currentDate();
String d = DateUtil.format(currentDate,"yyyy-MM-dd HH:mm:ss");
fpi.getParams().put("gte:endDate|datetime", d);  大于当前日期的翻页查询

聚合查询
1、Criteria idcardCriteria = new Criteria();
                idcardCriteria.orOperator(Criteria.where("idcard").regex(studentIdcard.toLowerCase()),
                        Criteria.where("idcard").regex(studentIdcard.toUpperCase()));
                criteria.andOperator(idcardCriteria);

Mongodb: Sort operation used more than the maximum 33554432 bytes of RAM
mongo中所有排序字段加上索引比较好

Criteria criteria = new Criteria();
criteria.andOperator(Criteria.where("shakyId").in(gshakyIds), Criteria.where("userId").in(userIds));//筛选条件（限定数据范围）
Aggregation aggregation = Aggregation.newAggregation(Aggregation.match(criteria),
        Aggregation.group("userId", "shakyId").count().as("count"),Aggregation.sort(Sort.Direction.ASC, "count"));//分组  单个字段分组，直接取count;两个字段key 确定唯一一条记录,后续需要手动统计
```


AggregationResults ar = mongoTemplate.aggregate(aggregation,
Ugclock.class, BasicDBObject.class);

```
for (BasicDBObject basicDBObject : ar) {
    String id = basicDBObject.getString("shakyId");
    int count = 1;
    if (shakyCount.containsKey(id)) {
        count = shakyCount.get(id);
        count++;
        shakyCount.put(id, count);
    } else {
        shakyCount.put(id, count);
    }
}

2、以id为group by计算num的总值sum（）
Aggregation agg = Aggregation.newAggregation(
                Aggregation.group(new String[] {"_id"}).sum("num").as("num")
                );
3、以id为group by找出num的最大值max()                
Aggregation agg = Aggregation.newAggregation(
                Aggregation.group(new String[] {"_id"}).max("num").as("num")
                );                
Aggregation agg = Aggregation.newAggregation(
                Aggregation.group(new String[] {"_id"}).min("num").as("num")
                );
4、以id为group by计算num的平均值avg(）
Aggregation agg = Aggregation.newAggregation(
                Aggregation.group(new String[] {"_id"}).avg("num").as("num")
                );
```

# 翻页

项目当中模拟插入了120W条数据，在同一个文档当中单纯查询数据的速度还不错，主要是对查询的文档字段添加了索引，但是对查询结果的前台分页确有问题。具体来说是不设置任何查询条件的时候，会查询出来将近120W条满足条件的结果，使用mongodb的limit()和skip() 来取出来 第一页前20条数据，这样在后台的java程序当中只是这20条数据占用内存。
代码具体形式类似于用mongodb客户端执行db.feedbackInfo.find(criteria).skip(0).limit(20) 获得第一页0-20条数据db.feedbackInfo.find(criteria).skip(20).limit(20) 获得第二页20-40条数据……db.feedbackInfo.find(criteria).skip(N).limit(20) 获得第二页N-N+20条数据但问题在于随着不断翻页，skip的值N会越来越大，前台的反应越来越慢。很直接的一个表现就是在前台从第一页直接跳转进入最后一页根本反应不过来。对于这个实际问题，原因就是本书这里所言的skip略过大量结果会带来性能问题，再根源地说是mongodb还不够完善，索引本身还比较简单。具体的这个分页效率的问题，有两种思路：第一，等mongodb升级，优化这个skip的执行效率。第二，不用skip()而实现分页效果。这个思路的基础就是mongodb本身对于where查询和limit()的效率还比较不错，也就是本来分页的那个查询用where和limit速度还可以的前提（一般就是需要建立必要的索引）。假如这个前提不成立，那没法讨论。本书接下来具体讨论了不使用skip对结果分页的实现例子，这个本质是对信息系统增加一个查询中间量——上次查询的业务数值，在逻辑上承担起跟skip相对等价的功能。比如说是第一页查询是按照一个日期date值查询，第一次用db.foo.find().sort({"date",-1}).limit(20)而点击下一页的时候，事先将上次查询的date的边界值给传递过去，第二页查询的时候就使用新的find条件查询db.foo.find({"date":{"$gt":latest.date}}); 而后再对查询结果排序即可这种绕过skip的方式评价：第一，很难比较方便地解决所有的分页问题，简单来说 对于使用正则表达式的查询，根本无法通过记录边界条件来实现。第二，不得不多传递上次查询的那个边界条件，增加了工作量，不够优雅。第三，只能够解决一页一页往下翻页的问题，如果我要从第1页直接跳到100页，就束手无策

## 翻页的实现一

### skip实现跳页（比较简单，数据量大了不行）

     /**
	 *db.getCollection('user').find({})是指查询全部。
	 *sort()设置排序，本示例是指以_id作为条件，正序排序。若将数字1改为-1，则为倒序。
	 *skip()设置跳页，本示例是指跳过前10条，从第11条开始显示。
	 *limit()设置每页的显示数量，本例是指每页限制显示10条。
	 */
	db.getCollection('user').find({}).sort({"ID":1}).skip(10).limit(10)

### 非skip实现跳页

	原理：以自增_id作为主条件，获取前一页的最后一条记录，查询之后的指定条记录
	
	//根据_id，查询前10条
	var a = db. getCollection('user').find({}).sort("_id",1).limit(10)
	//定义变量last
	var last  = null
	//循环遍历
	while(a.hasNext()){
	    last=a.next;//循环到最后，last接收的是最后一条的信息
	}
	//核心是"_id":{"$gt":last._id}，即查询大于最后一条的_id的后10条信息
	db.getCollection('slt').find({"_id":{"$gt":last._id}}).sort({_id:1}).limit(10)

## 翻页的实现二（非skip实现跳页，以自增_id作为主条件）

    private List<JSONObject> findOrderList(Map<String,Object> paramOptions,int page,int size,String sidx,String sord) throws Exception {
                //连接数据库
        MongoCollection<Document> mongoCollection = getMdbCollection();
        //进行第一次查询，条件中没有skip
        MongoCursor<Document> iterable = mongoCollection.find().limit(size).sort(new BasicDBObject("_id", 1)).iterator(); 
        //定义orderItems，用以接收查询的信息
        List<JSONObject> orderItems = new ArrayList<JSONObject>();
                
        //如果只有一页或第一页，则走此条件，否，则走else
	        if(page == 1){     
	　　　　  //遍历        
	            while (iterable.hasNext()) {
	              Document next = iterable.next();
	              JSONObject a =JSONObject.parseObject(next.toJson());
	              orderItems.add(a);
	            }
	　　　　　}else{ 
	　　　　　　MongoCursor<Document> iterable2 = mongoCollection.find().limit(size*(page-1)).sort(new BasicDBObject("_id", 1)).iterator(); 
	         //定义变量last，用以存储每页的最后一条记录 
	　　　　　Document last = null; 
	　　　　　while (iterable2.hasNext()) { 
	　　　　　　　　last = iterable2.next(); 
	　　　　　} 
	         //定义condition，用以添加【$gt:每页最后一条记录的_id值】，作为查询条件 
	　　　　　　Map<String, Object> condition = new HashMap<>(); 
	　　　　　　if (null != last.get("_id")) { 
	　　　　　　　　condition.put("$gt",last.get("_id")); 
	 　　　　　 } 
	　　　　　 MongoCursor<Document> iterable3 = mongoCollection.find(condition).limit(size).sort(new BasicDBObject("_id", 1)).iterator(); 
	　　　　　　//遍历 
	　　　　　　while (iterable.hasNext()) { 
	　　　　　　　　　　Document next = iterable.next(); 
	　　　　　　　　　　JSONObject a =JSONObject.parseObject(next.toJson()); 
	　　　　　　　　　　orderItems.add(a);
	　　　　　　} 
	　　　　　　return orderItems; 
	}
	 注：上面是按_id正序排列的，如果想要按照倒序排列，则需要将condition.put("$gt",last.get("_id"))改为condition.put("$lt",last.get("_id"))，将sort(new BasicDBObject("_id", 1)改为sort(new BasicDBObject("_id", -1)。

