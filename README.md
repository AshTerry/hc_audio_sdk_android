# 0.注意事项及准备工作

## 0.1 Android版本限制

android9.0 minsdk28

## 0.2 需提供的资料

请在集成sdk前，将以下app相关信息提供于惠诚。

*  application_id (包名)
## 0.3 支持语言

java，kotlin

# 1.集成方式

###  1.1.将sdk.aar 拷贝至app主工程libs目录下，并在build.gradle（app）中添加

```typescript
dependencies {
...
  implementation(fileTree("libs"))
  implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-android:1.3.9'
  implementation("com.squareup.okhttp3:okhttp:4.10.0")
}
```

### 
### 1.2.添加权限

```typescript
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.FOREGROUND_SERVICE"/>
<uses-permission android:name="android.permission.WAKE_LOCK"/>
```
如在非https环境下，请在AndroidManifest.xml中：
```typescript
<application
  android:usesCleartextTraffic="true"
>
```

### 
### 1.3.初始化

```typescript
   String secret = "xxxxxxxxxxxxxxxxxxxxxxxxx";
   String appID = "000000000000";
   Therapy therapy = new Therapy(
        secret,
        appID,
        this.getContext()
   );
```

### 
### 1.4.添加播放相关监听

```typescript
therapy.setPlayCallback(new IPlayerCallback() {

    //播放进度回掉
    //参数 currentProgress:当前进度
    @Override
    public void onTick(long currentProgress) {

    }
    
    //播放完成回掉
    @Override
    public void onFinished() {
        Log.e(TAG,"onFinished");
    }
    
    //播放启动回掉
    @Override
    public void onPlay() {
        Log.e(TAG,"onPlay");
    }
    
    //播放暂停回掉
    @Override
    public void onPause() {
        Log.e(TAG,"onPause");
    }
    
    //播放过程中出现异常
    @Override
    public void onErrorWhenPlaying(@NonNull Exception e) {
        Log.e(TAG,"onPause" + e.toString());
    }
});
```

### 
### 1.5. 播放准备

```typescript
therapy.prepare(solution, new IPrepareCallback() {
    //方案预加载完成
    @Override
    public void onPredownloadDone() {
        //可在方案预加载完成后立刻播放
       therapy.play();
    }

    //方案加载全部完成
    @Override
    public void onAllDone(@Nullable Exception e) {
        if (e == null) {
            //方案全部加载完成后，且无异常。
            therapy.play();
            return;
        }
        Log.e(TAG,"onAllDone" + e.toString());
        therapy.stop();
    }
});
```

# 
# 2.api

## 2.1 sdk 初始化(必须)

```typescript
 Therapy(
   "secret", // 需要由开发人员事先提供applicationId于惠诚
   "app id", // 需要由开发人员事先提供applicationId于惠诚 
   context //应用程序上下文
 )
```
注：尽量避免懒加载初始化sdk，防止因sdk未初始化导致的其他接口调用失败。
## 2.2  获取方案

```typescript
/*param*/
//duration：疗愈时长
TherapyApi().fetchCureSolutionSafe(duration, new IResponseCallback<String>() {
    //请求成功回掉
    //参数solution： 方案
    @Override
    public void onRequestSucceed(String solution) {
    }
    
    //请求成功回掉
    //参数call：基于okhttp的请求对象
    //参数e ： 异常
    @Override
    public void onRequestFailed(
                @NonNull Call call, 
                @NonNull Exception e) {
    }
});
```

## 2.3  加载方案（播放前必须完成加载）

```typescript
Therapy().prepare(
    @NotNull String solutionJSON, //方案 in json
    @NotNull IPrepareCallback callback//完成方案预加载后的回掉
) 
```

## 
## 2.4 播放

```typescript
//需要事先完成方案预加载。 
//如果当前正在播放，立刻return
Therapy().play() 
```

## 
## 2.5 暂停

```typescript
// 暂停后，调用play继续播放
Therapy().pause()
```

## 
## 2.6 结束

```typescript
Therapy().stop()
```

## 
## 2.7 获取缓存大小

```typescript
//返回：缓存总大小
long Therapy().getCacheSize()
```

## 
## 2.8 清理缓存

```typescript
Therapy().clearCache()
```

## 
## 2.9 当前疗愈是否可用

```typescript
bool Therapy().getEnable()
```

### 
### 2.10 是否播放中

```typescript
bool Therapy().isPlaying()
```

