
## 安装 Eclipse Scala Maven 插件
m2eclipse-scala

http://alchim31.free.fr/m2e-scala/update-site


~~~xml
	<properties>
		<java.version>1.8</java.version>
		<scala.version>2.12.8</scala.version>
	</properties>

    <build>
		<sourceDirectory>src/main/scala</sourceDirectory>
    		<testSourceDirectory>src/test/scala</testSourceDirectory>
		<plugins>
			<plugin>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>3.1</version>
				<configuration>
					<source>${java.version}</source>
					<target>${java.version}</target>
				</configuration>
			</plugin>
			<plugin>
				<groupId>net.alchim31.maven</groupId>
				<artifactId>scala-maven-plugin</artifactId>
				<version>4.1.0</version>
				<configuration>
					<scalaVersion>${scala.version}</scalaVersion>
				</configuration>
				<executions>
					<execution>
				      <goals>
				        <goal>compile</goal>
				        <goal>testCompile</goal>
				      </goals>
				    </execution>
				</executions>
			</plugin>
		</plugins>
	</build>
~~~