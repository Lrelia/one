.. Android 积分墙开发者文档

:tocdepth: 3


有米 Android 积分墙开发者文档
=============================

一、基本配置（重要）
--------------------

----

1、导入 SDK
~~~~~~~~~~~

将 sdk 解压后的 **libs** 目录下的 jar 文件导入到工程指定的 libs 目录。


2、权限配置
~~~~~~~~~~~

请将下面权限配置代码复制到 ``AndroidManifest.xml`` 文件中 :

.. code-block:: xml

    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.READ_PHONE_STATE" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW" />
    <uses-permission android:name="android.permission.GET_TASKS" />
    <!-- 以下为可选权限 -->
    <uses-permission android:name="com.android.launcher.permission.INSTALL_SHORTCUT" />


3、广告组件配置
~~~~~~~~~~~~~~~

请将以下配置代码复制到 ``AndroidManifest.xml`` 文件中:

.. code-block:: xml

    <activity
        android:name="net.youmi.android.Ym_Class_AdBrowser"
        android:configChanges="keyboard|keyboardHidden|orientation|screenSize"
        android:theme="@android:style/Theme.Light.NoTitleBar" >
    </activity>
    <service
        android:name="net.youmi.android.Ym_Class_AdService"
        android:exported="false" >
    </service>
    <service
        android:name="net.youmi.android.Ym_Class_ExpService"
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

.. danger::

    **注意：** Ym_Class_ExpService 是新增配置，请注意添加。


4、混淆配置
~~~~~~~~~~~

如果您的项目使用了 Proguard 混淆打包，为了避免 SDK 被二次混淆导致无法正常获取广告，请务必在 ``proguard-project.txt`` 中添加以下代码：

.. code-block:: java

    -dontwarn net.youmi.android.**
    -keep class net.youmi.android.** {
        *;
    }


5、打包（重要）
~~~~~~~~~~~~~~~

.. caution::

    **注意：** 打包时在配置文件中指定 target-sdk 为17以上。


二、积分墙代码集成（重要）
--------------------------

----

1、初始化
~~~~~~~~~

请务必在应用第一个 Activity（启动的第一个类）的 onCreate 中调用以下代码

.. code-block:: java

    import net.youmi.android.Ym_Class_AdManager;
    import net.youmi.android.offers.Ym_Class_OffersManager;
    ...
    Ym_Class_AdManager.getInstance(Context context).init("AppId", "AppSecret", false);
    Ym_Class_OffersManager.getInstance(Context context).ym_method_onAppLaunch();

.. Attention::

    * AppId 和 AppSecret 分别为应用的发布 ID 和密钥，由有米后台自动生成，\
      通过在有米后台 > `应用详细信息 <http://www.youmi.net/apps/view>`_  可以获得；
    * 最后的 boolean 值为是否开启测试模式，true 为是，false 为否。（上传有米审核及发布到市场版本，请设置为 false）


2、积分管理接口
~~~~~~~~~~~~~~~

2.1 查询积分余额
^^^^^^^^^^^^^^^^

调用以下接口，查询用户的积分账户余额：

.. code-block:: java

    import net.youmi.android.offers.Ym_Class_PointsManager;
    ...
    int myPointBalance = Ym_Class_PointsManager.getInstance(this).ym_method_queryPoints();

.. tip::


    **注意：** 该接口直接返回 int 型的积分余额。


2.2 扣除积分
^^^^^^^^^^^^

调用以下接口，扣除用户积分账户余额：

.. code-block:: java

    import net.youmi.android.offers.Ym_Class_PointsManager;
    ...
    int amount = 100; // 示例扣除100积分。
    boolean isSuccess = Ym_Class_PointsManager.getInstance(this).ym_method_spendPoints(amount);

.. tip::

    **注意：** 该接口直接返回扣除积分结果，成功扣除返回 true，否则返回 false。


2.3 增加积分
^^^^^^^^^^^^

调用以下接口，往用户积分账户余额增加积分：

.. code-block:: java

    import net.youmi.android.offers.Ym_Class_PointsManager;
    ...
    int amount = 100; // 示例增加100积分
    boolean isSuccess = Ym_Class_PointsManager.getInstance(this).ym_method_awardPoints(amount);

.. tip::

    **注意：** 该接口直接返回增加积分结果，成功返回 true，否则返回 false。


3、展示全屏积分墙
~~~~~~~~~~~~~~~~~

在 UI 线程中调用以下代码展示全屏积分墙：

.. code-block:: java

    import net.youmi.android.offers.Ym_Class_OffersManager;
    ...
    Ym_Class_OffersManager.getInstance(this).ym_method_showOffersWall();


4、展示悬浮半屏积分墙
~~~~~~~~~~~~~~~~~~~~~

在 UI 线程中调用以下代码展示悬浮半屏积分墙：

.. code-block:: java

    import net.youmi.android.offers.Ym_Class_OffersManager.
    ...
    Ym_Class_OffersManager.getInstance(this).ym_method_showOffersWallDialog(this);


三、积分墙高级功能（可选）
--------------------------

----

积分墙 SDK 提供了如下高级功能：

* 积分余额变动通知
* 客户端 SDK 获取订单信息
* 服务器获取订单信息（开发者直接通过自己设置的服务器监听订单信息）
* 验证积分墙配置是否正确
* 关闭有米 Debug Log

更多详情请参考 `积分墙高级功能 <offers_opt.html>`_


四、SDK 实用工具（可选）
------------------------

----

SDK 实用功能为您提供了便捷的实用工具：

* 检查更新
* 在线配置
* 用户数据统计

更多详情请参考 `SDK 实用工具 <functional.html>`_


五、其他
--------

----

SDK 常见问题
~~~~~~~~~~~~

1、环境配置问题
^^^^^^^^^^^^^^^

1.1 有米广告 SDK 使用哪种字符编码
+++++++++++++++++++++++++++++++++

有米广告 SDK 使用 UTF-8 字符编码，在嵌入广告以及导入示例程序的时候请使用 UTF-8 编程环境，否则会出现乱码情况。


1.2 有米广告 SDK 兼容 Android 系统 SDK 的哪些版本
+++++++++++++++++++++++++++++++++++++++++++++++++

有米广告 Android SDK 兼容 Android 系统 2.1及以上版本 SDK，对于2.1以下版本可能会有兼容性问题。


2、如何关闭Debug log
^^^^^^^^^^^^^^^^^^^^

如果需要关闭有米广告 SDK 的 Debug log，请调用 Ym_Class_AdManager.ym_method_setEnableDebugLog(false) 来关闭 SDK 的 log 输出。

*代码示例：*

.. code-block:: java

    import net.youmi.android.Ym_Class_AdManager
    ...
    // 调用以下接口关闭有米广告 SDK 相关的 log
    Ym_Class_AdManager.getInstance(this).ym_method_setEnableDebugLog(false);
    ...

.. tip::

    **注意：** 上传到有米主站进行审核时务必开启 Debug log,这样才能保证通过审核。

3、关于测试模式
^^^^^^^^^^^^^^^

广告运行在非发布状态下的情况属于测试模式。

以下情况下属于测试模式：

1. 在初始化接口设置测试模式为 true
2. 应用未上传、待审核的情况下属于测试模式
3. 已上传并通过审核，但是后续版本应用 ID 和密钥与应用的包名不对应

该模式下可以获得更多的测试广告，已经安装过的广告卸载后可以重复安装，但只能结算积分，不结算收入。

正式发布前请务必将初始化接口的测试模式参数设置为 flase，并且上传应用到 `有米主站 <http://www.youmi.net/>`_ 进行审核。
