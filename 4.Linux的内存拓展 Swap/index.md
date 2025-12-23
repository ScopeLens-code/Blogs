### Swap的意义

Swap本质上就是将硬盘空间暂时当作内存来使用,所以弊端也很明显就是慢,不过能够短暂提供ram给提供,防止oom已经是巨大的提升了,毕竟走得慢总比不走要好.

### 创建Swap

#### 1.查看自己的Swap空间

查看Linux内存:`free -h`

#### 2.计算所需Swap空间
   
经验法则(来自网络):  

|RAM|Swap|
|---|----|
|RAM<2GB|Swap=RAM*2|
|2GB<=RAM<=8GB|Swap=RAM|
|RAM>8GB|4GB<=Swap<=8GB|

#### 3.创建Swap文件

```
# 创建一个 4GB 的 Swap 文件
sudo fallocate -l 4G /swapfile
```

#### 4.设置文件权限

```
# 设置文件权限为 600
sudo chmod 600 /swapfile
```

#### 5.设置Swap区域

```
# 格式化文件为 Swap 区域
sudo mkswap /swapfile
```

#### 6.启用Swap文件

```
# 激活 Swap
sudo swapon /swapfile
```

#### 7.验证结果与设置永久生效

```
free -h
```

在`/etc/fstab`文件末尾加上`/swapfile none swap sw 0 0`

#### 8.优化Swappniess参数

##### 1.临时设置

```
# 临时将 swappiness 设置为 10
sudo sysctl vm.swappiness=10
```

##### 2.永久设置

编辑系统配置`/etc/sysctl.conf`,在文件末尾加上`vm.swappiness = 10`  
保存后运行`sudo sysctl -p`