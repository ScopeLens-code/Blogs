### Log的意义
在WEB项目中Log是程序中至关重要的模块  
它的作用主要体现在三点：  
1. 开发时调试
2. 程序出错时溯源(关键)
3. 记录WEB请求时留痕  

我在开发中已经离不开日志对我的帮助了  
我之前为我自己做了一个群聊BOT的小玩意，用来活跃群聊，其中包含了一些小指令和AI聊天功能，如果没有日志，我就难以去修复我对于指令的逻辑错误，以及AI聊天中上下文构建的缺陷，光是一个小项目日志都如此有用，自然不用说对于企业级的项目，更需要日志去了解自己的用户群体，针对性的优化自己的产品。  

而我在开发中一般是不会使用标准库提供的log的，太过于简陋，本着不重复造轮子的原则，自然是使用*logrus*这个更加专业的日志库  

### 为什么使用logrus库
为什么说logrus更加专业一些，我们做一个简单的对比，来展示它的优点

| 对比维度    | Gin 自带日志                      | 	Logrus                                        |
|---------|-------------------------------|------------------------------------------------|
| 日志等级    | 无等级概念                         | 	支持 Debug, Info, Warn, Error, Fatal, Panic等多级别 |
| 结构化日志   | 输出纯文本                         | 	支持 JSON、Text、Hook格式化输出                        |
| 日志分层    | 只有全局 DefaultWriter            | 	可以创建多个logger实例，分别用于业务、访问、系统模块等                |
| 上下文信息   | 仅限于请求信息	                      | 可通过WithFields()自定义上下文字段（如用户ID、模块名）             |
| 输出控制    | 只能通过DefaultWriter或io.Writer控制 | 支持文件切割（通过lumberjack等）、多输出目标                    |
| 性能与异步写入 | 同步阻塞输出                        | 可通过第三方库或 Hook 实现异步、批量写入                        |

从以上的几点中可以看出，自带的日志确实不适合生产级的项目，特别是不能结构化日志，性能问题。

### 简单的使用
#### 1. 日志文件切换  

日志如果只是输入到一份log文件中，必然会导致文件过大，难以打开，阅读查找，所以第一步就是要实现日志自动切换文件，分块的依据有很多，比如判断文件最大大小，超过就切换文件；创建定时任务；根据业务种类区分文件。  

我这里采用的是定时任务的方式，我个人比较推荐**先根据业务种类区分文件，然后设置定时任务去切换文件**，这样架构更清晰，我这是抛砖引玉就不说那么多了。
```
//如果有多个业务就创建多个实例
var Log *logrus.Logger
var logFile *os.File

// Init 初始化 定时更新日志
func Init(myCron *cron.Cron) { //我这里使用Cron包
	//定时任务： 打开/创建 日志文件
	_, err := myCron.AddFunc("0 0 * * *", func() {
		Log.Info("每天凌晨 0 点执行任务，当前时间:", time.Now())
		// 切换日志文件
		Log.Info("切换日志文件...")
		CloseLogger()
		if err := SetupLogger("pkg/logger/"); err != nil {
			Log.Fatalf("日志启动失败: %v", err)
		}
	})
	if err != nil {
		return
	}

	// 初始化日志
	if err := SetupLogger("pkg/logger/"); err != nil {//设置日志文件目录
		Log.Fatalf("日志启动失败: %v", err)
	}
}

// SetupLogger 设置日志输出
func SetupLogger(filePath string) error {

	Log = logrus.New()
	//获取当前日期用于创建日志文件名
	currentDate := time.Now().Format("2006-01-02")
	fileName := fmt.Sprintf(filePath+"logs/handle-%s.log", currentDate)

	//打开或者创建文件
	var err error
	logFile, err = os.OpenFile(fileName, os.O_APPEND|os.O_CREATE|os.O_WRONLY, 0666)
	if err != nil {
		Log.Fatalf("打开日志文件失败：%v", err)
	}

	//设置输出和格式
	Log.SetOutput(logFile)
	Log.SetFormatter(&logrus.JSONFormatter{})

	return nil
}

// CloseLogger 关闭日志文件
func CloseLogger() {
	if logFile != nil {
		err := logFile.Close()
		if err != nil {
			return
		}
	}
}
```
以上就是日志实例的初始化模块

#### 2. 简单使用
```
// 发送请求
logger.Log.WithField("请求体", config).Info("发送请求")
//日志文件中效果： {"level":"info","msg":"发送请求","time":"2025-09-30T12:50:44+08:00","请求体":{"Method":"POST","Url":"http://xxxxx/send_private_msg","Header":null,"Params":null,"Body":{"message":[{"type":"text","data":{}}],"user_id":XXX}}}

// 记录路由记录
logger.Log.Infof("匹配消息路由：{%v}", route.Command)
...
```
这里两种分别对应了使用了存入结构化数据到日志中和直接打出文本，这在记录api请求的参数和记录结果尤为有用，还可以为日志分级，例如info，error等等  
在你使用loki这种日志系统中，定位问题更加迅速准确。

### 小结
没有日志的程序我只能称之为脚本（爆论）