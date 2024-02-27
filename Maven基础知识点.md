- [前言](#前言)
- [一, 仓库: 用于存储资源, 包含各种jar包(本地仓库, 私服仓库, 中央仓库)](#一-仓库-用于存储资源-包含各种jar包本地仓库-私服仓库-中央仓库)
- [二, 坐标: Maven中的坐标用于描述仓库中资源的位置](#二-坐标-maven中的坐标用于描述仓库中资源的位置)
- [三, 本地仓库配置](#三-本地仓库配置)
- [四, 第一个Maven程序, Maven项目结构](#四-第一个maven程序-maven项目结构)
  - [1, 在src同层目录下创建pom.xml](#1-在src同层目录下创建pomxml)
  - [2, 在pom.xml所在文件夹下进行cmd运行指令](#2-在pomxml所在文件夹下进行cmd运行指令)
- [五, 插件创建工程(要求所在位置不是maven工程的结构, 没有pom.xml的文件)](#五-插件创建工程要求所在位置不是maven工程的结构-没有pomxml的文件)
  - [1, java工程插件创建:](#1-java工程插件创建)
  - [2, web工程创建:](#2-web工程创建)
- [六, 依赖管理](#六-依赖管理)
  - [1, 依赖配置:](#1-依赖配置)
  - [2, 依赖传递:](#2-依赖传递)
  - [3, 依赖传递冲突问题:](#3-依赖传递冲突问题)
  - [4, 可选依赖: 指对外隐藏当前所依赖的资源--不透明](#4-可选依赖-指对外隐藏当前所依赖的资源--不透明)
  - [5, 排除依赖:指项目主动断开依赖的资源, 被排除的资源无需指定版本](#5-排除依赖指项目主动断开依赖的资源-被排除的资源无需指定版本)
  - [6, 依赖范围: 通过限定jar包的适用范围](#6-依赖范围-通过限定jar包的适用范围)
  - [7, 依赖范围传递性: 带有依赖范围的资源在进行传递时, 作用范围将受到影响](#7-依赖范围传递性-带有依赖范围的资源在进行传递时-作用范围将受到影响)
- [七, 生命周期与插件](#七-生命周期与插件)
  - [1, 项目构建生命周期:](#1-项目构建生命周期)
  - [2, 插件:](#2-插件)


---
# 前言
>学习maven的理由和maven自身的优点
项目构建:  提供标准的, 跨平台的自动化构建方式
依赖管理: 方便快捷的管理项目依赖的资源(jar包), 避免资源见的版本冲突问题
统一开发: 提供标准的,统一的项目结构
POM: project object model
必备网站: https://mvnrepository.com

---

# 一, 仓库: 用于存储资源, 包含各种jar包(本地仓库, 私服仓库, 中央仓库)

1. 本地仓库, 远程仓库(私服, 中央)

2. 私服的作用: 

- 保存具有版权的资源, 包含购买或自主研发的jar包, 中央仓库的jar都是开源的, 不能存储具有版权的资源
- 一定范围内共享资源, 仅对内部开放, 不对外共享

# 二, 坐标: Maven中的坐标用于描述仓库中资源的位置
>maven的下载地址: https://repo1.maven.org/maven2/

主要组成:  
- groupId: 定义当前maven项目的隶属组织名称(域名反写:org.myproject)
- artifactId: 定义当前项目名称(如:CRM, SMS)
- version: 定义当前版本号
- packaging: 定义该项目的打包方式

**坐标的作用:** 使用唯一标识, 唯一性定位资源位置, 通过该标识可以将资源的 识别与下载工作交由机器完成

# 三, 本地仓库配置
在maven的conf文件里的settings.xml里配置国内镜像资源
```xml
    <mirrors>
    <mirror>
        <id>alimaven</id>
        <mirrorOf>central</mirrorOf>
        <name>aliyun maven</name>
        <url>http://maven.aliyun.com/nexus/content/groups/public/</url>       
    </mirror>
   </mirrors>
```

# 四, 第一个Maven程序, Maven项目结构
## 1, 在src同层目录下创建pom.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project 
    xmlns="http://maven.apache.org/POM/4.0.0" 
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.itstudy</groupId>
    <artifactId>project-java</artifactId>
    <version>1.0</serion>
    <packaging>jar</packaging>

    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>
    </dependencies>

</project>
```

## 2, 在pom.xml所在文件夹下进行cmd运行指令
Maven指令构建项目:  
- mvn compile  	#编译产生文件夹
- mvn clean      	#清理编译的文件夹
- mvn test        	#测试相当于调试了一下, 产生的报告在 target\surefire-reports
- mvn package	#打包, 只打包源程序(会执行前面的一些编译测试指令, 编译无误再打包)
- mvn install	#安装到本地仓库


# 五, 插件创建工程(要求所在位置不是maven工程的结构, 没有pom.xml的文件)
## 1, java工程插件创建:

mvn指令如下
```cmd
mvn archetype:generate -DgroupId=com.itstudy -DartifactId=java-project -DarchetypeArtifactId=maven-archetype-quickstart -Dversion=0.0.1-snapshot -DinteractiveMode=false
```

## 2, web工程创建:
mvn指令如下
```cmd
mvn archetype:generate -DgroupId=com.itstudy -DartifactId=web-project -DarchetypeArtifactId=maven-archetype-webapp -Dversion=0.0.1-snapshot -DinteractiveMode=false
```

# 六, 依赖管理
## 1, 依赖配置:
```xml
<dependencies>
    <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13.1</version>
            <scope>test</scope>
    </dependency>
</dependencies>
```
## 2, 依赖传递:
依赖具有传递性: 
- 直接依赖:再当前项目中通过依赖配置建立的依赖关系
- 间接依赖:被资源的资源如果依赖其他资源,当前项目间接依赖其他资源
## 3, 依赖传递冲突问题:
- 路径优先: 当依赖中出现相同资源时,层级越深,优先级越低,反之越高
- 声明优先: 当资源在相同层级被依赖时,配置顺序靠前的覆盖配置顺序靠后的
- 特殊优先: 当同级配置了相同资源的不同版本, 后配置的覆盖先配置的
## 4, 可选依赖: 指对外隐藏当前所依赖的资源--不透明 
加上option属性`<optional>true</optional>`
```xml
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.12</version>
    <optional>true</optional>
</dependency>
```
## 5, 排除依赖:指项目主动断开依赖的资源, 被排除的资源无需指定版本
属性标签为`<exclusions></exclusions>`
```xml
<dependency>
    <groupId>com.itstudy</groupId>
    <artifactId>java02</artifactId>
    <version>1.0-SNAPSHOT</version>
    <exclusions>
        <exclusion>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```
## 6, 依赖范围: 通过<scope></scope>限定jar包的适用范围
作用范围:` main文件夹范围内, test文件夹范围内, 是否参与打包(package指令范围内)`

- scope: compile(默认main test package都能用)
- test(test里能用)	
- provided(main test里能用)
- runtime(package可以打包但main test不用, 所以很少配)

## 7, 依赖范围传递性: 带有依赖范围的资源在进行传递时, 作用范围将受到影响

# 七, 生命周期与插件
## 1, 项目构建生命周期: 
描述构建项目经过了几个过程
- clean: 清理工作
- default: compile  test-compile  test  package - install
- site: 产生报告, 发布站点等
## 2, 插件: 
插件与生命周期的阶段绑定, 执行生命周期时执行对应的插件, 通过插件可以自定义其他功能
```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-source-plugin</artifactId>
            <version>3.2.1</version>
            <executions>
                <execution>
                    <goals>
                        <goal>jar</goal>
                        <goal>test-jar</goal>
                    </goals>
                    <phase>generate-test-resources</phase>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```