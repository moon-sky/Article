

## 本地语法构建失败导致小P无法使用客诉问题复盘

### 事件回顾

- 9-29 质量部门梁创发反馈E28 16860车主升级到1.4.4后语音指令执行异常
- 9-29 语音部门分析日志得出结论是用户同步通讯录时，数量较大，导致语法文件构建超时，进而导致本地识别无法使用。在用户接收到车机端AI推送后，小P会将识别模式改为端云混合识别，而这个场景中需要等待本地识别结果返回，所以导致语音表现异常。
- 9-29 与思必驰沟通，确定是思必驰本地语法SDK针对英文通讯录存在构建时长较长，交由他们语音部门负责分析以及制定解决方案。
- 9-30 与质量部门沟通，针对当前问题，制定应对方案，以防大量客诉出现。
- 10-7 质量部门反馈第二例9844车主也在升级1.4.4以后，语音指令异常。
- 10-8 同类问题客诉升至4例，内部组会确定问题原因、临时解决方案以及永久解决方案
- 10-9 测试人员开始在D22\D55\E28\D21B车型上尝试复现，确定该问题影响范围
- 10-9 验证在protocol sdk中过滤英文联系人过渡方案，并确定发版本计划
- 10-9 提交最新hot fix代码，随D55 10-15 1.0.2 D22 11-05 1.0.3版本发布 E28 1.5版本

### 原因分析

#### 用户日志分析

客诉用户日志中都存在如下信息

```java
RecognizerImplMix: 跨进程 build grammar 失败, code = 302, msg = timeout!!!
```

进一步分析，可以得知该日志是在用户使用蓝牙电话同步通讯录以后，会触发本地语法构建。

```java
    @Override
    protected void onParameterChanged(String key, Object value) {
        LogUtils.i(TAG, "onParameterChanged, key: " + key + "value: " + value);
      ………………………………

            case LEXICON:
      //通讯录有更新
                onLexiconChanged();
                break;
            
      ……………………………………
private void onLexiconChanged() {
    LogUtils.i(TAG, "on lexicon changed");
    final Runnable runnable = new Runnable() {
        @Override
        public void run() {
          
            final String newBnf = getBnf(grammarBnfPath);
            if (TextUtils.isEmpty(newBnf)) {
                LogUtils.w(TAG, "new bnf is empty");
                bnfMd5 = null;
                return;
            }
            bnfMd5 = Utils.getMD5(newBnf);
            LogUtils.i(TAG, "new bnf md5: " + bnfMd5);

            if (Configs.Recog.Local.isEnableLexicon() && !isGrammarExist()) {
                reInitNeeded.set(true);
              //重新初始化
                reInit();
            } else if (Configs.Recog.Cloud.isEnableLexicon() && initialized && !reInitNeeded.get()) {
                updateCloudLexicon();
            } else {
                LogUtils.w(TAG, "all lexicons are disabled or is not initialized");
            }
        }
    };

    ThreadPool.execute(runnable);
}
      
      
      
    //  ————————————————————重新初始化————————-
       private void initEngine() {
……………………
  //构建本地语法文件
            buildLocalGrammarWithRemote();
……………………
    }
      
    // ————————BuildGrammarClient——————
       public class BuildGrammarClient {
         //构建超时设置为60s

    private static final long BUILD_GRAMMAR_TIME_OUT_MILLIS = 60 * 1000;
         
         
         //如果构建超时，则将识别模式改为在线识别
                             public void onFailed(int code, String msg) {
                LogUtils.i(TAG, "跨进程 build grammar 失败, code = %s, msg = %s", code, msg);
  
               client.unbindService();
                LogUtils.i(TAG, "跨进程 build grammar 失败, 初始化 cloudEngine，即本地不行可以继续使用云端引擎", code, msg);
                engineMode = MODE_CLOUD;
                // 初始化成功了就不要再初始化, 避免云引擎一直不能用
                initCloudEngine();
            }
         
```

通过以上代码可以看出，客诉用户的构建全部是构建语法文件超时导致离线ASR初始化失败，进而导致离线ASR无法正常使用，将识别模式设置为在线识别模式。

但是当用户触发AI推送、蓝牙电话接听、自动泊车场景时，会再次将识别模式切换为DEFAULT也就是混合模式。导致在线识别结果返回后，由于离线识别未成功，进而无法进入NLU请求阶段，表现为小P无响应。

```java
@Override
public void feed(final NLUFeed feed) {
    super.feed(feed);
    NLUTriggerIntent triggerIntent = feed.triggerIntent;
  //1.4.4版本蓝牙电话、自动泊车、智能推送都会触发triggerIntent逻辑
    if (triggerIntent != null) {
        try {
            //设置为本地识别模式
            if (mIsLocalSkill.get()) {
                mIsAsrModeOffline.set(true);
            }
            final String asrMode = (mIsAsrModeOffline.get()) ? ASR_OFFLINE : ASR_DEFAULT;
          //切换引擎为本地的识别
            JarvisBus.instance().publish(SDKEvent.ASR_MODE, asrMode);
        } catch (Throwable e) {
            e.printStackTrace();
        }
        return;
    }
  
  
  
  ---------DDSBaseNLU------------
        @Override
    public void start(String reason) {
        super.start(reason);
        LogUtils.d(TAG, "start");
        doStart(reason);
        resetAsrMode();
    }

    @Override
    public void stop() {
        super.stop();
        resetAsrMode();
    }
//在nlu执行完成后，重置识别模式为默认识别模式
    protected void resetAsrMode() {
        if (mIsAsrModeOffline.get()) {
            JarvisBus.instance().publish(SDKEvent.ASR_MODE, ASR_DEFAULT);
        }
        mIsAsrModeOffline.set(false);
    }
```

```java
//识别结果返回
        public void onResults(AIResult aiResult) {
               if (MODE_MIX == engineMode&&localEngine!=null) {
                    resultPicker.appendResult(jsResult, local);
                }
        }

public void appendResult(final JSONObject result, final boolean local) {
    LogUtils.d(TAG, "appendResult, text: "+result+", local: "+local);
    if (local) {
        localResult = result;
        putResultMap(localResultMarker);
    } else {
        putResultMap(cloudResultMarker);
        cloudResult = result;
    }
  //识别模式为混合
    if (MAX_RESULT <= getResultMapSize()) {
        parseResult();
    }
}
```



#### 根本原因分析

思必驰语法构建SDK使用G2P模型，该模型针对无意义英文名称构建未做优化，平均构建一个英文联系人名称大约在70ms（E28平台测试）左右，客户用户存在普遍1000+的英文联系人，所以总体构建耗时> 70*1000，超过我们设置的构建语法时间的超时阈值。

### 解决方案

#### 临时解决方案

1. Lib_speech_protocl sdk中增加在同步联系人时过滤英文联系人，不传入语法构建流程的逻辑

#### 最终解决方案

1. CarSpeechService更新思必驰1.4.5SDK，将构建联系人与构建应用名称、通话记录等语法文件分开开来，将构建联系人语法文件工作放在后台，等到构建完成再重新初始化离线ASR，失败则不影响现有离线ASR功能（P0）
2. 去掉构建语法超时的逻辑，直到语法构建回调成功或者失败(P1)
3. 去掉构建语法文件时左上角‘通讯录同步中，请稍后再唤醒我’的逻辑，不阻塞用户正常使用。(P1)
4. 小鹏提供常用英文联系人名称列表给思必驰，用于构建新版本语法构建模型，减少用户构建英文联系人耗时（P1）
5. 思必驰调研切换语法构建新引擎方案，更新语法构建SDK（P1）

### 会议结论











