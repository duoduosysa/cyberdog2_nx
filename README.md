# cyberdog_ws

## 项目名称
本项目是基于小米铁蛋四足开发者平台的主要功能包。
## 仓库介绍
该仓库为四足开发平台项目代码主仓库，已将原有的 9 个子仓库合并为单一仓库，直接 clone 即可获取全部代码，无需再执行 `vcs import`。

原始子仓库来源：[MiRoboticsLab/cyberdog_ws](https://github.com/MiRoboticsLab/cyberdog_ws)（rolling 分支）



| 仓库名称               | 仓库地址                                                | 主要功能                                                     | 设计文档                                                     |
| ---------------------- | --------------------------- | ------------------------------- | ------------------------------------------- |
| cyberdog_ws            | https://github.com/MiRoboticsLab/cyberdog_ws            | 启动模块                                                     | [启动模块](https://miroboticslab.github.io/blogs/#/cn/cyberdog_bringup_cn)<br/> |
| manager                | https://github.com/MiRoboticsLab/manager                | 全局管理节点                                                 | [管理模块](https://miroboticslab.github.io/blogs/#/cn/cyberdog_manager_cn)<br/> |
| bridges                | https://github.com/MiRoboticsLab/bridges                | ros消息服务定义文件<br/>与app端通讯程序<br/>can数据收发封装库 | [grpc通信模块](https://miroboticslab.github.io/blogs/#/cn/cyberdog_grpc_cn) |
| devices                | https://github.com/MiRoboticsLab/devices                | 设备管理节点<br/>bms数据发布插件<br/>led设置插件<br/>touch插件<br/>uwb插件 | [设备管理模块](https://miroboticslab.github.io/blogs/#/cn/device_manager_cn)<br/>[bms模块](https://miroboticslab.github.io/blogs/#/cn/cyberdog_bms_cn)<br/>[LED模块](https://miroboticslab.github.io/blogs/#/cn/cyberdog_led_cn)<br/>[touch模块](https://miroboticslab.github.io/blogs/#/cn/cyberdog_touch_cn)<br/>[uwb模块](https://miroboticslab.github.io/blogs/#/cn/cyberdog_uwb_cn)<br/> |
| motion                 | https://github.com/MiRoboticsLab/motion                 | 运控管理                                                     | [运动管理模块](https://miroboticslab.github.io/blogs/#/cn/motion_manager_cn)<br/> |
| sensors                | https://github.com/MiRoboticsLab/sensors                | 传感器节点<br/>gps插件<br/>雷达插件<br/>tof插件<br/>超声插件 | [传感器模块](https://miroboticslab.github.io/blogs/#/cn/sensor_manager_cn)<br/>[gps模块](https://miroboticslab.github.io/blogs/#/cn/cyberdog_gps_cn)<br/>[雷达模块](https://miroboticslab.github.io/blogs/#/cn/cyberdog_lidar_cn)<br/>[tof模块](https://miroboticslab.github.io/blogs/#/cn/cyberdog_tof_cn)<br/>[超声模块](https://miroboticslab.github.io/blogs/#/cn/cyberdog_ultrasonic_cn)<br/> |
| interaction            | https://github.com/MiRoboticsLab/interaction            | 语音节点<br/>可视化编程节点<br/>小爱训练词节点<br/>图传节点<br/>快连节点 | [语音模块]()<br/>[可视化编程模块]()<br/>[语音训练词模块]()<br/>[图传模块]()<br/>[快连模块]()<br/> |
| cyberdog_nav2          | https://github.com/MiRoboticsLab/cyberdog_nav2           | 算法任务管理相关                                             | [算法任务管理](https://miroboticslab.github.io/blogs/#/cn/algorithm_manager_cn)<br/> |
| cyberdog_tracking_base | https://github.com/MiRoboticsLab/cyberdog_tracking_base | 存放了基于navigation2实现的docking， navigation， tracking功能相关的参数<br/>附加模块等 |                                                              |
| utils                  | https://github.com/MiRoboticsLab/utils                  | 通用接口库                                                   | [通用接口库](https://miroboticslab.github.io/blogs/#/cn/cyberdog_common_cn)<br/> |



## 开发者模式解锁（前置条件）

CyberDog 2 出厂默认未开启 SSH 访问，需要先解锁才能进行开发。由于小米官方已停止维护，在线申请解锁已不可用，可通过以下方式手动解锁：

### 方式一：USB 直连 SSH（先试这个）

用 USB 数据线连接电脑与机器狗，尝试直接 SSH：

```bash
ssh mi@192.168.55.1   # 密码 123
```

如果能连上，说明你的机器狗已处于解锁状态，可直接跳到编译章节。

### 方式二：NX 核心板 + 开发套件 + Debug 串口（一定能成功）

如果 USB 直连无法 SSH，则需要通过硬件调试串口进入系统：

1. 拆开机器狗，取出 Jetson Xavier NX 核心板（SOM 模块）
2. 将核心板安装到 NVIDIA Jetson Xavier NX 开发套件（Developer Kit）上
3. 用 USB-TTL 串口转接器连接开发套件上的 Debug UART 串口（TX/RX/GND，3.3V 电平）
4. 电脑端使用串口工具（PuTTY / minicom / MobaXterm），波特率 115200
5. 上电后即可在串口终端看到 Linux 启动日志和 login 提示
6. 用 `mi` / `123` 登录后，手动开启 SSH：

```bash
sudo systemctl enable ssh
sudo systemctl start ssh
```

7. 将核心板装回机器狗后，即可通过 USB 直连 SSH 访问

> **注意**：Debug 串口是硬件级别的调试接口，不受任何软件解锁状态限制，因此一定能成功。

### 方式三：修改开源代码绕过解锁检查（软件方案）

如果已经能 SSH 进入 NX（通过上述任一方式），可以通过修改开源代码彻底消除解锁限制，使系统始终认为已解锁。

**原理**：系统通过闭源工具 `/usr/bin/cyberdog_get_info unlock-state` 查询解锁状态（返回 0 = 未解锁，非 0 = 已解锁）。该工具只在开源代码的两个位置被调用，修改这两处即可。

**修改 1：`unlock_request` 节点**

文件：`manager/unlock_request/include/unlock_request/unlock_request.hpp`（源码中此文件位于 `nx_robot/cyberdog/include/unlock_request/unlock_request.hpp`）

将 `GetUnlockStatus()` 和 `UnlockAccess()` 函数体改为直接返回成功：

```cpp
// 修改前
int GetUnlockStatus()
{
    std::string parament = " unlock-state; echo $?";
    std::string ShellCommand = "/usr/bin/cyberdog_get_info" + parament;
    int result = GetShellCommandResult(ShellCommand);
    return result;
}

// 修改后：始终返回"已解锁"
int GetUnlockStatus()
{
    INFO("GetUnlockStatus() bypassed, always return unlocked");
    return 1;  // 非0 = 已解锁
}
```

```cpp
// 修改前
int UnlockAccess()
{
    std::string parammentKey = "/SDCARD/cyberdog-key; echo $?";
    std::string ShellCommand = "/usr/bin/cyberdog_get_info unlock-request " + parammentKey;
    int result = GetShellCommandResult(ShellCommand);
    return result;
}

// 修改后：始终返回"解锁成功"
int UnlockAccess()
{
    INFO("UnlockAccess() bypassed, always return success");
    return 0;  // 0 = 成功
}
```

**修改 2：OTA 脚本（可选）**

文件：`cyberdog_bringup` 部署后位于 NX 上的 `/opt/ros2/cyberdog/share/cyberdog_ota/scripts/utils/cyberdog_systems.sh`

```bash
# 修改前
function get_not_unlocked()
{
    state=$(sudo /usr/bin/cyberdog_get_info unlock-state | tail -1 | awk '{print $NF}')
    ...
}

# 修改后：始终返回已解锁
function get_not_unlocked()
{
    echo 1  # 非0 = 已解锁
}
```

> **说明**：`/usr/bin/cyberdog_get_info` 是闭源二进制，仅用于解锁状态管理，不影响运动控制、导航、语音等任何实际功能。修改后可选择保留或删除该文件。编译修改后的 `unlock_request` 包并部署到 NX 即可永久生效。

## 编译与部署

### 在 NX 上直接编译 vs Docker 交叉编译

| | **在 NX 上直接编译** | **在电脑上 Docker 交叉编译** |
|---|---|---|
| **前提** | 能 SSH 进 NX | Docker + 小米依赖包可下载 |
| **优势** | 环境已就绪，不需要额外配置 | 不占用机器人资源 |
| **劣势** | NX 算力有限，全量编译较慢 | 小米服务器可能已关闭 |
| **推荐度** | ⭐⭐⭐⭐⭐ | ⭐⭐（有风险） |

### 推荐路径：在 NX 上直接编译

NX 上已经有完整的编译环境（ROS2 Galactic + colcon + 所有依赖库都预装了），不需要 Docker，不需要交叉编译，不需要从小米服务器下载任何东西（Dockerfile 里那些链接可能已失效）。

```bash
# 1. SSH 进 NX（前提：已通过上述方式解锁）
ssh mi@192.168.55.1   # 密码 123

# 2. 克隆本仓库
cd ~
git clone https://github.com/duoduosysa/cyberdog2.git cyberdog_ws
cd cyberdog_ws

# 3. 加载编译环境
source /opt/ros2/galactic/setup.bash
source /opt/ros2/cyberdog/setup.bash   # 加载已安装的 cyberdog 依赖

# 4a. 只编译单个你修改的包（推荐，快）
colcon build --merge-install --packages-select <包名>
# 几分钟搞定

# 4b. 或编译所有包（慢，NX 上可能要 30-60 分钟）
colcon build --merge-install

# 5. 替换到系统目录
sudo cp -rf install/lib/<包名> /opt/ros2/cyberdog/lib/
sudo cp -rf install/share/<包名> /opt/ros2/cyberdog/share/

# 6. 重启生效
sudo reboot
```

> **额外提醒**：NX 的存储空间有限（eMMC），`git clone` 全量代码 + 编译产物会占不少空间。建议先 `df -h` 查看剩余空间，如果不够可以只编译需要修改的包（`--packages-select`）。

### Docker 交叉编译（备选）

进入 tools 目录下，可使用 Dockerfile 文件编译镜像，具体步骤可参考：[镜像编译](https://github.com/MiRoboticsLab/blogs/blob/rolling/docs/cn/dockerfile_instructions_cn.md)

> **注意**：Dockerfile 中的依赖包从 `cnbj2m.fds.api.xiaomi.com` 下载，该服务器可能已关闭。如遇下载失败，请优先使用 NX 上直接编译的方式。

## 使用示例
可作为开发用参考demo，非原生功能

| demo名称                | 功能描述     | github地址                                             |
| ----------------------- | ------------ | ------------------------------------------------------ |
| grpc_demo               | grpc通信例程 | https://github.com/WLwind/grpc_demo                    |
| audio_demos             | 语音例程     | https://github.com/jiayy2/audio_demos                  |
| cyberdog_ai_sports_demo | 运动计数例程 | https://github.com/Ydw-588/cyberdog_ai_sports_demo     |
| cyberdog_face_demo      | 人脸识别demo | https://github.com/Ydw-588/cyberdog_face_demo          |
| nav2_demo               | 导航         | https://github.com/duyongquan/nav2_demo                |
| cyberdog_vp_demo        | 可视化编程   | https://github.com/szh-cn/cyberdog_vp_demo             |
| cyberdog_action_demo    | 手势动作识别 | https://github.com/liangxiaowei00/cyberdog_action_demo |


## 文档

架构设计请参考： [平台架构](https://miroboticslab.github.io/blogs/#/cn/cyberdog_platform_software_architecture_cn)

详细文档请参考：[项目博客文档](https://miroboticslab.github.io/blogs/#/)

刷机请参考：[刷机包下载](https://s.xiaomi.cn/c/8JEpGDY8)

## 版权和许可

四足开发者平台遵循Apache License 2.0 开源协议。详细的协议内容请查看 [LICENSE.txt](./LICENSE.txt)

thirdparty：[第三方库](https://github.com/MiRoboticsLab/blogs/blob/rolling/docs/cn/third_party_library_management_cn.md)

## 联系方式

dukun1@xiaomi.com

liukai21@xiaomi.com

tianhonghai@xiaomi.com

wangruheng@xiaomi.com
