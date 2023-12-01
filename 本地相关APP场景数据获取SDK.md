## 本地相关APP场景数据获取SDK

[TOC]



### 项目介绍

为了方便本地对话引擎以及其他语音相关业务获取地图APP的场景数据，该SDK提供了相关接口方便调用

### 集成方法

在build.gradle中添加

```groovy
dependencies {
    api 'com.xiaopeng.lib:lib_speech_resourcemanager:0.0.1.01-SNAPSHOT'
}
```

### 初始化

首先要初始化CarSpeechResourceManager

```java
CarSpeechResourceManager.getInstance(context)
```

### 监听导航数据

添加导航数据动态监听

```java
mResourceManager.addNaviDataChangeListener(new IResourceDataChangeListener() {
    @Override
    public void onDataChange(ResourceChangeTypeInfo info) {
        mTest1.setText("测试数据:\n");
        mTest1.append("本次变更类型："+info.changeType + "\n");
        mTest1.append("本次修改数据："+info.data + "\n");
        mTest1.append("所有导航缓存数据："+mResourceManager.getAllNavDataFromCache());
    }
});
```

### 从缓存中获取历史记录

```java
mResourceManager.getHistoryListFromCache();
```

### 从缓存中获取收藏记录

```java
mResourceManager.getHistoryListFromCache();
```

### 获取家庭住址

```java
CarSpeechResourceManager.getInstance().getHomeAddress();
```

### 获取公司地址

```java
CarSpeechResourceManager.getInstance().getHomeAddress();
```

### 获取ASR增强地图全量数据

```java
CarSpeechResourceManager.getInstance().getHomeAddress();
```

### 释放资源

```java
CarSpeechResourceManager.getInstance().destory();
```

