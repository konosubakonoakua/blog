---
number: 176
title: "[zynq] interrupt on zynq"
state: open
url: https://github.com/konosubakonoakua/blog/issues/176
author: konosubakonoakua
createdAt: 2025-05-01T05:25:05Z
updatedAt: 2025-05-09T08:55:42Z
labels: []
---

# 176 [zynq] interrupt on zynq

## uio
- https://yurovsky.github.io/2014/10/10/linux-uio-gpio-interrupt.html
- https://spacewire.co.uk/tutorial/leds_switches_add_buttons_int/
- https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18842490/Testing+UIO+with+Interrupt+on+Zynq+Ultrascale
## fasync
- diagram
  ```mermaid
  sequenceDiagram
      participant 用户进程
      participant 内核
      participant 驱动
  
      Note over 用户进程,驱动: 初始化阶段
      用户进程->>内核: 1. fcntl(F_SETOWN, pid)
      用户进程->>内核: 2. fcntl(F_SETFL, FASYNC)
      内核->>驱动: 3. 调用.fasync()
      驱动->>内核: 4. fasync_helper()注册回调
  
      Note over 用户进程,驱动: 数据就绪阶段
      驱动->>驱动: 5. 设备数据到达(中断/轮询)
      驱动->>内核: 6. kill_fasync(SIGIO, POLL_IN)
      内核->>用户进程: 7. 发送SIGIO信号
  
      Note over 用户进程: 响应阶段
      用户进程->>驱动: 8. read()读取数据
      驱动->>用户进程: 9. 返回数据
  ```
- https://blog.csdn.net/qq_42330920/article/details/119911517
- https://www.cnblogs.com/yuanqiangfei/p/16145657.html
- https://www.cnblogs.com/physworld/p/14839320.html
## others
- https://jvns.ca/blog/2017/06/03/async-io-on-linux--select--poll--and-epoll/
- https://www.zynqnotes.com/linux-irq-mapping
- https://www.zynqnotes.com/pl-ps-interrupt-1
- https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18842020/Cascade+Interrupt+Controller+support+in+DTG
- https://ax7010-20231-v101.readthedocs.io/zh-cn/latest/7010_S6_RSTdocument_CN/8_%E4%B8%AD%E6%96%AD_CN.html

---

## Comments

### konosubakonoakua · 2025-05-08T07:20:47Z

## axi gpio regs for interrupt

<details>

![Image](../assets/176/1.png)

![Image](../assets/176/2.png)

![Image](../assets/176/3.png)

</details>

### konosubakonoakua · 2025-05-09T08:46:00Z

## realdigital
- [Project 5 Using Interrupts](https://www.realdigital.org/doc/f37660e196405e98d115d8bf56806ea4)
- [Setup and use of Blackboard's GPIO devices](https://www.realdigital.org/doc/25bc9b4ca6fbcb20e44a0ae17631943a)
- [Setting up and using ARM's Generic Interrupt Controller](https://www.realdigital.org/doc/87be521204ed2c4447d567b666961ce8)
