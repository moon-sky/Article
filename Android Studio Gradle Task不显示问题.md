## Android Studio Gradle Task不显示问题

今天打开AS,准备通过gradlew xxxtask来编一个apk版本，但是我想不起来名字了，于是准备去侧边栏的Gradle找一下task名称，谁知居然没有了

![image-20210728110620673](https://gitee.com/moonsky/image-bed/raw/master/image-20210728110620673.png)并且告诉我task list is not built during sync

Google了一下，原来得知这是AS故意为之

> Gradle task list is large and slow to populate in Android projects. This feature by default is disabled for performance reasons. You can re-enable it in: `Settings | Experimental | Do not build Gradle task list during Gradle sync`.



