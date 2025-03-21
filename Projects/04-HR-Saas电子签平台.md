---
title: HR Saas电子签平台构建
category: project
tags:
  - project
publishedAt: 2024-06-15
description: 电子签署能力在人事SaaS领域的商业价值不仅体现在效率提升和成本优化上，更通过法律合规、数据智能化、生态协同及可持续发展等维度，这是一个智能化的HR Saas产品的必备能力，也是商业化、增强产品力的很好的一个功能模块。同时人事业务本身就是电子签署的最大的应用场景，人员流动、合同管理都需要高效的电子签署来支持。
---

电子签署能力在人事SaaS领域的商业价值不仅体现在效率提升和成本优化上，更通过法律合规、数据智能化、生态协同及可持续发展等维度，这是一个智能化的HR Saas产品的必备能力，也是商业化、增强产品力的很好的一个功能模块。

同时人事业务本身就是电子签署的最大的应用场景，人员流动、合同管理都需要高效的电子签署来支持；

# 1. 项目总览

1. 整体系统领域和结构设计
	1. 文件模板、法人公司、电子签署3个领域模型的独立设计；实现高内聚、低耦合；
	2. 平台能力构建：
		1. 开放的人事档案接入设计；支持配置化的多种协议（HSF、HTTP等）
		2. 外部依赖服务的切换能力，依赖接口而不是具体实现，方便切换底层服务的提供商，有更好的扩展性；
	3. 外部服务的防腐层设计；
2. 核心链路性能考量
	1. 数据库设计；
		1. MySQL分库分表；64个分库，每个分库1024张分表；根据企业id由数据库代理进行路由；（水平拆分）
		2. MySQL合理的建表、索引设计；
		3. 强一致性场景使用MySQL关系型数据库；如核心链路的读写；
		4. 弱一致性场景使用分析型数据库，如搜索场景下的复杂模糊查询；
	2. 任务MQ异步化并发处理核心链路；
		1. 如批量电子签署发起，将任务通过MQ分发、并发处理，
		2. 批量的打包下载；当开启一个多文件的打包下载，先创建主任务，然后将子任务通过MQ分发，每个子任务单独执行文件的处理，同时开启一个轮询，轮询所有的子任务是否完成，完成之后再将子任务进行打包；
3. 系统的稳定性设计
	1. Redis分布式限流；
		1. 单企业的核心链路的并发量限制；
		2. 单用户的核心链路的并发量限制；
	2. 外部强依赖的降级措施；如该项目强依赖MQ，针对MQ可能发生的消息堆积情况，提前做一些准备；
	3. 核心链路的监控告警；
		1. 流量监控；关注流量的突发、恶意流量等对系统的冲击；
		2. 监控核心链路的错误、失败等异常；
	4. 系统中的强依赖有哪些？弱依赖有哪些？弱依赖是否有降级、熔断；
4. 项目管理过程中的考量
	1. 项目开始前的技术调研、外部依赖调研；
		1. 文件模板能力：WPS、钉钉内部的文档；
		2. 底层电子签署能力：直连e签宝开放平台、钉钉内部的签署服务；
	2. 项目并行开发的任务分解；
		1. 将项目内涉及的多个模块分解，并行开发，评估模块工作量、关键联调节点；
	3. 项目风险把控；
		1. 项目前：首先是项目的技术方案要足够细，才能更准确识别风险可能所在；
		2. 项目中：每日进度对其，随时对人力资源进行调整、倾斜；
		3. 关键联调测试质量把控；

# 2. 业务逻辑架构图

![](/images/hr-saas-esign-architecture-design.png)

- 电子签平台以租户的形式对外提供服务；电子签平台仅是数据组装、功能通道，不存储用户数据，租户需要以Http或RPC的形式提供数据源；
- 电子签内部3个核心模块：法人公司、文件模板、签署中心；法人公司、文件模板相互独立，互不感知；签署模块依赖以上2个模块的能力。
- 电子签底层依赖文档能力、文件存储服务、电子签服务提供商；

# 3. Usecase

一份合同的发起需要3个基本元素：
- 模板文件：企业使用哪一份合同模板文件
- 签署方1：给哪一个员工发起签署；（档案数据）
- 签署方2：签署的相关法人公司；（电子签署印章）

因此只需要准备好「合同模板」，在员工列表中，选定员工即可一键发起合同：

![](/images/hr-saas-esign-usecase-1.png)

在真正发起之前，HR可以手动调整一些数据，再执行发起动作，发起之后
1. 根据合同模板文件内的档案字段，拉取员工的对应的档案数据；
2. 渲染数据到文件中，并生成真实发起的PDF文件；
3. 将文件发送到「电子签机构服务」。并维护签署记录的状态流转；
4. SMS、协同办公软件、邮件等通知员工方、企业方执行签署；

![](/images/hr-saas-esign-usecase-2.png)

# 4. 签署发起的流程图

![](/images/hr-saas-esign-start.png)

- HR（管理员）选中员工 + 文件模板后，可以直接发起电子签，平台会根据租户，以及提前配置好的Hook，拉取对应的员工档案。
- 数据 + 模板 => 生成最终要签署的合同文件。最终将文件及印章坐标（图中简化了）发送到电子签服务接口。
- 根据配置在服务提供商的webhook，维护签署文件状态的流转；


# 5. 领域模型和数据源

![](/images/hr-saas-esign-domain.png)

上面是系统内存领域模型，再下一层的存储层使用MySQL数据库；
- 对于强一致性的业务，走MySQL；
- 对于弱一致性复杂查询场景，如台账搜索这类超多查询参数，走分析型数据库查询；（需要做数据导流，异步将MySQL数据同步到对应的分析型数据库）



# 6. 外部服务防腐层

## 核心目的
- **隔离外部服务的变化对当前系统的侵蚀**，确保自己服务的领域模型的纯粹性；对于外部的数据结构、协议的变化，可以统一在防腐层处理，也不会影响到自己的领域结构；
- 在防腐层统一对外部服务的异常进行处理，可以做异常的转换映射、限流、熔断等逻辑；
- 针对依赖的「文档能力」、「电子签能力」可以在防腐层进行适配和切换，只需要适配已有的内部领域模型，对上层业务无感知。保持内部逻辑的清晰；

## 如何设计

以电子签的发起签署为例：

1. 对接电子签服务提供商的外部接口：
```java
public interface EsignClient {

    EsignStartResponse startEsign(EsignStartRequest request);

	// ...
}
```
2. 外部服务定义的数据模型，只会传递到防腐层，不应向内部模型渗透；通常使用`Adaptor`或者`manager`来隔离；
	1. 将外部服务的签署状态转为内部状态；
	2. 映射外部服务的错误码到内部服务；
	3. 对外部服务执行限流、降级策略；
```java
@Service
public class EsignAdapterImpl implements EsignAdapter {
    
    @Resource
    private EsignClient esignClient;

    @Override
    @SentinelResource(fallback = "startEsignFallback")
    public HrmEsignRecord startEsign(HrmEsignRecord esignRecord) {
        // 针对要发起的签署记录构建外部请求
        EsignStartRequest esignStartRequest = buildEsignStartRequest(esignRecord);

		// 执行请求发送
		EsignStartResponse esignStartResponse = esignClient.startEsign(esignStartRequest);

		// 处理服务方的响应，构建内部领域的发起后的签署记录（将订单号、状态填充到内部模型）
		HrmEsignRecord hrmEsignRecord = parseEsignResponse(esignStartResponse);	

		return hrmEsignRecord;
    }

	// fallback
    public HrmEsignRecord startEsignFallback(HrmEsignRecord esignRecord) {
        // 降级策略

		return hrmEsignRecord;
    }
}
```

# 7. 开放能力设计

电子签平台作为一个数据通道，对接入的租户提供的服务为：<u>数据组装</u>、<u>文件生成</u>、<u>签署文件的管理</u>等等；

![](/images/hr-saas-esign-open-api.png)

在整个租户参与的流程中，有从租户拉取数据的钩子，也有对租户开放的数据接口：
- 在接入平台时，需要租户配置接入的数据提供协议（HTTP、HSF内部RPC协议等）、服务调用地址；
- 平台也提供相应的数据查询接口（HTTPS、RPC）

这一块在代码的设计中，主要是<font color="#f79646">简单工厂 + 动态配置 + RPC泛化调用</font>

```java
public interface HrmEsignTenantSpi {

	// 获取用于模板制作的租户档案字段
    TenantArchives fetchTenantFields(TenantFieldRequest request);

	// 获取用于发起的租户用户数据
    TenantUserData fetchTenantUserData(TenantUserDataRequest request);
}

// HTTP方式接入的租户
public interface HrmEsignTenantSpiHTTPAdaptor {

    // 租户配置
    private TenantConfig tenantConfig;

    // 获取用于模板制作的租户档案字段
    public TenantArchives fetchTenantFields(TenantFieldRequest request) {

        // 获取租户配置
        String tenantId = request.getTenantId();
        TenantConfig tenantConfig = tenantConfig.getConfig(tenantId)

		// HTTP 调用
    }
}

// RPC方式接入的租户
public interface HrmEsignTenantSpiRPCAdaptor {

	// 租户配置
    private TenantConfig tenantConfig;
	
    // 获取用于模板制作的租户档案字段
    public TenantArchives fetchTenantFields(TenantFieldRequest request) {
        // 获取租户配置
        String tenantId = request.getTenantId();
        TenantConfig tenantConfig = tenantConfig.getConfig(tenantId)

        // RPC 泛化调用：根据动态配置的group、service、method
    
    }
}
```



# 7. 系统的稳定性风险及应对

## 核心组件MQ的稳定性

MQ是系统的核心链路中的强依赖，消息堆积如何处理？
1. 是否流量突发引发的堆积？
	1. 如果是<font color="#de7802">正常流量突发</font>，<font color="#de7802">优先考虑紧急扩容</font>，第一时间保证业务正常处理，增强消费能力，防止系统雪崩；
	2. 如果是<font color="#4bacc6">异常流量</font>，恶意企业、恶意攻击等，造成生产端产生无效消息，根据<font color="#4bacc6">企业标识，过滤掉恶意流量</font>；
2. 如果不是流量高引发的堆积，比如<u>重试机制导致不断重新消费</u>，引发的堆积，需要明确重试原因，是否能够重试解决？
	1. 可以重试解决，可以降低重试次数，比如只重试1-2次；
	2. 重试不可以解决，针对关键字、错误码不进行重试，直接fast-fail；
3. 以上都应该提前准备好对应的<u>降级开关</u>、<u>错误响应映射</u>等；

## 接口限流

接口限流的系统保护是如何处理的？这里分为2部分考量：
1. 对外部提供服务的限流（QPS）
	1. 目的：<font color="#f79646">恶意流量</font>、<font color="#f79646">保护系统不会外部流量打垮</font>；
	2. 明确限流的维度：企业、个人；
	3. 限流的技术实现：基于Redis的分布式限流；也可以是使用Sentinel的热点规则限流；使用什么方式，取决于当前系统的结构是否合适、基建是否完善、成本等因素；
2. 对内部依赖的服务的限流（线程数）
	1. 目的：同样是保护系统，但是是保护系统<font color="#f79646">不被外部服务的不稳定拖垮</font>；
	2. 方式：这里要使用线程数限流，而非QPS，当外部服务RT不稳定，如果不做线程数控制，会导致启动更多的线程，占用系统资源；
	3. 具体可以自行在业务中构建线程池，设定上限；也可以使用Sentinel的线程数限流能力，直接配置化对应的接口即可；



