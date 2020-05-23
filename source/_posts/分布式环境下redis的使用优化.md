---
title: 分布式环境下redis的使用优化
date: 2020-03-10 08:50:07
tags: redis
categories: redis
---

# 分布式与集群

单机处理到达瓶颈的时候，你就把单机复制几份，这样就构成了一个“集群”。集群中每台服务器就叫做这个集群的一个“节点”，所有节点构成了一个集群。每个节点都提供相同的服务，那么这样系统的处理能力就相当于提升了好几倍（有几个节点就相当于提升了这么多倍）。但问题是用户的请求究竟由哪个节点来处理呢？最好能够让此时此刻负载较小的节点来处理，这样使得每个节点的压力都比较平均。要实现这个功能，就需要在所有节点之前增加一个“调度者”的角色，用户的所有请求都先交给它，然后它根据当前所有节点的负载情况，决定将这个请求交给哪个节点处理。这个“调度者”有个牛逼了名字——负载均衡服务器。集群结构的好处就是系统扩展非常容易。如果随着你们系统业务的发展，当前的系统又支撑不住了，那么给这个集群再增加节点就行了。但是，当你的业务发展到一定程度的时候，你会发现一个问题——无论怎么增加节点，貌似整个集群性能的提升效果并不明显了。这时候，你就需要使用微服务结构了。

从单机结构到集群结构，你的代码基本无需要作任何修改，你要做的仅仅是多部署几台服务器，每台服务器上运行相同的代码就行了。但是，当你要从集群结构演进到微服务结构的时候，之前的那套代码就需要发生较大的改动了。所以对于新系统我们建议，系统设计之初就采用微服务架构，这样后期运维的成本更低。但如果一套老系统需要升级成微服务结构的话，那就得对代码大动干戈了。所以，对于老系统而言，究竟是继续保持集群模式，还是升级成微服务架构，这需要你们的架构师深思熟虑、权衡投入产出比。OK，下面开始介绍所谓的分布式结构。分布式结构就是将一个完整的系统，按照业务功能，拆分成一个个独立的子系统，在分布式结构中，每个子系统就被称为“服务”。这些子系统能够独立运行在web容器中，它们之间通过RPC方式通信。

举个例子，假设需要开发一个在线商城。按照微服务的思想，我们需要按照功能模块拆分成多个独立的服务，如：用户服务、产品服务、订单服务、后台管理服务、数据分析服务等等。这一个个服务都是一个个独立的项目，可以独立运行。如果服务之间有依赖关系，那么通过RPC方式调用。这样的好处有很多：系统之间的耦合度大大降低，可以独立开发、独立部署、独立测试，系统与系统之间的边界非常明确，排错也变得相当容易，开发效率大大提升。系统之间的耦合度降低，从而系统更易于扩展。我们可以针对性地扩展某些服务。假设这个商城要搞一次大促，下单量可能会大大提升，因此我们可以针对性地提升订单系统、产品系统的节点数量，而对于后台管理系统、数据分析系统而言，节点数量维持原有水平即可。服务的复用性更高。比如，当我们将用户系统作为单独的服务后，该公司所有的产品都可以使用该系统作为用户系统，无需重复开发。

![](redis0.png)

# 集群环境下redis锁的进一步优化

## 死锁的问题
![](redis1.png)

解决：
![](redis2.png)

## 假如程序被kill了，删除锁同样无法执行得到

![](redis3.png)

解决：给锁增加过期时间

![](redis4.png)

## 过期时间的确定
 假如程序1执行到删除锁的时间大于了过期时间，也就是程序1还没执行完锁就被删掉了，此时另外一个线程也可以拿到锁。当线程1执行完去删除锁的时候删掉的就是线程2的锁了，程序就乱了。

解决：
保证当前线程只删除当前线程加的锁

![](redis5.png)

![](redis6.png)

## redisson.org的使用

	<dependency>
            <groupId>org.redisson</groupId>
            <artifactId>redisson</artifactId>
            <version>3.6.5</version>
        </dependency>


	  @Bean
	    @ConditionalOnProperty(name="redisson.address")
	    RedissonClient redissonSingle() {
	        Config config = new Config();
	        SingleServerConfig serverConfig = config.useSingleServer()
	                .setAddress(redssionProperties.getAddress())
	                .setTimeout(redssionProperties.getTimeout())
	                .setConnectionPoolSize(redssionProperties.getConnectionPoolSize())
	                .setConnectionMinimumIdleSize(redssionProperties.getConnectionMinimumIdleSize());
	        
	        if(StringUtils.isNotBlank(redssionProperties.getPassword())) {
	            serverConfig.setPassword(redssionProperties.getPassword());
	        }
	
	        return Redisson.create(config);
	    }







