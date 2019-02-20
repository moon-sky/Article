## 智娃Logger日志规范 ##
### 日志级别 ###
**分为5级**

1. FATAL(致命)：软件系统发生崩溃
2. ERROR(错误)：软件核心模块运转不正常，需要调试追踪具体原因，修改才能正常工作
3. WARN（警告：软件一般模块出现问题，不影响系统运行
4. INFO (通知)：此信息输出后，主要是记录系统运行状态等关联信息，例如电量，activity 生命周期等。系统配置、系统运行环境信息，有助于安装实施人员检查配置是否正确
5. DEBUG(调试)：除却上面各种情况后，你希望输出的相关信息，都可以在这里输出。主要用于调试一些想看到的信息

### 日志书写 ###
1. 日志需要简洁、只暴露必要的元素、参数
2. 正确使用对应的级别标识
3. log格式：
	1. yyyy-MM-dd HH:mm:ss:SSS	LEVEL:FATAL	INFO:class.method.info
	2. info内容如果需要呈现多个信息，则用|分割，例如hostname_www.baidu.com pingtime:300ms|hostname:sina.cn pingtime:200ms
	3. FATAL级别的info 要展示尽可能相关的参数，
	4. ERROR级别的info要展示 ERROR的类型，ERROR
### 日志输出 ###
1.  可以代码控制选择LOG信息，例如 ALL 是展示所有的log， FATAL 是只输出FATAL级别的log

