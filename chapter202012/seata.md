# 分布式事务解决方案-Seata

## 1. 三个角色
### 1.1 TC (Transaction Coordinator) - 事务协调者
> 维护全局和分支事务的状态，驱动全局事务提交或回滚。  

### 1.2 TM (Transaction Manager) - 事务管理器
> 定义全局事务的范围：开始全局事务、提交或回滚全局事务。  

### 1.3 RM (Resource Manager) - 资源管理器
> 管理分支事务处理的资源，与TC交谈以注册分支事务和报告分支事务的状态，并驱动分支事务提交或回滚。

## 2. 事务模式
> 支持AT、TCC、SAGA和XA事务模式，最终目标是在分布式环境下实现ACID，并且隐藏所有底层实现，尽量让开发者只关注于自身业务，开发者可以根据业务场景去选择不同的事务模式。  

### 2.1 AT
> 具体工作流程去官网看。

* 角色  
	- @GlobalTransactional的发起方节点作为TM。
	- 每个节点自己的ACID作为RM。
	- Seata Server作为TC。
* 数据源代理
	- 在数据源上层做了一层代理
		- 处理RM的**commit**
		- 创建或删除**undo log**
		- **向TC汇报** *xid*、*branchId*、*rollback_info* 等关键字段信息。
	- 使用数据库表实现了undo log，用来做RM的回滚。
	- TM持有全局锁，只有当TM决定本次事务可以全部提交的时候才会释放锁。
	- 抢到锁的任何RM都可以提交，而没拿到锁的任何一个RM都无法提交，并且要注意超时时间。
	- xid是全局事务id，每个子事务都会有个branchId并且在TC上与xid相关联。
* 问题
	- Seata Server一定要做高可用，并且它本身的吞吐量决定了事务过程的吞吐量。
	- 调用链很长并且远程服务响应时间不是很快的场景。
		- 每个RM节点的timeout要有一定的规划。
		- 请求的响应时间会过长。
		- 可能会由某一个很轻量并且在整个业务调用链又不是很重要的服务报错，导致调用此服务之前执行了部分很重的业务全部回滚，得不偿失。
	- 读和写
		- 写入过程对部分数据加了x锁，导致数据一直被锁定，如果调用链时间非常长会锁定相当长的时间
		- 读取过程由于一般都采用**提交读**策略，所以如果对读写顺序要求很高的话可能会读到脏数据，如果采用select for update，势必影响吞吐量。

### 2.2 TCC
> 具体工作流程去官网看。

除了没有

### 2.3 SAGA

### 2.4 XA

## 3. 配套支持
### 3.1 注册中心
* eureka
* zookeeper
* nacos
* consul
* etcd
* 其他略

### 3.2 微服务框架
* Dubbo: seata-dubbo
* Spring Cloud: spring-cloud-starter-alibaba-seata
* 其他略

### 3.3 ORM框架
* Mybatis
* Mybatis Plus
* 其他略

### 3.4 数据库
* MySQL
* 其他略