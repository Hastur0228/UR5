# UR5 机器人 TCP/IP 脚本操作手册

基于 scriptManual_3.15.4.pdf 和相关技术文档整理

## 目录

1. [概述](#概述)
2. [TCP/IP 连接基础](#tcpip-连接基础)
3. [套接字通信功能](#套接字通信功能)
4. [客户端-服务器通信](#客户端-服务器通信)
5. [数据读写操作](#数据读写操作)
6. [远程控制接口](#远程控制接口)
7. [实时数据交换](#实时数据交换)
8. [错误处理与调试](#错误处理与调试)
9. [实际应用示例](#实际应用示例)
10. [最佳实践](#最佳实践)

---

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

---

## 套接字通信功能

### 1. 套接字创建与连接

#### socket_open()
```urscript
# 创建TCP套接字连接
socket_handle = socket_open(host, port, socket_name="")

# 参数说明：
# host: 目标主机IP地址 (字符串)
# port: 目标端口号 (整数)
# socket_name: 套接字名称 (可选)

# 示例：
connection = socket_open("192.168.1.50", 12345, "my_socket")
```

#### socket_close()
```urscript
# 关闭套接字连接
socket_close(socket_name="")

# 参数说明：
# socket_name: 要关闭的套接字名称

# 示例：
socket_close("my_socket")
```

### 2. 套接字状态检查

#### socket_get_var()
```urscript
# 获取套接字状态变量
status = socket_get_var(variable_name, socket_name="")

# 参数说明：
# variable_name: 变量名称
# socket_name: 套接字名称

# 示例：
is_connected = socket_get_var("connected", "my_socket")
```

---

## 客户端-服务器通信

### 1. 机器人作为客户端

#### 基本客户端连接
```urscript
# 客户端连接示例
def connect_to_server():
    host_ip = "192.168.1.100"
    host_port = 30000
    
    # 建立连接
    connection = socket_open(host_ip, host_port)
    
    if connection:
        textmsg("连接服务器成功")
        return connection
    else:
        textmsg("连接服务器失败")
        return False
    end
end
```

#### 发送请求数据
```urscript
# 向服务器发送请求
def send_request(request_data):
    if socket_send_string(request_data):
        textmsg("数据发送成功")
    else:
        textmsg("数据发送失败")
    end
end
```

### 2. 机器人作为服务器

#### 服务器设置
```urscript
# 创建服务器套接字
server_socket = socket_open("0.0.0.0", 12345)  # 监听所有接口

# 等待客户端连接
while not socket_get_var("connected", server_socket):
    sleep(0.1)
end

textmsg("客户端已连接")
```

---

## 数据读写操作

### 1. 字符串数据操作

#### socket_send_string()
```urscript
# 发送字符串数据
result = socket_send_string(string_data, socket_name="")

# 参数说明：
# string_data: 要发送的字符串
# socket_name: 套接字名称

# 示例：
socket_send_string("Hello Server", "my_socket")
```

#### socket_read_string()
```urscript
# 读取字符串数据
received_string = socket_read_string(socket_name="", timeout=2.0)

# 参数说明：
# socket_name: 套接字名称
# timeout: 超时时间（秒）

# 示例：
message = socket_read_string("my_socket", 5.0)
```

### 2. 数值数据操作

#### socket_send_int()
```urscript
# 发送整数数据
result = socket_send_int(integer_value, socket_name="")

# 示例：
socket_send_int(12345, "my_socket")
```

#### socket_read_ascii_float()
```urscript
# 读取ASCII格式的浮点数数组
float_array = socket_read_ascii_float(number_of_values, socket_name="", timeout=2.0)

# 参数说明：
# number_of_values: 期望读取的数值个数
# socket_name: 套接字名称
# timeout: 超时时间

# 示例：接收6个浮点数（位姿数据）
pose_data = socket_read_ascii_float(6, "my_socket", 3.0)
```

#### socket_send_line()
```urscript
# 发送一行数据（自动添加换行符）
result = socket_send_line(line_data, socket_name="")

# 示例：
socket_send_line("movej(waypoint_1)", "my_socket")
```

### 3. 二进制数据操作

#### socket_send_byte()
```urscript
# 发送字节数据
result = socket_send_byte(byte_value, socket_name="")

# 示例：
socket_send_byte(255, "my_socket")
```

#### socket_read_binary_integer()
```urscript
# 读取二进制整数
integer_value = socket_read_binary_integer(number_of_bytes, socket_name="", timeout=2.0)

# 参数说明：
# number_of_bytes: 字节数（1, 2, 4）
# socket_name: 套接字名称
# timeout: 超时时间

# 示例：读取4字节整数
value = socket_read_binary_integer(4, "my_socket")
```

---

## 远程控制接口

### 1. 通过端口30002发送命令

```python
# Python端发送运动命令
import socket

def send_ur_command(command):
    HOST = "192.168.1.100"
    PORT = 30002
    
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect((HOST, PORT))
    s.send((command + "\n").encode())
    s.close()

# 运动命令示例
send_ur_command("movej([-1.95, -1.58, 1.16, -1.15, -1.55, 1.18], a=1.0, v=0.1)")
send_ur_command("movel(p[0.3, 0.3, 0.4, 2.22, -2.22, 0.0], a=1.2, v=0.3)")

# I/O控制命令
send_ur_command("set_digital_out(1, True)")
send_ur_command("set_digital_out(1, False)")
```

### 2. 批量命令发送

```python
# 发送多个命令的完整程序
import socket
import time

def execute_ur_program():
    HOST = "192.168.1.100"
    PORT = 30002
    
    commands = [
        "set_digital_out(1, True)",
        "movej([-1.95, -1.58, 1.16, -1.15, -1.55, 1.18], a=1.0, v=0.1)",
        "sleep(2.0)",
        "movej([-1.95, -1.66, 1.71, -1.62, -1.56, 1.19], a=1.0, v=0.1)",
        "set_digital_out(1, False)"
    ]
    
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect((HOST, PORT))
    
    for cmd in commands:
        s.send((cmd + "\n").encode())
        time.sleep(0.1)  # 短暂延迟
    
    # 等待执行完成
    time.sleep(15)
    s.close()

execute_ur_program()
```

---

## 实时数据交换

### 1. 监听机器人状态

```python
# 通过端口30003获取实时数据
import socket
import struct

def monitor_robot_state():
    HOST = "192.168.1.100"
    PORT = 30003
    
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect((HOST, PORT))
    
    while True:
        try:
            data = s.recv(1024)
            if len(data) > 0:
                # 解析实时数据包
                parse_robot_data(data)
        except KeyboardInterrupt:
            break
    
    s.close()

def parse_robot_data(data):
    # 解析机器人状态数据
    # 具体解析格式参考客户端接口文档
    print(f"接收到数据长度: {len(data)}")
```

### 2. 位置数据获取

```urscript
# 在UR脚本中获取当前位置
current_pose = get_actual_tcp_pose()
current_joints = get_actual_joint_positions()

# 通过套接字发送位置数据
pose_string = str(current_pose)
joint_string = str(current_joints)

socket_send_string(pose_string, "data_socket")
socket_send_string(joint_string, "data_socket")
```

---

## 错误处理与调试

### 1. 连接错误处理

```urscript
# 连接重试机制
def robust_connect(host, port, max_retries=3):
    retry_count = 0
    
    while retry_count < max_retries:
        connection = socket_open(host, port)
        
        if connection:
            textmsg("连接成功")
            return connection
        else:
            retry_count = retry_count + 1
            textmsg("连接失败，重试中...")
            sleep(1.0)
        end
    end
    
    textmsg("连接失败，已达到最大重试次数")
    return False
end
```

### 2. 数据传输错误处理

```urscript
# 带错误检查的数据发送
def safe_send_data(data, socket_name):
    if socket_get_var("connected", socket_name):
        result = socket_send_string(data, socket_name)
        if not result:
            textmsg("数据发送失败")
            return False
        end
    else:
        textmsg("套接字未连接")
        return False
    end
    
    return True
end
```

### 3. 超时处理

```urscript
# 带超时的数据接收
def receive_with_timeout(socket_name, timeout_seconds):
    start_time = get_system_time()
    
    while (get_system_time() - start_time) < timeout_seconds:
        if socket_get_var("readable", socket_name):
            return socket_read_string(socket_name)
        end
        sleep(0.1)
    end
    
    textmsg("接收数据超时")
    return ""
end
```

---

## 实际应用示例

### 1. 视觉系统集成

```urscript
# 与视觉系统通信获取目标位置
def get_vision_target():
    # 连接视觉服务器
    vision_socket = socket_open("192.168.1.200", 8080)
    
    if vision_socket:
        # 请求目标位置
        socket_send_string("GET_TARGET_POSE", vision_socket)
        
        # 接收位置数据 (x, y, z, rx, ry, rz)
        pose_data = socket_read_ascii_float(6, vision_socket, 5.0)
        
        if pose_data[0] == 6:  # 检查接收到的数据个数
            target_pose = p[pose_data[1], pose_data[2], pose_data[3], 
                           pose_data[4], pose_data[5], pose_data[6]]
            
            # 移动到目标位置
            movel(target_pose, a=1.2, v=0.3)
            
            textmsg("已移动到视觉目标位置")
        else:
            textmsg("视觉数据接收错误")
        end
        
        socket_close(vision_socket)
    else:
        textmsg("无法连接视觉系统")
    end
end
```

### 2. PLC通信示例

```urscript
# 与PLC系统通信
def communicate_with_plc():
    plc_ip = "192.168.1.50"
    plc_port = 502  # Modbus TCP端口
    
    plc_socket = socket_open(plc_ip, plc_port)
    
    if plc_socket:
        # 发送启动信号
        socket_send_string("START_CYCLE", plc_socket)
        
        # 等待PLC响应
        response = socket_read_string(plc_socket, 3.0)
        
        if response == "READY":
            textmsg("PLC已准备就绪")
            return True
        else:
            textmsg("PLC响应异常")
            return False
        end
        
        socket_close(plc_socket)
    else:
        textmsg("无法连接PLC")
        return False
    end
end
```

### 3. 数据记录系统

```urscript
# 实时数据记录
thread data_logger():
    log_socket = socket_open("192.168.1.300", 9999)
    
    while True:
        # 获取当前状态
        current_pose = get_actual_tcp_pose()
        current_force = get_tcp_force()
        timestamp = get_system_time()
        
        # 构建数据包
        log_data = str(timestamp) + "," + str(current_pose) + "," + str(current_force)
        
        # 发送到日志服务器
        socket_send_line(log_data, log_socket)
        
        sleep(0.1)  # 10Hz记录频率
    end
end
```

### 4. 远程监控界面

```python
# Python端监控程序
import socket
import threading
import json

class URMonitor:
    def __init__(self, robot_ip):
        self.robot_ip = robot_ip
        self.monitoring = False
        
    def start_monitoring(self):
        self.monitoring = True
        monitor_thread = threading.Thread(target=self._monitor_loop)
        monitor_thread.start()
        
    def _monitor_loop(self):
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.connect((self.robot_ip, 30003))
        
        while self.monitoring:
            try:
                data = s.recv(1024)
                robot_state = self._parse_state_data(data)
                self._update_display(robot_state)
            except Exception as e:
                print(f"监控错误: {e}")
                break
                
        s.close()
        
    def _parse_state_data(self, data):
        # 解析机器人状态数据
        return {"timestamp": time.time(), "data_length": len(data)}
        
    def _update_display(self, state):
        print(f"机器人状态: {state}")

# 使用示例
monitor = URMonitor("192.168.1.100")
monitor.start_monitoring()
```

---

## 最佳实践

### 1. 连接管理

- **使用连接池**：对于频繁通信，维护持久连接而非每次重新连接
- **心跳检测**：定期发送心跳包检查连接状态
- **优雅断开**：程序结束时正确关闭所有套接字连接

```urscript
# 连接管理示例
global persistent_connections = {}

def get_connection(name, host, port):
    if not persistent_connections[name]:
        persistent_connections[name] = socket_open(host, port, name)
    end
    return persistent_connections[name]
end

def cleanup_connections():
    # 程序结束时清理所有连接
    socket_close("vision_socket")
    socket_close("plc_socket")
    socket_close("log_socket")
end
```

### 2. 数据格式标准化

- **使用JSON格式**：便于解析和调试
- **数据校验**：发送前验证数据格式
- **版本兼容**：考虑协议版本兼容性

```urscript
# 标准化数据格式
def send_structured_data(command, parameters, socket_name):
    data_packet = {
        "command": command,
        "parameters": parameters,
        "timestamp": get_system_time(),
        "version": "1.0"
    }
    
    json_string = to_str(data_packet)  # 转换为JSON字符串
    socket_send_string(json_string, socket_name)
end
```

### 3. 错误恢复策略

- **自动重连**：网络中断后自动尝试重连
- **数据缓存**：网络不稳定时缓存待发送数据
- **状态同步**：重连后同步系统状态

```urscript
# 自动恢复机制
def auto_recovery_send(data, socket_name, max_retries=3):
    retry_count = 0
    
    while retry_count < max_retries:
        if socket_get_var("connected", socket_name):
            if socket_send_string(data, socket_name):
                return True
            end
        else:
            # 尝试重新连接
            textmsg("尝试重新连接...")
            # 重新连接逻辑
        end
        
        retry_count = retry_count + 1
        sleep(1.0)
    end
    
    return False
end
```

### 4. 性能优化

- **批量处理**：合并多个小数据包为一个大包
- **异步处理**：使用线程处理网络通信
- **缓冲区管理**：适当设置缓冲区大小

```urscript
# 批量数据处理
def batch_send_commands(commands, socket_name):
    batch_data = ""
    
    # 合并命令
    for i in range(len(commands)):
        batch_data = batch_data + commands[i] + "\n"
    end
    
    # 一次性发送
    socket_send_string(batch_data, socket_name)
end
```

### 5. 安全考虑

- **访问控制**：限制可连接的IP地址
- **数据加密**：敏感数据传输加密
- **输入验证**：验证接收到的命令合法性

```urscript
# 简单的访问控制
allowed_ips = ["192.168.1.100", "192.168.1.200"]

def validate_connection(client_ip):
    for i in range(len(allowed_ips)):
        if client_ip == allowed_ips[i]:
            return True
        end
    end
    return False
end
```

---

## 总结

UR5机器人的TCP/IP通信功能为工业自动化提供了强大的网络集成能力。通过合理使用套接字函数和遵循最佳实践，可以构建稳定可靠的机器人网络应用系统。

### 关键要点：

1. **选择合适的端口**：根据应用需求选择对应的通信端口
2. **处理网络异常**：实现完善的错误处理和恢复机制  
3. **优化通信性能**：使用批量处理和连接复用技术
4. **确保系统安全**：实施适当的访问控制和数据验证
5. **标准化协议**：使用统一的数据格式和通信协议

通过这些TCP/IP脚本功能，UR5机器人可以无缝集成到复杂的工业4.0生产环境中，实现智能制造和远程监控等先进应用。