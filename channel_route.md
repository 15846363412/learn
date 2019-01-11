## 接口调用 ##
#### com.jiupai.channel.route.api.auth.IAuthOrderCostService ####
### 1.定义接口 ###
ResultInfo authOrderRecord(RequestInfo<StatsOrderCost> orderCost);
#### 2。class AuthOrderCostServiceProvider implements IAuthOrderCostService {} ####

在AuthOrderCostServiceProvider实现了authOrderRecord接口 

#### （1）autowired自动注入CrStatsOrderCostService ####

#### （2）重写authOrderRecord方法。 ####

 - 用crStatsOrderCostService服务用authOrderRecord（订单记录）方法 
 - 返回成功码和成功信息 return ResultInfo.successResult(null) 


#### com.jiupai.channel.route.api.auth.IAuthRouteService#authRoute ####

### 1.定义接口 ###

#### ResultInfo<ArrayList<AuthRoute>>  authRoute(RequestInfo<AuthRouteQuery> authRouteQueryRequestInfo); ####
#### 2.public class AuthRouteProvider  implements IAuthRouteService。 ####
#### （1）autowired CmmtAuthRoutService服务 ####
#### （2）实现authRoute方法  ####
- 调用getbody
- 请求参数非空校验
- 调用queryAuthRout方法查询授权信息
- 	getGroupIdByMercId根据商户号获取路由组信息
- 	selectEnableAuthRout 查询可用的路由组信息
- 	过滤非有效的路由组
- 	queryEableOrgAuth查询路由机构信息
- 	处理认证渠道名称。双层for循环进行遍历
- 	策略处理  采用ArrayListMultimap.create();用于创建一个一键多值的multimap集合 
- 	按照规则分组mapper
- 	往multimap里put值
- 	获取multimap中不重复的key
- 	对这个key进行for循环遍历
- 	获取路由优先级规则 调用枚举类型ECmmtRulInf_RulTyp.getEnum(rulTyp)
- 	分别把分流和优先级的code存入到认证路由中
- 	清理已选择路由信息
- 	其他策略类型排序处理
- 	认证路由非空校验
- 	返回认证路由 cmmtAuthRouts
		
#### 订单费用服务 接口####
#### com.jiupai.channel.route.api.stats.IStatsOrderCostService ####
    StatsOrderCostServiceImpl implements IStatsOrderCostService {}

1. queryRequestInfo.getBody()获取body
2. 校验订单集合
3. 根据订单列表查询订单
4. Collections2.transform批量