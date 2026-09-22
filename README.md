# 松应仿真训练营 · 第一篇：从环境搭建到机器人仿真控制

从键盘控制轮式底盘，到运行 G1 运动策略：用三个 Demo，串起 **环境安装、资产订阅、场景搭建、Python 控制与仿真运行**。

本篇从 OrcaLab 的基本操作开始，先用差速底盘跑通控制流程，再通过阿克曼车辆观察不同的转向方式，最后选学 G1 预训练策略推理，为第二篇的定点导航做准备。

[![差速轮式底盘运行效果](docs/images/chasu.gif)](docs/images/差速轮式底盘.webm)

## 资料导航

| 资料 | 入口 |
| --- | --- |
| 第一篇完整图文与教学视频 | [飞书详细文档](https://ucnj8k63v5wn.feishu.cn/wiki/TnwZwVptdi9r9MkmtSUc4OfCnzh?from=from_copylink) |
| 仓库内完整教程 | [安装、界面操作、资产准备与实验步骤](docs/tutorial.md) |
| 工作坊技术方案 | [课程目标、技术路线与完成标准](docs/tutorial.md#workshop-plan) |
| Demo 视频 | [差速轮式底盘](docs/images/差速轮式底盘.webm) · [阿克曼车辆](docs/images/Ackerman底盘.webm) · [G1 运控动图](docs/images/g1.gif) |
| 本文操作入口 | [运行准备](#运行准备) · [资产与场景](#资产与场景) · [三个 Demo](#demo-展示与运行) · [FAQ](#常见问题) |
| 第二篇实战 | [从场景资产搭建到 G1 定点导航](https://github.com/Xbotics-Embodied-AI-club/Orcaplayground-G1-routing-inspection/tree/feature/demo1-v1-green-table) |
| 官方项目说明 | [OrcaPlayground 官方 README（26.7.1）](https://github.com/openverse-orca/OrcaPlayground/blob/release/26.7.1/README.md) |

首次学习可按飞书中的图文和视频逐步操作；本页用于快速查找准备事项、启动命令和运行效果。图片、动图和两段完整 Demo 视频已保存在仓库中。

## 你将完成什么

| 实验 | 任务 | 完成标准 |
| --- | --- | --- |
| 差速轮式底盘 | 通过 W/A/S/D 控制移动与转向 | 完成前进、后退和左右转向，理解左右轮速度差的作用 |
| 阿克曼车辆 | 控制黄色越野车，观察前轮转角 | 能沿弧线行驶，并说明它与差速底盘的区别 |
| G1 运控（选学） | 加载已有 ONNX 运动策略 | 完成初始化、基本站立与策略行走 |

差速底盘是本篇主实验。完成后，保存自己的 Layout，并录制一段控制效果，作为本次实践记录。G1 实验使用已有模型进行推理，不涉及重新训练。

## 技术路线

Python 程序读取键盘输入或运行已有运动策略，通过 OrcaGym 与仿真交换状态和控制指令，最终由 OrcaLab 中的执行器驱动机器人运动。

| 组件 | 职责 |
| --- | --- |
| OrcaLab | 加载资产与场景，执行物理仿真并显示机器人运动 |
| OrcaGym | 通过 gRPC 连接仿真，交换状态和控制指令 |
| Python Example | 处理控制输入、计算动作并推动仿真循环 |

<details>
<summary>展开查看组件关系图</summary>

![OrcaLab、OrcaGym 与 Python 控制程序的关系](docs/images/orca_relation.jpg)

</details>

本篇先完成同一台计算机上的仿真控制，默认通信地址为 `localhost:50051`。视觉取流、路径点导航和到点拍照在第二篇展开。

## 运行准备

### 1. 确认课程环境

| 项目 | 本篇使用版本 |
| --- | --- |
| 操作系统 | Ubuntu / Linux |
| Python | 3.12 |
| OrcaLab / OrcaGym | 26.7.1 |
| 仓库分支 | `release/26.7.1` |

本地安装从[环境准备](docs/tutorial.md#setup)开始；使用预装云镜像时，先检查已有环境，无需重复安装。下面沿用第一篇教程的环境名 `orcalab`；如果已有配置完成的 `orca` 环境，请激活实际环境。

```bash
conda activate orcalab
python --version
python -m pip show orca-lab orca-gym
nvidia-smi
# 检查对应的版本和驱动
```

### 2. 获取代码与依赖

首次下载时，在终端执行：

```bash
mkdir -p ~/ORCA
cd ~/ORCA
git clone --branch release/26.7.1 \
  https://github.com/Xbotics-Embodied-AI-club/Orcaplayground-G1-routing-inspection.git OrcaPlayground
cd OrcaPlayground
python -m pip install -r requirements.txt
```

已有仓库时，进入自己的仓库目录，确认当前分支是 `release/26.7.1`。后续命令均在仓库根目录执行；这里的 `~/ORCA` 是示例存放位置，可替换为自己的路径。

### 3. 启动 OrcaLab

在已激活课程环境的终端中执行：

```bash
cd ~/ORCA/OrcaPlayground
orcalab .
```

`.` 指定当前项目目录，使 OrcaLab 读取仓库中的 [外部程序配置](.orcalab/config.toml)。第一次启动可能需要下载并初始化运行组件；等待完成后按提示重新启动。

<details>
<summary>展开查看界面区域与仿真程序选择</summary>

![OrcaLab 主界面及主要区域](docs/images/main_region_orca.png)

![选择仿真程序](docs/images/2.png)

</details>

## 资产与场景

在资产中心订阅 **OrcaPlaygroundAssets**，等待状态显示为 `Up to Date`，再将目标资产拖入场景。仅完成订阅，还没有生成供程序控制的机器人。

| 概念 | 操作中的含义 |
| --- | --- |
| Asset | 资产面板中可重复使用的资源模板 |
| Actor | 把 Asset 拖入场景后生成的具体对象 |
| Layout | 保存场景对象及环境配置的布局文件 |

每个 Demo 建议使用独立 Layout，并只放置一台匹配的机器人。当前示例会扫描场景中的实际实例，识别执行器和机器人；找不到匹配对象或出现多个匹配对象时，应先检查场景。

| Demo | 需要拖入的 Asset | 启动入口 |
| --- | --- | --- |
| 差速轮式底盘 | `openloong_gripper_2f85_mobile_base_usda` | 界面选择 `run_wheeled_chassis` |
| 阿克曼车辆 | `hummer_h2_usda`（黄色越野车） | 界面选择 `run_ackerman` |
| G1 自动行走（选学） | `g1_29dof_old_usda` | 手动启动后运行 `examples.g1.run_g1_random_walk` |

资产订阅、Actor 检查与 Layout 保存的图文步骤见[场景搭建教程](docs/tutorial.md#assets)。第一篇的 G1 资产与第二篇不同，复现时请使用本篇表格中的型号。

## Demo 展示与运行

<a id="demo-展示"></a>

轮式 Demo 可直接在 OrcaLab 的“启动仿真”窗口选择对应程序。若需要在终端观察日志，则选择“无仿真程序（手动启动）”，再在另一个已激活课程环境、位于仓库根目录的终端中执行下方命令。两种启动方式选其一。

### Demo 1：差速轮式底盘

**目标：**用键盘控制前进、后退与转向，观察左右轮速度差如何改变底盘方向。

将差速底盘资产拖入场景并保存 Layout，选择 `run_wheeled_chassis` 启动。终端启动命令为：

```bash
python -m examples.wheeled_chassis.run_wheeled_chassis
```

启动后单击仿真视口，切换为英文输入法，再依次测试 W、S、A、D。

| 按键 | 动作 |
| --- | --- |
| W / S | 前进 / 后退 |
| A / D | 左转 / 右转 |

**运行对照：**机器人能够响应四个方向的输入，终端无持续报错。页首动图中的移动协作机器人是本 Demo 的控制对象，黄色越野车属于下一个实验。

[查看完整视频](docs/images/差速轮式底盘.webm) · [详细实验步骤](docs/tutorial.md#demo-differential) · [入口代码](examples/wheeled_chassis/run_wheeled_chassis.py)

### Demo 2：阿克曼车辆

**目标：**控制黄色越野车运动，观察前轮转角与车辆行驶轨迹。

在独立 Layout 中加入 `hummer_h2_usda`，选择 `run_ackerman` 启动。终端启动命令为：

```bash
python -m examples.wheeled_chassis.run_ackerman
```

W / S 控制前进与后退，A / D 控制转向。将前进和转向配合使用，观察车辆沿弧线行驶，并与差速底盘对比。

[![阿克曼车辆运行预览](docs/images/ackerman.gif)](docs/images/Ackerman底盘.webm)

[查看完整视频](docs/images/Ackerman底盘.webm) · [详细实验步骤](docs/tutorial.md#demo-ackerman) · [入口代码](examples/wheeled_chassis/run_ackerman.py)

### Demo 3：G1 运动策略（选学）

**目标：**加载已有 ONNX 策略，观察 G1 初始化、站立与自动行走。

在课程环境中安装额外依赖：

```bash
python -m pip install -r examples/g1/requirements.txt
```

将 `g1_29dof_old_usda` 加入场景，选择“无仿真程序（手动启动）”，再执行：

```bash
python -m examples.g1.run_g1_random_walk
```

![G1 运动策略运行效果](docs/images/g1.gif)

**运行对照：**模型加载后，机器人能够完成初始化、保持基本站立并行走。仅看到策略加载成功，还不能判断整个仿真已正常运行。

如需手动控制，可结束当前程序后，在 OrcaLab 程序列表选择 `run_g1`；它与上面的自动行走入口不同。

[详细实验步骤](docs/tutorial.md#demo-g1) · [自动行走入口代码](examples/g1/run_g1_random_walk.py)

## 常见问题

按“版本与环境 → 项目目录 → 资产与 Layout → 仿真状态 → gRPC → 控制程序”的顺序排查，并保留完整终端日志。

| 现象 | 优先检查 |
| --- | --- |
| 看不到外部程序 | 是否从仓库根目录运行 `orcalab .`，是否存在 `.orcalab/config.toml` |
| 找不到机器人或执行器 | 是否使用表格中的准确资产、已拖入场景，且只有一台匹配机器人 |
| 无法连接 `localhost:50051` | 仿真服务是否启动，控制程序与 OrcaLab 是否在同一台计算机 |
| W/A/S/D 无反应 | 仿真视口是否获得焦点、是否使用英文输入法、程序是否仍在运行 |
| 提示缺少 Python 模块 | 当前环境是否正确，基础依赖和对应示例依赖是否安装 |
| ONNX 已加载但 G1 未运行 | 继续检查后续报错、G1 资产型号及仿真连接状态 |

完整检查步骤见 [FAQ 与排查顺序](docs/tutorial.md#faq)。记录版本、启动命令、资产标识和完整日志，可以更快定位问题。

## 继续探索

完成第一篇后，进入第二篇，将控制流程扩展到相机取流、绿色桌子绕障和工厂定点导航。

| 分支 | 学习内容 |
| --- | --- |
| [release/26.7.1](https://github.com/Xbotics-Embodied-AI-club/Orcaplayground-G1-routing-inspection/tree/release/26.7.1) | 第一篇：环境、资产与基础仿真控制 |
| [feature/demo1-v1-green-table](https://github.com/Xbotics-Embodied-AI-club/Orcaplayground-G1-routing-inspection/tree/feature/demo1-v1-green-table) | 第二篇：绿色桌子绕障与目标点导航 |
| [demo1-v1-factory-navi](https://github.com/Xbotics-Embodied-AI-club/Orcaplayground-G1-routing-inspection/tree/demo1-v1-factory-navi) | 第二篇：工厂路径点导航与柜前拍照 |

## 官方资料与致谢

本训练营基于 OrcaPlayground 与 OrcaGym 开发。更多示例、平台安装说明与扩展开发方法请参考：

- [OrcaPlayground 官方 README（26.7.1）](https://github.com/openverse-orca/OrcaPlayground/blob/release/26.7.1/README.md)
- [OrcaGym](https://github.com/openverse-orca/OrcaGym)
- [ORCA 官方文档](https://docs.orca3d.cn/)
- [ORCA 资产中心](https://simassets.orca3d.cn/)

感谢上游项目及其贡献者。许可证见 [LICENSE](LICENSE)。
