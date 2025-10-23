### Cron的意义
Gin项目开发中定时任务对我来说是必须的，我参与过的Gin项目也基本都使用了定时任务  

例如
1. 定时对数据库的数据进行处理(创建新数据，清理表)
2. 日志分页(每天更换日志文件，相当于文件分块)
3. 定时清理无用缓存文件(文件服务器中存在的缓存)

### 开发推荐

我基本也不会使用标准time包中提供的方法进行开发  
而我最常用的就是第三方库：**Cron/v3**  
安装方法：`go get github.com/robfig/cron/v3@v3.0.0`  

安装之后使用起来非常简单，但是功能却很强大  

1. 初始化cron实例  
   ```
   myCron := cron.New() //类型是*cron.Cron
   ```
2. 添加定时任务
   ```
   myCron.AddFunc() 
   函数实例：func (c *Cron) AddFunc(spec string, cmd func()) (EntryID, error)
   第一个参数是cron表达式,描述什么时候执行函数
   第二个参数是需要执行的操作

   e.g.
   	//发送日报
	_, err = myCron.AddFunc("5 0 * * *", func() {
		groups, err := table.ListGroup()
		if err != nil {
			logger.Log.Error("初始化groupMap失败:", err)
			return
		}
		go bangumi.SendDayAnime(groups)
	})
    这是我BOT中一个定时任务，每日的0点5分的时候去执行下面的函数发送bangumi的更新日报

    只要你想，可以为这个实例挂在任意多的定时任务，配置够就行
   ```
3. 启动定时任务
   ```
   myCron.Start()
   ```

这就完成了，定时任务就会按照你的需求在特定时间触发了
