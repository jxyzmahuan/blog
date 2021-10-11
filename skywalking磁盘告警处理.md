---
title: SkyWalking磁盘告警处理
date: 2021-10-08 14:23:13
tags:
    - SkyWalking
    - APM
    - Java
    - 链路追踪
    - 微服务治理
---

# SkyWalking磁盘告警处理

## 服务器现状

1. 磁盘总量40G

2. （20210921前）每日日志量750M左右，新微服务上线后每日1.6G

3. 日志存留31天

## 处理方案

1. 更改日志存留策略 31天改为10天

    ```
      #修改skywalking OAP server配置，然后重起OAP Server
      recordDataTTL: ${SW_CORE_RECORD_DATA_TTL:10} # Unit is day
    ```
    
2. 详细分析日志暴涨原因
   
   
   
   
   
   

