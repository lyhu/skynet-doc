# 3.1.1 服务开发

开发步骤

1. 引入 Skynet起步依赖
2. 实现插件接口
3. 插件Action配置
4. 编译打包
5. 部署测试

## 1. Skynet起步依赖

目前skynet所依赖的包全部放在Maven仓库上面，配置仓库地址，则可以获得skynet全部依赖。

> pom.xml maven 依赖

```markup
<parent>
    <groupId>com.iflytek.skynet</groupId>
    <artifactId>skynet-cloud-starter-parent</artifactId>
    <version>{版本号}</version>
</parent>

<properties>
    <mvn-repo>http://pl.maven.iflytek.com/nexus</mvn-repo>
</properties>

<repositories>
    <repository>
        <id>repo1-cache</id>
        <name>repo1-cache</name>
        <url>${mvn-repo}/content/groups/public</url>
        <snapshots>
            <enabled>false</enabled>
        </snapshots>
    </repository>
    <repository>
        <id>repo2-cache</id>
        <name>repo2-cache</name>
        <url>${mvn-repo}/content/repositories/skynet-releases</url>
        <snapshots>
            <enabled>false</enabled>
        </snapshots>
    </repository>
    <repository>
        <id>repo3-cache</id>
        <name>repo3-cache</name>
        <url>${mvn-repo}/content/repositories/skynet-snapshots</url>
        <snapshots>
            <enabled>true</enabled>
        </snapshots>
    </repository>
</repositories>
```

## 2. 实现插件接口

以Web服务为例，需要实现RestSvcContext接口，下面是接口定义

```java
public interface RestSvcContext extends AutoCloseable {

    /**
     * 服务 名称
     */
    String getSvcName();

    /**
     * 初始化
     */
    void init(AppContext appContext, Map<String, Object> params) throws Exception;

    /**
     * 服务状态
     */
    Map<String, Object> getState() throws Exception;
}
```

实现类示例

```java
@AntAction(name = RestSvc.SvcName, type = AntActionType.rest)
@Component(RestSvc.SvcName)
public class RestSvc implements RestSvcContext {

    public static final String SvcName = "sample.rest.app";

    @Override
    public String getSvcName() {
        return null;
    }

    @Override
    public void init(AppContext appContext, Map<String, Object> params) throws Exception {
        // 开始进行引擎初始化
    }

    @Override
    public Map<String, Object> getState() {
        return null;
    }

    @Override
    public void close() throws Exception {

    }
}
```

## 3. Action配置

skynet服务开发完成后，需要对配置zk节点，下面是zk节点配置示例

```text
/skynet/plugin/sample/action/rest-app-v10=_atitle=[Sample]Web应用
/skynet/plugin/sample/action/rest-app-v10=_desc=Cloud-Sample站点服务
/skynet/plugin/sample/action/rest-app-v10=_param={\n            "name": "skynet.rest.host",\n            "context": {\n                "svc": "sample.rest.app"\n            }\n        }/skynet/plugin/sample/service=_desc=skynet基础服务插件_服务定义
/skynet/plugin/sample/service=skynet.rest.host=RestWeb宿主服务
/skynet/plugin/sample/setting=_desc=skynet基础服务插件_自定义配置
/skynet/plugin/sample/setting=_mq_config={\n    "mq_type": "amq",\n    "mq_uri": "{$ref@setting:mq_uri}",\n    "inq": "amq_starter_message",\n    "concurrency": 1\n}
/skynet/plugin/sample/setting=mq_uri=tcp://192.168.84.190:61616
/skynet/plugin/sample=_desc=Skynet二次开发示例，包括一些开发测试服务。
/skynet/plugin/sample=_name=Skynet示例
```

## 4.编译打包

服务开发完成后一般用maven的assembly插件，将工程打成相应的zip或者jar包，下面是maven使用assembly设置，在pom文件里引入如下内容

```markup
<build>
		<plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-assembly-plugin</artifactId>
				<configuration>
					<descriptors>
						<descriptor>assembly.xml</descriptor>
					</descriptors>
				</configuration>
				<executions>
					<execution>
						<id>make-assembly</id>
						<phase>package</phase>
						<goals>
							<goal>single</goal>
						</goals>
					</execution>
				</executions>
			</plugin>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-deploy-plugin</artifactId>
				<configuration>
					<skip>${maven.deploy.skip}</skip>
				</configuration>
			</plugin>
		</plugins>
	</build>
```

下面是assembly配置示例

```markup
<assembly xmlns="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.2"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/plugins/mavenassembly-plugin/assembly/1.1.2 http://maven.apache.org/xsd/assembly-1.1.2.xsd">
    <id>RELEASE</id>
    <formats>
        <format>dir</format>
        <format>zip</format>
    </formats>
    <dependencySets>
        <dependencySet>
            <useProjectArtifact>false</useProjectArtifact>
            <outputDirectory>./skynet/lib</outputDirectory>
        </dependencySet>
    </dependencySets>

    <fileSets>
        <fileSet>
            <directory>misc</directory>
            <outputDirectory>/</outputDirectory>
            <includes>
                <include>**/*</include>
            </includes>
        </fileSet>  
        
        <fileSet>
            <directory>../skynet-boot-project/skynet-cloud-xservice/target</directory>
            <outputDirectory>./skynet/plugin/ant/lib</outputDirectory>
            <includes>
                <include>*.jar</include>
            </includes>
            <excludes>
                <exclude>*javadoc.jar</exclude>
                <exclude>*sources.jar</exclude>
            </excludes>
        </fileSet>
        <fileSet>
            <directory>../skynet-boot-project/skynet-boot-jagent/target</directory>
            <outputDirectory>./skynet/lib/jagent</outputDirectory>
            <includes>
                <include>*.jar</include>
            </includes>
            <excludes>
                <exclude>*javadoc.jar</exclude>
                <exclude>*sources.jar</exclude>
            </excludes>
        </fileSet> 
                
    </fileSets>
</assembly>

```

## 5. 部署测试

打包完成后，将打成的包放在/iflytek/server/skynet/plugin下面，然后在skynet控制台页面，分配该服务，再对此服务进行测试即可。

