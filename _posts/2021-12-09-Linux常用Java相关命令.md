---
layout: post
title:  "Linux常用Java相关命令"
date:   2021-12-09 18:14:54
categories: Linux
tags: Linux
excerpt: Linux常用Java相关命令
mathjax: true


---

* content
{:toc}


## 命令：ps -ef|grep java

这个命令可以查看进行，其中java是类型，显示启动行，进行过多不太方便

```
[root@tools ~]# ps -ef|grep java
root        2566       1  0 12月09 ?      00:04:25 /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.312.b07-1.el8_4.x
86_64/jre/bin/java -server -Dinstall4j.jvmDir=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.312.b07-1.el8_4.x86_64/jre -Dexe4j.moduleName=/home/nexus-3.37.0-01/bin/nexus -XX:+UnlockDiagnosticVMOptions -Dinstall4j.launcherId=245 -Dinstall4j.swt=false -Di4jv=0 -Di4jv=0 -Di4jv=0 -Di4jv=0 -Di4jv=0 -Xms2703m -Xmx2703m -XX:MaxDirectMemorySize=2703m -XX:+UnlockDiagnosticVMOptions -XX:+LogVMOutput -XX:LogFile=../sonatype-work/nexus3/log/jvm.log -XX:-OmitStackTraceInFastThrow -Djava.net.preferIPv4Stack=true -Dkaraf.home=. -Dkaraf.base=. -Dkaraf.etc=etc/karaf -Djava.util.logging.config.file=etc/karaf/java.util.logging.properties -Dkaraf.data=../sonatype-work/nexus3 -Dkaraf.log=../sonatype-work/nexus3/log -Djava.io.tmpdir=../sonatype-work/nexus3/tmp -Dkaraf.startLocalConsole=false -Djdk.tls.ephemeralDHKeySize=2048 -Djava.endorsed.dirs=lib/endorsed -Di4j.vpt=true -classpath /home/nexus-3.37.0-01/.install4j/i4jruntime.jar:/home/nexus-3.37.0-01/lib/boot/nexus-main.jar:/home/nexus-3.37.0-01/lib/boot/activation-1.1.1.jar:/home/nexus-3.37.0-01/lib/boot/jakarta.xml.bind-api-2.3.3.jar:/home/nexus-3.37.0-01/lib/boot/jaxb-runtime-2.3.3.jar:/home/nexus-3.37.0-01/lib/boot/txw2-2.3.3.jar:/home/nexus-3.37.0-01/lib/boot/istack-commons-runtime-3.0.10.jar:/home/nexus-3.37.0-01/lib/boot/org.apache.karaf.main-4.3.2.jar:/home/nexus-3.37.0-01/lib/boot/osgi.core-7.0.0.jar:/home/nexus-3.37.0-01/lib/boot/org.apache.karaf.specs.activator-4.3.2.jar:/home/nexus-3.37.0-01/lib/boot/org.apache.karaf.diagnostic.boot-4.3.2.jar:/home/nexus-3.37.0-01/lib/boot/org.apache.karaf.jaas.boot-4.3.2.jar com.install4j.runtime.launcher.UnixLauncher start 9d17dc87 0 0 org.sonatype.nexus.karaf.NexusMainroot       18664   18616  0 08:40 pts/0    00:00:00 grep --color=auto java
```

## 命令：pgrep java | xargs pwdx

```
[root@tools ~]# pgrep java | xargs pwdx
2566: /home/nexus-3.37.0-01
```

