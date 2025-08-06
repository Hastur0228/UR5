# UR5 机器人概述和TCP/IP连接基础

基于 scriptManual_3.15.4.pdf 和相关技术文档整理

## 概述

UR5 机器人支持通过 TCP/IP 协议进行网络通信，可以实现：
- 远程控制机器人运动
- 实时数据监控
- 与外部设备通信
- 客户端-服务器架构应用

### 支持的通信端口

| 端口 | 频率 | 功能描述 | 数据方向 |
|------|------|----------|----------|
| 30001 | 10Hz | 主要客户端接口 | 发送URScript命令，接收状态数据 |
| 30002 | 10Hz | 次要客户端接口 | 发送URScript命令，接收状态数据 |
| 30003 | 125Hz/500Hz | 实时客户端接口 | 发送URScript命令，接收实时数据 |
| 30004 | 125Hz/500Hz | 实时数据接口 | 仅接收实时数据 |
| 29999 | - | 仪表板服务器 | 机器人状态控制 |

---

## TCP/IP 连接基础

### 1. 基本网络配置

```urscript
# 设置机器人IP地址（通过示教器设置）
# 默认子网掩码: 255.255.255.0
# 示例IP: 192.168.1.100
```

### 2. 网络连接测试

```python
# Python客户端连接测试
import socket

HOST = "192.168.1.100"  # 机器人IP地址
PORT = 30002            # 目标端口

try:
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect((HOST, PORT))
    print("连接成功")
    s.close()
except socket.error as e:
    print(f"连接失败: {e}")
```