---
title: Android系统修改之Notification布局修改(一)
author: luo0612
layout: post
---

源码基于Android4.4

**相关布局文件的位置:**

	frameworks/base/core/res目录下:
		1. notification_template_base.xml
		2. notification_template_big_base.xml	
		3. notification_template_big_picture.xml
		4. notification_template_big_text.xml
		5. notification_template_inbox.xml


**相关类位置:**

	frameworks/base/core/java/android/app目录下:
		1. Notification.java


**重点**

	RemoteViews类
	Binder机制

**待分析...**

	