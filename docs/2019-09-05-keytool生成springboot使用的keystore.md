
	搭建本地https开发环境，需要使用密钥对，可以使用keytool来创建管理本地的keystore

	创建本地存放密钥的任意地址，D:D:/keystore

	经cmd，进入keytool的文件目录，我的是在C:\Program Files\Java\jdk1.8.0_171\bin

---

# 1. 为服务器生成证书
>
> 	keytool -genkey -alias tomcat -keypass 123456 -keyalg RSA -keysize 1024 -validity 365 -keystore D:/keystore/tomcat.keystore -storepass 123456 -dname CN=localhost,OU=localhost,L=SH,ST=SH,C=CN

	参数			|		说明
	----		|	:-----:
	-genkey		|	创建密钥对
	-keypass	|	指定别名tomcat密钥的私钥密码
	-keyalg 	|	指定密钥的算法，默认DSA 
	-keysize	|	指定密钥长度，默认1024
	-validity	|	证书有效期，默认90天
	-keystore	|	密钥的文件名与路径
	-storepass	|	获取密钥库内信息的密码
	-dname		|	指定证书拥有者信息,CN=名字与姓氏,OU=组织单位名称,O=组织名称,L=城市或区域名称,ST=州或省份名称,C=单位的两字母国家代码

# 2. 转换证书的格式为PKCS12
	
> keytool -importkeystore -srckeystore D:/keystore/tomcat.keystore -destkeystore D:/keystore/tomcat.keystore -deststoretype pkcs12

# 3. 生成客户端证书

>  keytool -genkey -alias client -keypass 123456 -keyalg RSA -storetype PKCS12 -keypass 123456 -storepass 123456 -keystore D:/keystore/client.p12 -dname CN=localhost,OU=localhost,L=SH,ST=SH,C=CN


	由于不能直接将PKCS12格式的证书库导入，必须先把客户端证书导出为一个单独的CER文件，使用如下命令：

>  keytool -export -alias client -keystore D:/keystore/client.p12 -storetype PKCS12 -keypass 123456 -file D:/keystore/client.cer


	将该文件导入到服务器的证书库，添加为一个信任证书：

>  keytool -import -v -file D:/keystore/client.cer -keystore D:/keystore/tomcat.keystore

	查看服务器证书库，会有两个，一个时服务器证书，一个是受信任的客户端证书
>  keytool -list -v -keystore D:/keystore/tomcat.keystore

	由于是双向SSL认证，客户端也要验证服务器证书，因此，必须把服务器证书添加到浏览器的“受信任的根证书颁发机构”。
	不能直接将keystore格式的证书库导入，必须先把服务器证书导出为一个单独的CER文件

>  keytool -keystore D:/keystore/tomcat.keystore -export -alias ordering -file D:/keystore/server.cer

# 4. chrome浏览器安装证书

双击server.cer文件，按照提示安装证书
将证书填入到“受信任的根证书颁发机构”。具体方法：（我用的谷歌浏览器）：
- 打开谷歌浏览器 --> 设置--> 高级 --> 管理证书 --> 受信任的根证书颁发机构 --> 导入server.cer

由于chrome加入了一种新的HSTS策略，使用该策略的网站，会强制浏览器使用HTTPS协议与该网站通信。

在chrome的地址栏里输入 chrome://net-internals/#hsts，把localhost从HSTS中删除
chrome://flags/#allow-insecure-localhost enable后重启，https://localhost就能访问了。


> 另：HTTPS和HTTP的区别在于，用HTTPS协议时传输的数据是加密的（TSL和SSL），而用HTTP传输时是明文传输。
> 
> 具体来说，HTTPS协议对传输内容使用的是对称加密算法，也即通信双方使用相同的密钥。但是对于密钥分发过程则使用的是公钥加密。但是我如何确认服务器不是别人伪造的呢？——我需要验证他的身份，即验证他的公钥。在服务器给我提供的证书中，有他声明的公钥Kp1，也有第三方用第三方自己的私钥(Ks0)加密服务器公钥(Kp1)后的密文(Ck1)。假如我相信第三方的身份是真实可信的，那么我用第三方的公钥(Kp0)，解密服务器的被第三方加密的公钥，和服务器直接发给我的公钥比较。如果相同，则验证成功；不同则验证失败。 
> 数学公式：证书构成（Kp1, Ck1, Kp0) ，其中Ck1==Eks0(Kp1)，若Kp1 == Dkp0(Ck1)则验证成功。
> 
> 这里一个重要的假设是我相信第三方，也即我相信证书是有效的（证书还包含其它信息来供大家确定是否有效，上段描述只是简化）。然而这里我自己搭建的服务器所提供的证书并不被chrome所信任，所以验证失败。
> 
> 造成证书不受信的可能的情况有： 
> 1. 第三方证书没有及时更新 
> 2. 第三方服务器不安全 
> 3. 证书不是由可信第三方颁布

# spring cloud 配置 #
1. 所有工程内指定keystore的位置

	server:
      ssl:
	    key-store: D:/keystore/tomcat.keystore
	    key-store-password: 123456
	    key-store-type: JKS
	    key-alias: ordering


或者使用PKCS12 格式
	server:
		ssl:
			key-store: classpath:server-dev.p12
		    key-store-password: 123456
		    key-store-type: PKCS12
		    key-alias: server

注意：classpath路径，maven在编译时会在p12文件后面追加字符，造成密钥文件出错，需要在pom文件内使用resource标签引入文件
或者将文件放在外部路径。


2. eureka配置
    
    	eureka:
			      instance:
				      hostname: localhost   # 本地开发使用hostname需要在系统文件hosts内添加IP与hostname的对应
				      secure-port-enabled: true
				      non-secure-port-enabled: false
				      secure-port: ${server.port}
				      status-page-url: https://${eureka.instance.hostname}:${server.port}/info
				      prefer-ip-address: true
				      health-check-url: https://${eureka.instance.hostname}:${server.port}/health
				      home-page-url: https://${eureka.instance.hostname}:${server.port}/
				   server:
					    renewal-percent-threshold: 0.49
					    eviction-interval-timer-in-ms: 60000
				   client:
					    register-with-eureka: false #false表示不向注册中心注册自己。
					    fetch-registry: false #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
					    service-url:
	      					defaultZone: https://admin:123@${eureka.instance.hostname}:${server.port}/eureka/
    

3. 微服务连接eureka
		
		eureka:
		  client:
		    enabled: true
		    register-with-eureka: true
		    fetch-registry: true
		    service-url:
		      defaultZone: https://admin:123@localhost:7001/eureka/
		  instance:
		    prefer-ip-address: true
		    instance-id: ${spring.application.name}:${spring.application.ip-address:${server.port}}
		    secure-port: ${server.port}
		    non-secure-port-enabled: false
		    secure-port-enabled: true


