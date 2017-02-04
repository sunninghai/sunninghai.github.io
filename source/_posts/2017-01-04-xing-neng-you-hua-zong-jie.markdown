---
layout: post
title: "性能优化策略总结"
date: 2017-01-04 17:42:19 +0800
comments: true
categories: 性能
---

　　提到应用程序性能优化，我们都习惯性的想到下面的方式，缓存、异步、集群、jvm调优等。其实我们应该首先从我们的代码入手进行优化，然后再考虑其他的方式。
　　
## 代码层面优化

绝大多数的性能问题都是我们代码写的不合理引起的，在平时的编码中有些我们可以作为规范来规避性能问题

* 不必要耗时逻辑判断,深层次调用封装不合理导致重复的逻辑执行；
* for循环次数过多或者不必要的for循环；我们的代码中多次出现for循环RPC调用
* 尽量重用对象、使用局部变量
* 减少变量的重复计算，典型的就是数组、集合遍历时大小的计算
* 慎用异常。异常对性能不利,不要用来控制程序流程。影响主要体现在
	1. 抛出异常首先要创建一个新的对象。
	2. Throwable接口的构造函数调用名为fillInStackTrace()的Native方法，fillInStackTrace()方法检查堆栈，收集调用跟踪信息。只要有异常被抛出，VM就必须调整调用堆栈，因为在处理过 程中创建了一个新的对象。
* 能估计到的数组、集合大小初始化大小
* 除非线程安全需要，尽量用HashMap、ArrayList、StringBuilder等非线程安全的类
* 不要使用synchronized关键字，synchronized虽然可以达到同步效果，但仅限于单机环境，现在大部分服务都是分布式服务，如果需要同步需要用分布式锁
* 顺序插入和随机访问比较多的场景使用ArrayList，元素删除和中间插入比较多的场景使用LinkedList
* 把一个基本数据类型转为字符串，基本数据类型.toString()是最快的方式、String.valueOf(数据)次之、数据+""最慢。String.valueOf() 在调用toString()前会先做非空判断， + ""底层使用了StringBuilder实现，先用append方法拼接，再用toString()方法获取字符串
* 使用最有效率的方式去遍历Map

		public static void main(String[] args){
			HashMap<String, String> hm = new HashMap<String, String>();
		   hm.put("111", "222");
		   Set<Map.Entry<String, String>> entrySet = hm.entrySet();
		   Iterator<Map.Entry<String, String>> iter = entrySet.iterator();
		   while (iter.hasNext()){
		   Map.Entry<String, String> entry = iter.next();
		   System.out.println(entry.getKey() + "\t" + entry.getValue());
		   }
		}
		
还有很多其他的我们需要注意的地方

##数据库优化

数据库优化主要包括 sql优化、架构优化、连接池优化（暂时没有用到，只是简单调整连接数）

* sql 优化.主要的是根据慢sql记录进行分析优化
* 架构优化.主要是读写分离、一主多从、数据库拆分（根据业务进行拆分）


##缓存使用

缓存主要用在频繁访问且更新频率不是很大的数据场景。分为本地缓存、分布式缓存，本地缓存现在应用相对较少，主要还是分布式缓存。

使用缓存比较纠结的是缓存的更新策略，什么时候失效缓存。主要是主动更新（适用于更新即可见的场景）、失效重新加载（适用于更新后不需要立即可见场景，通过失效时间来控制）

### 缓存更新策略
 
* 失效：应用程序先从cache取数据，没有得到,数据库中取数据，成功后，放到缓存中。
* 命中：应用程序从cache中取数据，取到后返回。
* 更新：先把数据存到数据库中，成功后，再让缓存失效. 为什么不直接更新而是失效呢？主要是因为如果两个并发写导致缓存脏数据。如下图
	![顺序](../images/redis1.png)
	
### 缓存被击穿

高并发请求如果同时检测到缓存失效，大量请求同时去请求数据库容易压垮数据库，我们期望的是只有一个请求访问数据库，其他请求继续请求缓存。

解决方式

	public String get(key) {
	      String value = redis.get(key);
	      if (value == null) { //代表缓存值过期
	          //设置一定的超时时间，防止del操作失败的时候，下次缓存过期一直不能load db
	          if (redis.setnx(key_mutex, 1, 3 * 60) == 1) {  //代表设置成功
	               value = db.get(key);
	                      redis.set(key, value, expire_secs);
	                      redis.del(key_mutex);
	              } else {  //这个时候代表同时候的其他线程已经load db并回设到缓存了，这时候重试获取缓存值即可
	                      sleep(50);
	                      get(key);  //重试
	              }
	          } else {
	              return value;      
	          }
	  }
	　　　


##异步使用

对于实时性要求不高，而且相对耗时的操作可以采用异步的方式；

可以直接在请求内部单起一条线程，执行异步操作，也可以利用消息队列或者任务队列统一处理；


异步虽好，但也有需要注意的地方

* 事务支持不好；如果需要事务操作需要全部放到异步中去
* 注意任务数量的控制

##jvm调优