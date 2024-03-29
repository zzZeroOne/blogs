layout: paga
title: maven打包 代码依赖库分离
date: 2024-01-18 17:47:56
tags:

```
<build>
        <finalName>${project.artifactId}</finalName>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <fork>true</fork>
                    <maxAttempts>500</maxAttempts>
                    <jvmArguments>-Dfile.encoding=UTF-8</jvmArguments>
                    <mainClass>${start-class}</mainClass>
                    <layout>ZIP</layout>
                    <includes>
                        <!-- 这里只包含一个不存在的项nothing,即代表什么都不包含 -->
                        <include>
                            <groupId>nothing</groupId>
                            <artifactId>nothing</artifactId>
                        </include>
                    </includes>
                </configuration>
                <executions>
                    <execution>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
                <dependencies>
                    <dependency>
                        <groupId>org.springframework</groupId>
                        <artifactId>springloaded</artifactId>
                        <version>1.2.5.RELEASE</version>
                    </dependency>
                </dependencies>
            </plugin>
            <plugin>
                <artifactId>maven-assembly-plugin</artifactId>
                <configuration>
                    <appendAssemblyId>false</appendAssemblyId>
                    <descriptors>
                        <descriptor>src/assembly/package.xml</descriptor>
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

            <!-- 指定启动类，将依赖打成外部jar包 -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <configuration>
                    <archive>
                        <!-- 生成的jar中，不要包含pom.xml和pom.properties这两个文件 -->
                        <addMavenDescriptor>false</addMavenDescriptor>
                        <manifest>
                            <!-- 是否要把第三方jar加入到类构建路径 -->
                            <addClasspath>false</addClasspath>
                            <!-- 外部依赖jar包的最终位置 -->
                            <classpathPrefix>lib/</classpathPrefix>
                            <!-- 项目启动类 -->
                            <mainClass>${start-class}</mainClass>
                        </manifest>
                    </archive>
                </configuration>
            </plugin>
            <!--拷贝依赖到jar外面的lib目录-->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-dependency-plugin</artifactId>
                <executions>
                    <execution>
                        <id>copy-lib</id>
                        <phase>package</phase>
                        <goals>
                            <goal>copy-dependencies</goal>
                        </goals>
                        <configuration>
                            <outputDirectory>${project.build.directory}/lib</outputDirectory>
                            <excludeTransitive>false</excludeTransitive>
                            <stripVersion>false</stripVersion>
                            <includeScope>compile</includeScope>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
            <!--指定配置文件，将resources打成外部resource-->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <configuration>
                    <archive>
                        <!-- 指定配置文件目录，这样jar运行时会去找到同目录下的resources文件夹下查找 -->
                        <manifestEntries>
                            <Class-Path>resources/</Class-Path>
                        </manifestEntries>
                    </archive>
                    <!-- 打包时忽略的文件(也就是不打进jar包里的文件) -->
                    <excludes>
                        <exclude>*.yml</exclude>
                        <!--<exclude>*.xml</exclude>-->
                    </excludes>
                </configuration>
            </plugin>
            <!-- 拷贝资源文件 外面的resource目录-->
            <plugin>
                <artifactId>maven-resources-plugin</artifactId>
                <configuration>
                    <!-- 过滤后缀为pfx、cer、pfx的证书文件 -->
                    <nonFilteredFileExtensions>
                        <nonFilteredFileExtension>pfx</nonFilteredFileExtension>
                        <nonFilteredFileExtension>jks</nonFilteredFileExtension>
                        <nonFilteredFileExtension>cer</nonFilteredFileExtension>
                    </nonFilteredFileExtensions>
                </configuration>
                <executions>
                    <execution>
                        <id>copy-dependencies</id>
                        <phase>package</phase>
                        <goals>
                            <goal>copy-resources</goal>
                        </goals>
                        <configuration>
                            <!-- 资源文件输出目录 -->
                            <outputDirectory>${project.build.directory}/resources</outputDirectory>
                            <resources>
                                <resource>
                                    <directory>src/main/resources</directory>
                                </resource>
                            </resources>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
            <!--<plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-ssh-plugin</artifactId>
                <version>3.0.0</version>
                <executions>
                    <execution>
                        <id>upload-jar</id>
                        <phase>deploy</phase>
                        <goals>
                            <goal>exec</goal>
                        </goals>
                        <configuration>
                            <sshHost>192.168.10.240</sshHost>
                            <sshPort>22</sshPort>
                            <username>root</username>
                            <password>root</password>
                            <commands>
                                <command>scp target/myproject.jar root@192.168.10.240:/home/app</command>
                                <command>sh /home/app/bin/server.sh</command>
                            </commands>
                        </configuration>
                    </execution>
                </executions>
            </plugin>-->

        </plugins>

        <!--<extensions>
            <extension>
                <groupId>org.apache.maven.wagon</groupId>
                <artifactId>wagon-ssh</artifactId>
                <version>2.8</version>
            </extension>
        </extensions>-->
    </build>
```
