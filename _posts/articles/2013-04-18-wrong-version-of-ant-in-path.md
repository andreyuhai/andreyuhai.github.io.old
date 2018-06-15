---
layout: post
title: "Wrong version of Ant in path"
excerpt: "Recently, working on a Windows box, I had a problem where my ANT_HOME and ant -version result did not match."
modified: 2013-04-18 18:26:17 +0300
categories: articles
tags: [ant, path, windows, java, weblogic]
image:
  feature:
  credit:
  creditlink:
comments: true
share: true
aging: true
---

Recently, working on a Windows box, I had a problem where my `ANT_HOME` and `ant -version` result did not match.
Problem resided in the order of path variables. `ANT_HOME` was declared last but when the system looks for a command in the path, it uses the first result it finds.

For example, the following was in my path:

* C:\Windows\system32;
* C:\Windows;
* C:\Windows\System32\Wbem;
* C:\Windows\System32\WindowsPowerShell\v1.0\;
* C:\bea\weblogic92\server\bin;
* C:\Program Files (x86)\Subversion\bin;
* C:\oracle\instantclient_11_2;
* C:\Program Files\TortoiseHg\;
* C:\apache-ant-1.9.0\bin;
* C:\Program Files (x86)\Git\bin;
* C:\apache-maven-3.0.5\bin

As it can be seen, `apache-ant-1.9.0` is almost the last one in the list. Little did I know that Weblogic folder contanied an older version of `ant` as well. I reordered my path, and everything worked like a charm afterwards.

The same thing can happen with Java. You have your `JAVA_HOME` set, but for some reason `java -version` returns a different version number. Check `windows\system32\` folder, because `java.exe` might exist there as well. Add your Java location before `windows\` path declarations.
