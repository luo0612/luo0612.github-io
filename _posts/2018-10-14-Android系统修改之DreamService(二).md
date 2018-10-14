---
title: 2018-10-08-Android系统修改之DreamService(二)
author: luo0612
layout: post
---

**源码基于Android4.4**  

最近要实现一个时间屏保功能, 在话机息屏之后, 需要在屏保上显示时间和日期.  
查找一番, 网上说的最多的就是基于 **DreamService** 实现, 后面发觉Android已经实现了该功能.    
通过 **设置->显示->互动屏保->时钟** 可以开启该时间屏保功能, 但是该互动屏保有些限制就是只能在 **插入基座**或**充电** 时, 才会启动.  
后续经过简单修改, 实现正常情况下, 息屏之后显示该时间屏保.

### 开启互动屏保: 

**设置->显示->互动屏保:**

![](https://i.imgur.com/KZXufBm.png)  

![](https://i.imgur.com/xk6iwJL.png)  

![](https://i.imgur.com/yzXJHIX.png)  

![](https://i.imgur.com/OxLB5ZS.png)

### 相关源码位置

**设置应用:**  

	packages/apps/Settings

**设置->显示选项:**

	<!-- 设置界面 -->
	packages/apps/Settings/src/com.android.settings/Settings.java

	<!-- 显示选项 -->
	packages/apps/Settings/res/xml/settings_headers.xml
	<!-- 通过以选项进入显示界面 -->
    <!-- DisplaySettings: 显示界面 -->
    <header
        android:id="@+id/display_settings"
        android:icon="@drawable/ic_settings_display"
        android:fragment="com.android.settings.DisplaySettings"
        android:title="@string/display_settings" />

**设置->显示->互动屏保选项:**  

	<!-- 显示界面 -->
	packages/apps/Settings/src/com.android.settings/DisplaySettings.java

	<!-- 互动屏保选项 -->
	packages/apps/Settings/res/xml/display_settings.xml
	<!-- 通过以下选项进入互动屏保界面 -->
	<!-- DreamSettings: 互动屏保界面 -->
	<PreferenceScreen
    	android:key="screensaver"
    	android:title="@string/screensaver_settings_title"
    	android:fragment="com.android.settings.DreamSettings" />	
	
**设置->显示->互动屏保->时钟选项:**  

	<!-- 互动屏保界面 -->
	packages/apps/Settings/src/com.android.settings.DreamSettings.java

### 源码分析

**1. 开启/关闭互动屏保**  

![](https://i.imgur.com/MciqJnJ.png)  

	源码位置:
	packages/apps/Settings/src/com.android.settings.DreamSettings.java
	 
互动屏保 **打开/关闭** 的开关是通过代码动态添加的, 在 **DreamSettings.onCreate()** 方法:  

	// 创建Switch控件
	mSwitch = new Switch(activity);
	...省略代码...
	final int padding = activity.getResources().getDimensionPixelSize(R.dimen.action_bar_switch_padding);
	//设置 PaddingEnd
	mSwitch.setPaddingRelative(0, 0, padding, 0);
	//自定义设置ActionBar的Options选项
	activity.getActionBar().setDisplayOptions(ActionBar.DISPLAY_SHOW_CUSTOM,ActionBar.DISPLAY_SHOW_CUSTOM);
	//设置ActionBar的Options选项自定义布局, 该处即Switch控件
	activity.getActionBar().setCustomView(mSwitch, new ActionBar.LayoutParams(
                ActionBar.LayoutParams.WRAP_CONTENT,
                ActionBar.LayoutParams.WRAP_CONTENT,
                Gravity.CENTER_VERTICAL | Gravity.END));



**2. 互动屏保列表**

![](https://i.imgur.com/cQZiGPx.png)  

	源码位置:
	packages/apps/Settings/src/com.android.settings.DreamSettings.java
	packages/apps/Settings/src/com.android.settings.DreamBackend.java

互动屏保的列表是通过 **DreamBackend.getDreamInfos()** 方法获取到所有注册的互动屏保

	//获取已经开启的互动屏保
	ComponentName activeDream = getActiveDream();
	PackageManager pm = mContext.getPackageManager();
	//查询所有继承DreamService类, 实现自定义互动屏保的类的相关信息
	Intent dreamIntent = new Intent(DreamService.SERVICE_INTERFACE);
	List<ResolveInfo> resolveInfos = pm.queryIntentServices(dreamIntent,PackageManager.GET_META_DATA);
	List<DreamInfo> dreamInfos = new ArrayList<DreamInfo>(resolveInfos.size());
	for (ResolveInfo resolveInfo : resolveInfos) {
		if (resolveInfo.serviceInfo == null)
			continue;
		DreamInfo dreamInfo = new DreamInfo();
		dreamInfo.caption = resolveInfo.loadLabel(pm);
		dreamInfo.icon = resolveInfo.loadIcon(pm);
		dreamInfo.componentName = getDreamComponentName(resolveInfo);
		dreamInfo.isActive = dreamInfo.componentName.equals(activeDream);
		//该互动屏保的设置界面的相关信息
		dreamInfo.settingsComponentName = getSettingsComponentName(pm, resolveInfo);
		dreamInfos.add(dreamInfo);
	}
	Collections.sort(dreamInfos, mComparator);



