---
title: maven与gradle下cucumber-jvm环境搭建
date: 2018-05-31 08:46:04
tags: [cucumber，gradle，maven]
---

## 前言

cucumber不需多说，BDD工具，目前被我们用于实例化接口文档。由于实施的两个项目分别使用了maven和gradle构建工具，所以需要考虑maven和gradle下的cucumber-jvm框架的搭建。

## maven

### 配置

- `pom.xml`文件中添加依赖：

```xml
        <cucumber.version>1.2.5</cucumber.version>
        <cucumber-runner.version>1.3.3</cucumber-runner.version>
        <junit.version>4.12</junit.version>

  <!--  cucumber -->
        <dependency>
            <groupId>info.cukes</groupId>
            <artifactId>cucumber-java</artifactId>
            <version>${cucumber.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>info.cukes</groupId>
            <artifactId>cucumber-junit</artifactId>
            <version>${cucumber.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>${junit.version}</version>
            <scope>test</scope>
        </dependency>
```

- 添加插件配置

```xml
  <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>2.19.1</version>
                <configuration>
                    <skip>${skipSurefire}</skip>
                    <excludes>
                        <exclude>**/RunBDDTest.java</exclude>
                    </excludes>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-failsafe-plugin</artifactId>
                <version>2.19.1</version>
                <configuration>
                    <includes>
                        <include>**/RunBDDTest.java</include>
                    </includes>
                </configuration>
                <executions>
                    <execution>
                        <id>integration-test</id>
                        <goals>
                            <goal>integration-test</goal>
                        </goals>
                    </execution>
                    <execution>
                        <id>verify</id>
                        <goals>
                            <goal>verify</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
```
### 实现

- `src/test/resouces`下新建`feature`文件（略过）

- `src/test/java`下新建`step`实现（略过）

- `src/test/java`下新建测试类`RunBDDTest`：

```java
package com.angrytest.stepdefs;

import com.github.mkolisnyk.cucumber.runner.ExtendedCucumber;
import com.github.mkolisnyk.cucumber.runner.ExtendedCucumberOptions;
import cucumber.api.CucumberOptions;
import org.junit.runner.RunWith;

@RunWith(ExtendedCucumber.class)
@ExtendedCucumberOptions(jsonReport = "target/cucumber.json",
        overviewReport = true,
        outputFolder = "target")
@CucumberOptions(
        features = "src/test/resouces/features",
        plugin = {"pretty", "html:target/cucumber-html-report", "json:target/cucumber.json"},
        glue = {"com.angrytest.stepdefs"}
)
public class RunBDDTest {

}
```

### 执行

可通过ide直接执行，也可通过以下命令执行：

```
mvn integration-test -DskipSurefire=true
```

加`skipSurefire=true`主要是为了将BDD测试和原有单元测试区分开

报告部分还可以参考：http://mkolisnyk.github.io/cucumber-reports/

## gradle

### 配置

- `build.gradle`文件中添加依赖

```
dependencies {
    testCompile 'io.cucumber:cucumber-java:2.4.0'
    testCompile 'io.cucumber:cucumber-junit:2.4.0'
    testCompile 'com.vimalselvam:cucumber-extentsreport:3.0.1'
    testCompile 'com.aventstack:extentreports:3.0.6'
}

```

- 添加执行配置

```
task cucumber() {
    dependsOn assemble, compileTestJava
    doLast {
        javaexec {
            main = "cucumber.api.cli.Main"
            classpath = configurations.cucumberRuntime + sourceSets.main.output + sourceSets.test.output
            args = ['--plugin', 'html:cucumber-html-report', '--plugin', 'json:build/cucumber-report.json', '--glue', 'com.angrytest.stepdefs', 'src/test/resources']
        }
    }
}
```
### 实现

与maven基本一致

### 执行

```
gradle cucumber
```
## 后记

- 建议ide使用idea，插件更强大，调试更方便。

- eclipse插件在结合rest-assured使用时可能会出现依赖冲突，可加入如下依赖解决：

```
    <dependency>
        <groupId>org.codehaus.groovy</groupId>
        <artifactId>groovy-xml</artifactId>
        <version>2.4.13</version>
    </dependency>
```

- 建议BDD测试代码与单元测试代码放在一起，可以和项目代码一起做版本控制。