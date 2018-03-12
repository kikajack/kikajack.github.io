[监控系统及zabbix介绍](https://www.jianshu.com/p/143cb6e3167c)

#1. Zabbix
[Zabbix 官网](https://www.zabbix.com)
[Zabbix 官网中文文档](https://www.zabbix.com/documentation/3.4/zh/manual/introduction/manual_structure)，zabbix 3.4 开始文档有中文的了。
[linux--监控系统之Zabbix简介](http://blog.51cto.com/584014981/1411173)
[Zabbix 和 Nagios 对比](https://www.zhihu.com/question/19973178)
[Zabbix 监控 Nginx 示例](https://yq.aliyun.com/articles/14813)
[Zabbix 维基百科](https://zh.wikipedia.org/wiki/Zabbix)

Zabbix 是由 Alexei Vladishev 开发的一种网络监视、管理系统，基于 Server-Client 架构。可用于监视各种网络服务、服务器和网络机器等状态。主要是由 Alexei Vladishev 所设立的 Zabbix SIA 做开发与维护。

Zabbix 系统由以下各种独立的模组组成，配置比较简单（基本上在 Web 上配置），高度集成开箱即用。

- Zabbix Server
- Zabbix Agent
- Zabbix Frontend（可视化界面）
- Zabbix Proxy (非必要)

Server 端与 Agent 端是以 C 语言开发，Web 管理端 Frontend 是以 PHP 及 Javascript 构成。可以使用各种数据库如 MySQL, PostgreSQL, SQLite, Oracle 或 IBM DB2 储存资料。Zabbix 可以使用多种方式监视。可以只使用 Simple Check 不需要安装 Client 端，亦可基于 SMTP 或 HTTP 等各种协定做死活监视。在客户端如 UNIX、Windows 中安装 Zabbix Agent 之后，可监视 CPU Load、网络使用状况、硬盘容量等各种状态。而就算没有安装 Agent 在监视对象中，Zabbix 也可以经由 SNMP、TCP、ICMP，利用 IPMI、SSH、telnet 对目标进行监视。另外，Zabbix 包含 XMPP 等各种 Item 警示功能。

Zabbix 的授权是属于 GPLv2，商业开源。

#2. nagios
生产环境中可以使用 nagios+cacti（图表环境搭起来可能需要耗费一些时间）。

nagios 轻量灵活，报警机制强，基于插件机制，监控相对独立，复杂监控功能可能需要使用多个插件。如果只是监控服务器/服务是否在运行，nagios 可以满足。
nagios 中，mod_gearman 插件做分布式，pnp 插件做绘图（如果觉得 pnp4nagios 太丑，可以用 grafana），nagios-api 做 restful 交互。

nagios 配置的时候不需要数据库，但在可视化的时候，一般都会从数据库里读监控数据，此时可能有问题。
#3. open-falcon
[GitHub](https://github.com/open-falcon/falcon-plus/)
[中文文档](https://book.open-falcon.org/zh_0_2/)
[博客资料](http://blog.csdn.net/mbugatti/article/details/53405574)

旧的 [GitHub](https://github.com/XiaoMi/open-falcon)
旧的 [中文文档](http://book.open-falcon.org/zh/intro/)

Open-Falcon，为前后端分离的架构，包含 backend 和 frontend 两部分。由小米运维部开源出来的互联网企业级监控系统解决方案。Open Falcon是完全用GO语言编写的，主要特性如下：

- 数据采集免配置：agent自发现、支持Plugin、主动推送模式
- 容量水平扩展：生产环境每秒50万次数据收集、告警、存储、绘图，可持续水平扩展。
- 告警策略自发现：Web界面、支持策略模板、模板继承和覆盖、多种告警方式、支持回调动作。
- 告警设置人性化：支持最大告警次数、告警级别设置、告警恢复通知、告警暂停
- 不同时段不同阈值、支持维护周期，支持告警合并。
- 历史数据高效查询：秒级返回上百个指标一年的历史数据。
- Dashboard人性化：多维度的数据展示，用户自定义Dashboard等功能。
- 架构设计高可用：整个系统无核心单点，易运维，易部署。