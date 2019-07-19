### deploy 包到Maven 官方仓库

1、创建账号
https://issues.sonatype.org/

2、提交 Community Support - Open Source Project Repository Hosting 申请，此处如果有域名一定绑定域名，没有域名的话可以用github来托管代码 io.github.username pom.xml 中的 groupId也得按这种格式来

3、配置pom.xml 和 settings.xml 文件

setting.xml
~~~
    <server>
      <id>ossrh</id>
      <username>XXX</username>
      <password>XXX</password>
    </server>

      <profile>
      <id>ossrh</id>
      <activation>
        <activeByDefault>true</activeByDefault>
      </activation>
      <properties>
        <gpg.executable>gpg</gpg.executable>
        <gpg.passphrase>XXX</gpg.passphrase>
      </properties>
    </profile>

    <activeProfiles>
    <activeProfile>ossrh</activeProfile>
  </activeProfiles>
~~~

pom.xml
~~~
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>io.github.snowthinker</groupId>
	<artifactId>config-helper</artifactId>
	<version>0.0.4-RELEASE</version>
	<packaging>jar</packaging>
	
	<name>config-helper</name>
	<url>https://github.com/snowThinker/config-helper</url>
	<description>Simplify property file load</description>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.apache.commons</groupId>
			<artifactId>commons-configuration2</artifactId>
			<version>2.4</version>
		</dependency>
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-api</artifactId>
			<version>1.7.25</version>
		</dependency>
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>4.12</version>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>2.5</version>
				<configuration>
					<source>1.8</source>
					<target>1.8</target>
					<encoding>UTF-8</encoding>
				</configuration>
			</plugin>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-source-plugin</artifactId>
				<version>2.2.1</version>
				<executions>
					<execution>
						<id>attach-sources</id>
						<goals>
							<goal>jar-no-fork</goal>
						</goals>
					</execution>
				</executions>
			</plugin>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-javadoc-plugin</artifactId>
				<version>2.9.1</version>
				<executions>
					<execution>
						<id>attach-javadocs</id>
						<goals>
							<goal>jar</goal>
						</goals>
					</execution>
				</executions>
			</plugin>
			<plugin>
				<groupId>org.sonatype.plugins</groupId>
				<artifactId>nexus-staging-maven-plugin</artifactId>
				<version>1.6.7</version>
				<extensions>true</extensions>
				<executions>
	                <execution>
	                   <id>default-deploy</id>
	                   <phase>deploy</phase>
	                   <goals>
	                      <goal>deploy</goal>
	                   </goals>
	                </execution>
	             </executions>
				<configuration>
					<serverId>ossrh</serverId>
					<nexusUrl>https://oss.sonatype.org/</nexusUrl>
					<autoReleaseAfterClose>true</autoReleaseAfterClose>
				</configuration>
			</plugin>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-gpg-plugin</artifactId>
				<version>1.6</version>
				<executions>
					<execution>
						<id>sign-artifacts</id>
						<phase>verify</phase>
						<goals>
							<goal>sign</goal>
						</goals>
					</execution>
				</executions>
			</plugin>
		</plugins>
	</build>

	<licenses>
		<license>
			<name>Apache License, Version 2.0</name>
			<url>http://www.apache.org/licenses/LICENSE-2.0.txt</url>
			<distribution>repo</distribution>
		</license>
	</licenses>

	<scm>
		<url>https://github.com/snowThinker/config-helper</url>
	</scm>
	
	<developers>
	    <developer>
	      <name>Andrew Wong</name>
	      <email>wj.andrew@yahoo.com</email>
	      <organization>Sonatype</organization>
	      <organizationUrl>http://www.sonatype.com</organizationUrl>
	    </developer>
    </developers>

	<distributionManagement>
		<snapshotRepository>
			<id>ossrh</id>
			<url>https://oss.sonatype.org/content/repositories/snapshots</url>
		</snapshotRepository>
		<repository>
			<id>ossrh</id>
			<url>https://oss.sonatype.org/service/local/staging/deploy/maven2/</url>
		</repository>
	</distributionManagement>
</project>

~~~


4、安装gpg4win 并配置证书信息
下载地址：https://www.gpg4win.org/download.html 

~~~
gpg --gen-key
gpg --keyserver hkp://keyserver.ubuntu.com:11371 --send-keys CF21873A  #上传到服务器
gpg --keyserver hkp://keyserver.ubuntu.com:11371 --recv-keys  CF21873A #查看是否上传整个
~~~

5、执行部署
mvn clean deploy