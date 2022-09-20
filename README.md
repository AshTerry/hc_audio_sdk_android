# 0.注意事项及准备工作

## 0.1 Android版本限制

android9.0 minsdk28

## 0.2 获取sdk授权

 开发人员需要提前提供app的application id于惠诚；后获得对应的 appid 和 secret，用以激活sdk。

## 0.3 支持语言

java，kotlin

# 1.集成方式

###  1.1.将sdk.aar 拷贝至app主工程libs目录下，并在build.gradle（app）中添加

```plain
dependencies {
...
  implementation(fileTree("libs"))
  implementation("com.squareup.okhttp3:okhttp:4.10.0")
}
```
###  1.2.初始化

```plain
 private Therapy therapy;

 void init() {
   String secret = "xxxxxxxxxxxxxxxxxxxxxxxxx";
   String appID = "000000000000";
   therapy = new Therapy(
        secret,
        appID,
        this.getContext()
   );
 }
```
### 1.3.添加播放进度监听

```plain
therapy.setOnITimeTickCallback(new ITickCallback() {

    //进度活动回掉
    //参数 currentProgress：当前进度 (秒)
    @Override
    public void hc_onASMRElapsed(long currentProgress) {
        System.out.println("hc_onASMRElapsed");
    }
    
    //试听进度完毕回掉
    @Override
    public void hc_onASMRTimeUp() {
        System.out.println("hc_onASMRTimeUp");
    }
});
```
### 1.4.添加权限

```plain
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.FOREGROUND_SERVICE"/>
<uses-permission android:name="android.permission.WAKE_LOCK"/>
```
如在非https环境下：
```plain
<application
  android:usesCleartextTraffic="true"
>
```
# 2.api

## 2.1 sdk 初始化(必须)

```plain
 val therapy = Therapy(
   "secret", // 需要由开发人员事先提供applicationId于惠诚
   "app id", // 需要由开发人员事先提供applicationId于惠诚 
   context //应用程序上下文
 )
```
注：尽量避免懒加载初始化sdk，防止因sdk未初始化导致的其他接口调用失败。
## 2.2  获取方案

```plain
//返回：若无异常返回方案json字符串；否则返回异常。

TherapyApi.fetchCureSolutionSafe(
    duration: Duration,//疗愈时长
    callback: IResponseCallback<String>,//获得方案的回掉
) 
```

## 2.3  预加载方案（播放前必须完成预加载）

```plain
Therapy.prepare(
    solutionJSON: String, //方案 in json
    callback: ITherapyCallback //完成方案预加载后的回掉
)
```

 eg:

```plain
String solution = loadSolution();
therapy.prepare(solution, new ITherapyCallback() {

    @Override
    public void onPrepared() {
        therapy.play();
    }
    
    @Override
    public void onPrepareFailed() {
    }
});
```
## 2.4 播放

```plain
Therapy.play() //需要事先完成方案预加载
```
## 2.5 暂停

```plain
Therapy.pause()// 暂停后，调用play继续播放
```
## 2.6 结束

```plain
 Therapy.stop()
```
## 2.7 获取缓存大小

```plain
long Therapy.getCacheSize()
```
## 2.8 清理缓存

```plain
Therapy.clearCache()
```

