
有米Android积分墙开发者文档
===========================

一、基本配置 
--------------

1.导入SDK
~~~~~~~~~~~~~~~~~~~~~~~~
将sdk解压后的 **libs** 目录下的jar文件导入到工程指定的libs目录。 


2.权限配置
~~~~~~~~~~~~~~~~~~~~~~~~

**请将下面权限配置代码复制到 AndroidManifest.xml 文件中 :**::
	 

    <uses-permission android:name="android.permission.INTERNET"/> 
    <uses-permission android:name="android.permission.READ_PHONE_STATE"/>
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" /> 
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE"/>
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/> 
    <uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW"/>
    <uses-permission android:name="android.permission.GET_TASKS"/>
    <!--以下为可选权限-->
    <uses-permission android:name="com.android.launcher.permission.INSTALL_SHORTCUT"/>

3.广告组件配置
~~~~~~~~~~~~~~~~~~~~~~~~

**请将以下配置代码复制到 AndroidManifest.xml 文件中:**::

	<activity
		android:name="net.youmi.android.Ym_Class_AdBrowser"
		android:configChanges="keyboard|keyboardHidden|orientation"            
		android:theme="@android:style/Theme.Light.NoTitleBar" >
	</activity>
	<service
		android:name="net.youmi.android.Ym_Class_AdService"
		android:exported="false" >
	</service>
	<receiver 
		android:name="net.youmi.android.Ym_Class_AdReceiver" >
		<intent-filter>
			<action android:name="android.intent.action.PACKAGE_ADDED" />
			<data android:scheme="package" />
		</intent-filter>
	</receiver> 
	<receiver
		android:name="net.youmi.android.offers.Ym_Class_OffersReceiver"
		android:exported="false" >
	</receiver>

4.混淆配置
~~~~~~~~~~~~~~~~~~~~~~~~
**如果您的项目使用了Proguard混淆打包，为了避免SDK被二次混淆导致无法正常获取广告，请务必在proguard-project.txt中添加以下代码:**::

    -dontwarn net.youmi.android.**
	-keep class net.youmi.android.** {
	*;  
	}  

二、积分墙代码集成
-------------------

1.初始化
~~~~~~~~~~~~~~~~~~~~~~~~
请务必在应用第一个Activity(启动的第一个类)的onCreate中调用以下代码::

	net.youmi.android.Ym_Class_AdManager.getInstance(this).init("AppId","AppSecret", false); 
	net.youmi.android.offers.Ym_Class_OffersManager.getInstance(this).ym_method_onAppLaunch(); 

其中，AppId和AppSecret分别为应用的发布ID和密钥，这两个值通过有米后台自动生成，通过在有米后台-`应用详细信息 <http://www.youmi.net/apps/view>`_  可以获得。


2.积分管理接口
~~~~~~~~~~~~~~~~~~~~~~~~
2.1 查询积分余额
*************************

调用以下接口，查询用户的积分账户余额: ::

	int myPointBalance = net.youmi.android.offers.Ym_Class_PointsManager.getInstance(this).ym_method_queryPoints();
	
注意，该接口直接返回int型的积分余额。
	

2.2 扣除积分
*************************

调用以下接口，扣除用户积分账户余额: ::
    
	int amount=100;//示例扣除100积分。
	boolean isSuccess = net.youmi.android.offers.Ym_Class_PointsManager.getInstance(this).ym_method_spendPoints(amount);
	
注意，该接口直接返回扣除积分结果，成功扣除返回true，否则返回false。


2.3 增加积分
*************************

调用以下接口，往用户积分账户余额增加积分: ::

	int amount=100;//示例增加100积分。
	boolean isSuccess = net.youmi.android.offers.Ym_Class_PointsManager.getInstance(this).ym_method_awardPoints(amount);
	
注意，该接口直接返回增加积分结果，成功返回true，否则返回false。

3.展示全屏积分墙
~~~~~~~~~~~~~~~~~~~~~~~~
在UI线程中调用以下代码展示全屏积分墙: ::

    net.youmi.android.offers.Ym_Class_OffersManager.getInstance(this).ym_method_showOffersWall();

4.展示悬浮半屏积分墙
~~~~~~~~~~~~~~~~~~~~~~~~
在UI线程中调用以下代码展示悬浮半屏积分墙: ::

	net.youmi.android.offers.OffersManager.getInstance(this).ym_method_showOffersWallDialog(this);	

三、积分墙高级功能
------------------
积分墙SDK提供了积分余额变动通知、订单到账通知等高级功能，更多详情请参考 `积分墙高级功能 <offers_opt.html>`_ 。
	
四、SDK实用工具
---------------
SDK实用功能提供了检查更新和在线配置等功能，可以为您提供便捷的实用工具。`更多详情 <functional.html>`_
 
