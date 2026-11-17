# 环境搭建

## 1. 操作系统与 ROS 2

| Ubuntu | ROS 2 |
|---|---|
| 24.04 | Jazzy |
| 22.04 | Humble |

安装完 ROS 2 后，确认：

```bash
source /opt/ros/$ROS_DISTRO/setup.bash
ros2 run demo_nodes_cpp talker
```

## 2. 依赖

```bash
pip install -r requirements.txt
```

## 3. 编译

```bash
colcon build --symlink-install
source install/setup.bash
```

## 4. 常见问题

（在这里记录踩过的坑，一条一行：现象 → 原因 → 解决）