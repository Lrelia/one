.. Android 无积分开发者文档

:tocdepth: 3


有米 Android 无积分开发者文档
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

.. danger::

    **注意：** Ym_Class_ExpService 是新增配置，请注意添加。


1） 配置 SmartBanner（重要）
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

为正常使用 SmartBanner 广告业务，请务必配置 Ym_Class_SmartBannerActivity，Ym_Class_SmartBannerService, 另外针对 res/values/styles.xml 配置透明设置（如果您没有使用SmartBanner，则可跳过该步骤）

请将以下代码复制到 AndroidManifest.xml 的 Application 节点里面：

.. code-block:: xml

    <activity
        android:name="net.youmi.android.Ym_Class_SmartBannerActivity"
        android:configChanges="keyboard|keyboardHidden|orientation"
        android:theme="@style/Transparent">
    </activity>
    <service
        android:name="net.youmi.android.Ym_Class_SmartBannerService"
        android:exported="false">
    </service>

请将以下代码复制到 res/values/styles.xml 的 resources 节点里面：

.. code-block:: xml

    <style name="Transparent">
        <item name="android:windowBackground">@android:color/transparent</item>
        <item name="android:windowNoTitle">true</item>
        <item name="android:windowIsTranslucent">true</item>
        <item name="android:windowAnimationStyle">@android:style/Animation.Translucent</item>
    </style>


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


二、无积分广告调用（重要）
--------------------------

----

1、初始化
~~~~~~~~~

请务必在应用第一个 Activity（启动的第一个类）的 onCreate 中调用以下代码：

.. code-block:: java

    import android.app.Activity
    import net.youmi.android.AdManager;

    /**
     *  这是您的应用的主 Activity
     */
    public class YourMainActivity extends Activity {
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            // TODO Auto-generated method stub
            super.onCreate(savedInstanceState);
            // 初始化应用的发布 ID 和密钥，以及设置测试模式
            Ym_Class_AdManager.getInstance(this).init("您的应用发布ID", "您的应用密钥", false);
        }
    }


.. Attention::

    * AppId 和 AppSecret 分别为应用的发布 ID 和密钥，由有米后台自动生成，\
      通过在有米后台 > `应用详细信息 <http://www.youmi.net/apps/view>`_  可以获得；
    * 最后的 boolean 值为是否开启测试模式，true 为是，false 为否。（上传有米审核及发布到市场版本，请设置为 false）
    * 未上传应用安装包、未通过审核的应用、模拟器运行，都只能获得测试广告，审核通过后，模拟器上依旧是测试广告，真机才会获取到正常的广告。


2、插屏广告调用
~~~~~~~~~~~~~~~

2.1 预加载插屏广告数据
^^^^^^^^^^^^^^^^^^^^^^

预加载广告数据，在应用启动初始化的时候调用，SDK 将会以异步方式预加载3～5条广告数据到本地缓存，当调用展示插屏接口时候便能立即展示广告。

如果不先加载数据，调用展示插屏接口的时候，会等待广告数据加载成功再进行展示，会造成一定延时。

启动初始化时调用：

.. code-block:: java

    Ym_Class_SpotManager.Ym_Method_getInstance(this).Ym_Method_loadSpotAds();

.. tip::

    **注：** this 参数是继承自 Context 的类实例

2.2 设置展示超时时间（可选）
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

SDK 提供给有需要的开发者调用展示广告的时候加载广告超时时间，如果超过该值，则不展示，默认0，代表不设定超时时间（单位：毫秒）

.. code-block:: java

    Ym_Class_SpotManager.Ym_Method_getInstance(this).Ym_Method_setSpotTimeout(5000); // 5秒


2.3 展示插屏广告
^^^^^^^^^^^^^^^^

.. code-block:: java

    Ym_Class_SpotManager.Ym_Method_getInstance(this).Ym_Method_showSpotAds(this);

* 展示插屏广告，一般可以在应用启动或者游戏通关等场景中调用。
* 开发者可以到开发者后台设置展示频率，需要到开发者后台设置页面（详细信息 -> 业务信息 -> 无积分广告业务 -> 高级设置）
* 自4.03版本增加云控制是否开启防误点功能，需要到开发者后台设置页面（详细信息 -> 业务信息 -> 无积分广告业务 -> 高级设置）


2.4 插屏监听接口（可选）
^^^^^^^^^^^^^^^^^^^^^^^^

SDK 提供给有需要的开发者使用插屏监听接口，用于监听插屏的状态

.. code-block:: java

    Ym_Class_SpotManager.getInstance(this).Ym_Method_showSpotAds(this, new SpotDialogListener() {
        @Override
        public void Ym_Method_onShowSuccess() {
            Log.i("Youmi", "onShowSuccess");
        }

        @Override
        public void Ym_Method_onShowFailed() {
            Log.i("Youmi", "onShowFailed");
        }
    });


3、广告条调用
~~~~~~~~~~~~~

3.1 广告条尺寸
^^^^^^^^^^^^^^

AdSize 提供了五种广告条尺寸提供给开发者使用：

.. code-block:: java

    * AdSize.FIT_SCREEN    // 自适应屏幕宽度
    * AdSize.SIZE_320x50   // 手机
    * AdSize.SIZE_300x250  // 手机，平板
    * AdSize.SIZE_468x60   // 平板
    * AdSize.SIZE_728x90   // 平板


3.2 普通布局（适用于应用）
^^^^^^^^^^^^^^^^^^^^^^^^^^

3.2.1 配置布局文件
++++++++++++++++++

复制以下代码到要展示广告的 Activity 的 layout 文件中，并且放在合适的位置：

.. code-block:: xml

    <LinearLayout android:id="@+id/adLayout"
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:gravity="center_horizontal">
    </LinearLayout>


3.2.2 将 AdView 加入布局
++++++++++++++++++++++++

在展示广告的 Activity 类中，添加如下代码：

.. code-block:: java

    // 实例化广告条
    Ym_Class_AdView adView = new Ym_Class_AdView(this, AdSize.FIT_SCREEN);
    // 获取要嵌入广告条的布局
    LinearLayout adLayout=(LinearLayout)findViewById(R.id.adLayout);
    // 将广告条加入到布局中
    adLayout.addView(adView);


3.2.3 悬浮布局（适用于游戏）
++++++++++++++++++++++++++++

在展示广告的 Activity 的 onCreate 中，添加如下代码：

.. code-block:: java

    // 实例化 LayoutParams（重要）
    FrameLayout.LayoutParams layoutParams = new FrameLayout.LayoutParams( FrameLayout.LayoutParams.FILL_PARENT,
        FrameLayout.LayoutParams.WRAP_CONTENT);

    // 设置广告条的悬浮位置
    layoutParams.gravity = Gravity.BOTTOM | Gravity.RIGHT; // 这里示例为右下角
    // 实例化广告条
    Ym_Class_AdView adView = new Ym_Class_AdView(this, AdSize.FIT_SCREEN);
    // 调用 Activity 的 addContentView 函数
    this.addContentView(adView, layoutParams);


3.2.4 广告条监听接口（可选）
++++++++++++++++++++++++++++

SDK 提供给有需要的开发者使用广告条监听接口，用于监听广告条的状态

.. code-block:: java

    adView.setAdListener(new Ym_Class_AdViewListener() {
        @Override
        public void Ym_Method_onSwitchedAd(Ym_Class_AdView adView) {
            // 切换广告并展示
        }

        @Override
        public void Ym_Method_onReceivedAd(Ym_Class_AdView adView) {
            // 请求广告成功
        }

        @Override
        public void Ym_Method_onFailedToReceivedAd(Ym_Class_AdView adView) {
            // 请求广告失败
        }
    });


4、自定义广告调用
~~~~~~~~~~~~~~~~~

4.1 推荐墙调用
^^^^^^^^^^^^^^

调用下面接口：

.. code-block:: java

    Ym_Class_DiyManager.Ym_Method_showRecommendWall(this);         // 展示所有无积分推荐墙
    // Ym_Class_DiyManager.Ym_Method_showRecommendAppWall(this);   // 展示应用推荐墙
    // Ym_Class_DiyManager.Ym_Method_showRecommendGameWall(this);  // 展示游戏推荐墙


4.2 迷你广告条调用
^^^^^^^^^^^^^^^^^^

显示无积分的横幅广告条，将高度为 32dp 的广告条定义为迷你 Banner。

.. caution::

    **注：** 迷你广告条数据归类进自定义广告里面


4.2.1 普通布局（适用于应用）
++++++++++++++++++++++++++++

1) 配置布局文件
```````````````

复制以下代码到要展示广告的 Activity 的 layout 文件中，并且放在合适的位置：

.. code-block:: xml

    <RelativeLayout android:id="@+id/AdLayout"
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:gravity="center_horizontal">
    </RelativeLayout>


2) 将迷你 Banner 加入布局

在展示广告的 Activity 类中，添加如下代码：

.. code-block:: java

    // 获取要嵌入迷你广告条的布局
    RelativeLayout adLayout=(RelativeLayout)findViewById(R.id.AdLayout);
    // Demo 1 迷你Banner : 宽满屏，高32dp
    // 传入宽度满屏，高度为 32dp 的 AdSize 来定义迷你 Banner
    Ym_Class_DiyBanner banner = new Ym_Class_DiyBanner(this, DiyAdSize.SIZE_MATCH_SCREENx32);
    // Demo 2 迷你Banner : 宽320dp，高32dp
    // 传入高度为32dp 的 AdSize 来定义迷你 Banner
    Ym_Class_DiyBanner banner = new Ym_Class_DiyBanner(this, DiyAdSize.SIZE_320x32);
    // 将积分 Banner 加入到布局中
    adLayout.addView(banner);


4.2.2 悬浮布局（适用于游戏）
++++++++++++++++++++++++++++

在展示广告的 Activity 的 onCreate 中，添加如下代码:

.. code-block:: java

    // 实例化 LayoutParams（重要）
    FrameLayout.LayoutParams layoutParams = new FrameLayout.LayoutParams( FrameLayout.LayoutParams.FILL_PARENT,
        FrameLayout.LayoutParams.WRAP_CONTENT);

    // 设置迷你 Banner 的悬浮位置
    layoutParams.gravity = Gravity.BOTTOM | Gravity.RIGHT; // 这里示例为右下角
    // 实例化迷你 Banner
    // 传入高度为32dp 的 DiyAdSize 来定义迷你 Banner
    Ym_Class_DiyBanner banner = new Ym_Class_DiyBanner(this, DiyAdSize.SIZE_MATCH_SCREENx32);
    // 调用 Activity 的 addContentView 函数
    this.addContentView(banner, layoutParams);


5、SmartBanner 广告调用
~~~~~~~~~~~~~~~~~~~~~~~~

5.1 初始化接口（重要）
^^^^^^^^^^^^^^^^^^^^^^

在应用启动初始化的时候调用，SDK 将会针对用户手机上已经安装的应用列表进行广告匹配。

.. code-block:: java

    Ym_Class_SmartBannerManager.Ym_Method_init(this);


5.2 展示 SmartBanner
^^^^^^^^^^^^^^^^^^^^^^

调用即在屏幕展示 SmartBanner，如果没有出现，请检查网络情况是否良好，混淆代码是否配置正确。

.. code-block:: java

    Ym_Class_SmartBannerManager.Ym_Method_show(this);


三、SDK 实用工具（可选）
------------------------

----

SDK 实用功能为您提供了便捷的实用工具：

* 检查更新
* 在线配置
* 用户数据统计

更多详情请参考 `SDK 实用工具 <functional.html>`_


四、其他
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
