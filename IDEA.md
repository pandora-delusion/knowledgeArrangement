### idea使用maven下载jar包，出现证书校验问题问题

每次从github上下载下来的项目都报如下错误
could not transfer artifact org.springframework.boot:spring-boot-starter-parent:pom:2.2.5.RELEASE from/to nexus-aliyun (https://maven.aliyun.com/nexus/content/groups/public): PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target
解决方案：

在Maven命令后加入参数“-Dmaven.wagon.http.ssl.insecure=true -Dmaven.wagon.http.ssl.allowall=true”

参考：https://blog.csdn.net/xxaann/article/details/104794669

### IDEA 配置Maven以及修改默认Repository

参考：[IntellIJ IDEA 配置 Maven 以及 修改 默认 Repository](https://www.cnblogs.com/phpdragon/p/7216626.html)