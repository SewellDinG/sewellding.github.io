---
layout: post
title: Windows网络认证机制（NTLMv2、Kerberos）分析
description: ""
keywords: ""
---

## 认证机制

本地认证、网络认证（工作组环境：NTLMv2协议）

![NTLMv2](/assets/images/2020-04-02/NTLMv2.jpg)

网络认证（域环境：Kerberos协议）

![Kerberos](/assets/images/2020-04-02/Kerberos.jpg)

## 抓包分析

NTLMv2：远程登录SMB共享目录（从本机抓取）

[NTLMv2_SMB2.pcap](/assets/images/2020-04-02/NTLMv2_SMB2.pcap)

Kerberos：登陆域内主机（从域控抓取）

[Kerberos_logon.pcap](/assets/images/2020-04-02/Kerberos_logon.pcap)