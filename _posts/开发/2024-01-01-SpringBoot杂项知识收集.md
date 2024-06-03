---
title: SpringBoot 杂项知识收集
categories: 开发
toc: true
permalink: /springboot-mix-note.html
---

## 1. 自定义 Banner

### 创建字符画

可以到 <http://patorjk.com/software/taag/> 创建，个人比较喜欢的字体是 Big、Doom、Standard、Star Wars、Ivrit，这几款字体比较规整，Character Width 和 Character Height 配置为 Full。字符画保存到 src/main/resources/banner.txt 文件中

```text
  _    _   _____  __          __  ______   _____
 | |  | | |_   _| \ \        / / |___  /  / ____|
 | |__| |   | |    \ \  /\  / /     / /  | |
 |  __  |   | |     \ \/  \/ /     / /   | |
 | |  | |  _| |_     \  /\  /     / /__  | |____
 |_|  |_| |_____|     \/  \/     /_____|  \_____|
```

### 使用变量

| 变量                             | 值示例            |
| -------------------------------- | ----------------- |
| ${spring-boot.formatted-version} | (v2.1.11.RELEASE) |
| ${spring-boot.version}           | 2.1.11.RELEASE    |
| ${application.formatted-version} | (v1.0.0)          |
| ${application.version}           | 1.0.0             |
| ${application.title}             | My application    |

### 使用颜色

使用类似 `${AnsiColor.BRIGHT_RED}` 指定前景色;

使用类似 `${AnsiBackground.WHITE}` 指定背景色彩;

使用类似 `${AnsiStyle.BOLD}` 指定字体；

这些可用的值定义在 `org.springframework.boot.ansi` 包下相应的类里。

## 2. Window 命令行启动 SpringBoot 应用


主要是方便本地调试。

### 使用 spring-boot 插件

spring-boot-maven-plugin-2.2.2.RELEASE

```bat
@ECHO OFF
SETLOCAL

REM 请设置基本环境变量
SET JAVA_HOME=C:\Program Files (x86)\Java\jdk1.8.0_45
SET M2_HOME=C:\Program Files (x86)\apache-maven-3.6.3
SET DEBUG_PORT=8888
SET LOG_PREFIX=D:\log
SET CLEAN_CMD=

REM 自动检测当前目录
SET APP_HOME=%~dp0

REM 自动获取当前目录名
SET APP_NAME=
SET TMP_ARRAY=%APP_HOME:\= %
FOR %%a in (%TMP_ARRAY%) do SET APP_NAME=%%a

REM 日志目录
SET LOG_HOME=%LOG_PREFIX%\%APP_NAME%

REM 设置变量
SET JAVA_OPTS=-Xms1024M -Xmx1024M -Dlog.home=%LOG_HOME%
SET JAVA_DEBUG=-Xdebug -Xnoagent -Xrunjdwp:transport=dt_socket,address=%DEBUG_PORT%,server=y,suspend=n
SET jvmArguments="%JAVA_OPTS% %JAVA_DEBUG%"

ECHO=
ECHO JAVA_HOME=%JAVA_HOME%
ECHO M2_HOME=%M2_HOME%
ECHO APP_HOME=%APP_HOME%
ECHO APP_NAME=%APP_NAME%
ECHO LOG_HOME=%LOG_HOME%
ECHO DEBUG_PORT=%DEBUG_PORT%

IF "%1%"=="c" SET CLEAN_CMD=clean
ECHO CLEAN=%CLEAN_CMD%
IF "%1%"=="h" GOTO HELP

:MAIN
REM 执行程序
CD %APP_HOME% && CALL "%M2_HOME%\bin\mvn" -q -Dmaven.test.skip=true -Dspring-boot.run.noverify=true -Dspring-boot.run.jvmArguments=%jvmArguments% %CLEAN_CMD% spring-boot:run
GOTO END

:HELP
ECHO=
ECHO Usage:
ECHO     start.cmd      compile and start
ECHO     start.cmd c    clean, compile and start
ECHO     start.cmd h    show help
GOTO END

:END
ENDLOCAL
```

### 使用 java 命令

```bat
@ECHO OFF
SETLOCAL

REM 请设置基本环境变量
SET JAVA_HOME=C:\Program Files (x86)\Java\jdk1.8.0_45
SET M2_HOME=C:\Program Files (x86)\apache-maven-3.6.3
SET DEBUG_PORT=8888
SET LOG_PREFIX=D:\log
SET CLEAN_CMD=

REM 自动检测当前目录
SET APP_HOME=%~dp0
SET JAR_HOME=%APP_HOME%target
SET CFG_HOME=D:\wzctmp\java-demo\config

REM 自动获取当前目录名
SET APP_NAME=
SET TMP_ARRAY=%APP_HOME:\= %
FOR %%a in (%TMP_ARRAY%) DO SET APP_NAME=%%a

REM 日志目录
SET LOG_HOME=%LOG_PREFIX%\%APP_NAME%

REM 设置变量
SET JAVA_OPTS=-Xms1024M -Xmx1024M -Dlog.home=%LOG_HOME%
SET JAVA_DEBUG=-Xdebug -Xnoagent -Xrunjdwp:transport=dt_socket,address=%DEBUG_PORT%,server=y,suspend=n
SET jvmArguments="%JAVA_OPTS% %JAVA_DEBUG%"

ECHO=
ECHO JAVA_HOME=%JAVA_HOME%
ECHO M2_HOME=%M2_HOME%
ECHO APP_HOME=%APP_HOME%
ECHO APP_NAME=%APP_NAME%
ECHO JAR_HOME=%JAR_HOME%
ECHO CFG_HOME=%CFG_HOME%
ECHO LOG_HOME=%LOG_HOME%
ECHO DEBUG_PORT=%DEBUG_PORT%

IF "%1%"=="c" SET CLEAN_CMD=clean
ECHO CLEAN=%CLEAN_CMD%
IF "%1%"=="h" GOTO HELP

:MAIN
REM 打包
CD %APP_HOME% && CALL "%M2_HOME%\bin\mvn" -q -Dmaven.test.skip=true %CLEAN_CMD% package

REM 查找JAR_FILE
IF EXIST %JAR_HOME% CD %JAR_HOME% && FOR %%i in (*.jar) DO SET JAR_FILE=%%i
IF "%JAR_FILE%"=="" echo ****** COMPILE ERROR ****** && GOTO END
ECHO JAR_FILE=%JAR_FILE%

REM 设置CLASSPATH
SET CLASSPATH=%CFG_HOME%;%JAR_HOME%\%JAR_FILE%
ECHO CLASSPATH=%CLASSPATH%

REM 执行程序
CALL "%JAVA_HOME%\bin\java" %JAVA_OPTS% %JAVA_DEBUG% -cp %CLASSPATH% org.springframework.boot.loader.JarLauncher

GOTO END

:HELP
ECHO=
ECHO Usage:
ECHO     start.cmd      compile and start
ECHO     start.cmd c    clean, compile and start
ECHO     start.cmd h    show help
GOTO END

:END
ENDLOCAL
```

## 3. SpringBoot 自动配置原理

SpringBoot 自动配置实际上是一种 注解 + SPI 机制，本文以 Spring Boot 2.2.0.RELEASE 为例，结合一个 Starter 示例来说明自动配置原理。

1. 一个典型的 SpringBoot 程序都会使用 @SpringBootApplication；

2. @SpringBootApplication 被 @EnableAutoConfiguration 注解；

3. @EnableAutoConfiguration 被 @Import(AutoConfigurationImportSelector.class) 注解

4. AutoConfigurationImportSelector 是一个 ImportSelector，Import 注解会调用其 selectImports 方法，并将返回的字符串作为类名创建 Bean；

5. selectImports -> getAutoConfigurationEntry -> getCandidateConfigurations -> SpringFactoriesLoader.loadFactoryNames

6. SpringFactoriesLoader.loadFactoryNames 的功能主要是从 "META-INF/spring.factories" 文件中读取第一个参数指定的配置， EnableAutoConfiguration.class;

至此，Spring Boot 自动配置的原理已经清楚了。
