![license](https://img.shields.io/github/license/yedf/dtm)
[![Build Status](https://travis-ci.com/yedf/dtm.svg?branch=main)](https://travis-ci.com/yedf/dtm)
[![Coverage Status](https://coveralls.io/repos/github/yedf/dtm/badge.svg?branch=main)](https://coveralls.io/github/yedf/dtm?branch=main)
[![Go Report Card](https://goreportcard.com/badge/github.com/yedf/dtm)](https://goreportcard.com/report/github.com/yedf/dtm)
[![Go Reference](https://pkg.go.dev/badge/github.com/yedf/dtm.svg)](https://pkg.go.dev/github.com/yedf/dtm)

[English](https://github.com/yedf/dtm/blob/main/README-en.md)

# GO语言分布式事务管理服务

DTM是首款golang的开源分布式事务管理器，优雅的解决了幂等、空补偿、悬挂等分布式事务难题。提供了高性能和简单易用的分布式事务解决方案。

受邀参加中国数据库大会分享[多语言环境下分布式事务实践](http://dtcc.it168.com/yicheng.html#b9)

## 与其他框架对比

目前开源的分布式事务框架，暂未看到非Java语言有成熟的框架。而Java语言的较多，有阿里的SEATA、华为的ServiceComb-Pack，京东的shardingsphere，以及himly，tcc-transaction，ByteTCC等等，其中以seata应用最为广泛。

下面是dtm和seata的主要特性对比：

|  特性| DTM | SEATA |备注|
|:-----:|:----:|:----:|:----:|
| 支持语言 |  <font color=green>Golang、python、php及其他</font> | <font color=orange>Java</font> |dtm可轻松接入一门新语言|
|异常处理| <font color=green>[子事务屏障自动处理](https://zhuanlan.zhihu.com/p/388444465)</font>|<font color=orange>手动处理</font> |dtm解决了幂等、悬挂、空补偿|
| TCC事务| <font color=green>✓</font>|<font color=green>✓</font>||
| XA事务|<font color=green>✓</font>|<font color=green>✓</font>||
|AT事务|<font color=red>✗</font>|<font color=green>✓</font>|AT与XA类似，性能更好|
| SAGA事务 |  <font color=orange>简单模式</font> |  <font color=green>状态机复杂模式</font> |dtm的状态机模式在规划中|
|事务消息|<font color=green>✓</font>|<font color=red>✗</font>|dtm提供类似rocketmq的事务消息|
|通信协议|HTTP|dubbo等协议，无HTTP|dtm后续将支持grpc类协议|
|star数量|<img src="https://img.shields.io/github/stars/yedf/dtm.svg?style=social" alt="github stars"/>|<img src="https://img.shields.io/github/stars/seata/seata.svg?style=social" alt="github stars"/>|dtm从20210604发布0.1，发展快|

从上面对比的特性来看，如果您的语言栈包含了Java之外的语言，那么dtm是您的首选。如果您的语言栈是Java，您也可以选择接入dtm，使用子事务屏障技术，简化您的业务编写。

## [各语言客户端及示例](./doc/sdk.md)

## 快速开始

### 安装

`git clone https://github.com/yedf/dtm`

### dtm依赖于mysql

配置mysql：

`cp conf.sample.yml conf.yml # 修改conf.yml`

### 启动并运行saga示例
`go run app/main.go saga`

## 开始使用

### 使用
``` GO
  // 具体业务微服务地址
  const qsBusi = "http://localhost:8081/api/busi_saga"
	req := &gin.H{"amount": 30} // 微服务的载荷
	// DtmServer为DTM服务的地址，是一个url
	saga := dtmcli.NewSaga("http://localhost:8080/api/dtmsvr").
		// 添加一个TransOut的子事务，正向操作为url: qsBusi+"/TransOut"， 补偿操作为url: qsBusi+"/TransOutCompensate"
		Add(qsBusi+"/TransOut", qsBusi+"/TransOutCompensate", req).
		// 添加一个TransIn的子事务，正向操作为url: qsBusi+"/TransOut"， 补偿操作为url: qsBusi+"/TransInCompensate"
		Add(qsBusi+"/TransIn", qsBusi+"/TransInCompensate", req)
	// 提交saga事务，dtm会完成所有的子事务/回滚所有的子事务
  err := saga.Submit()
```

成功运行后，可以看到TransOut、TransIn依次被调用，完成了整个分布式事务

### 时序图

上述saga分布式事务的时序图如下：

<img src="https://pic3.zhimg.com/80/v2-b7d98659093c399e182a0173a8e549ca_1440w.jpg" height=428 />

### 完整示例
参考[examples/quick_start.go](./examples/quick_start.go)

## 文档与介绍(更新中)
  * [分布式事务简介](https://zhuanlan.zhihu.com/p/387487859)
  * 分布式事务模式
    - [XA事务模式](https://zhuanlan.zhihu.com/p/384756957)
    - [SAGA事务模式](https://zhuanlan.zhihu.com/p/385594256)
    - [TCC事务模式](https://zhuanlan.zhihu.com/p/388357329)
    - 可靠消息事务模式
  * [子事务屏障](https://zhuanlan.zhihu.com/p/388444465)
  * [通信协议](./doc/protocol.md)
  * [FAQ](https://github.com/yedf/dtm/issues/1)
  * 部署指南

## 交流群
请加 yedf2008 好友或者扫码加好友，验证回复 dtm 按照指引进群

![yedf2008](http://service.ivydad.com/cover/dubbingb6b5e2c0-2d2a-cd59-f7c5-c6b90aceb6f1.jpeg)

如果您觉得此项目不错，或者对您有帮助，请赏颗星吧！

## 谁在使用
<div style='vertical-align: middle'>
    <img alt='常青藤爸爸' height='40'  src='https://www.ivydad.com/_nuxt/img/header-logo.2645ad5.png'  /img>
    <img alt='镜小二' height='40'  src='https://img.epeijing.cn/official-website/assets/logo.png'  /img>
</div>
