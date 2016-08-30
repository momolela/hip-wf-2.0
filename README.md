该框架为bsoft公司内部所使用，部分依赖包位于公司内部maven库
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

### 1.1.使用maven进行依赖
```xml
  <dependency>
		<groupId>com.bsoft</groupId>
		<artifactId>hip-wf</artifactId>
		<version>2.0</version>
  </dependency>
```

## 2.使用分布式服务调用功能。
### 2.1.编写远程服务

```java
/*远程方法可被java后端程序远程调用，也可被前端js调用（框架的核心功能）
 *远程方法的入参和出参可使用任意数据类型，包括自定义java类型。
 */
@RpcClass(serviceID = "demoServcie")//用于扫描中确认 servcieID
public class remoteServcie {

        
        
	@RpcService //定义一个方法为远程方法
	public void setSome(String parentID){
	 
	}

	@RpcService
	public Map<String, Object> getSome(){
		Map mi = new HashMap<String, Object>();
		mi.put("name", "病人");
		mi.put("age", "53");

		Map addr = new HashMap<String, Object>();
		addr.put("addr", "杭州市文三路199号");

		mi.put("addr", addr);
		return mi;
	}

}
```

### 2.2.配置spring
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:ssdev="http://www.bsoft.com.cn/schema/ssdev"
	xsi:schemaLocation="http://www.springframework.org/schema/beans   
	http://www.springframework.org/schema/beans/spring-beans.xsd         
	http://www.bsoft.com.cn/schema/ssdev
	http://www.bsoft.com.cn/schema/ssdev/ssdev.xsd">
	
        <!-- 配置服务发布与注册 -->
        <!-- name:应用域名，一个javaweb应用为一个域，同一个服务器下一个域名只允许有一个，多个服务器可重复并且进行负载 -->
        <!-- registryAddress:zookeeper地址，用于服务注册发现协调的zookeeper的地址（必须部署zookeeper） -->
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

### 2.3.调用远程服务
 调用远程服务和被调用者必须使用相同的hip-wf框架，并且做响应spring配置.
 调用远程方法在参数传递和异常处理方面同调用本地方法。
```java
 //调用civ 域下的服务 demoServcie 中的getSome方法（如果该域部署多个，该调用将会随机负载）
 Map<String, Object> data = ( Map<String, Object>) Client.rpcInvoke("civ.demoServcie", "getSome");
 //传递参数
 Map<String, Object> data = ( Map<String, Object>) Client.rpcInvoke("civ.demoServcie", "getSome",arg1,arg2,arg3);
 //指定调用某ip下的服务
 Map<String, Object> data = ( Map<String, Object>) Client.rpcInvoke("civ.demoServcie", "getSome",new IpBalance("ip地址"),arg1,arg2,arg3);
 ```
 
该部分内容详细用法详见[ssdev wiki](http://)  
 
## 3.使用javaScript面向对象扩展
使用hip-wf开发的项目里所有资源文件（js,css）必须位于项目的classpath下（包括web项目，和依赖的jar中）
### 3.1.在页面上引入boot.js
   *boot.js作为引导js必须通过htlm标签的形式进行加载
```html
<!-- dependency.hip.boot为 boot.js 的classpath， script是加载js文件固定的写法 -->
<script type="text/javascript" src="script/dependency.hip.boot.js"></script>
```

### 3.2.使用$import引入js与css
以下JavaScript代码引用了jquery-191.js  与 demo.css 文件:
```js
//dependency.jquery.jquery-191 为 jquery-191.js 的classpath路径 （因为采用类似java中包的命名方式，而使用.作为目录的分割，所以文件名中禁止使用.）
//引用js文件可以省略.js 后缀，而引用.css必须使用后缀
$import([ "dependency.jquery.jquery-191", "demo.demo.css" ]);
//引用单个文件可直接使用字符串
$import("dependency.jquery.jquery-191");
```
由于所有的加载过程为异步，如果某些代码需要依赖某些文件，那么可以采用如下写法:
```js
$import([ "dependency.jquery.jquery-191", "demo.demo.css" ],function(){
    //这里可以使用jquery 的 $ 了
});
//由于query 尚未加载完 这里不能使用 $ 
```
当需要引入的多个js文件存在依赖关系时，为了避免多个$import嵌套的情况，可采用如下写法：
```js
//jqueryUI加载时依赖于jquery 
//将有依赖关系的文件分别至于数组中，那么他们就会按照前后位置进行依赖加载
$import([ ["dependency.jquery.jquery-191"] , [ "dependency.jqueryUI.jqueryUI" ] ],function(){
    
})；
```
### 3.3.使用$define定义js类
$define("类名称",Objcet);<br/>
以下为一个最基本的类的定义（满足所有OO中所有基本要素，可继承，可重写，可进行构造链执行等等）<br/>
一个物理文件只能定义一个类
```js
$define("demo.demoClass", { //类的名称(该名称必须和该文件的路径完全相同，类似于java class 的完整路径)
	extend : "dependency.hip.mod.modBase",//继承的类，以 ; 分割可继承多个类 
	init : function(args) {//构造函数
	},
	method1:function(){//方法1
	   this.method2();
	   this.$callSuper(arguments)();//调用父类的method1方法
	},
	method2:function(){//方法2
	   alert(this.attribute1);
	},
	attribute1:"0000"//字段
	....
)};
```
使用$ref引入某个实例
```js
$define("demo.demoClass", { //类的名称(该名称必须和该文件的路径完全相同，类似于java class 的完整路径)
	extend : "dependency.hip.mod.modBase",//继承的类，以 ; 分割可继承多个类 
	init : function(args) {//构造函数
	},
	demoClass2:$ref("demo.demoClass2")//当demo.demoClass 被实例化后，demoClass2将被赋值new demo.demoClass2();
)};
```
一般情况下在定义类前需要引入类执行过程中需要依赖的其他类
```js
$import(["dependency.jquery.jquery-191" , "demo.demoClass2" ]);//一般情况$define并不需要写入$import的callback中，因为在大部分的依赖都是执行过程中才需要的。
$define("demo.demoClass", { //类的名称(该名称必须和该文件的路径完全相同，类似于java class 的完整路径)
        ...
	method2:function(){//方法2
	    var demoClass2= new demo.demoClass2();//使用前必须先$import
	},
	...
)};
```

### 3.4.使用js类
使用 js 关键字 new 进行实例化：
```js
$import("demo.demoClass",function(){//使用某个类前必须保证该类已经被引入
    var demoClass = new demo.demoClass(“构造参数”);//使用完整类名进行实例化
    demoClass.method1();
})；
```
使用 $new 进行实例化
```js
$new("demo.demoClass","构造参数").then(function(clas) {
   clas.method1();
});
```

## 4.加载HTML片段
```xml
  1.该功能由类dependency.hip.mod.modBase实现。
  2.一个html片段的加载可能需要3个文件  1.js文件，继承dependency.hip.mod.modBase类，2.html文件，3.css文件（可选）
  3.3个文件必须是同名的位于相同的路径下（后缀不同），例如 demo.demoDiv.js  demo.demoDiv.html   demo.demoDiv.css 
```
```html
  <!--demo.demoDiv.html-->
  <div class="demoDiv" ></div>
```

```css
  /*demo.demoDiv.css*/
 .demoDiv {
     color: #FFF;
  }
```

### 4.1.定义用于处理HTML片段的类

```js
//demo.demoDiv.js
$import([ "dependency.jquery.jquery-191"]);
$define("demo.demoDiv", {
	extend : "dependency.hip.mod.modBase",//必须要继承该类
	onLoad : function(dom, args) {//当demo.demoDiv.html中定义的DOM被添加到页面上后触发 （dom ：HTMLDOM,args：自定义传入的参数）
             $(dom).text("123");
	} 
});
```

### 4.2.定义使用angular 1.x 处理HTML片段的类

```html
  <!--demo.demoDiv.html-->
  <div class="demoDiv" >{{name}}</div>
```

```js
//demo.demoDiv.js
$import([ "dependency.jquery.jquery-191"]);
$define("demo.demoDiv", {
	extend : "dependency.hip.mod.angularModBase",//必须要继承该类（不同于3.1中）
	init : function() {
	    this.angInject = [ "$injector" ];// 指定需要注入的模块
	},
	onLoad : function(dom, args, $scope,$injector) { 
            $scope.name = "123";
	} 
});
```

### 4.3.加载并使用HTML片段以及处理的类

