# 智娃SDK #
## SDK需求 ##
### 语义接口 ###
1. 使用智娃的接口 URL 为http://toysdk.tuling123.com/openapi/api ，请求参数参考智娃
### 语音识别接口 ###
1. 继续使用百度语音识别 参考图灵SDK 接口
### 语音朗读接口 ###
### 主动询问接口 ###
1. 参考智娃中的VolleyManager	public void startNetRequest(String cmd, int reqtype,boolean isOfficialServer)方法


### 注意事项 ###
1. 在调用语义接口的时候，初始化的时候需要获取userid，并且存放在本地根目录下的uid.txt目录，只有初始化完成的 时候才可以调用，startRequestSmartBoy 这种请求智娃语义的接口，否则提示初始化未完成。