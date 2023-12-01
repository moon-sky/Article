### SpeechEngine知多少



扩展自ISpeechEngine.Stub，说明这是提供服务的服务端

实现了ISpeechEngine接口



```java
public interface ISpeechEngine extends IInterface {
  //跨进程通讯用的actor与sender
    IActorBridge getActorBridge() throws RemoteException;

    ISubscriber getSubscriber() throws RemoteException;

    ITTSEngine getTTSEngine() throws RemoteException;

    IWakeupEngine getWakeupEngine() throws RemoteException;

    IAgent getAgent() throws RemoteException;

    IAppMgr getAppMgr() throws RemoteException;

    ISpeechState getSpeechState() throws RemoteException;

    ISoundLockState getSoundLockState() throws RemoteException;

    IASREngine getASREngine() throws RemoteException;

    IQueryInjector getQueryInjector() throws RemoteException;

    IRecordEngine getRecordEngine() throws RemoteException;

    IWindowEngine getWindowEngine() throws RemoteException;

    IRecognizer getRecognizer() throws RemoteException;

    IHotwordEngine getHotwordEngine() throws RemoteException;

    ICarSystemProperty getCarSystemProperty() throws RemoteException;

    IVADEngine getVadEngine() throws RemoteException;
```

#### SDKAdapter

负责初始化各种engine（）

![image-20210712114222275](https://gitee.com/moonsky/image-bed/raw/master/image-20210712114222275.png)

![image-20210712114244780](https://gitee.com/moonsky/image-bed/raw/master/image-20210712114244780.png)

![image-20210712114306692](https://gitee.com/moonsky/image-bed/raw/master/image-20210712114306692.png)



#### SpeechState

根据电话当前的状态，来处理唤醒等相关的事务

#### SoundLockManager

音区锁定管理类

#### AppMgr

系统应用管理类

#### SpeechAgent



### Lib_speech_protocol

#### SpeechClient---aidl---DameonService

onConnect()通知各个proxy，并获取到carspeechservice 的speechengine对象



##### DDSWakeUpEngine

#### AIWakeupEngine



#### SpeechClient

#### 

