---
layout: post
title:  "SpringBoot服务一种打包技术"
date:   2021-12-09 18:14:54
categories: SpringBoot
tags: SpringBoot
excerpt: SpringBoot服务一种打包技术
mathjax: true

---

* content
{:toc}
## 针对企业级的面向微服务的打包

针对maven的pom，将资源进行分离拷贝，输出脚本，和jar包，这是在企业经常使用的手段，如果我们的工程众多，也可以最后将jar包统一配置管理。从以下几方面说明。

## 配置打包输出路径配置

```
  <properties>
  		<project.out.version>${project.build.directory}/${project.build.finalName}/version</project.out.version>
  </properties>
```

## 配置资源路径和打包命令

以下在build配置

```
<!-- 打包后的jar包名称 -->
<finalName>epoch-system-service</finalName>
<resources>
    <!--这里配置，本地启动编译使用，否则导致启动无法构建配置，可以根据自己路径多个配置-->
    <resource>      
    	<directory>src/main/resources</directory>    
    </resource>
    <!--这里配置，本地启动编译使用，否则导致启动无法构建配置，可以根据自己路径多个配置-->
    <resource>    
    	<directory>src/main/resources</directory>    
    	<includes>
    		<include>**/*</include>
    	</includes>
    	<targetPath>${project.out.version}</targetPath>
    </resource>
    <!-- 拷贝的资源路径，以及输出的目标路径,这里是面向了启动脚本 -->
    <resource>
    	<directory>${basedir}/script</directory>    
    	<includes>
    		<include>*.bat</include>
    		<include>*.sh</include>
    	</includes>
    	<targetPath>${project.out.version}</targetPath>
    </resource> 
</resources>
```

## plugins配置

### 配置编译JDK版本

```
<plugin>
	<artifactId>maven-compiler-plugin</artifactId>
	<configuration>
		<source>1.8</source>
		<target>1.8</target>
		<encoding>UTF-8</encoding>
	</configuration>
</plugin>
```

### 配置核心启动类内部

```
<plugin>
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-jar-plugin</artifactId>
	<configuration>
		<archive>
			<manifest>
				<mainClass>com.epoch.sys.Application</mainClass>
				<addClasspath>false</addClasspath>
				<classpathPrefix>lib/</classpathPrefix>
				<addDefaultSpecificationEntries>true</addDefaultSpecificationEntries>
			</manifest>
		</archive>
	    <outputDirectory>${project.out.version}</outputDirectory>
	</configuration>
</plugin>
```

### 配置拷贝jar包

这里在spring tool suite可能会有红X，但是其实是一个误报BUG

```
<plugin>
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-dependency-plugin</artifactId>
	<executions>
		<execution>
			<id>copy-dependencies</id>
			<phase>package</phase>
			<goals>
				<goal>copy-dependencies</goal>
			</goals>
			<configuration>
				<outputDirectory>
					${project.out.version}/lib
				</outputDirectory>
				<overWriteReleases>false</overWriteReleases>
				<overWriteSnapshots>false</overWriteSnapshots>
				<overWriteIfNewer>true</overWriteIfNewer>
			</configuration>
		</execution>
	</executions>
</plugin>
```

说一下统一管理jar包，其实无非就是将第三方包放到统一目录，比如../..父目录即可。针对内部的包可以单独放置。后面会进行说明。