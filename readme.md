# UR5 机器人 TCP/IP 脚本操作手册

基于 scriptManual_3.15.4.pdf 和相关技术文档整理

## 目录

1. [概述](#概述)
2. [TCP/IP 连接基础](#tcpip-连接基础)
3. [套接字通信功能](#套接字通信功能)
4. [客户端-服务器通信](#客户端-服务器通信)
5. [数据读写操作](#数据读写操作)
6. [远程控制接口](#远程控制接口)
7. [UR Script 命令参考](#ur-script-命令参考)
8. [实时数据交换](#实时数据交换)
9. [错误处理与调试](#错误处理与调试)
10. [实际应用示例](#实际应用示例)
11. [最佳实践](#最佳实践)

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

## UR Script 命令参考

### 1. 运动控制命令

#### 1.1 关节空间运动

##### movej()
```urscript
# 关节空间运动 - 机器人在关节空间中移动到目标位置
movej(q, a=1.4, v=1.05, t=0, r=0)

# 参数说明：
# q: 目标关节位置 [q1, q2, q3, q4, q5, q6] (弧度)
# a: 关节加速度 (弧度/秒²)，默认 1.4
# v: 关节速度 (弧度/秒)，默认 1.05  
# t: 运动时间 (秒)，0表示不使用时间控制
# r: 混合半径 (米)，用于路径混合

# 示例：
# 移动到预定义关节位置
movej([-1.95, -1.58, 1.16, -1.15, -1.55, 1.18], a=1.0, v=0.5)

# 移动到当前位置的变体
current_joints = get_actual_joint_positions()
current_joints[0] = current_joints[0] + d2r(10)  # 第一关节转动10度
movej(current_joints, a=0.8, v=0.3)
```

##### movej() - 位姿目标
```urscript
# 使用位姿作为目标的关节运动
movej(pose, a=1.4, v=1.05, t=0, r=0)

# 参数说明：
# pose: 目标位姿 p[x, y, z, rx, ry, rz]

# 示例：
movej(p[0.3, 0.3, 0.4, 2.22, -2.22, 0.0], a=1.0, v=0.5)
```

#### 1.2 直线运动

##### movel()
```urscript
# 直线运动 - TCP沿直线路径移动到目标位置
movel(pose, a=1.2, v=0.25, t=0, r=0)

# 参数说明：
# pose: 目标位姿 p[x, y, z, rx, ry, rz] (米, 弧度)
# a: 工具加速度 (米/秒²)，默认 1.2
# v: 工具速度 (米/秒)，默认 0.25
# t: 运动时间 (秒)，0表示不使用时间控制
# r: 混合半径 (米)

# 示例：
# 基本直线运动
movel(p[0.5, 0.2, 0.3, 0, 3.14, 0], a=0.5, v=0.1)

# 相对运动
current_pose = get_actual_tcp_pose()
target_pose = pose_trans(current_pose, p[0.1, 0, 0, 0, 0, 0])
movel(target_pose, a=0.3, v=0.05)
```

#### 1.3 圆弧运动

##### movec()
```urscript
# 圆弧运动 - TCP沿圆弧路径运动
movec(pose_via, pose_to, a=1.2, v=0.25, r=0, mode=0)

# 参数说明：
# pose_via: 圆弧中间点位姿 p[x, y, z, rx, ry, rz]
# pose_to: 圆弧终点位姿 p[x, y, z, rx, ry, rz]
# a: 工具加速度 (米/秒²)
# v: 工具速度 (米/秒)
# r: 混合半径 (米)
# mode: 圆弧模式 (0=不受约束, 1=受约束)

# 示例：
# 基本圆弧运动
start_pose = get_actual_tcp_pose()
via_pose = p[0.4, 0.3, 0.3, 0, 3.14, 0]
end_pose = p[0.5, 0.4, 0.3, 0, 3.14, 0]
movec(via_pose, end_pose, a=0.5, v=0.1)

# 画圆弧
center = p[0.4, 0.3, 0.3, 0, 3.14, 0]
radius = 0.05
via_point = pose_trans(center, p[radius, 0, 0, 0, 0, 0])
end_point = pose_trans(center, p[0, radius, 0, 0, 0, 0])
movec(via_point, end_point, a=0.3, v=0.05)
```

#### 1.4 进程运动

##### movep()
```urscript
# 进程运动 - 沿预定义路径运动
movep(pose, a=1.2, v=0.25, r=0)

# 参数说明：
# pose: 目标位姿
# a: 工具加速度 (米/秒²)
# v: 工具速度 (米/秒)  
# r: 混合半径 (米)

# 示例：
movep(p[0.3, 0.3, 0.4, 2.22, -2.22, 0.0], a=1.0, v=0.2)
```

#### 1.5 伺服运动

##### servoj()
```urscript
# 关节伺服运动 - 实时关节位置控制
servoj(q, a=0, v=0, t=0.008, lookahead_time=0.1, gain=300)

# 参数说明：
# q: 目标关节位置 [q1, q2, q3, q4, q5, q6] (弧度)
# a: 不使用，保留参数
# v: 不使用，保留参数
# t: 控制周期 (秒)，默认 0.008
# lookahead_time: 前瞻时间 (秒)
# gain: 控制增益

# 示例：
# 实时关节控制
target_joints = get_actual_joint_positions()
target_joints[0] = target_joints[0] + 0.01
servoj(target_joints, t=0.008, lookahead_time=0.1, gain=300)
```

##### servoc()
```urscript
# 笛卡尔伺服运动 - 实时位姿控制
servoc(pose, a=0, v=0, r=0)

# 参数说明：
# pose: 目标位姿 p[x, y, z, rx, ry, rz]
# a: 不使用，保留参数
# v: 不使用，保留参数
# r: 不使用，保留参数

# 示例：
current_pose = get_actual_tcp_pose()
target_pose = pose_trans(current_pose, p[0.001, 0, 0, 0, 0, 0])
servoc(target_pose)
```

### 2. 输入输出控制命令

#### 2.1 数字输出控制

##### set_digital_out()
```urscript
# 设置数字输出端口状态
set_digital_out(n, b)

# 参数说明：
# n: 数字输出端口号 (0-7)
# b: 输出状态 (True/False)

# 示例：
set_digital_out(0, True)   # 设置DO0为高电平
set_digital_out(1, False)  # 设置DO1为低电平

# 批量设置
set_digital_out(0, True)
set_digital_out(1, True)
set_digital_out(2, False)
```

##### set_configurable_digital_out()
```urscript
# 设置可配置数字输出
set_configurable_digital_out(n, b)

# 参数说明：
# n: 可配置输出端口号 (0-7)
# b: 输出状态 (True/False)

# 示例：
set_configurable_digital_out(0, True)
```

##### set_tool_digital_out()
```urscript
# 设置工具数字输出
set_tool_digital_out(n, b)

# 参数说明：
# n: 工具输出端口号 (0-1)
# b: 输出状态 (True/False)

# 示例：
set_tool_digital_out(0, True)  # 工具输出0置高
set_tool_digital_out(1, False) # 工具输出1置低
```

#### 2.2 模拟输出控制

##### set_analog_out()
```urscript
# 设置模拟输出电压
set_analog_out(n, f)

# 参数说明：
# n: 模拟输出端口号 (0-1)
# f: 输出电压 (0.0-10.0V)

# 示例：
set_analog_out(0, 5.0)    # AO0输出5V
set_analog_out(1, 2.5)    # AO1输出2.5V

# 渐变输出
for voltage in range(0, 101):
    set_analog_out(0, voltage / 10.0)
    sleep(0.1)
end
```

#### 2.3 输入读取

##### get_digital_in()
```urscript
# 读取数字输入状态
state = get_digital_in(n)

# 参数说明：
# n: 数字输入端口号 (0-7)
# 返回值: True/False

# 示例：
if get_digital_in(0):
    textmsg("输入0为高电平")
else:
    textmsg("输入0为低电平")
end

# 等待输入信号
while not get_digital_in(1):
    sleep(0.1)
end
textmsg("输入1信号到达")
```

##### get_analog_in()
```urscript
# 读取模拟输入电压
voltage = get_analog_in(n)

# 参数说明：
# n: 模拟输入端口号 (0-1)
# 返回值: 电压值 (0.0-10.0V)

# 示例：
voltage_0 = get_analog_in(0)
textmsg("AI0电压: " + to_str(voltage_0))

# 电压监控
while True:
    v = get_analog_in(0)
    if v > 8.0:
        textmsg("电压过高警告")
        break
    end
    sleep(0.5)
end
```

### 3. 传感器与反馈命令

#### 3.1 位置反馈

##### get_actual_tcp_pose()
```urscript
# 获取TCP实际位姿
pose = get_actual_tcp_pose()

# 返回值: p[x, y, z, rx, ry, rz]
# x, y, z: 位置坐标 (米)
# rx, ry, rz: 旋转向量 (弧度)

# 示例：
current_pose = get_actual_tcp_pose()
textmsg("当前位置: " + to_str(current_pose))

# 提取位置信息
x_pos = current_pose[0]
y_pos = current_pose[1]
z_pos = current_pose[2]
```

##### get_actual_joint_positions()
```urscript
# 获取实际关节位置
joints = get_actual_joint_positions()

# 返回值: [q1, q2, q3, q4, q5, q6] (弧度)

# 示例：
current_joints = get_actual_joint_positions()
textmsg("关节位置: " + to_str(current_joints))

# 转换为角度
joint_degrees = []
for i in range(6):
    joint_degrees[i] = r2d(current_joints[i])
end
```

##### get_target_tcp_pose()
```urscript
# 获取TCP目标位姿
target_pose = get_target_tcp_pose()

# 示例：
target = get_target_tcp_pose()
actual = get_actual_tcp_pose()
error = pose_dist(target, actual)
textmsg("位置误差: " + to_str(error))
```

#### 3.2 力和力矩反馈

##### get_tcp_force()
```urscript
# 获取TCP处的力和力矩
force_torque = get_tcp_force()

# 返回值: [Fx, Fy, Fz, Mx, My, Mz]
# Fx, Fy, Fz: 力 (牛顿)
# Mx, My, Mz: 力矩 (牛顿·米)

# 示例：
ft = get_tcp_force()
force_magnitude = norm([ft[0], ft[1], ft[2]])
textmsg("合力大小: " + to_str(force_magnitude) + " N")

# 力控制应用
while True:
    force = get_tcp_force()
    if force[2] > 10.0:  # Z方向力超过10N
        textmsg("检测到接触")
        break
    end
    sleep(0.01)
end
```

#### 3.3 速度反馈

##### get_actual_tcp_speed()
```urscript
# 获取TCP实际速度
speed = get_actual_tcp_speed()

# 返回值: [vx, vy, vz, wx, wy, wz]
# vx, vy, vz: 线速度 (米/秒)
# wx, wy, wz: 角速度 (弧度/秒)

# 示例：
tcp_speed = get_actual_tcp_speed()
linear_speed = norm([tcp_speed[0], tcp_speed[1], tcp_speed[2]])
textmsg("线速度: " + to_str(linear_speed) + " m/s")
```

##### get_actual_joint_speeds()
```urscript
# 获取实际关节速度
joint_speeds = get_actual_joint_speeds()

# 返回值: [v1, v2, v3, v4, v5, v6] (弧度/秒)

# 示例：
speeds = get_actual_joint_speeds()
max_speed = 0
for i in range(6):
    if abs(speeds[i]) > max_speed:
        max_speed = abs(speeds[i])
    end
end
textmsg("最大关节速度: " + to_str(max_speed) + " rad/s")
```

### 4. 系统与控制命令

#### 4.1 时间控制

##### sleep()
```urscript
# 程序暂停指定时间
sleep(t)

# 参数说明：
# t: 暂停时间 (秒)

# 示例：
sleep(1.0)      # 暂停1秒
sleep(0.5)      # 暂停500毫秒
sleep(2.5)      # 暂停2.5秒

# 动作间隔
movel(pose1, a=1.0, v=0.1)
sleep(2.0)
movel(pose2, a=1.0, v=0.1)
```

##### get_system_time()
```urscript
# 获取系统时间戳
timestamp = get_system_time()

# 返回值: 系统时间 (秒)

# 示例：
start_time = get_system_time()
# ... 执行某些操作 ...
end_time = get_system_time()
duration = end_time - start_time
textmsg("执行时间: " + to_str(duration) + " 秒")
```

#### 4.2 程序控制

##### halt()
```urscript
# 立即停止程序执行
halt()

# 示例：
if get_digital_in(0):  # 急停信号
    textmsg("检测到急停信号")
    halt()
end
```

##### textmsg()
```urscript
# 显示文本消息
textmsg(s)

# 参数说明：
# s: 要显示的字符串

# 示例：
textmsg("程序开始执行")
textmsg("当前位置: " + to_str(get_actual_tcp_pose()))

# 调试信息
current_force = get_tcp_force()
textmsg("力传感器读数: Fz = " + to_str(current_force[2]))
```

#### 4.3 条件与循环

##### 条件语句
```urscript
# if-else 条件判断
if condition:
    # 条件为真时执行
else:
    # 条件为假时执行
end

# 示例：
input_state = get_digital_in(0)
if input_state:
    set_digital_out(0, True)
    textmsg("输入激活，输出开启")
else:
    set_digital_out(0, False)
    textmsg("输入未激活，输出关闭")
end
```

##### 循环语句
```urscript
# while 循环
while condition:
    # 循环体
end

# 示例：
counter = 0
while counter < 10:
    textmsg("计数: " + to_str(counter))
    counter = counter + 1
    sleep(1.0)
end

# 等待条件
while not get_digital_in(1):
    textmsg("等待启动信号...")
    sleep(0.5)
end
```

### 5. 数学与坐标变换

#### 5.1 位姿变换

##### pose_trans()
```urscript
# 位姿变换
result_pose = pose_trans(p_from, p_to)

# 参数说明：
# p_from: 基准位姿
# p_to: 相对位姿
# 返回值: 变换后的位姿

# 示例：
base_pose = get_actual_tcp_pose()
offset = p[0.1, 0, 0, 0, 0, 0]  # X方向偏移10cm
target_pose = pose_trans(base_pose, offset)
movel(target_pose, a=0.5, v=0.1)
```

##### pose_inv()
```urscript
# 位姿逆变换
inverse_pose = pose_inv(pose)

# 示例：
current_pose = get_actual_tcp_pose()
inverse = pose_inv(current_pose)
textmsg("逆变换位姿: " + to_str(inverse))
```

#### 5.2 数学函数

##### norm()
```urscript
# 计算向量模长
magnitude = norm(vector)

# 示例：
force = get_tcp_force()
force_magnitude = norm([force[0], force[1], force[2]])
textmsg("合力大小: " + to_str(force_magnitude))
```

##### d2r() / r2d()
```urscript
# 角度与弧度转换
radians = d2r(degrees)  # 角度转弧度
degrees = r2d(radians)  # 弧度转角度

# 示例：
angle_deg = 90
angle_rad = d2r(angle_deg)
textmsg("90度 = " + to_str(angle_rad) + " 弧度")

joint_positions = get_actual_joint_positions()
joint_0_degrees = r2d(joint_positions[0])
textmsg("关节0角度: " + to_str(joint_0_degrees) + " 度")
```

### 6. 命令快速参考

#### 6.1 运动命令速查表

| 命令 | 功能 | 主要参数 | 运动类型 |
|------|------|----------|----------|
| `movej()` | 关节空间运动 | q, a, v, t, r | 关节插值 |
| `movel()` | 直线运动 | pose, a, v, t, r | 直线插值 |
| `movec()` | 圆弧运动 | pose_via, pose_to, a, v, r | 圆弧插值 |
| `movep()` | 进程运动 | pose, a, v, r | 进程路径 |
| `servoj()` | 关节伺服 | q, t, lookahead_time, gain | 实时控制 |
| `servoc()` | 笛卡尔伺服 | pose | 实时控制 |

#### 6.2 I/O命令速查表

| 命令 | 功能 | 参数 | 范围 |
|------|------|------|------|
| `set_digital_out()` | 设置数字输出 | n, b | n: 0-7 |
| `get_digital_in()` | 读取数字输入 | n | n: 0-7 |
| `set_analog_out()` | 设置模拟输出 | n, f | n: 0-1, f: 0-10V |
| `get_analog_in()` | 读取模拟输入 | n | n: 0-1 |
| `set_tool_digital_out()` | 工具数字输出 | n, b | n: 0-1 |

#### 6.3 传感器命令速查表

| 命令 | 功能 | 返回值 | 单位 |
|------|------|--------|------|
| `get_actual_tcp_pose()` | TCP实际位姿 | [x,y,z,rx,ry,rz] | 米, 弧度 |
| `get_actual_joint_positions()` | 关节实际位置 | [q1,q2,q3,q4,q5,q6] | 弧度 |
| `get_tcp_force()` | TCP力和力矩 | [Fx,Fy,Fz,Mx,My,Mz] | 牛顿, 牛顿·米 |
| `get_actual_tcp_speed()` | TCP实际速度 | [vx,vy,vz,wx,wy,wz] | 米/秒, 弧度/秒 |

#### 6.4 常用参数说明

##### 运动参数
- **a (加速度)**: 
  - 关节运动: 弧度/秒² (典型值: 0.5-1.4)
  - 直线运动: 米/秒² (典型值: 0.3-1.2)
- **v (速度)**:
  - 关节运动: 弧度/秒 (典型值: 0.1-1.05)
  - 直线运动: 米/秒 (典型值: 0.05-0.25)
- **r (混合半径)**: 米 (典型值: 0.001-0.01)

##### 位姿表示
- **p[x, y, z, rx, ry, rz]**: 位姿向量
  - x, y, z: 位置坐标 (米)
  - rx, ry, rz: 旋转向量 (弧度)

#### 6.5 常用代码模板

##### 基本运动序列
```urscript
# 运动到安全位置
movej([-1.95, -1.58, 1.16, -1.15, -1.55, 1.18], a=1.0, v=0.5)

# 直线接近目标
movel(p[0.3, 0.3, 0.4, 2.22, -2.22, 0.0], a=0.5, v=0.1)

# 精确定位
movel(p[0.3, 0.3, 0.35, 2.22, -2.22, 0.0], a=0.3, v=0.05)
```

##### I/O控制模板
```urscript
# 等待启动信号
while not get_digital_in(0):
    sleep(0.1)
end

# 执行动作并设置输出
set_digital_out(1, True)  # 开启输出
# ... 执行运动 ...
set_digital_out(1, False) # 关闭输出
```

##### 力控制模板
```urscript
# 力控接触检测
while True:
    force = get_tcp_force()
    if norm([force[0], force[1], force[2]]) > 5.0:
        textmsg("检测到接触")
        break
    end
    # 继续运动
    current_pose = get_actual_tcp_pose()
    target_pose = pose_trans(current_pose, p[0, 0, -0.001, 0, 0, 0])
    servoc(target_pose)
    sleep(0.008)
end
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

本手册提供了UR5机器人TCP/IP通信和URScript命令的完整参考，涵盖了从基础网络连接到高级机器人控制的各个方面。

### 手册内容概览：

#### TCP/IP通信功能
- **网络连接管理**：套接字创建、连接和状态监控
- **数据传输**：字符串、数值和二进制数据的发送接收
- **客户端-服务器架构**：双向通信和实时数据交换
- **错误处理与恢复**：网络异常处理和自动重连机制

#### URScript命令系统
- **运动控制**：movej, movel, movec等运动命令的详细使用
- **I/O控制**：数字和模拟输入输出的完整操作方法
- **传感器反馈**：位置、力、速度等传感器数据获取
- **系统控制**：时间管理、程序控制和数学运算函数
- **快速参考**：命令速查表和常用代码模板

### 关键应用场景：

1. **远程控制**：通过网络发送运动命令实现远程操作
2. **视觉集成**：与视觉系统通信获取目标位置信息
3. **PLC通信**：与工厂自动化系统进行数据交换
4. **力控应用**：基于力反馈的精密装配和接触检测
5. **实时监控**：连续获取机器人状态和传感器数据

### 最佳实践要点：

1. **网络通信**：选择合适端口、处理网络异常、优化通信性能
2. **运动控制**：合理设置速度加速度参数、使用混合半径优化路径
3. **I/O操作**：正确配置输入输出、实现可靠的信号处理
4. **安全考虑**：实施访问控制、输入验证和异常处理
5. **代码结构**：使用标准化数据格式和模块化程序设计

通过本手册的指导，开发者可以充分利用UR5机器人的TCP/IP通信能力和丰富的URScript命令集，构建稳定可靠的工业自动化应用系统，实现智能制造和工业4.0的先进生产模式。
