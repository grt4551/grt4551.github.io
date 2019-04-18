---
layout: post
title: 解决maven报错[转]
tags:
  - java
  - maven
categories: maven
published: true
---
#### 导入Myabtis源码后，POM文件会报出如下异常：
```code
Plugin execution not covered by lifecycle configuration org.sonatype.plugins:jarjar-maven-plugin  
Plugin execution not covered by lifecycle configuration org.apache.felix:maven-bundle-plugin
```

#### 在这里找到了问题的原因和解决办法：
>http://wiki.eclipse.org/M2E_plugin_execution_not_covered

m2e在eclipse中执行maven生命周期构建，配置完毕后执行Maven构建后的项目。这是被多个不同Maven目标控制的。有些目标在workspace层面控制，有些在project/.setting下控制。

<!--more-->

但是在特殊情况下还是会有异常。主要原因有2个：

>1、workspace外部的资源修改了，使得Maven插件构建workspace出现异常。  

>2、在不同的JVM和系统下，maven插件可能会导致内存泄露。
为了解决这些长期存在的问题，m2e插件需要知道每个Maven插件的生命周期。这就需要用到"project build lifecycle mapping" 或者 "lifecycle mapping"。

由于Mybatis的牛人们都不用m2e插件，而是自己用指令控制Maven操作。所以有些插件对于m2e来说是没有用到的。现在只需要告诉m2e插件忽略检查这些插件的生命周期就好。

在文章的结尾我也找到了解决办法：
```xml
Window-Perferences-Maven-Lifecycle Mapping
点击Open workspace lifecycle mappings metadata
加入如下内容：
<lifecycleMappingMetadata>
    <pluginExecutions>
        <pluginExecution>
            <pluginExecutionFilter>
                <groupId>org.sonatype.plugins</groupId>
                <artifactId>jarjar-maven-plugin</artifactId>
                <versionRange>[1.7,)</versionRange>
                <goals>
                    <goal>jarjar</goal>
                </goals>
            </pluginExecutionFilter>
            <action>
                <ignore />
            </action>
        </pluginExecution>
        <pluginExecution>
            <pluginExecutionFilter>
                <groupId>org.apache.felix</groupId>
                <artifactId>maven-bundle-plugin</artifactId>
                <versionRange>[2.3.7,)</versionRange>
                <goals>
                    <goal>manifest</goal>
                </goals>
            </pluginExecutionFilter>
            <action>
                <ignore />
            </action>
        </pluginExecution>
        <pluginExecution>
            <pluginExecutionFilter>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-enforcer-plugin</artifactId>
                <versionRange>[1.0.0,)</versionRange>
                <goals>
                    <goal>enforce</goal>
                </goals>
            </pluginExecutionFilter>
            <action>
                <ignore />
            </action>
        </pluginExecution>
    </pluginExecutions>
</lifecycleMappingMetadata>
```