# 面向室内语义建图的多帧标签融合与不确定性度量

> 一台差速小车，走一圈就能记住"屋里有什么"，并且知道自己记错没有。

[![ROS 2](https://img.shields.io/badge/ROS%202-Jazzy%20%2F%20Humble-blue)](https://docs.ros.org/)
[![Ubuntu](https://img.shields.io/badge/Ubuntu-24.04%20%2F%2022.04-orange)](https://ubuntu.com/)
[![License](https://img.shields.io/badge/License-MIT-green)](LICENSE)

---

## 一、这是什么

巡检机器人需要的不只是"我在哪"，而是"那里有什么"。本项目建设一套面向室内场景的语义建图系统：

- 用 **2D 激光 SLAM** 建立环境的栅格地图；
- 用 **视觉检测模型** 识别物体，并通过相机—激光外参把二维检测框投影到三维点云，形成**物体实体**；
- 对每个实体做**多帧贝叶斯融合**，维护类别后验与**不确定性**；
- 当不确定性高于阈值时，触发**主动重观测**（换视角补测）。

最终，地图不仅告诉你"那里是灭火器"，还告诉你"这句话有多可信"。

## 二、系统架构

```
        ┌──────────────┐
/scan ──►│  SLAM 建图    │──► map.pgm / map.yaml
        └──────┬───────┘
               │ T_world_cam（相机位姿）
               ▼
┌─────────┐   ┌──────────────┐   ┌──────────────┐   ┌──────────────┐
│ 语义检测 │──►│ 2D→3D 投影    │──►│ 实体聚类/关联 │──►│ 贝叶斯融合    │
└─────────┘   └──────────────┘   └──────────────┘   └──────┬───────┘
                                                          │ 后验 + 不确定性
                                                          ▼
                                                   ┌──────────────┐
                                                   │ 主动重观测    │
                                                   └──────┬───────┘
                                                          │
                                            ┌─────────────┴─────────────┐
                                            ▼                           ▼
                                    语义地图（graph.json）        查询 / 导航
```

## 三、目录结构

```
semantic-slam-car/
├── README.md
├── docs/                   # 设计文档、标定报告、实验记录
├── src/
│   ├── perception/         # 检测与 2D→3D 投影
│   ├── association/        # 聚类与跨帧关联
│   ├── fusion/             # 贝叶斯融合与不确定性
│   ├── active/             # 主动重观测策略
│   └── eval/               # 基线、消融、指标统计
├── launch/                 # 启动文件
├── config/                 # 参数与标定结果
├── scripts/                # 工具脚本（数据采集、可视化）
├── data/                   # 数据集（不入库，见 .gitignore）
└── tests/                  # 单元测试
```

## 四、环境依赖

| 项 | 版本 |
|---|---|
| 操作系统 | Ubuntu 24.04（推荐）/ 22.04 |
| ROS 2 | Jazzy（对应 24.04）/ Humble（对应 22.04） |
| Python | 3.10+ |
| 关键库 | numpy、opencv-python、open3d、scikit-learn |
| 硬件 | 差速底盘（带编码器）、2D 激光雷达、单目相机 |

## 五、安装

```bash
# 1. 安装 ROS 2（略，见 docs/setup.md）
source /opt/ros/$ROS_DISTRO/setup.bash

# 2. 克隆仓库
git clone https://github.com/<你的用户名>/semantic-slam-car.git
cd semantic-slam-car

# 3. 安装 Python 依赖
pip install -r requirements.txt

# 4. 编译
colcon build --symlink-install
source install/setup.bash
```

## 六、运行

```bash
# 建图
ros2 launch slam_toolbox online_async_launch.py

# 保存地图
ros2 run nav2_map_server map_saver_cli -f maps/lab_room

# 语义建图（检测 + 投影 + 融合）
ros2 launch semantic_slam semantic_build.launch.py

# 查询
ros2 service call /query_object semantic_slam/srv/Query "{category: 'fire_extinguisher'}"
```

## 七、结果

| 方法 | 标签正确率 | 多帧一致率 | 查询定位误差 |
|---|---|---|---|
| B1 单帧检测 | 约 72% | — | 约 0.45 m |
| B2 多帧硬投票 | 约 79% | 约 76% | 约 0.38 m |
| **本方法** | **≥ 86%** | **≥ 88%** | **≤ 0.30 m** |

> 数值为本项目目标值，实际结果见 `docs/results.md`。

## 八、团队分工

| 成员 | 负责 | 交付物 |
|---|---|---|
| 【姓名】 | 统筹、集成、贝叶斯融合 | 系统跑通、融合模块 |
| 【姓名】 | 检测部署、2D→3D 投影 | 实体坐标、投影误差报告 |
| 【姓名】 | 聚类、跨帧关联 | 物体实体表 |
| 【姓名】 | 置信度、校准、主动复检 | 校准曲线、复检策略 |
| 【姓名】 | 基线、消融、标注、指标 | 对比表、消融表 |

## 九、进度

- [ ] 第 1–2 月：ROS 2 训练、平台搭建、标定
- [ ] 第 3–4 月：建图定位跑通、检测部署
- [ ] 第 5–6 月：投影与实体关联，基线实现
- [ ] 第 7–8 月：贝叶斯融合与不确定性
- [ ] 第 9–10 月：校准、主动复检、消融实验
- [ ] 第 11–12 月：指标评测、软著、结题

## 十、参考工作

- Kimera — [arXiv:1910.02490](https://arxiv.org/abs/1910.02490)
- ConceptGraphs — [arXiv:2309.16650](https://arxiv.org/abs/2309.16650)
- AutoX-SemMap — [arXiv:2409.15493](https://arxiv.org/abs/2409.15493)

## 十一、License

MIT License，详见 `LICENSE`。
