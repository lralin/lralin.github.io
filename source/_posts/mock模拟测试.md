---
title: Mock模拟测试
date: 2019/05/24 12:50:25
updated: 2019/05/24 12:50:25
tags:
- mockito
- 测试
---

##### 简介

mock测试就是在测试过程中，对于某些不容易构造或者不容易获取的对象，用一个虚拟的对象来创建以便测试的测试方法。而在java中mock工具有很多例如Mockito、EasyMock、PowerMock，我们这里主要介绍Mockito。

##### 示例
<!--more-->
1. pom包：
   ```xml
   <dependency>
       <groupId>org.mockito</groupId>
       <artifactId>mockito-all</artifactId>
       <version>1.9.5</version>
   </dependency>
   ```
2. Java代码中的实际调用关系
```java
public interface AirTravelDubboService {
	public AirTravelRes airTravel(AirTravelReq req);
}

public class AirTravelDubboServiceImpl implements AirTravelDubboService {
	@Autowired
	private A0032Service a0032Service;
	
	public AirTravelRes airTravel(AirTravelReq req){
		AirTravelRes res = new AirTravelRes();
		// xxx 业务代码
		a0032Service.flight(data);
		// xxx 业务代码
		return res;
	}
}

public interface A0032Service {
	public boolean flight(RouteData data,List<String> list);
}

@Service
public class A0032ServiceImpl implements A0032Service{

	//RPC 远程服务
	@Autowired
	private FlightInfoDubboService flightInfoDubboService;
	
	public boolean flight(RouteData data,List<String> list){
		//xxx业务代码
 		FlightInfoA0032Resp resp = flightInfoDubboService.flightInfo(req);
		//xxx业务代码
	}
}
```
该样例使用的是两个dubbo服务之间的调用。AirTravelDubboService服务通过A0032Service接口调用flightInfoDubboService远程服务
2. Junit中进行模拟测试
```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = {"classpath:applicationContext.xml"})
public class AirTravelDubboTest extends BaseTest {

	@Autowired
	private AirTravelDubboService airTravelDubboService;

	@Autowired
	private A0032Service a0032Service;
	
	@Mock
	private FlightInfoDubboService mockApiService;

	@Before
	public void TestBefore() {
		MockitoAnnotations.initMocks(this);
		FlightInfoA0032Resp resp = new FlightInfoA0032Resp();
		resp.setThirdCode("200");
		resp.setResult("交易成功");
		//设置a0032Service接口实现类里面的flightInfoDubboService属性为mockApiService对象。（进行属性注入）
		ReflectionTestUtils.setField(a0032Service, "flightInfoDubboService", mockApiService);
		when(mockApiService.flightInfo(any(RouteBean.class),anyList())).thenReturn(resp);
//		doCallRealMethod().when(a0032Service).flight();
	}

	@Test
	public void Test1() {
		AirTravelReq req = new AirTravelReq();
		req.setAAA("xxxxxxxx");
		req.setBBB("xxxxx");
		AirTravelRes res = airTravelDubboService.airTravel(req);
		//该测试类启用后，a0032Service接口中的FlightInfoDubboService远程服务返回的数据就是{thirdCode:200,result:"交易成功"}。
	}
}
```
##### mockito 常用方法
```java
//mockApiService服务输入任意值都返回模拟结果
when(mockApiService.flightInfo(any(RouteBean.class),anyList())).thenReturn(resp);
//mockApiService服务输入任意值都返回null
when(mockApiService.flightInfo(any(RouteBean.class),anyList())).thenReturn(null);
//mockApiService服务输入任意值都抛出异常
doThrow(new RuntimeException()).when(mockApiService.flightInfo(any(FlightInfoA0032Req.class),any(RouteBean.class)));
//a0032Service接口部分方法使用真实调用
doCallRealMethod().when(a0032Service).flight(data)
```
