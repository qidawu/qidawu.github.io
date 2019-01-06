---
title: Tomcat 容器如何查找类？
date: 2019-01-06 01:11:40
updated:
tags: Java
---

# catalina.sh

catalina.sh 中有一段 CLASSPATH 的代码：

```bash
# Ensure that any user defined CLASSPATH variables are not used on startup,
# but allow them to be specified in setenv.sh, in rare case when it is needed.
CLASSPATH=

if [ -r "$CATALINA_BASE/bin/setenv.sh" ]; then
  . "$CATALINA_BASE/bin/setenv.sh"
elif [ -r "$CATALINA_HOME/bin/setenv.sh" ]; then
  . "$CATALINA_HOME/bin/setenv.sh"
fi
```



```bash
# Get standard Java environment variables
if $os400; then
  # -r will Only work on the os400 if the files are:
  # 1. owned by the user
  # 2. owned by the PRIMARY group of the user
  # this will not work if the user belongs in secondary groups
  . "$CATALINA_HOME"/bin/setclasspath.sh
else
  if [ -r "$CATALINA_HOME"/bin/setclasspath.sh ]; then
    . "$CATALINA_HOME"/bin/setclasspath.sh
  else
    echo "Cannot find $CATALINA_HOME/bin/setclasspath.sh"
    echo "This file is needed to run this program"
    exit 1
  fi
fi
```



```bash
# Add on extra jar files to CLASSPATH
if [ ! -z "$CLASSPATH" ] ; then
  CLASSPATH="$CLASSPATH":
fi
CLASSPATH="$CLASSPATH""$CATALINA_HOME"/bin/bootstrap.jar

# Add tomcat-juli.jar to classpath
# tomcat-juli.jar can be over-ridden per instance
if [ -r "$CATALINA_BASE/bin/tomcat-juli.jar" ] ; then
  CLASSPATH=$CLASSPATH:$CATALINA_BASE/bin/tomcat-juli.jar
else
  CLASSPATH=$CLASSPATH:$CATALINA_HOME/bin/tomcat-juli.jar
fi
```

`start` command：

```bash
eval $_NOHUP "\"$_RUNJAVA\"" "\"$LOGGING_CONFIG\"" $LOGGING_MANAGER $JAVA_OPTS $CATALINA_OPTS \
  -D$ENDORSED_PROP="\"$JAVA_ENDORSED_DIRS\"" \
  -classpath "\"$CLASSPATH\"" \
  -Dcatalina.base="\"$CATALINA_BASE\"" \
  -Dcatalina.home="\"$CATALINA_HOME\"" \
  -Djava.io.tmpdir="\"$CATALINA_TMPDIR\"" \
  org.apache.catalina.startup.Bootstrap "$@" start \
  >> "$CATALINA_OUT" 2>&1 "&"
```

`stop` command：

```bash
eval "\"$_RUNJAVA\"" $LOGGING_MANAGER $JAVA_OPTS \
  -D$ENDORSED_PROP="\"$JAVA_ENDORSED_DIRS\"" \
  -classpath "\"$CLASSPATH\"" \
  -Dcatalina.base="\"$CATALINA_BASE\"" \
  -Dcatalina.home="\"$CATALINA_HOME\"" \
  -Djava.io.tmpdir="\"$CATALINA_TMPDIR\"" \
  org.apache.catalina.startup.Bootstrap "$@" stop
```

# catalina.properties

Common class loader 加载位于 `$CATALINA_HOME/lib` 目录下的所有类和 jar 文件，这些资源对所有应用程序和 Tomcat 可见。



最佳实践

1、避免在 Tomcat 里使用 Commons loader 加载不属于Tomcat标准发布的库和包，这可能会引起兼容错误。如果你需要在多个应用程序间共享一个库或包，创建 `shared/lib` 和 `shared/classes` 目录，然后配置到 `catalina.properties` 文件的 Shared loader。

2、以上规则的一个例外就是常用的第三方共享库，例如 JDBC driver。这些应该放到 `$CATALINA_HOME/lib` 目录下。

3、可能的话，尽可能按照Tomcat的开发者推荐的方式使用 Tomcat，如果你发现你不得不频繁配置 classpath，你可能需要重新考虑你的开发过程了。

# 启动 Tomcat

`sh bin/startup.sh` 或 `sh catalina.sh start` 启动 Tomcat，日志输出如下：

```bash
Using CATALINA_BASE:   /e/Developer/Apache/apache-tomcat-8.5.37
Using CATALINA_HOME:   /e/Developer/Apache/apache-tomcat-8.5.37
Using CATALINA_TMPDIR: /e/Developer/Apache/apache-tomcat-8.5.37/temp
Using JRE_HOME:        E:\Developer\Java\jdk1.8.0_191
Using CLASSPATH:       /e/Developer/Apache/apache-tomcat-8.5.37/bin/bootstrap.jar:/e/Developer/Apache/apache-tomcat-8.5.37/bin/tomcat-juli.jar
Tomcat started.
```

从启动日志可见，CLASSPATH 变量设置为两个 jar 包：`bin/bootstrap.jar` 和 `bin/tomcat-juli.jar`。

当执行启动脚本时，分析 `catalina.sh` 脚本源码可见，简化来说就是执行了命令 `java -classpath bootstrap.jar:tomcat-juli.jar org.apache.catalina.startup.Bootstrap start `，也就是运行了 `bootstrap.jar` 中的类文件 `org/apache/catalina/startup/Bootstrap.class`，为 `main` 方法传入命令行参数  `start`。

`META-INF/MANIFEST.MF`：

```
Manifest-Version: 1.0
Ant-Version: Apache Ant 1.9.9
Created-By: 1.7.0_80-b15 (Oracle Corporation)
Main-Class: org.apache.catalina.startup.Bootstrap
Specification-Title: Apache Tomcat Bootstrap
Specification-Version: 8.5
Specification-Vendor: Apache Software Foundation
Implementation-Title: Apache Tomcat Bootstrap
Implementation-Version: 8.5.37
Implementation-Vendor: Apache Software Foundation
X-Compile-Source-JDK: 1.7
X-Compile-Target-JDK: 1.7
Class-Path: commons-daemon.jar
```



# 参考

http://tomcat.apache.org/whichversion.html

http://tomcat.apache.org/tomcat-8.5-doc/class-loader-howto.html

https://www.mulesoft.com/tcat/tomcat-classpath

https://blog.csdn.net/u010858001/article/details/52399506