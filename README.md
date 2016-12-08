# ADroi海外聚合广告SDK
ADroi-SDK 说明文档
---

#简介
ADroi海外聚合广告SDK是卓易科技为出海客户进行打造的变现平台，实现了多平台SDK的聚合，实时动态切换，包含了强大的控制功能，让出海客户有更多选择的同时，也简化了接入流程！
    
#安装
由于目前部分海外广告平台不再支持Eclipse,所以本SDK目前未给出支持Eclipse的方式。
##Android Studio 安装SDK请参照以下步骤：
    
1.在项目app的build.gradle中添加如下代码：
```
repositories {
    jcenter()
    
    //增加maven仓库
    maven {
        url "https://github.com/DroiBaaS/DroiOversea-AD-SDK/raw/master/"
    }
}

dependencies {
    compile fileTree(include: ['*.jar'], dir: 'libs')
    compile 'com.android.support:appcompat-v7:24.2.1'
    testCompile 'junit:junit:4.12'
    //增加ADroiSDK依赖
    compile 'com.droi:adroi:2.0.1'   
    compile 'com.squareup.picasso:picasso:2.5.2'
    //增加Facebook-Ad-SDK依赖
    compile 'com.facebook.android:audience-network-sdk:4.+'
    compile "com.facebook.fresco:fresco:0.5.0"
    //增加AdMob-Ad-SDK依赖
    compile 'com.google.android.gms:play-services-ads:+'
}
```

2.在项目的AndroidManifest.xml中增加基础配置： 
```
<application>
    ...

    <!-- 增加Admob-AD相关配置 -->
    <activity
        android:name="com.google.android.gms.ads.AdActivity"
        android:configChanges="keyboard|keyboardHidden|orientation|screenLayout|uiMode|screenSize|smallestScreenSize"
        android:theme="@android:style/Theme.Translucent" />
    <meta-data
        android:name="com.google.android.gms.version"
        android:value="@integer/google_play_services_version" />

    <!-- 增加Facebook-AD相关配置 -->
    <activity
        android:name="com.facebook.ads.InterstitialAdActivity"
        android:configChanges="keyboardHidden|orientation|screenSize"/>
    <activity
        android:name="com.facebook.ads.AudienceNetworkActivity"
        android:configChanges="keyboardHidden|orientation|screenSize"/>

    <!--配置应用ID&渠道号，必配，数据由Droi运营给出-->
    <meta-data android:name="DROI_APPID" android:value="您的应用ID"/>
    <meta-data android:name="DROI_CHANNEL" android:value="您的渠道号"/>
</application>
```

3.在项目的AndroidManifest.xml中配置广告控制相关参数（选配）：
```
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    ...>
    <application>
        ...
        <!--如有需要，配置控制参数，参数由Droi运营给出 -->
        <meta-data android:name="DROI_CUSTOMER" android:value="卓易客户项"/>
        <meta-data android:name="DROI_BRANDS" android:value="卓易品牌项"/>
        <meta-data android:name="DROI_PROJECT" android:value="卓易工程项"/>
        <meta-data android:name="DROI_CPU" android:value="卓易平台项"/>
        <meta-data android:name="DROI_OSVERSION" android:value="卓易版本项"/>
    </application>
</manifest>
```

#使用
更多用例可以参考我们在GitHub上的示例Demo，地址：https://github.com/DroiBaaS/DroiOverseaSDK-Demo-Android

##初始化
在应用Application的onCreate()中添加SDK初始化代码：
```
    try {
        DroiSdk.initialize(this);
    } catch (Exception e) {
        e.printStackTrace();
    }
```

##横幅广告
横幅广告接入分为两步：

1.在xml中增加banner视图：
```
    <com.droi.mobileads.DroiView
        android:id="@+id/adview"
        android:layout_width="fill_parent"
        android:layout_height="50dp"
        />
```

2.在对应的Activity中增加以下code：
```
public class MainActivity extends AppCompatActivity implements DroiView.BannerAdListener{
    private DroiView droiView;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        //1.创建横幅广告
        droiView = (DroiView) findViewById(R.id.adview);
        //2.设置广告单元ID
        droiView.setAdUnitId("YOUR_BANNER_AD_UNIT_ID_HERE");
        //3.请求广告
        droiView.loadAd();
        //4.设置广告监听
        droiView.setBannerAdListener(this);
    }

    @Override
    public void onBannerLoaded(DroiView banner) {
        Log.d("Droi","droi bannner onBannerLoaded");
        // Sent when the banner has successfully retrieved an ad.
    }

    @Override
    public void onBannerFailed(DroiView banner, DroiErrorCode errorCode) {
        Log.d("Droi","droi bannner onBannerFailed"+errorCode.toString());
        // Sent when the banner has failed to retrieve an ad. 
        //You can use the DroiErrorCode value to diagnose the cause of failure.
    }

    @Override
    public void onBannerClicked(DroiView banner) {
        Log.d("Droi","droi bannner onBannerClicked");
        // Sent when the user has tapped on the banner.
    }

    @Override
    public void onBannerExpanded(DroiView banner) {
        Log.d("Droi","droi bannner onBannerExpanded");
        // Sent when the banner has just taken over the screen.
    }

    @Override
    public void onBannerCollapsed(DroiView banner) {
        Log.d("Droi","droi bannner onBannerCollapsed");
        // Sent when an expanded banner has collapsed back to its original size.
    }

    @Override
    protected void onDestroy() {
        //5.销毁广告视图
        if(droiView!= null){
            droiView.destroy();
            droiView = null;
        }
        super.onDestroy();
    }
}
```

##插屏广告

插屏的实现可以参考以下Code：
```
public class MainActivity extends AppCompatActivity implements DroiInterstitial.InterstitialAdListener{
    //1.声明插屏广告对象
    private DroiInterstitial mInterstitial;
    String TAG="Droi";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        //2.创建插屏广告对象，并设置对应的广告位ID；
        mInterstitial = new DroiInterstitial(this, "YOUR_INTERSTITIAL_AD_UNIT_ID_HERE");
        //3.设置插屏广告的监听；
        mInterstitial.setInterstitialAdListener(MainActivity.this);
        //4.请求广告；
        mInterstitial.load();
    }
    
    // 自行定义调用显示广告的方法
    void yourAppsShowInterstitialMethod() {
        //5.广告请求成功时，显示广告；
        if (mInterstitial.isReady()) {
            mInterstitial.show();
        } else {
            // Caching is likely already in progress if `isReady()` is false.
            // Avoid calling `load()` here and instead rely on the callbacks as suggested below.
        }
    }
    
    @Override
    public void onInterstitialLoaded(DroiInterstitial interstitial) {
        Log.d(TAG,"droi onInterstitialLoaded");
        // The interstitial has been cached and is ready to be shown.
    }

    @Override
    public void onInterstitialFailed(DroiInterstitial interstitial, DroiErrorCode errorCode) {
        Log.d(TAG,"droi onInterstitialFailed");
        // The interstitial has failed to load. Inspect errorCode for additional information.
        // This is an excellent place to load more ads.    
    }

    @Override
    public void onInterstitialShown(DroiInterstitial interstitial) {
        Log.d(TAG,"droi onInterstitialShown");
        // The interstitial has been shown. Pause / save state accordingly.
    }

    @Override
    public void onInterstitialClicked(DroiInterstitial interstitial) {
        Log.d(TAG,"droi onInterstitialClicked");
    }

    @Override
    public void onInterstitialDismissed(DroiInterstitial interstitial) {
        Log.d(TAG,"droi onInterstitialDismissed");
         // The interstitial has being dismissed. Resume / load state accordingly.
        // This is an excellent place to load more ads.
    }

    @Override
    protected void onDestroy() {
        //6.退出时销毁广告视图；
        if(mInterstitial != null){
            mInterstitial.destroy();
            mInterstitial = null;
        }
        super.onDestroy();
    }
}
```

##原生广告
原生广告可让您通过与现有设计一致的方式轻松地通过应用获利。AdroiSDK可让您访问广告的各个素材资源，以便您按照自己的方式进行排列; 这样，广告看起来就像应用内容的一部分。 SDK会自动处理图片缓存和指标跟踪，以便您可以专注于展示广告的方式，时间和位置。

Adroi-SDK提供了一个类（DroiAdAdapter）:一个包装现有Adapter的子类将广告插入到ListView、GridView或其他视图所用的Adapter的实现对象（包括CursorAdapter）。

在Android应用程式中加入原生广告需要三个简单的步骤：

1.为原生广告创建XML布局;
2.定义应在信息流中放置广告的位置;
3.创建DroiAdAdapter来包装您现有的Adapter子类并开始加载广告;

**关于隐私信息图标**
您的原生广告必须显示隐私权信息图标，ADroiSDK会自动处理隐私信息图标上的点击事件，如何在广告中添加隐私权信息图标可以参考下节说明。

###设置原生广告布局

1.首先，定义一个XML布局，了解广告在应用Feed中的外观。这个例子布局包含两个TextView S代表标题和其他文本，再加上三个ImageView：一个图标图像，主图像和隐私信息的图标；从广告中选择最适合您应用Feed中最无缝的资源。
**不过，您的广告必须包含隐私信息的图标** 。建议的尺寸为40 dp大小的方块包含10dp的padding（即图标显示面积只有20dp*20dp，但点击面积更大些）。 此图标链接到一个重要的隐私声明，并且是必须的。

例如：res/layout/native_ad_layout.xml
```
    <RelativeLayout ...>
      <ImageView android:id="@+id/native_ad_main_image"
        ... />
      <ImageView android:id="@+id/native_ad_icon_image"
        ... />
      <TextView android:id="@+id/native_ad_title"
        ... />
      <TextView android:id="@+id/native_ad_text"
        ... />
      <ImageView android:id="@+id/native_ad_privacy_information_icon_image"
        android:layout_width="40dp"
        android:layout_height="40dp"
        android:padding="10dp"
        ... />
    </RelativeLayout>
```

**注意**：您应该设置你的ImageViews为null背景属性，以便处理PNG透明度：
```
    <ImageView
    android:background="@null"
    ... />
```

2.接下来，创建一个ViewBinder对象，指定布局XML和广告内容之间的绑定。

```
    ViewBinder viewBinder = new ViewBinder.Builder(R.layout.native_ad_layout)
        .mainImageId(R.id.native_ad_main_image)
        .iconImageId(R.id.native_ad_icon_image)
        .titleId(R.id.native_ad_title)
        .textId(R.id.native_ad_text)
        .privacyInformationIconImageId(R.id.native_ad_privacy_information_icon_image)
        .build();
```

注意：如果你直接显示原生广告，你可能有文字或图像提供额外的子视图,可以通过以下方式添加额外字段：
```
    new ViewBinder.Builder().addExtras(Map<String, Integer> resIds)
```
或者
```
    new ViewBinder.Builder().addExtra(String key, int resId)
```

示例：假设在Droi Web UI中添加了自定义字段“sponsoredtext”和“sponsoredimage”：
```
     "sponsoredtext" : "Sponsored",
     "sponsoredimage" : "http://www.droi.com/sponsored.jpg"
```

您可以将这些额外字段添加到您的XML布局，然后将它们绑定到视图，如下所示：
```
    ViewBinder viewBinder = new ViewBinder.Builder(R.layout.native_ad_layout)
      .mainImageId(R.id.native_ad_main_image)
      .iconImageId(R.id.native_ad_icon_image)
      .titleId(R.id.native_ad_title)
      .textId(R.id.native_ad_text)
      .privacyInformationIconImageId(R.id.native_ad_privacy_information_icon_image)
      // Your custom fields are bound with the addExtra() call.
      .addExtra("sponsoredtext", R.id.sponsored_text)
      .addExtra("sponsoredimage", R.id.sponsored_image)
      .build();
```
 
注意：如果要向附加功能添加图像，键必须以“image”结尾。

3.DroiAdAdapter通过类DroiStaticNativeAdRenderer在您的信息流中加载的广告。 创建DroiStaticNativeAdRenderer的实例：
```
    DroiStaticNativeAdRenderer adRenderer = new DroiStaticNativeAdRenderer(viewBinder);
```

###在信息流中放置广告

接下来，通过DroiNativeAdPositioning.DroiServerPositioning对象来设置广告在信息流中显示的位置。 您可以在原生广告单元的设置界面设置广告在信息流中显示的位置以及显示间隔。

如果更新了设置，需要创建DroiNativeAdPositioning.DroiServerPositioning对象：
```
    DroiNativeAdPositioning.DroiServerPositioning adPositioning =
        DroiNativeAdPositioning.serverPositioning();
```

###创建DroiAdAdapter

DroiAdAdapter类根据在ADroi UI设置的规则放置广告并处理广告缓存。

创建DroiAdAdapter实例的参数包含：当前的Context、当前List的Adapter的子类以及您在上一步中创建的DroiNativeAdPositioning对象。
```
    mAdAdapter = new DroiAdAdapter(this, adapter, adPositioning);
```

接下来，注册广告渲染器（在上一节的步骤3中通过ViewBinder创建的DroiStaticNativeAdRenderer实例），使适配器通过您创建的布局渲染广告：
```
    mAdAdapter.registerAdRenderer(adRenderer);
```

最后，设置信息流视图的适配器（DroiAdAdapter的实例）：
```
    myListView.setAdapter(mAdAdapter);
```

###开始加载广告

到这一步，DroiAdAdapter已准备好根据您的设置来加载广告。为了改善您的应用程序显示广告的相关性，你可以设置位置或者额外的关键字数据。您还可以精确指定所需的广告物料来节省带宽。
```
    final EnumSet<NativeAdAsset> desiredAssets = EnumSet.of(
                        NativeAdAsset.TITLE,
                        NativeAdAsset.TEXT,
                        // Don't pull the ICON_IMAGE
                        // NativeAdAsset.ICON_IMAGE,
                        NativeAdAsset.MAIN_IMAGE,
                        NativeAdAsset.CALL_TO_ACTION_TEXT);
```
然后创建一个新的RequestParameters使用对象Builder：
```
    mRequestParameters = new RequestParameters.Builder()
                        .location(location)
                        .keywords(keywords)
                        .desiredAssets(desiredAssets)
                        .build();
```
现在您可以通过原生广告位以及上一步的请求参数来请求原生广告了：
```
    mAdAdapter.loadAds(AD_UNIT_ID, mRequestParameters);
```

要加载全新的广告到信息流可以调用DroiAdAdapter#refreshAds。

示例代码：
```
 private ListView mListView;
  private DroiAdAdapter mAdAdapter;
  private static final String MY_AD_UNIT_ID = "myAdUnitId";

  @Override
  public void onCreate(Bundle savedInstanceState) {
      // Set up your adapter as usual.
      Adapter myAdapter;

      // Set up a ViewBinder and DroiStaticNativeAdRenderer as above.
      ViewBinder viewBinder = new ViewBinder.Builder(R.layout.native_ad_layout)
              .mainImageId(R.id.native_ad_main_image)
              .iconImageId(R.id.native_ad_icon_image)
              .titleId(R.id.native_ad_title)
              .textId(R.id.native_ad_text)
              .privacyInformationIconImageId(R.id.native_ad_privacy_information_icon_image)
              .addExtra("sponsoredText", R.id.sponsored_text)
              .addExtra("sponsoredImage", R.id.sponsored_image)
              .build();

      // Set up the positioning behavior your ads should have.
      DroiNativeAdPositioning.DroiServerPositioning adPositioning =
            DroiNativeAdPositioning.serverPositioning();
      DroiStaticNativeAdRenderer adRenderer = new DroiStaticNativeAdRenderer(viewBinder);

      // Set up the DroiAdAdapter
      mAdAdapter = new DroiAdAdapter(this, myAdapter, myAdPositioning);
      mAdAdapter.registerAdRenderer(adRenderer);

      myListView.setAdapter(mAdAdapter);
  }

  @Override
  public void onResume() {
      // Set up your request parameters
      myRequestParameters = RequestParameters.Builder()
              .keyWords("my targeting keywords")
              .build();

      // Request ads when the user returns to this activity.
      mAdAdapter.loadAds(MY_AD_UNIT_ID, myRequestParameters);
      super.onResume();
  }
```

###销毁DroiAdAdapter

当DroiAdapter的Activity销毁时，DroiAdapter也必须销毁。你应该在创建DroiAdapter相反的生命周期方法中销毁DroiAdapter。 如果你在Activity#onCreate创建了适配器，你应该在Activity#onDestroy毁掉它。如果你在Activity#onResume创建了适配器，你应该在Activity#onPause毁掉它。
示例代码：
```
     @Override
      protected void onDestroy() {
          mAdAdapter.destroy();
          super.onDestroy();
      }
```

##其他说明

###调试模式
开发者接入SDK时，可以通过以下方法开启Debug模式来打印日志以及请求测试广告（开关Debug模式的代码建议添加在Application中的初始化代码之后）。
```
    DroiSdk.setDebugMode(true);
``` 
已正式发布的应用，可以在手机的/sdcard/AdSDK/目录下修改dev_data.json文件,替换内容 {"debug_mode":true} 后重启应用即可开启SDK的日志打印功能。  
**注：**    *正式发布的应用请关闭调试模式，否则会影响广告计费！*  

###关于混淆
如果您的应用需要混淆，请在混淆的配置文件中加入以下代码：
```
-dontwarn com.droi.**
-keep class  com.droi.nativeads.*{*;}
-keep class  com.droi.common.**{*;}
-keep class  com.droi.network.**{*;}

-keepclassmembers class com.droi.** { public *; }
-keep public class com.droi.**
-keep public class android.webkit.JavascriptInterface {}

-keep class com.droi.mobileads.CustomEventBannerAdapter {*;}
-keep class com.droi.mobileads.CustomEventInterstitialAdapter {*;}
-keep class com.droi.mobileads.CustomEventNativeAdapter {*;}

# Explicitly keep any custom event classes in any package.
-keep class * extends com.droi.mobileads.CustomEventBanner {}
-keep class * extends com.droi.mobileads.CustomEventInterstitial {}
-keep class * extends com.droi.nativeads.CustomEventNative {}

# Keep methods that are accessed via reflection
-keepclassmembers class ** { @com.droi.common.util.ReflectionTarget *; }

# Support for Android Advertiser ID.
-keep class com.google.android.gms.common.GooglePlayServicesUtil {*;}
-keep class com.google.android.gms.ads.identifier.AdvertisingIdClient {*;}
-keep class com.google.android.gms.ads.identifier.AdvertisingIdClient$Info {*;}

# Filter out warnings that refer to legacy Code.
-dontwarn org.apache.http.**
-dontwarn com.droi.volley.toolbox.**

# Support for Google Play Services
# http://developer.android.com/google/play-services/setup.html
-keep class * extends java.util.ListResourceBundle {
    protected Object[][] getContents();
}

-keep public class com.google.android.gms.common.internal.safeparcel.SafeParcelable {
    public static final *** NULL;
}

-keepnames @com.google.android.gms.common.annotation.KeepName class *
-keepclassmembernames class * {
    @com.google.android.gms.common.annotation.KeepName *;
}

-keepnames class * implements android.os.Parcelable {
    public static final ** CREATOR;
}

#inmobi
-keepattributes SourceFile,LineNumberTable
-keep class com.inmobi.** { *; }
-dontwarn com.inmobi.**
-keep public class com.google.android.gms.**
-dontwarn com.google.android.gms.**
-dontwarn com.squareup.picasso.**
-keep class com.google.android.gms.ads.identifier.AdvertisingIdClient{
     public *;
}
-keep class com.google.android.gms.ads.identifier.AdvertisingIdClient$Info{
     public *;
}
# skip the Picasso library classes
-keep class com.squareup.picasso.** {*;}
-dontwarn com.squareup.picasso.**
-dontwarn com.squareup.okhttp.**
# skip Moat classes
-keep class com.moat.** {*;}
-dontwarn com.moat.**

#facebook
-keep class com.facebook.** { *; }
-keep interface com.facebook.** { *; }
-dontwarn com.facebook.**

#admob
-dontwarn com.google.android.gms.**
-keep class com.google.android.gms.** { *;}
-keep class com.cmcm.adsdk.nativead.AdmobNativeLoader{
      <fields>;
      <methods>;
}
-keep public class com.google.android.gms.ads.**{
    public *;
}
# For old ads classes
-keep public class com.google.ads.**{
    public *;
}

-keep class org.apache.http.**{*;}
-keep public class android.net.http.SslError
-keep public class android.webkit.WebViewClient

-dontwarn android.webkit.WebView
-dontwarn android.net.http.SslError
-dontwarn android.webkit.WebViewClient
```

##FAQ
Q1.参数太多，不知道如何填写？  
A1.运营给出数据时会以Excel表格给出，会详细给出各个字段对应的替换值，开发者拿到表格后，直接把数据填写到对应的 meta-data 项中即可(**下方数据为示例数据**)。  
必须填写的数值：  

| name | value |  
|:-----|-------|  
|DROI_APPID|meituan01|  
|DROI_CHANNEL|mt_001|  
以下数据就运营如果没有给，可以不用在meta-data中配置：  

| name | value |  
|:-----|-------|  
|DROI_CUSTOMER|hwdsf0001|  
|DROI_BRANDS|thirdparty|  
|DROI_PROJECT|tp_01|  
|DROI_CPU|tp1111|  
|DROI_OSVERSION|6.0|  

Q2.无法获取到广告？  
A2.首先确认手机网络连接状态，网络是否可用；然后确认您的DROI_APPID以及DROI_CHANNEL是否填正确填写；然后确认当前应用的包名是不是您提供给Droi运营的包名；最后再检查一下使用的广告位及对应的类型是否一致，比如不要在横幅广告创建方法中调用插屏类广告的广告位。如果以上都检查完还是无法获取广告请联系Droi运营，确认是否给您的应用增加了相关配置！  

Q3.集成aar时出现错误？  
A3.aar内集成了fb,inmobi,admob广告相关配置，如果您在AndroidManifest中还做了上述广告平台的相关配置，可以删除本地配置。另外部分广告平台SDK有相关的依赖，如果与您本地的集成有冲突，可以通过以下方式移除相关依赖：

V4包依赖冲突：  
```
compile ('com.facebook.android:audience-network-sdk:4.+')
{
        exclude module: 'support-v4'
        exclude group: 'com.android.support'
}
```
