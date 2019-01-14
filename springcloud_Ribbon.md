学习总结
# 什么是ribbon #

![](https://i.imgur.com/fPI1NpJ.png)

# 入门 #

>引入依赖
>
	<!-- ribbon -->
	<dependency>
	    <groupId>org.springframework.cloud</groupId>
	    <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
	</dependency>

实现方式一
	
	 //注入  RibbonLoadBalancerClient
	    @Autowired
	    private LoadBalancerClient client;
	...
	    //通过serviceId 拉取服务列表
	    //List<ServiceInstance> instances = discoveryClient.getInstances("user-service");
	    //ServiceInstance instance = instances.get(0);
	    ServiceInstance instance = client.choose("user-service");
	String jsonStr = restTemplate.getForObject("http://"+instance.getHost()+":"+instance.getPort()+"/api/v1/user/2", String.class);

之前我们是通过serviceId（“user-service”）获取到的是服务列表，现在我们直接可以通过serviceId，返回的是单个实例，不是因为列表里面只有一个实例，是因为choose方法中已经帮我们做了复杂均衡了。

实现方式二

	/**
	 * 把RestTemplate注入到Spring容器中
	 */
	@Bean
	@LoadBalanced //增加注解   让RestTemplate内置一个负载均衡器
	public RestTemplate restTemplate(){
	    return new RestTemplate();
	}


	调用代码
    String jsonStr = restTemplate.getForObject("http://user-service/api/v1/user/2", String.class);
	
修改轮询策略
application.yml：

	#--负载均衡轮询策略 
	# serviceId
	user-service:
	  ribbon:
	    #负载均衡策略的className
	    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule

# 重载机制 #
为什么要有重载机制
一个正常的eureka客户端，每间隔30秒没有给服务器发送心跳，如果90秒服务器还没有收到心跳，服务器就会认为这个客户端已经宕机，但是eureka不会马上剔除，没间隔60会同意剔除这些失效的客户端，这样导致，我们服务实际上已经宕机了但是服务列表里面还有，这样服务的消费者在调用的时候就会报错，假设我们服务的提供方有五个，虽然只宕机了一台，但是还有四台是正常的，在这个时候我们条用这个服务如果出现报错信息肯定不是我们希望看到的，所有我们就有了这个重载机制，Spring Cloud 整合了Spring Retry 来增强RestTemplate的重试能力，当一次服务调用失败后，不会立即抛出一次，而是再次重试下一个服务。

实现代码
application.yml

	#服务名称
	spring:
	  application:
	    name: order-service
	  cloud:
	    loadbalancer:
	      retry:
	        enabled: true # 开启Spring Cloud的重试功能
	        
	#负载均衡轮询策略
	user-service:
	  ribbon:
	    #负载均衡策略的className
	    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule
	    ConnectTimeout: 250 # Ribbon的连接超时时间
	    ReadTimeout: 1000 # Ribbon的数据读取超时时间
	    OkToRetryOnAllOperations: true # 是否对所有操作都进行重试
	    MaxAutoRetriesNextServer: 1 # 切换实例的重试次数
	    MaxAutoRetries: 1 # 对当前实例的重试次数