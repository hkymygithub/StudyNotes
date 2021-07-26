# Consul配置中心持久化

### Windows环境

#### 前提条件

1. 已经安装consul，即拥有consul.exe文件。（consul下载地址：[Downloads | Consul by HashiCorp](https://www.consul.io/downloads)）

#### 创建必要文件

​	在consul.exe文件所在目录创建"consul_start.bat"脚本文件(用于将consul注册为服务)，创建"data"文件夹（用于保存数据）。

#### 编写脚本文件

```
@echo.service startup......  
@echo off  
@sc create Consul binpath= "D:\Program\consul\consul.exe agent -server -ui -bind=127.0.0.1 -client=0.0.0.0 -bootstrap-expect  1  -data-dir D:\Program\consul\data"
@net start Consul
@sc config Consul start= AUTO  
@echo.Consul starting success! 
@pause
```

（其中需要将consul.exe和data更改为相应目录）

#### 执行脚本

这里需要以管理员身份执行脚本，执行完成后可以在系统服务管理中看到consul服务，默认开机自启。
