## 解决Android打包过程中出现的slf4j依赖冲突问题

### 背景交代：

昨天同事说xxx项目Jenkins构建失败，并给我发了一个构建失败的日志文件

```groovy
2021-12-21T10:03:29.090Z]   java.lang.RuntimeException: Duplicate class org.slf4j.ILoggerFactory found in modules lib_xxxx-0.1.xxx-xxx-SNAPSHOT-runtime.jar (com.xxxx.lib:lib_xxxx:0.1.xxx-SNAPSHOT:20211221.094703-1) and slf4j-api-1.7.21.jar (org.slf4j:slf4j-api:1.7.21)
[2021-12-21T10:03:29.090Z]   Duplicate class org.slf4j.IMarkerFactory found in modules lib_xxxx-0.1.xxx-xxx-SNAPSHOT-runtime.jar (com.xxxx.lib:lib_xxxx:0.1.xxx-SNAPSHOT:20211221.094703-1) and slf4j-api-1.7.21.jar (org.slf4j:slf4j-api:1.7.21)
[2021-12-21T10:03:29.090Z]   Duplicate class org.slf4j.Logger found in modules lib_xxxx-0.1.xxx-xxx-SNAPSHOT-runtime.jar (com.xxxx.lib:lib_xxxx:0.1.xxx-SNAPSHOT:20211221.094703-1) and slf4j-api-1.7.21.jar (org.slf4j:slf4j-api:1.7.21)
[2021-12-21T10:03:29.090Z]   Duplicate class org.slf4j.LoggerFactory found in modules lib_xxxx-0.1.xxx-xxx-SNAPSHOT-runtime.jar (com.xxxx.lib:lib_xxxx:0.1.xxx-SNAPSHOT:20211221.094703-1) and slf4j-api-1.7.21.jar (org.slf4j:slf4j-api:1.7.21)
[2021-12-21T10:03:29.090Z]   Duplicate class org.slf4j.MDC found in modules lib_xxxx-0.1.xxx-xxx-SNAPSHOT-runtime.jar (com.xxxx.lib:lib_xxxx:0.1.xxx-SNAPSHOT:20211221.094703-1) and slf4j-api-1.7.21.jar (org.slf4j:slf4j-api:1.7.21)
[2021-12-21T10:03:29.091Z]   Duplicate class org.slf4j.MDC$1 found in modules lib_xxxx-0.1.xxx-xxx-SNAPSHOT-runtime.jar (com.xxxx.lib:lib_xxxx:0.1.xxx-SNAPSHOT:20211221.094703-1) and slf4j-api-1.7.21.jar (org.slf4j:slf4j-api:1.7.21)
[2021-12-21T10:03:29.091Z]   Duplicate class org.slf4j.MDC$MDCCloseable found in modules lib_xxxx-0.1.xxx-xxx-SNAPSHOT-runtime.jar (com.xxxx.lib:lib_xxxx:0.1.xxx-SNAPSHOT:20211221.094703-1) and slf4j-api-1.7.21.jar (org.slf4j:slf4j-api:1.7.21)
[2021-12-21T10:03:29.091Z]   Duplicate class org.slf4j.Marker found in modules lib_xxxx-0.1.xxx-xxx-SNAPSHOT-runtime.jar (com.xxxx.lib:lib_xxxx:0.1.xxx-SNAPSHOT:20211221.094703-1) and slf4j-api-1.7.21.jar (org.slf4j:slf4j-api:1.7.21)
[2021-12-21T10:03:29.091Z]   Duplicate class org.slf4j.MarkerFactory found in modules lib_xxxx-0.1.xxx-xxx-SNAPSHOT-runtime.jar (com.xxxx.lib:lib_xxxx:0.1.xxx-SNAPSHOT:20211221.094703-1) and slf4j-api-1.7.21.jar (org.slf4j:slf4j-api:1.7.21)
[2021-12-21T10:03:29.091Z]   Duplicate class org.slf4j.event.EventConstants found in modules lib_xxxx-0.1.xxx-xxx-SNAPSHOT-runtime.jar (com.xxxx.lib:lib_xxxx:0.1.xxx-SNAPSHOT:20211221.094703-1) and slf4j-api-1.7.21.jar (org.slf4j:slf4j-api:1.7.21)
[2021-12-21T10:03:29.091Z]   Duplicate class org.slf4j.event.EventRecodingLogger found in modules lib_xxxx-0.1.xxx-xxx-SNAPSHOT-runtime.jar (com.xxxx.lib:lib_xxxx:0.1.xxx-SNAPSHOT:20211221.094703-1) and slf4j-api-1.7.21.jar (org.slf4j:slf4j-api:1.7.21)
[2021-12-21T10:03:29.091Z]   Duplicate class org.slf4j.event.Level found in modules lib_xxxx-0.1.xxx-xxx-SNAPSHOT-runtime.jar (com.xxxx.lib:lib_xxxx:0.1.xxx-SNAPSHOT:20211221.094703-1) and slf4j-api-1.7.21.jar (org.slf4j:slf4j-api:1.7.21)
[2021-12-21T10:03:29.091Z]   Duplicate class org.slf4j.event.LoggingEvent found in modules lib_xxxx-0.1.xxx-xxx-SNAPSHOT-runtime.jar (com.xxxx.lib:lib_xxxx:0.1.xxx-SNAPSHOT:20211221.094703-1) and slf4j-api-1.7.21.jar (org.slf4j:slf4j-api:1.7.21)
[2021-12-21T10:03:29.091Z]   Duplicate class org.slf4j.event.SubstituteLoggingEvent found in modules lib_xxxx-0.1.xxx-xxx-SNAPSHOT-runtime.jar (com.xxxx.lib:lib_xxxx:0.1.xxx-SNAPSHOT:20211221.094703-1) and slf4j-api-1.7.21.jar (org.slf4j:slf4j-api:1.7.21)
[2021-12-21T10:03:29.091Z]   Duplicate class org.slf4j.helpers.BasicMDCAdapter found in modules lib_xxxx-0.1.xxx-xxx-SNAPSHOT-runtime.jar (com.xxxx.lib:lib_xxxx:0.1.xxx-SNAPSHOT:20211221.094703-1) and slf4j-api-1.7.21.jar (org.slf4j:slf4j-api:1.7.21)
[2021-12-21T10:03:29.091Z]   Duplicate class org.slf4j.helpers.BasicMDCAdapter$1 found in modules lib_xxxx-0.1.xxx-xxx-SNAPSHOT-runtime.jar (com.xxxx.lib:lib_xxxx:0.1.xxx-SNAPSHOT:20211221.094703-1) and slf4j-api-1.7.21.jar (org.slf4j:slf4j-api:1.7.21)
[2021-12-21T10:03:29.091Z]   Duplicate class org.slf4j.helpers.BasicMarker found in modules lib_xxxx-0.1.xxx-xxx-SNAPSHOT-runtime.jar (com.xxxx.lib:lib_xxxx:0.1.xxx-SNAPSHOT:20211221.094703-1) and slf4j-api-1.7.21.jar (org.slf4j:slf4j-api:1.7.21)
[2021-12-21T10:03:29.091Z]   Duplicate class org.slf4j.helpers.BasicMarkerFactory found in modules lib_xxxx-0.1.xxx-xxx-SNAPSHOT-runtime.jar (com.xxxx.lib:lib_xxxx:0.1.xxx-SNAPSHOT:20211221.094703-1) and slf4j-api-1.7.21.jar (org.slf4j:slf4j-api:1.7.21)
[2021-12-21T10:03:29.091Z]   Duplicate class org.slf4j.helpers.FormattingTuple found in modules lib_xxxx-0.1.xxx-xxx-SNAPSHOT-runtime.jar (com.xxxx.lib:lib_xxxx:0.1.xxx-SNAPSHOT:20211221.094703-1) and slf4j-api-1.7.21.jar (org.slf4j:slf4j-api:1.7.21)
[2021-12-21T10:03:29.091Z]   Duplicate class org.slf4j.helpers.MarkerIgnoringBase found in modules lib_xxxx-0.1.xxx-xxx-SNAPSHOT-runtime.jar (com.xxxx.lib:lib_xxxx:0.1.xxx-SNAPSHOT:20211221.094703-1) and slf4j-api-1.7.21.jar (org.slf4j:slf4j-api:1.7.21)
[2021-12-21T10:03:29.091Z]   Duplicate class org.slf4j.helpers.MessageFormatter found in modules lib_xxxx-0.1.xxx-xxx-SNAPSHOT-runtime.jar (com.xxxx.lib:lib_xxxx:0.1.xxx-SNAPSHOT:20211221.094703-1) and slf4j-api-1.7.21.jar (org.slf4j:slf4j-api:1.7.21)
[2021-12-21T10:03:29.091Z]   Duplicate class org.slf4j.helpers.NOPLogger found in modules lib_xxxx-0.1.xxx-xxx-SNAPSHOT-runtime.jar (com.xxxx.lib:lib_xxxx:0.1.xxx-SNAPSHOT:20211221.094703-1) and slf4j-api-1.7.21.jar (org.slf4j:slf4j-api:1.7.21)
[2021-12-21T10:03:29.091Z]   Duplicate class org.slf4j.helpers.NOPLoggerFactory found in modules lib_xxxx-0.1.xxx-xxx-SNAPSHOT-runtime.jar (com.xxxx.lib:lib_xxxx:0.1.xxx-SNAPSHOT:20211221.094703-1) and slf4j-api-1.7.21.jar (org.slf4j:slf4j-api:1.7.21)
[2021-12-21T10:03:29.091Z]   Duplicate class org.slf4j.helpers.NOPMDCAdapter found in modules lib_xxxx-0.1.xxx-xxx-SNAPSHOT-runtime.jar (com.xxxx.lib:lib_xxxx:0.1.xxx-SNAPSHOT:20211221.094703-1) and slf4j-api-1.7.21.jar (org.slf4j:slf4j-api:1.7.21)
[2021-12-21T10:03:29.091Z]   Duplicate class org.slf4j.helpers.NamedLoggerBase found in modules lib_xxxx-0.1.xxx-xxx-SNAPSHOT-runtime.jar (com.xxxx.lib:lib_xxxx:0.1.xxx-SNAPSHOT:20211221.094703-1) and slf4j-api-1.7.21.jar (org.slf4j:slf4j-api:1.7.21)
[2021-12-21T10:03:29.091Z]   Duplicate class org.slf4j.helpers.SubstituteLogger found in modules lib_xxxx-0.1.xxx-xxx-SNAPSHOT-runtime.jar (com.xxxx.lib:lib_xxxx:0.1.xxx-SNAPSHOT:20211221.094703-1) and slf4j-api-1.7.21.jar (org.slf4j:slf4j-api:1.7.21)
[2021-12-21T10:03:29.091Z]   Duplicate class org.slf4j.helpers.SubstituteLoggerFactory found in modules lib_xxxx-0.1.xxx-xxx-SNAPSHOT-runtime.jar (com.xxxx.lib:lib_xxxx:0.1.xxx-SNAPSHOT:20211221.094703-1) and slf4j-api-1.7.21.jar (org.slf4j:slf4j-api:1.7.21)
[2021-12-21T10:03:29.091Z]   Duplicate class org.slf4j.helpers.Util found in modules lib_xxxx-0.1.xxx-xxx-SNAPSHOT-runtime.jar (com.xxxx.lib:lib_xxxx:0.1.xxx-SNAPSHOT:20211221.094703-1) and slf4j-api-1.7.21.jar (org.slf4j:slf4j-api:1.7.21)
[2021-12-21T10:03:29.091Z]   Duplicate class org.slf4j.helpers.Util$1 found in modules lib_xxxx-0.1.xxx-xxx-SNAPSHOT-runtime.jar (com.xxxx.lib:lib_xxxx:0.1.xxx-SNAPSHOT:20211221.094703-1) and slf4j-api-1.7.21.jar (org.slf4j:slf4j-api:1.7.21)
[2021-12-21T10:03:29.091Z]   Duplicate class org.slf4j.helpers.Util$ClassContextSecurityManager found in modules lib_xxxx-0.1.xxx-xxx-SNAPSHOT-runtime.jar (com.xxxx.lib:lib_xxxx:0.1.xxx-SNAPSHOT:20211221.094703-1) and slf4j-api-1.7.21.jar (org.slf4j:slf4j-api:1.7.21)
[2021-12-21T10:03:29.091Z]   Duplicate class org.slf4j.spi.LocationAwareLogger found in modules lib_xxxx-0.1.xxx-xxx-SNAPSHOT-runtime.jar (com.xxxx.lib:lib_xxxx:0.1.xxx-SNAPSHOT:20211221.094703-1) and slf4j-api-1.7.21.jar (org.slf4j:slf4j-api:1.7.21)
[2021-12-21T10:03:29.091Z]   Duplicate class org.slf4j.spi.LoggerFactoryBinder found in modules lib_xxxx-0.1.xxx-xxx-SNAPSHOT-runtime.jar (com.xxxx.lib:lib_xxxx:0.1.xxx-SNAPSHOT:20211221.094703-1) and slf4j-api-1.7.21.jar (org.slf4j:slf4j-api:1.7.21)
[2021-12-21T10:03:29.091Z]   Duplicate class org.slf4j.spi.MDCAdapter found in modules lib_xxxx-0.1.xxx-xxx-SNAPSHOT-runtime.jar (com.xxxx.lib:lib_xxxx:0.1.xxx-SNAPSHOT:20211221.094703-1) and slf4j-api-1.7.21.jar (org.slf4j:slf4j-api:1.7.21)
[2021-12-21T10:03:29.091Z]   Duplicate class org.slf4j.spi.MarkerFactoryBinder found in modules lib_xxxx-0.1.xxx-xxx-SNAPSHOT-runtime.jar (com.xxxx.lib:lib_xxxx:0.1.xxx-SNAPSHOT:20211221.094703-1) and slf4j-api-1.7.21.jar (org.slf4j:slf4j-api:1.7.21)
```

lib_xxxx-0.1.xxx-xxx-SNAPSHOT-runtime.jar与slf4j-api-1.7.21.jar中都引入了以上这些类，导致出现冲突。

### 开始分析

#### 找到罪魁祸首

1. 通过查看最近的lib库提交记录，果然发现昨天在lib库中增加了slf4j的包![image-20211222115643726](https://gitee.com/moonsky/image-bed/raw/master/image-20211222115643726.png)

2. 接着找到第二个包，在打包的apk项目根目录下，使用命令

   ```shell
   ./gradlew :***:***:dependencies
   ```

   在一个danikula的库下发现了其踪迹

   ![image-20211222115959324](https://gitee.com/moonsky/image-bed/raw/master/image-20211222115959324.png)

#### 制定解决方案

这种问题解决方案一般有两种策略

1. 两方引用该库时统一版本号，比如都用比较新的1.7.25，但是由于danikula属于第三方库，并且已经三年不更新了，要改也是我们自己这边改成1.7.21

2. 打包时移除指定引用，该方案也是我们最终采用的方案

   ```groovy
       implementation (rootProject.ext.dependencies["videocache"]){
           exclude group: 'org.slf4j'
       }
   ```

当然，除了以上方案，还有强制使用一个固定版本等，

参考文章：

https://www.zhyea.com/2018/02/08/gradle-exclude-dependencies.html

https://tomgregory.com/how-to-exclude-gradle-dependencies/