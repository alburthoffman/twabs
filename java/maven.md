Maven Jar package
==============
## simple jar package
### plugin code
```xml
<plugin>  
    <groupId>org.apache.maven.plugins</groupId>  
    <artifactId>maven-jar-plugin</artifactId>  
    <version>2.4</version>  
    <configuration>  
        <archive>  
            <manifest>  
                <addClasspath>true</addClasspath>  
                <classpathPrefix>lib/</classpathPrefix>  
                <mainClass>com.sysware.HelloWorld</mainClass>  
            </manifest>  
        </archive>  
    </configuration>  
</plugin>
```
### command
```bash
$ mvn package
```

## package with dependency
### plugin code
```xml
<plugin>  
    <groupId>org.apache.maven.plugins</groupId>  
    <artifactId>maven-assembly-plugin</artifactId>  
    <version>2.3</version>  
    <configuration>  
        <appendAssemblyId>false</appendAssemblyId>  
        <descriptorRefs>  
            <descriptorRef>jar-with-dependencies</descriptorRef>  
        </descriptorRefs>  
        <archive>  
            <manifest>  
                <mainClass>com.juvenxu.mvnbook.helloworld.HelloWorld</mainClass>  
            </manifest>  
        </archive>  
    </configuration>  
    <executions>  
        <execution>  
            <id>make-assembly</id>  
            <phase>package</phase>  
            <goals>  
                <goal>assembly</goal>  
            </goals>  
        </execution>  
    </executions>  
</plugin>
```
### command
```bash
$ mvn assembly:assembly
```

## package resources and source files
### plugin code
```xml
<build>  
        <finalName>...</finalName>  
        <sourceDirectory>src/main/java</sourceDirectory>  
        <resources>  
            <!-- 控制资源文件的拷贝 -->  
            <resource>  
                <directory>src/main/resources</directory>  
                <targetPath>${project.build.directory}</targetPath>  
            </resource>  
        </resources>  
        <plugins>  
            <!-- 设置源文件编码方式 -->  
            <plugin>  
                <groupId>org.apache.maven.plugins</groupId>  
                <artifactId>maven-compiler-plugin</artifactId>  
                <configuration>  
                    <defaultLibBundleDir>lib</defaultLibBundleDir>  
                    <source>1.6</source>  
                    <target>1.6</target>  
                    <encoding>UTF-8</encoding>  
                </configuration>  
            </plugin>  
            <!-- 打包jar文件时，配置manifest文件，加入lib包的jar依赖 -->  
            <plugin>  
                <groupId>org.apache.maven.plugins</groupId>  
                <artifactId>maven-jar-plugin</artifactId>  
                <configuration>  
                    <archive>  
                        <manifest>  
                            <addClasspath>true</addClasspath>  
                            <classpathPrefix>lib/</classpathPrefix>  
                            <mainClass>.....MonitorMain</mainClass>  
                        </manifest>  
                    </archive>  
                </configuration>  
            </plugin>  
            <!-- 拷贝依赖的jar包到lib目录 -->  
            <plugin>  
                <groupId>org.apache.maven.plugins</groupId>  
                <artifactId>maven-dependency-plugin</artifactId>  
                <executions>  
                    <execution>  
                        <id>copy</id>  
                        <phase>package</phase>  
                        <goals>  
                            <goal>copy-dependencies</goal>  
                        </goals>  
                        <configuration>  
                            <outputDirectory>  
                                ${project.build.directory}/lib  
                            </outputDirectory>  
                        </configuration>  
                    </execution>  
                </executions>  
            </plugin>  
            <!-- 解决资源文件的编码问题 -->  
            <plugin>  
                <groupId>org.apache.maven.plugins</groupId>  
                <artifactId>maven-resources-plugin</artifactId>  
                <version>2.3</version>  
                <configuration>  
                    <encoding>UTF-8</encoding>  
                </configuration>  
            </plugin>  
            <!-- 打包source文件为jar文件 -->  
            <plugin>  
                <artifactId>maven-source-plugin</artifactId>  
                <version>2.1</version>  
                <configuration>  
                    <attach>true</attach>  
                    <encoding>UTF-8</encoding>  
                </configuration>  
                <executions>  
                    <execution>  
                        <phase>compile</phase>  
                        <goals>  
                            <goal>jar</goal>  
                        </goals>  
                    </execution>  
                </executions>  
            </plugin>  
        </plugins>  
    </build>
```
### command
```bash
$ mvn package
```
it will create xxx.jar, xxx-sources.jar, lib and resources directory.

## package with maven shade plugin
### plugin code`
```xml
<build>  
    <resources>  
        <resource>  
            <targetPath>${project.build.directory}/classes</targetPath>  
            <directory>src/main/resources</directory>  
            <filtering>true</filtering>  
            <includes>  
                <include>**/*.xml</include>  
            </includes>  
        </resource>  
    </resources>  
    <plugins>  
        <plugin>  
            <groupId>org.apache.maven.plugins</groupId>  
            <artifactId>maven-compiler-plugin</artifactId>  
            <version>3.0</version>  
            <configuration>  
                <source>1.6</source>  
                <target>1.6</target>  
                <encoding>UTF-8</encoding>  
            </configuration>  
        </plugin>  
        <plugin>  
            <groupId>org.apache.maven.plugins</groupId>  
            <artifactId>maven-shade-plugin</artifactId>  
            <version>2.0</version>  
            <executions>  
                <execution>  
                    <phase>package</phase>  
                    <goals>  
                        <goal>shade</goal>  
                    </goals>  
                    <configuration>  
                        <transformers>  
                            <transformer  
                                implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">  
                                <mainClass>com.test.testguava.app.App</mainClass>  
                            </transformer>  
                            <transformer  
                                implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">  
                                <resource>applicationContext.xml</resource>  
                            </transformer>  
                        </transformers>  
                        <shadedArtifactAttached>true</shadedArtifactAttached>  
                        <shadedClassifierName>executable</shadedClassifierName>  
                    </configuration>  
                </execution>  
            </executions>  
        </plugin>  
    </plugins>  
</build>
```

### command
```bash
mvn package
```
it will create xxx-...-executable.jar, xxx.jar. The first one is executable, has all dependencies packaged in.
