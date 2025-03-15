---
title: 微服务：分布式限流组件Sentinel
category: DistSystem
tags:
  - MicroService
  - Sentinel
publishedAt: 2024-03-01
description: 分布式限流组件Sentinel的原生接入、MVC接入、Dubbo接入的方式，以及控制台操作；
---

# Sentinel的接入

接入方式：
1. 接入`原生sentinel`，硬编码资源和限流逻辑。
2. 接入`原生sentinel` + `Sentinel控制台`，不需要硬编码限流逻辑，代码中只需要定义好资源即可，可以直接在控制台动态配置限流；
3. 注解配置sentinel资源
4. 接入`Spring adapter`，让sentinel拦截所有的http资源，统一在`Sentinel控制台`配置限流规则。
5. dubbo接入sentinel

所有代码：https://github.com/huiru-wang/backend-code-snippet/tree/main/07-springboot-sentinel

通常在日常的工作中，最常见的就是Http、RPC这类的通讯接口的限流。偶尔会需要MQ的分布式限流，MQ本身可以配置消费者的拉取批次和频率，但是如果要做到分布式限流，并且灵活的配置化，还是需要借助Sentinel

以下只有接入和最简单的使用，具体的`fallback`、`熔断`等功能在此基础上看官方文档即可
## 1. 原生Sentinel

（1）在Springboot的基础上引入`sentinel-core`依赖即可

```xml
<dependency>  
	<groupId>com.alibaba.csp</groupId>  
	<artifactId>sentinel-core</artifactId>  
</dependency>  
```

（2）本地定义规则和资源

```java
@Component  
public class SentinelFlowRuleConfig implements InitializingBean {  
  
	@Override  
	public void afterPropertiesSet() {  
		List<FlowRule> rules = new ArrayList<>();  
		FlowRule rule = new FlowRule();  
		rule.setResource("hello-resource");  
		rule.setGrade(RuleConstant.FLOW_GRADE_QPS);  
		rule.setCount(2);  
		rules.add(rule);  
		FlowRuleManager.loadRules(rules);  
	}  
}
```

（3）使用对应的规则拦截业务逻辑

```java
@RestController("/")  
public class HelloController {  
  
	@GetMapping("/hello")  
	public String hello() {  
		try (Entry entry = SphU.entry("hello-resource")) {  
			// 执行正常的业务逻辑 
			return "hello world";  
		} catch (BlockException ex) {  
			// 处理被流控的逻辑
			return "Sentinel block";  
		}  
	}  
}
```


## 2. 原生Sentinel + Sentinel控制台

（1）在Springboot的基础上引入`sentinel-core`、`transport`依赖即可

```xml
<dependency>  
	<groupId>com.alibaba.csp</groupId>  
	<artifactId>sentinel-core</artifactId>  
</dependency>  
<dependency>  
	<groupId>com.alibaba.csp</groupId>  
	<artifactId>sentinel-transport-simple-http</artifactId>  
</dependency>
```


（2）启动控制台
参考官网文档，下载控制台 jar 包并在本地启动：可以参见 [此处文档](https://sentinelguard.io/zh-cn/docs/dashboard.html)

```shell
java -jar .\sentinel-dashboard-1.8.8.jar --server.port=8080
```


（3）控制台相关配置（后续所有跟控制台相关的配置，都为下面的配置）

参考官方文档：https://sentinelguard.io/zh-cn/docs/general-configuration.html

应用启动时，会默认读取：classpath下的`sentinel.properties`，也可以配置读取其他的文件，可参考文档，我这里的配置是：

```shell
project.name=web-app  
# 单文件最大50MB、文件数上限、
csp.sentinel.metric.file.single.size=52428800
csp.sentinel.metric.file.total.count=1
# log日志在项目根目录下
csp.sentinel.log.dir=logs/csp/  
  
# sentinel-transport-common 控制台配置  
csp.sentinel.dashboard.server=127.0.0.1:8080  
# 客户端暴露此端口，以读取各种规则信息、状态
csp.sentinel.api.port=8719
```

（4）不再需要硬编码FlowRule，可以直接定义资源：

```java
@RestController("/")  
public class HelloController {  
  
	@GetMapping("/hello")  
	public String hello() {  
		try (Entry entry = SphU.entry("hello-resource")) {  
			// 执行正常的业务逻辑 
			return "hello world";  
		} catch (BlockException ex) {  
			// 处理被流控的逻辑
			return "Sentinel block";  
		}  
	}  
}
```

（5）启动项目，配置限流

如果不触发资源，Sentinel默认不会启动，所以这里需要触发一下，可以postman触发一下接口，就可以在控制台看到对应的app：

![](/images/systemdesign-sentinel.png)

可以对资源进行流控的配置了：

![](/images/system-design-sentinel-qps.png)

配置完成后，观察本机的`csp`日志，可以看到接收到的限流配置更新，对应`resource = "hello-resource"`

![](/images/system-design-sentinel-log.png)


## 3. 注解配置Sentinel资源

（1）增加注解适配依赖，并配置开启AspectJ

```xml
<!-- Sentinel限流组件 -->  
<dependency>  
	<groupId>com.alibaba.csp</groupId>  
	<artifactId>sentinel-core</artifactId>  
</dependency>  
<!-- transport依赖用于和sentinel控制台通讯 -->  
<dependency>  
	<groupId>com.alibaba.csp</groupId>  
	<artifactId>sentinel-transport-simple-http</artifactId>  
</dependency>  
<!-- sentinel注解需要 -->  
<dependency>  
	<groupId>com.alibaba.csp</groupId>  
	<artifactId>sentinel-annotation-aspectj</artifactId>  
</dependency>
```

同时需要注入一个Bean：
```java
@Bean  
public SentinelResourceAspect sentinelResourceAspect() {  
	return new SentinelResourceAspect();  
}
```

（2）使用注解定义资源：

参考官网文档：https://sentinelguard.io/zh-cn/docs/annotation-support.html

使用`@SentinelResource`注解定义资源（只能用于方法），并设置对应的`fallback`方法，当限流触发时，会执行对应的`fallback`
注意事项：
1. value：定义资源名；
2. blockHandler：定义在触发降级规则后的执行逻辑；比如下面的例子，如果接口的qps大于设定的值，就会走到`blockHandler`中；如果是流控触发的则会抛出`FlowException`，降级触发则会抛出`DegradeException`，它们都是`BlockException`的之类；
3. fallback：定义在资源抛出异常后的降级策略，不会处理规则的降级，只会处理异常的降级；比如下面的例子，当`"Test".equals(message)`抛出异常后，就会走到`fallback`
4. blockHandler和fallback对应的方法，必须和资源的返回值一样，且入参必须和资源（原方法）的入参一致，可以额外接受一个`BlockException`；比如下面的例子，`blockHandler`、`fallback`方法定义都需要带上`String message`

```java
@GetMapping("/greet")  
@SentinelResource(value = "greet-resource", blockHandler = "greetBlockHandler", fallback = "greetFallback")  
public String greet(@RequestParam("message") String message) {  
	Assert.isTrue(!"Test".equals(message), "Test Not Accept");  
	return "hello [" + message + "]";  
}  
  
public String greetBlockHandler(String message, BlockException ex) {  
	ex.printStackTrace();  
	return "Sentinel block [" + message + "]";  
}  

public String greetFallback(String message, Throwable e) {  
	return "Sentinel fallback [" + message + "]";  
}
```


## 3. SpringMVC接入Sentinel

在SpringMVC的基础上，增加sentinel相关依赖，并增加：`sentinel-spring-webmvc-adapter`

>`sentinel-spring-webmvc-adapter`中interceptor使用的`HttpServletRequest`仍然是：`javax.servlet.http.HttpServletRequest`，在Spring3.x中，已经切换成了：`jakarta.servlet.http.HttpServletRequest`，因此使用adapter，需要Springboot版本为2.x，暂不支持3.x；

```xml
<!-- Sentinel限流组件 -->  
<dependency>  
	<groupId>com.alibaba.csp</groupId>  
	<artifactId>sentinel-core</artifactId>  
</dependency>  
<!-- transport依赖用于和sentinel控制台通讯 -->  
<dependency>  
	<groupId>com.alibaba.csp</groupId>  
	<artifactId>sentinel-transport-simple-http</artifactId>  
</dependency>  
<dependency>  
	<groupId>com.alibaba.csp</groupId>  
	<artifactId>sentinel-spring-webmvc-adapter</artifactId>  
</dependency>
```

（1）通过注册`Interceptor`，统一在`preHandler`中通过将HTTP接口的`method`、`方法名`组合定义为资源；
```java
@Configuration  
public class InterceptorConfig implements WebMvcConfigurer {  
  
	@Override  
	public void addInterceptors(InterceptorRegistry registry) {  
		SentinelWebMvcConfig config = new SentinelWebMvcConfig();  
		// 添加自定义BlockExceptionHandler处理器
		config.setBlockExceptionHandler(new CustomBlockExceptionHandler());  
		config.setHttpMethodSpecify(true);  
		config.setOriginParser(request -> request.getHeader("S-user"));
		// SentinelWebInterceptor 拦截所有接口（"/**"）  
		registry.addInterceptor(new SentinelWebInterceptor(config)).addPathPatterns("/**");  
	}  
}

// 自定义一个BlockExceptionHandler，返回自定义的JSON格式的响应数据
@ControllerAdvice  
public class CustomBlockExceptionHandler implements BlockExceptionHandler {  
  
	@Override  
	public void handle(HttpServletRequest request, HttpServletResponse response, BlockException e) throws Exception {  
		// 设置响应头  
		response.setContentType("application/json;charset=UTF-8");  
		ServiceResult<Object> fail = ServiceResult.fail("429", "请求被限流，请稍后再试");  
		try {  
			response.getWriter().write(JSON.toJSONString(fail));  
		} catch (IOException ioException) {  
			ioException.printStackTrace();  
		}  
	}  
}
```

![](/images/system-design-sentinel-spring-mvc.png)

（2）定义一个简单API，启动项目，直接可以在Sentinel控制台看到所有的HTTP接口资源；

```java
@RestController("/")  
public class HelloController {  
  
	@GetMapping("/hi")  
	public String hi(@RequestParam("message") String message) {  
		Assert.isTrue(!"Test".equals(message), "Test Not Accept");  
		return "hello [" + message + "]";  
	}  
}
```

![](/images/system-design-sentinel-mvc-dashboard.png)



## 4. dubbo接入Sentinel

（1）在dubbo的基础上增加依赖：
```xml
<dependency>  
	<groupId>com.alibaba.csp</groupId>  
	<artifactId>sentinel-core</artifactId>  
</dependency>  
<dependency>  
	<groupId>com.alibaba.csp</groupId>  
	<artifactId>sentinel-dubbo-adapter</artifactId>  
</dependency>  
<dependency>  
	<groupId>com.alibaba.csp</groupId>  
	<artifactId>sentinel-transport-simple-http</artifactId>  
</dependency>
```

（2）和SpringMVC类似，`sentinel-dubbo-adapter`通过Filter，对所有的`provider`、`consumer`的`interface`、`method`都定义了资源；

![](/images/system-design-sentinel-dubbo.png)

定义一个简单的dubbo的Provider

dubbo项目启动后，自动将所有的接口、方法都定义了资源，直接在Sentinel控制台就可以看到。
```java
public interface GreetService {  
	String greet(String message);  
}

@DubboService(group = "demo", version = "1.0.0")  
public class GreetServiceImpl implements GreetService {  
	@Override  
	public String greet(String message) {  
		return "dubbo app [" + message + "]";  
	}  
}
```

![](/images/system-design-sentinel-dubbo-app.png)
