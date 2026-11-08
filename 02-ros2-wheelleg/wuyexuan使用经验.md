# wuyexuan 使用经验与实验记录



---

## 0. 环境：我的机器和教程不一样

| 项目 | 教程假设 | **我的实际环境** |
|---|---|---|
| 系统 | Ubuntu 22.04 | **Ubuntu 24.04.5 LTS** |
| ROS2 版本 | Humble | **Jazzy** |
| Python | 3.10 | **3.12.16**（conda 环境 `rm_ros2`） |
| 工作区 | `ros2_ws` | `~/PROJECT/RoboMaster/training/02-ros2-wheelleg/Tutorial/ros2_ws` |



### 已装的依赖（和教程清单完全一致）

| 包 | 版本 | 备注 |
|---|---|---|
| `mujoco` | 3.16.0 | 第 9 章仿真 |
| `numpy` | 2.2.6 | |
| `pyserial` | 3.5 | 第 8 章串口 |
| `colcon-common-extensions` | 0.3.0 | 在环境里构建 |
| `setuptools` | 71.1.0 | 必须 `<72` |
| `empy` | 3.3.4 | 必须 `<4` |
| `lark` | 1.1.1 | 消息解析 |

`PYTHONNOUSERSITE=1` 已设，`~/.local` 是空的，环境是干净的。

---

## 1. 每天开工的固定流程（贴在显示器边上）

```text
新开一个终端
    │
    │  ① conda activate rm_ros2               ← 决定"用哪个 python 编"
    │     ✔ 自检1：which colcon 必须是 .../rm_ros2/bin/colcon
    │        看到 /usr/bin/colcon → 停！环境没激活
    │
    │  ② source /opt/ros/jazzy/setup.bash     ← 决定"能不能生成消息"
    │      （~/.bashrc 里已有，可跳过）
    │
    │  ③ cd 到 ros2_ws 并 colcon build         ← 生成 install/
    │      （只在改了代码时跑）
    │     ✔ 自检2：head -1 可执行文件 必须是 .../rm_ros2/bin/python3.12
    │        看到 #!/usr/bin/python3 → 这次构建废了，回 ①
    │
    │  ④ source install/setup.bash            ← 让 ros2 看见我的包
    │
    ▼
 ⑤ ros2 run <包名> <可执行名>

※ 装过任何 pip 包之后，额外跑一次 ✔ 自检3（见下）
```

### 复制粘贴版（从新终端一路贴到运行）

```bash
# ① 激活 Python 环境（决定用哪个 python 编译）
conda activate rm_ros2

# ✔ 自检1：colcon 必须来自环境（显示 /usr/bin/colcon 就别往下走）
which colcon | grep -q miniconda || echo "⚠️  环境没激活，不要构建！"
which colcon

# ② 让 ROS2 可用（.bashrc 里已有，一般可跳过）
source /opt/ros/jazzy/setup.bash

# ③ 进入工作区并构建（改了需要重编的东西才跑）
cd ~/PROJECT/RoboMaster/training/02-ros2-wheelleg/Tutorial/ros2_ws
colcon build --symlink-install

# ✔ 自检2：shebang 必须指向环境（显示 /usr/bin/python3 就回 ① 重编）
head -1 install/wheel_leg_sim/lib/wheel_leg_sim/wheel_leg_sim

# ④ 把工作区挂进当前终端（每开一个新终端都要做）
source install/setup.bash

# ⑤ 运行
ros2 run wheel_leg_sim wheel_leg_sim
```

### ✔ 自检3：装完任何 pip 包之后跑一次

```bash
python -c "import setuptools, em, lark; print(setuptools.__version__, em.__version__, lark.__version__)"
# 合格线：setuptools < 72 | empy 3.x | lark 1.1.x
```

### 为什么自检1 最要命

本机同时存在两套 colcon（`/usr/bin/colcon` 和 conda 环境里的），
而且系统 python 里 `empy` / `lark` / `setuptools` 也全都有。
所以忘记激活环境时，`colcon build` **不报错、显示成功**，
只把 shebang 悄悄写成 `#!/usr/bin/python3`（已实测复现）。
系统 python 没有 `mujoco`，问题要等到第 9 章才爆出来。

**自检3 的必要性**：`pip install` 装任何新包，都可能顺手把 `setuptools` 升到 72+
或把 `empy` 升到 4.x，而报错信息完全不提这两个名字。

**修复很简单**：在环境里重新 `colcon build --symlink-install` 一次，shebang 就回来了
（实测 1.19 秒，**不需要** `rm -rf build install log`）。



---



## 2. 常用命令速查



---

---

## 3. 实验记录

> 格式：日期 → 目的 → 命令 → 结果 → 结论。以后每次做实验往下加一条。

### 2026-11-08　环境体检

- **目的**：确认换到 Jazzy 后，教程第 1 章的环境能跑通
- **命令**：
  ```bash
  conda activate rm_ros2
  python -V                                   # Python 3.12.16
  python -c "import rclpy, mujoco; print('ok')"
  cd .../ros2_ws && colcon build --symlink-install
  ```
- **结果**：
  - `rclpy` / `mujoco 3.16.0` 都能 import
  - 6 个包全部构建成功，耗时 3.09 秒
  - shebang 指向 `.../rm_ros2/bin/python3.12` ✅
  - `ros2 pkg list` 共 288 个包，`turtlesim` / `rviz2` / `teleop_twist_keyboard` 都在
- **结论**：环境健康，可以开始按教程往下学。
  ⚠️ 唯一注意：教程是 Humble/3.10 写的，命令里的发行版名和 Python 版本要自己换。

---
