# 0.注意事项及准备工作

## 0.1 Android版本限制

android9.0 minsdk28

## 0.2 需提供的资料

请在集成sdk前，将以下app相关信息提供于惠诚。

*  application_id (包名)
## 0.3 支持语言

java，kotlin

# 1.集成方式

## 1.1 预备工作

###  1.1.1.将sdk.aar 拷贝至app主工程libs目录下，并在build.gradle（app）中添加

```typescript
dependencies {
...
  implementation(fileTree("libs"))
  implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-android:1.3.9'
  implementation("com.squareup.okhttp3:okhttp:4.10.0")
}
```
### 1.1.2.添加权限

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
## 1.2 流程概述

1. 授权当前用户
2. 以当前用户身份（userid）登录
3. 获取所有疗愈类型
4. 初始化 Therapy并监听播放回掉
5. 根据用户选择的疗愈相关配置参数，获取方案
6. 根据获取的方案json字符串，预备Therapy
7. 预备完成后，播放
# 2.授权登录
## 2.1 授权

```typescript
AuthorizeApi().authorize(
        channelID,//渠道id
        new IResponseCallback<Boolean>() {
            @Override
            public void onRequestSucceed(
              Boolean isAuthorizeSuccessful //渠道授权是否成功
            ) {
            }

            @Override
            public void onRequestFailed(@NonNull Call call, @NonNull Exception exception) {
            }
        });
```
## 2.2 登录

```plain
 未授权情况下调用，抛出AuthException (不建议频繁调用。)
```

```typescript
//未授权情况下调用，抛出AuthException
AuthorizeApi().login(
        userID, //用户标识
        channelID,//渠道id
        context,//上下文
        new IResponseCallback<Boolean>() {
    @Override
    public void onRequestSucceed(Boolean isLoginSuccessful) {

    }

    @Override
    public void onRequestFailed(@NonNull Call call, @NonNull Exception exception) {
    }
});
```

# 
# 3.疗愈相关

## 3.1 获取所有疗愈

```typescript
// 无需授权
TherapyApi().fetchAllCures(
    channelID,// 渠道id
   new IResponseCallback<List<Cure>>() {
      @Override
      public void onRequestFailed(@NonNull Call call, @NonNull Exception exception) {
、      }

      @Override
      public void onRequestSucceed(List<Cure> cures) {
          //返回所有疗愈类型
、    }
});
```

## 3.2  获取方案

```plain
 未授权情况下调用，抛出AuthException。
 异地登录抛出：code为10026的HCException。
 如果主类型下没有分类则不传；否则一定要传其中一个。
```

```typescript
/*param*/
TherapyApi().fetchCureSolution(
        channelID,//渠道id
        userID,//当前用户标识
        cure,//疗愈主类型
        classification,//疗愈分类，如果主类型下没有分类则不传；否则一定要传其中一个。
        duration,//疗愈时长
        trackNum,//轨道数量，目前就是1
        level,//疗愈相关程度
        context,// 上下文
        new IResponseCallback<String>() {
            @Override
            public void onRequestFailed(@NonNull Call call,
                                       @NonNull Exception exception) {
    
            }
            @Override
            public void onRequestSucceed(String solutionJSON) {
                //方案的json字符串
            }
        });
```

# 4.播放

## 4.1 Therapy 初始化(必须)

```typescript
 Therapy(
   "secret", // 需要由开发人员事先提供applicationId于惠诚
   "app id", // 需要由开发人员事先提供applicationId于惠诚 
   context //应用程序上下文
 )
```
注：尽量避免懒加载初始化sdk，防止因sdk未初始化导致的其他接口调用失败。
### 4.2. 添加播放相关监听

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

## 4.3  加载方案（播放前必须完成加载）

```typescript
Therapy().prepare(
    @NotNull String solutionJSON, //方案 in json
    @NotNull IPrepareCallback callback//完成方案预加载后的回掉
) 
```
## eg：

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

## 4.4 播放

```typescript
//需要事先完成方案预加载。 
//如果当前正在播放，立刻return
Therapy().play() 
```
## 4.5 暂停

```typescript
// 暂停后，调用play继续播放
Therapy().pause()
```
## 4.6 结束

```typescript
Therapy().stop()
```
## 4.7 获取缓存大小

```typescript
//返回：缓存总大小
long Therapy().getCacheSize()
```
## 4.8 清理缓存

```typescript
Therapy().clearCache()
```
## 4.9 当前疗愈是否可用

```typescript
bool Therapy().getEnable()
```
## 4.10 是否播放中

```typescript
bool Therapy().isPlaying()
```

# 5.实体类

## 5.1 疗愈

```plain
class Cure {
   int id;   // id
   String title; // title
   String illustrate; //说明
   List<Long>  cureDurationOptions; //相关时间选项
   List<Level>  levels; //相关程度选项
   List<Classification>  classifications; //相关子项目(可能为空)
}
```


