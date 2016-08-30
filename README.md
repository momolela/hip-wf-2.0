该框架为bsoft所使用
# hip-wf 2.0
```xml
hip-wf编译后为jar，包含如下功能。
  1.基于ssdev的SOA框架，该框架提供了不同系统系统的分布式调用，服务注册，服务发现等基础功能。
  2.基于classpath的前端资源加载机制。
  3.javaScript面向对象扩展。
  4.javaScript依赖加载控制。
  5.前后台服务调用机制。
  6.js 扩展组件。
    页面片段加载
    复杂类型，如Map
    自定义事件通知
    前端日志框架log4js
  致力于尽量减少开发前端以及分布式服务中的工作量，对于不熟悉前端的Java开发人员有巨大帮助
```
## 1.使用hip-wf

* 使用maven进行依赖
```xml
  <dependency>
		<groupId>com.bsoft</groupId>
		<artifactId>hip-wf</artifactId>
		<version>2.0</version>
  </dependency>
```

## 2.使用分布式服务调用功能。
* 编写远程服务

* 配置spring
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:ssdev="http://www.bsoft.com.cn/schema/ssdev"
	xsi:schemaLocation="http://www.springframework.org/schema/beans   
	http://www.springframework.org/schema/beans/spring-beans.xsd         
	http://www.bsoft.com.cn/schema/ssdev
	http://www.bsoft.com.cn/schema/ssdev/ssdev.xsd">
	
        <!-- 配置服务发布与注册 -->
        <!-- name:应用域名，一个javaweb应用为一个域，同一个服务器下一个域名只允许有一个，多个服务器可重复并且进行负载 -->
        <!-- registryAddress:zookeeper地址，用于服务注册发现协调的zookeeper的地址 -->
        <!-- rpcServerWorkUrl:本应用服务发布的地址（注意端口号必须和javaweb容器相同） -->
	<ssdev:applicationDomain name="civ" registryAddress="zookeeper://localhost:2181" rpcServerWorkUrl="hessian://localhost:8300/civ/rpc/" enableLogger="false" />
  
        <!-- 通过扫描方式发布服务 ，扫描需要暴露为远程服务的类-->
	<ssdev:rpcService-scan base-package="civ.service.page,hip.wf" />
	
	<!-- 不通过扫描的方式发布服务 -->
	<ssdev:service id="serviceID" class="xx.xx.xx.xx" />
	
	<!-- devMode:false 强制浏览器不缓存页面，开发中使用 -->
	<bean id="appContextHolder" class="ctd.util.AppContextHolder">
		<property name="devMode" value="false"></property>
	</bean>
</beans>
```
