# HMonitor

## 简介

HMonitor是一个用于从Zabbix处接收告警后，进行告警管理、告警自动化处理等功能的一个工具平台。目前提供的功能包括：

1. 查看告警事件。使用者可以在“事件查看”标签下查看自己订阅的告警以及所有告警，并且了解有哪些告警需要自己处理
2. 告警通知的订阅管理。在“订阅管理”标签下使用者可以订阅告警，对告警进行屏蔽，并查看一段时间内的告警情况
3. 告警的自动修复。在“自动修复”标签下使用者可以查看目前提供的自动修复脚本，并且可以将告警事件和对应的修复脚本绑定，绑定后后续的告警会首先由修复脚本尝试修复，修复失败后才会告警。同时在该标签页下可以查看修复的成功率以及修复失败的原因。

## 系统架构

![image](https://github.com/QthCN/hmonitor/blob/master/docs/images/framework.jpg)

## 安装

1. 在Zabbix处配置script类型的媒体类型，脚本设置为HMonitor源码目录下的scripts/zabbix_hm.py
2. 配置trigger，trigger名需要以HM-打头
3. 配置相应的接收告警用户，用户的send to配置为HMonitor的地址
4. 在某台主机安装MySQL
5. 执行HMonitor源码目录下的db.sql建立对应的表
6. 执行hmonitor.py即可运行HMonitor/AutoFixer
7. 执行hmonitor_agent.py即可运行HMonitor Agent

HMonitor/AutoFixer属于无状态服务，如果集群中告警或需要自动处理的主机较多，可以启动多个HMonitor/AutoFixer并将他们放在如nginx之类的LB后面。

## 使用

### 订阅管理

点击*订阅告警*，即可查看所有可以被订阅的告警。如果对某个告警赶兴趣则可以在操作栏下点击*订阅告警*，如果不想再次接收这个告警则可以点击*取消订阅*。

![image](https://github.com/QthCN/hmonitor/blob/master/docs/images/subscribe_alerts.jpg)

订阅告警后，可以在*我的订阅*下面查看自己订阅的告警。不同级别的告警被不同颜色标记。

![image](https://github.com/QthCN/hmonitor/blob/master/docs/images/view_alerts.jpg)

如果某端时间某个主机在做类似扩容的操作，期间会触发很多IO wait的告警，则可以在这段时间内屏蔽这些告警。屏蔽告警可以在*告警屏蔽*中进行。

![image](https://github.com/QthCN/hmonitor/blob/master/docs/images/filter_alerts.jpg)

另外，可以在*告警统计*下查看最近一段时间的告警发送情况。

![image](https://github.com/QthCN/hmonitor/blob/master/docs/images/show_alerts.jpg)

### 事件查看

在订阅了告警后，就可以看到自己订阅的告警产生的事件了。这个在*我的事件*中可以查看。

![image](https://github.com/QthCN/hmonitor/blob/master/docs/images/my_events.jpg )

如果想查看所有事件，可以在*所有事件*中查看。

![image](https://github.com/QthCN/hmonitor/blob/master/docs/images/all_events.jpg )

### 告警自动化处理（告警自动修复）

在HMonitor中，可以预先上传一些自动修复的脚本（这个下面会专门讲到）。上传后可以在*修复脚本*中查看这些脚本。

![image](https://github.com/QthCN/hmonitor/blob/master/docs/images/autofix_list.jpg)

告警项需要和修复脚本关联后，相应的告警才能被脚本自动处理。关联操作可以在*绑定关系*中进行。

![image](https://github.com/QthCN/hmonitor/blob/master/docs/images/bind_autofix.jpg)

可以在*修复统计*中查看修复情况。

![image](https://github.com/QthCN/hmonitor/blob/master/docs/images/show_autofix.jpg)

## 告警自动化处理

所有的告警自动化处理的脚本都是Python格式的，在HMonitor启动的时候，会的去hmonitor/autofix/scripts中加载所有的.py文件。下面是一个样例文件。

```python
# -*- coding: utf-8 -*-
from hmonitor.autofix.scripts import AutoFixBase


class JustShowEventInfo(AutoFixBase):

    def do_fix(self, trigger_name, hostname, executor, event, *args, **kwargs):
        raise Exception("ERROR TEST")

    def get_author(self):
        return "Qin TianHuan"

    def get_version(self):
        return "1"

    def get_description(self):
        return u"测试用脚本"

    def get_create_date(self):
        return "2015-06-30 09:00:00"
```

