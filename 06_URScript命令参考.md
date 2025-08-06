# UR5 URScript命令参考

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