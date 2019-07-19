### maven 配置checkstyle


#### maven 配置
~~~
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-checkstyle-plugin</artifactId>
    <version>3.0.0</version>
    <dependencies>
        <dependency>
            <groupId>com.puppycrawl.tools</groupId>
            <artifactId>checkstyle</artifactId>
            <version>8.20</version>
        </dependency>
    </dependencies>
    <executions>
        <execution>
            <configuration>
                <configLocation>checkstyle.xml</configLocation>
                <encoding>UTF-8</encoding>
                <consoleOutput>true</consoleOutput>
                <failsOnError>true</failsOnError>
                <skip>false</skip>
            </configuration>
            <goals>
                <goal>check</goal>
            </goals>
        </execution>
    </executions>
</plugin>
~~~

#### checkstyle模板
https://github.com/checkstyle/checkstyle/tree/master/src/main/resources

可修改 severity 为 error，当编译违反规范的时候阻止编译打包
~~~
<property name="severity" value="error"/>
~~~
