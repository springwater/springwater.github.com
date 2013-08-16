---
layout: post
title: "中国 Android Studio 更新"
tagline: "idea"
description: ""
category : tech
tags: [idea, android]
---
{% include JB/setup %}

##demo
###Android Studio 0.2.2 更新到 0.2.4

1、查看现版本build编号
图中编号已经是0.2.4版本
<img src="/assets/img/2013081601.png" alt="form">

2、在google网站查询最新build编号
<a href='https://dl.google.com/android/studio/patches/updates.xml' target='_blank'>https://dl.google.com/android/studio/patches/updates.xml</a>
<img src="/assets/img/2013081602.png" alt="form">
<pre class="prettyPrint">
https://dl.google.com/android/studio/patches/updates.xml
</pre>

3、修改下载地址
<pre class="prettyPrint">
0.2.2 --> AI-130.754168
0.2.4 --> 130.777423
http://dl.google.com/android/studio/patches/AI-130.754168-130.777423-patch-win.jar
</pre>

4、复制更新包到Android Studio目录
<img src="/assets/img/2013081603.png" alt="form">

6、执行更新
<pre class="prettyPrint">
D:\Soft\Android\android-studio>java -classpath AI-130.754168-130.777423-patch-win.jar com.intellij.updater.Runner install .
注意最后的.表示在当前目录安装升级包，否则就应输入Android Studio的安装路径。
</pre>