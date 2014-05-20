.. Android 积分墙开发者文档

:tocdepth: 3


Android 积分墙源数据开发者文档
==============================

一、基本配置（重要）
--------------------

----

1、导入 SDK
~~~~~~~~~~~

将 sdk 解压后的 **libs** 目录下的 jar 文件导入到工程指定的 libs 目录。


2、权限配置
~~~~~~~~~~~

请将下面权限配置代码复制到 ``AndroidManifest.xml`` 文件中：

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

请将以下配置代码复制到 ``AndroidManifest.xml`` 文件中：

.. code-block:: xml

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



二、初始化及相关工作（重要）
----------------------------

----

请务必在应用第一个 Activity（启动的第一个类）的 onCreate 中调用以下代码：

.. code-block:: java

    import net.youmi.android.Ym_Class_AdManager;
    import net.youmi.android.offers.diyoffer.Ym_Class_DiyOfferWallManager;
    ...
    Ym_Class_AdManager.getInstance(Context context).init("AppId", "AppSecret", false);
    Ym_Class_DiyOfferWallManager.getInstance(Context context).ym_method_onAppLaunch();

.. Attention::

    * AppId 和 AppSecret 分别为应用的发布 ID 和密钥，由有米后台自动生成，\
      通过在有米后台 > `应用详细信息 <http://www.youmi.net/apps/view>`_  可以获得；
    * 最后的 boolean 值为是否开启测试模式，true 为是，false 为否。（上传有米审核及发布到市场版本，请设置为 false）


三、获取广告列表（重要）
------------------------

----

3.1 数据模型
~~~~~~~~~~~~

3.1.1 单个广告摘要信息的数据模型
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Ym_Class_AppSummaryObject 中集成了一条广告的摘要信息，通过使用 Ym_Class_AppSummaryObject，您可以获取广告的摘要信息，然后以列表形式展示出来：

.. code-block:: java

    import net.youmi.android.offers.diyoffer.Ym_Class_AppSummaryObject;
    ...

    Ym_Class_AppSummaryObject appSummaryObject;

    int id            =  appSummaryObject.ym_method_getAdId();         // 获取广告 id
    String adName     =  appSummaryObject.ym_method_getAppName();      // 获取 app 的名称
    String pn         =  appSummaryObject.ym_method_getPackageName();  // 获取 app 的包名
    int versionCode   =  appSummaryObject.ym_method_getVersionCode();  // 获取 app 的版本号
    String adIconUrl  =  appSummaryObject.ym_method_getIconUrl();      // 获取 app 的广告图标地址
    String adtext     =  appSummaryObject.ym_method_getAdSlogan();     // 获取广告标语
    String adSize     =  appSummaryObject.ym_method_getAppSize();      // 获取 app 的大小
    int points        =  appSummaryObject.ym_method_getPoints();       // 获取广告的积分（已完成状态下的广告积分返回值为0）
    String pointsUnit =  appSummaryObject.ym_method_getPointsUnit();   // 获取广告的积分单位
    int actionType    =  appSummaryObject.getActionType();             // 获取广告的类型
    int adStatus      =  appSummaryObject.ym_method_getAdTaskStatus(); // 获取广告的完成状态
    int dlStatus      =  appSummaryObject.ym_method_getAdDownloadStatus();  // 获取广告的下载状态
    String steps      =  appSummaryObject.ym_method_getTaskSteps();    // 任务步骤流程指引

**注：**

1. 广告的完成状态有2种，对应的值分别为：

.. code-block:: java

    <已完成>：net.youmi.android.offers.diyoffer.Ym_Class_AdTaskStatus.ALREADY_COMPLETE;
    <未完成>：net.youmi.android.offers.diyoffer.Ym_Class_AdTaskStatus.NOT_COMPLETE;

.. Attention::

    其中只有 <未完成> 状态下的广告才可以获取积分；<已完成> 状态下的广告是不能获取积分的，同时，<已完成> 状态下方法 Ym_Class_AppSummaryObject.ym_method_getPoints() 的返回值也为0


2. 广告的下载状态有3种，对应的值分别为：

.. code-block:: java

    <未下载>：net.youmi.android.offers.diyoffer.Ym_Class_AdDownloadStatus.NOT_DOWNLOAD;
    <正在下载>：net.youmi.android.offers.diyoffer.Ym_Class_AdDownloadStatus.DOWNLOADING;
    <已经下载>：net.youmi.android.offers.diyoffer.Ym_Class_AdDownloadStatus.ALERADY_DOWNLOAN;


3. 广告的类型有2种，对应的值分别为：

.. code-block:: java

    <体验类型>：net.youmi.android.offers.diyoffer.Ym_Class_AdType.EXPERIENCE;
    <注册类型>：net.youmi.android.offers.diyoffer.Ym_Class_AdType.REGISTER;


3.1.2 广告列表数据模型
^^^^^^^^^^^^^^^^^^^^^^

| Ym_Class_AppSummaryObjectList 中包含了每个广告的摘要信息 Ym_Class_AppSummaryObject，每次请求广告的时候都会返回这个列表数据模型，我们为这个列表数据模型提供以下几个方法：

.. code-block:: java

    public class Ym_Class_AppSummaryObjectList {
        /**
         *  添加广告
         */
        public boolean add(Ym_Class_AppSummaryObject object);

        /**
         *  获取指定索引的广告的摘要信息
         */
        public Ym_Class_AppSummaryObject get(int index);

        /**
         *  判断广告列表是否为空
         */
        public boolean isEmpty();

        /**
         *  获取广告列表的长度
         */
        public int size();
    }


3.2 获取方式
~~~~~~~~~~~~

获取积分墙列表数据有两种方式，一种为 **同步加载** ，一种为 **异步加载** 。


1. 同步加载方式（请注意在非 UI 线程中使用）：
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: java

    /**
     *  获取积分墙列表数据
     *  @param  requestType    请求类型
     *      Ym_Class_DiyOfferWallManager.ym_param_REQUEST_ALL          : 所有（默认值）
     *      Ym_Class_DiyOfferWallManager.ym_param_REQUEST_GAME         : 只请求游戏广告
     *      Ym_Class_DiyOfferWallManager.ym_param_REQUEST_APP          : 只请求应用广告
     *      Ym_Class_DiyOfferWallManager.ym_param_REQUEST_SPECIAL_SORT : 请求列表特殊排序，应用先于游戏显示
     *  @param  withAdDownloadUrl    广告是否携带url下载地址（可用于实现广告列表页实现下载功能）
     *      false:  不携带（默认值）
     *      true:   携带
     *  @return  Ym_Class_AppSummaryObjectList   广告摘要信息列表
     */
    Ym_Class_DiyOfferWallManager.getInstance(Context context).ym_method_getOfferWallAdList(int requestType, boolean withAdDownloadUrl);

*示例代码* ：

.. code-block:: java

    import net.youmi.android.offers.diyoffer.Ym_Class_AppSummaryObjectList;
    import net.youmi.android.offers.diyoffer.Ym_Class_DiyOfferWallManager;
    ...

    // 请求广告类型不限，广告附带 url 下载地址
    new Thread(new Runnable() {
         @Override
         public void run() {
             Ym_Class_AppSummaryObjectList data =
                 Ym_Class_DiyOfferWallManager.getInstance(this).ym_method_getOfferWallAdList(Ym_Class_DiyOfferWallManager.ym_param_REQUEST_ALL, true);
         }
    }).start();


2. 异步加载方式：
^^^^^^^^^^^^^^^^^

.. code-block:: java

    /**
     *  异步加载积分墙数据列表
     *  @param  requestType    请求类型
     *      Ym_Class_DiyOfferWallManager.ym_param_REQUEST_ALL          : 所有（默认值）
     *      Ym_Class_DiyOfferWallManager.ym_param_REQUEST_GAME         : 只请求游戏广告
     *      Ym_Class_DiyOfferWallManager.ym_param_REQUEST_APP          : 只请求应用广告
     *      Ym_Class_DiyOfferWallManager.ym_param_REQUEST_SPECIAL_SORT : 请求列表特殊排序，应用先于游戏显示
     *  @param  withAdDownloadUrl    广告是否携带url下载地址（可用于实现广告列表页实现下载功能）
     *      false:  不携带（默认值）
     *      true:   携带
     */
    Ym_Class_DiyOfferWallManager.getInstance(Context context).ym_method_loadOfferWallAdList(int requestType, boolean withAdDownloadUrl,
        Ym_Class_AppSummaryDataInterface appSummaryDataInterface);

*示例代码* ：

.. code-block:: java

    import net.youmi.android.offers.diyoffer.Ym_Class_AppSummaryDataInterface;
    import net.youmi.android.offers.diyoffer.Ym_Class_AppSummaryObject;
    import net.youmi.android.offers.diyoffer.Ym_Class_AppSummaryObjectList;
    import net.youmi.android.offers.diyoffer.Ym_Class_DiyOfferWallManager;
    ...

    /**
     *  请求第一页广告，广告类型不限，广告不附带下载地址
     */
    Ym_Class_DiyOfferWallManager.getInstance(this).ym_method_loadOfferWallAdList(Ym_Class_DiyOfferWallManager.ym_param_REQUEST_ALL, false,
        new Ym_Class_AppSummaryDataInterface() {
            /**
             *  当成功获取积分墙列表数据的时候会回调这个方法
             *  注意：本回调方法不在 UI 线程中执行，所以请不要在本接口中进行UI线程方面的操作
             */
            @Override
            public void ym_method_onLoadAppSumDataSuccess(Context context, Ym_Class_AppSummaryObjectList adList) {
                // TODO Auto-generated method stub
                for (int i = 0; i < adList.size(); ++i) {
                    Log.d("test", adList.get(i).toString());
                }
            }

            /**
             *  当获取积分墙数据失败的时候会回调这个方法
             *  注意：本回调方法不在 UI 线程中执行，所以请不要在本接口中进行 UI 线程方面的操作）
             */
            @Override
            public void ym_method_onLoadAppSumDataFailed() {
                // TODO Auto-generated method stub
                Log.d("test", "没有获取到数据");
            }
        }
    );


四、获取广告的详细数据（重要）
------------------------------

----

4.1 数据模型
~~~~~~~~~~~~

Ym_Class_AppDetailObject 中集成了一条广告的详细信息，通过 Ym_Class_AppDetailObject，您可以获取广告的详细信息，然后展示广告的详情页

.. code-block:: java

    import net.youmi.android.offers.diyoffer.Ym_Class_AppDetailObject;
    ...

    Ym_Class_AppDetailObject appDetailObject;

    int id              =  appDetailObject.ym_method_getAdId();           // 获取广告 id
    String adName       =  appDetailObject.ym_method_getAppName();        // 获取 app 的名称
    String pn           =  appDetailObject.ym_method_getPackageName();    // 获取 app 的包名
    int versionCode     =  appDetailObject.ym_method_getVersionCode();    // 获取 app 的版本号
    String versionName  =  appDetailObject.ym_method_getVersionName();    // 获取 app 的版本名
    String adIconUrl    =  appDetailObject.ym_method_getIconUrl();        // 获取 app 的图标地址
    String [] ssUrls    =  appDetailObject.ym_method_getScreenShotUrls(); // 获取 app 的截图地址列表
    String adSlogan     =  appDetailObject.ym_method_getAdSlogan();       // 获取广告标语
    String desc         =  appDetailObject.ym_method_getDescription();    // 获取广告的详细描述
    String size         =  appDetailObject.ym_method_getAppSize();        // 获取 app 的大小
    int points          =  appDetailObject.ym_method_getPoints();         // 获取 app 的积分
    String pointsUnit   =  appDetailObject.ym_method_getPointsUnit();     // 获取广告的积分单位
    String appCat       =  appDetailObject.ym_method_getAppCategory();    // 获取应用类型
    int actionType      =  appDetailObject.getActionType();               // 获取广告类型
    int adStatus        =  appDetailObject.ym_method_getAdTaskStatus();   // 获取广告的完成状态
    int dlStatus        =  appDetailObject.ym_method_getAdDownloadStatus(); // 获取广告的下载状态
    String author       =  appDetailObject.ym_method_getAuthor();         // 获取该 app 的作者名
    String steps        =  appDetailObject.ym_method_getTaskSteps();      // 任务步骤流程指引

**注：**

广告的完成状态、下载状态以及广告的类型值请参考上述第三点：获取广告列表中的描述


4.2 获取方式
~~~~~~~~~~~~

获取积分墙某个广告的详细数据有两种方式，一种为 **同步加载** ，一种为 **异步加载** 。


1. 同步加载方式（请注意在非 UI 线程中使用）：
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: java

    import net.youmi.android.offers.diyoffer.Ym_Class_AppDetailObject;
    import net.youmi.android.offers.diyoffer.Ym_Class_DiyOfferWallManager;
    ...

    /**
     *  获取广告的详细信息（请注意不要在 UI 线程中直接使用）
     *  @param  Ym_Class_AppSummaryObject 广告的摘要信息对象，广告的摘要信息对象请参考3.1节的描述
     */
    Ym_Class_DiyOfferWallManager.getInstance(Context context).ym_method_getAppDetailData(Ym_Class_AppSummaryObject appSummaryObject);


*示例代码* ：

.. code-block:: java

    import net.youmi.android.offers.diyoffer.Ym_Class_AppDetailObject;
    import net.youmi.android.offers.diyoffer.Ym_Class_DiyOfferWallManager;
    ...

    new Thread(new Runnable() {
        @Override
        public void run() {
            // 这里传入广告的摘要信息数据模型对象，以获取广告的详细数据
            Ym_Class_AppDetailObject data  = Ym_Class_DiyOfferWallManager.getInstance(this).ym_method_getAppDetailData(appSummaryObject);
        }
    }).start();


2. 异步加载方式：
^^^^^^^^^^^^^^^^^

.. code-block:: java

    /**
     *  获取广告的详细信息
     *  @param  appSumObject  要加载的广告的摘要信息对象
     *  @param  appDetailDataInterface  回调接口，当返回数据结果时回调本接口
     */
    Ym_Class_DiyOfferWallManager.getInstance(Context context).ym_method_loadAppDetailData(Ym_Class_AppSummaryObject appSummaryObject,
        Ym_Class_AppDetailDataInterface appDetailDataInterface);

*示例代码* ：

.. code-block:: java

    import net.youmi.android.offers.diyoffer.Ym_Class_AppSummaryObject;
    import net.youmi.android.offers.diyoffer.Ym_Class_AppDetailObject;
    import net.youmi.android.offers.diyoffer.Ym_Class_DiyOfferWallManager;
    import net.youmi.android.offers.diyoffer.Ym_Class_AppDetailDataInterface;
    ...

    /**
     *  异步加载积分墙某个广告的详细数据
     */
    Ym_Class_DiyOfferWallManager.getInstance(this).ym_method_loadAppDetailData(appSummaryObject,
        new Ym_Class_AppDetailDataInterface() {
            /**
             *  当成功加载到数据的时候，会回调本方法（注意：本回调方法不在 UI 线程中执行，所以请不要在本接口中进行 UI 线程方面的操作）
             */
            @Override
            public void ym_method_onLoadAppDetailDataSuccess(Context context, Ym_Class_AppDetailObject appDetailObject) {
                Log.d("test", appDetailObject.toString());
            }

            /**
             *  当加载数据失败的时候，会回调本方法（注意：本回调方法不在 UI 线程中执行，所以请不要在本接口中进行 UI 线程方面的操作）
             */
            @Override
            public void ym_method_onLoadAppDetailDataFailed() {
                Log.d("test", "没有获取到数据");
            }
    });


五、下载和打开应用（重要）
--------------------------

----

通过调用下面方法即可下载（或打开）广告，如果该广告的完成状态为 <未完成>，则可获取积分结算

.. caution::

    **注意：** 打开广告务必调用本方法，否则可能无法获取积分和结算

.. code-block:: java

    import net.youmi.android.offers.diyoffer.Ym_Class_DiyOfferWallManager;
    ...
	
    // 1、传入 Ym_Class_AppSummaryObject 对象
    Ym_Class_DiyOfferWallManager.getInstance(Context context).ym_method_openOrDownloadApp(Ym_Class_AppSummaryObject appSummaryObject);

    // 2、传入 Ym_Class_AppDetailObject 对象
    Ym_Class_DiyOfferWallManager.getInstance(Context context).ym_method_openOrDownloadApp(Ym_Class_AppDetailObject appDetailObject);


六、积分相关操作功能（重要）
----------------------------

----

6.1 查询积分余额
~~~~~~~~~~~~~~~~

调用以下接口，查询用户的积分账户余额：

.. code-block:: java

    import net.youmi.android.offers.Ym_Class_PointsManager;
    ...
    int myPointBalance = Ym_Class_PointsManager.getInstance(this).ym_method_queryPoints();

.. tip::

    **注意：** 该接口直接返回 int 型的积分余额。


6.2 扣除积分
~~~~~~~~~~~~

调用以下接口，扣除用户积分账户余额：

.. code-block:: java

    import net.youmi.android.offers.Ym_Class_PointsManager;
    ...
    int amount = 100; // 示例扣除100积分。
    boolean isSuccess = Ym_Class_PointsManager.getInstance(this).ym_method_spendPoints(amount);

.. tip::

    **注意：** 该接口直接返回扣除积分结果，成功扣除返回 true，否则返回 false。


6.3 增加积分
~~~~~~~~~~~~

调用以下接口，往用户积分账户余额增加积分：

.. code-block:: java

    import net.youmi.android.offers.Ym_Class_PointsManager;
    ...
    int amount = 100; // 示例增加100积分。
    boolean isSuccess = Ym_Class_PointsManager.getInstance(this).ym_method_awardPoints(amount);

.. tip::

    **注意：** 该接口直接返回增加积分结果，成功返回 true，否则返回 false。


七、监听应用的下载和安装（可选）
--------------------------------

----

app 下载安装监听器适用于当 app 下载安装状态改变时通知 UI 界面进行更新显示，比如下载进度的更新时 UI 界面应该显示进度条，当下载成功时隐藏进度条并提示用户安装等等，这些一般都只适用于 UI 交互。

通过实现 net.youmi.android.offers.diyoffer.DiyAppNotify 这个接口，并且在界面初始化后向 net.youmi.android.offers.diyoffer.Ym_Class_DiyOfferWallManager 的 registerListener 方法注册监听即可让界面随时获得 app 的下载安装状态，在界面销毁时，请务必调用 removeListener 方法注销监听。

DiyAppNotify 的定义：

.. code-block:: java

    /**
     *  app下载安装监听器
     */
    public interface DiyAppNotify {
        /**
         *  下载进度变更通知，在 UI 线程中执行。
         *  @param  id
         *  @param  contentLength
         *  @param  completeLength
         *  @param  percent
         *  @param  speedBytesPerS
         */
        public void onDownloadProgressUpdate(int id, long contentLength, long completeLength, int percent, long speedBytesPerS);

        /**
         *  下载成功通知，在 UI 线程中执行。
         *  @param  id
         */
        public void onDownloadSuccess(int id);

        /**
         *  下载失败通知，在 UI 线程中执行。
         *  @param  id
         */
        public void onDownloadFailed(int id);

        /**
         *  安装成功通知，在 UI 线程中执行。
         *  @param appObject
         */
        public void onInstallSuccess(int id);
    }

如果需要判断两个 app 是否为同一个，则可以通过获取它的广告 id 进行比较即可。

Ym_Class_DiyOfferWallManager 关于下载安装监听器的调用：

.. code-block:: java

    /**
     *  注册监听器
     */
    public void registerListener(DiyAppNotify listener);

    /**
     *  注释监听器
     */
    public void removeListener(DiyAppNotify listener);


八、其他功能（可选）
--------------------

----

8.1 设置请求广告的数量
~~~~~~~~~~~~~~~~~~~~~~

通过调用下面方法即可设置请求广告列表的长度，如果需要使用本方法，请在调用获取广告列表的方法之前调用本方法

.. code-block:: java

    import net.youmi.android.offers.diyoffer.Ym_Class_DiyOfferWallManager;
    ...
    Ym_Class_DiyOfferWallManager.getInstance(Context context).ym_method_setRequestCount(int count);


8.2 签到功能
~~~~~~~~~~~~

签到功能提供对 <已完成> 状态的广告进行签到，以提高广告的效果，开发者可以通过自己的服务器来做签到，sdk中也集成了签到功能的简单使用，用法如下：


首先通过调用下面方法获取签到列表，``请注意在非 UI 线程中调用本方法``。

*示例* ：

.. code-block:: java

    import net.youmi.android.offers.diyoffer.Ym_Class_AppSummaryDataInterface;
    import net.youmi.android.offers.diyoffer.Ym_Class_AppSummaryObjectList;
    import net.youmi.android.offers.diyoffer.Ym_Class_DiyOfferWallManager;
    ...
	
    new Thread(new Runnable() {
        @Override
        public void run() {
            // TODO Auto-generated method stub
            Ym_Class_AppSummaryObjectList list = Ym_Class_DiyOfferWallManager.getInstance(Context context).ym_method_getSignInAdList();
        }
    }).start();

然后通过调用下面方法之一，可以为签到列表上的广告进行签到，``下面三个方法可在任意线程中使用，请确保在使用这三个方法之前已经进行过至少一次的广告列表请求`` ：

.. code-block:: java

    // 1、通过传入 Ym_Class_AppSummaryObject 对象进行签到
    Ym_Class_DiyOfferWallManager.getInstance(Context context).ym_method_sendSignInActionType(Ym_Class_AppSummaryObject appSummaryObject);

    // 2、通过传入 Ym_Class_AppDetailObject 对象进行签到
    Ym_Class_DiyOfferWallManager.getInstance(Context context).ym_method_sendSignInActionType(Ym_Class_AppDetailObject appDetailObject);
	
    // 3、通过传入 广告ID 进行签到 （适用于开发者通过自己的服务器进行进行签到）
    Ym_Class_DiyOfferWallManager.getInstance(Context context).ym_method_sendSignInActionType(int adId);
	
如果你需要监听签到是否已经成功，可以在上面方法中，再传入一个参数，该参数的类型为Ym_Class_SignInInterface，通过这个接口可以监听签到的情况。

*示例* ：

.. code-block:: java

    import net.youmi.android.offers.diyoffer.Ym_Class_DiyOfferWallManager;
    import net.youmi.android.offers.diyoffer.Ym_Class_SignInInterface;
    ...

	// 对任务进行签到，appDetailObject为广告的详细信息对象
	Ym_Class_DiyOfferWallManager.getInstance(this).ym_method_sendSignInActionType(appDetailObject, new Ym_Class_SignInInterface() {
				
		/**
		 * 签到成功时会回调这个接口，本回调接口执行在UI线程中
		 */
		@Override
		public void ym_method_signInSuccess(Context context) {
			// TODO Auto-generated method stub
			Toast.makeText(context, "签到成功", Toast.LENGTH_SHORT).show();
		}
		
		/**
		 * 签到失败时会回调这个接口，本回调接口执行在UI线程中
		 * @param adId 广告ID
		 * @param errorCode 错误代码
		 */
		@Override
		public void ym_method_signInFailed(Context context, int adId, int errorCode) {
			// TODO Auto-generated method stub
			Toast.makeText(context, String.format("广告id = %d, 签到失败,错误代码：%d", adId, errorCode), Toast.LENGTH_LONG).show();
			
		}
	});
	
关于签到的错误代码描述如下：

.. code-block:: xml

	100：获取签到返回数据失败 
	101：广告任务还没有完成，不能签到
	102：签到请求参数缺失，请确定已经进行过广告列表的请求以获取广告请求参数
	103：没有查询到指定广告ID的相关广告记录


九、积分墙高级功能（可选）
--------------------------

----

积分墙 SDK 提供了如下高级功能：

* 积分余额变动通知
* 客户端 SDK 获取订单信息
* 服务器获取订单信息（开发者直接通过自己设置的服务器监听订单信息）
* 验证积分墙配置是否正确
* 关闭有米 Debug Log

更多详情请参考 `积分墙高级功能 <offers_opt.html>`_


十、SDK 实用工具（可选）
------------------------

----

SDK 实用功能为您提供了便捷的实用工具：

* 检查更新
* 在线配置
* 用户数据统计

更多详情请参考 `SDK 实用工具 <functional.html>`_


十一、其他
----------

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
