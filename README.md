# 凿岩台车项目组

## 简介

凿岩台车项目组专注于**地下工程凿岩台车（Rock Drilling Jumbo）**的软件开发与系统集成。台车是集钻孔、推进、导航、通信与控制于一体的工程装备，广泛应用于隧道开挖、矿山掘进等场景。

本仓库 **TeamDev** 是项目组的**主调度工程**：以 CMake 为核心，统一编排自研功能包与第三方依赖，支撑台车各子系统的模块化开发与联调验证。

---

## 项目目标

| 方向 | 说明 |
|------|------|
| 模块化 | 将通信协议、通用工具、网络传输等能力拆分为独立 Git 功能包，按需组合 |
| 可复用 | 功能包可在多台车项目、多个应用之间共享，降低重复开发成本 |
| 可集成 | 主工程通过清单配置拉取、编译、链接功能包，无需手工维护大量子模块路径 |
| 工程化 | 统一 C++17、CMake Presets、vcpkg 依赖管理与跨包构建顺序 |

---

## 业务领域

项目组软件覆盖凿岩台车核心控制与数据链路，主要包括：

- **运动与机构**：推进梁、凿岩机伸缩、各关节位姿与坐标变换
- **导航与测量**：棱镜定位、台车坐标系、断面与偏距计算
- **通信与协议**：CAN 报文、设备状态与工艺参数的消息定义（`Msg` 包）
- **系统通信**：RCS 网络传输、设备间数据发布与订阅（`RCSNet` 等）
- **基础能力**：数学运算、队列、文件 IO、数据拼接等通用模块（`Utils` 包）

---

## 技术架构

```
┌─────────────────────────────────────────────────────────────┐
│                     TeamDev 主调度工程                        │
│  src/main.cpp  ·  config/packages.yaml  ·  vcpkg.json        │
├──────────────────────────┬──────────────────────────────────┤
│   自研功能包（Git）        │   第三方库（vcpkg）                │
│   Msg    消息/协议定义     │   Boost · Eigen3 · fmt           │
│   Utils  数学/队列/工具    │                                  │
│   RCSNet 网络通信（可选）   │                                  │
└──────────────────────────┴──────────────────────────────────┘
```

### 配置阶段流程

1. 解析 `config/packages.yaml`，拉取远程 Git 功能包至 `packages/<包名>/`
2. 按 `depends_on` 确定编译顺序，`add_subdirectory` 注册各功能包
3. 生成 `build/<Preset>/generated/teamdev_packages.gen.h`，注入调度管线
4. 链接功能包 CMake 目标，编译主程序 `teamdev`

### 两类依赖

| 类型 | 配置位置 | 典型用途 |
|------|----------|----------|
| Git 功能包 | `config/packages.yaml` | Msg、Utils、RCSNet 等自研模块 |
| vcpkg 库 | `vcpkg.json` + `cmake/VcpkgDeps.cmake` | Boost、Eigen3、fmt 等通用第三方库 |

---

## 功能包一览

### Msg

凿岩台车**消息与协议定义**库，包含 CAN 报文结构、关节/机构数据类型、导航与工艺相关枚举等，是各控制模块共享的「数据语言」。

- 仓库：<https://github.com/Auzerx/Msg.git>
- CMake 目标：`Msg::msg`
- 依赖：`Utils`（Queue 等）

### Utils

项目组**通用工具集**，提供数学、队列、文件 IO、数据拼接、性能测试等基础能力。

- 仓库：<https://github.com/Auzerx/Utils.git>
- CMake 目标：`Utils::Math`、`Utils::Queue`、`Utils::FileIo` 等
- 子模块：Math、Queue、FileIo、DataConcatention、Benchmark 等

### RCSNet（可选）

RCS **网络通信库**，支持 TCP/UDP 等传输，用于设备间数据发布与订阅。

- 仓库：<https://github.com/Auzerx/RCS-Net.git>
- CMake 目标：`RcsNetwork::rcs_network`
- 说明：仓库根目录为应用工程时，需在 `packages.yaml` 中配置 `subdir: RcsNetwork`

---

## 目录结构

```
TeamDev/
├── CMakeLists.txt              # 主工程入口
├── CMakePresets.json           # Debug / Release / vcpkg 预设
├── vcpkg.json                  # 第三方库清单
├── config/
│   └── packages.yaml           # 功能包清单（核心配置）
├── cmake/
│   ├── FetchPackages.cmake     # Git 拉取与本地包解析
│   ├── RegisterFeatures.cmake  # 注册、排序、链接功能包
│   ├── VcpkgDeps.cmake         # vcpkg 依赖链接
│   ├── toolchain-mingw.cmake   # llvm-mingw 工具链
│   └── adapters/               # 功能包调度适配器
├── packages/                   # 功能包目录（Git 拉取 + 本地包）
├── src/
│   └── main.cpp                # 主程序入口
└── build/                      # 构建输出（不提交 Git）
```

---

## 开发环境

| 工具 | 说明 |
|------|------|
| 编译器 | llvm-mingw Clang（C++17） |
| 构建 | CMake 3.20+，MinGW Makefiles |
| IDE | VS Code / Cursor + CMake Tools |
| 包管理 | vcpkg（`x64-mingw-dynamic` triplet） |

### 环境变量（参考）

```text
VCPKG_ROOT=E:/Windows1/vcpkg
VCPKG_DEFAULT_TRIPLET=x64-mingw-dynamic
```

路径请按本机实际安装位置修改。`.vscode` 为本地 IDE 配置，不纳入版本库。

---

## 快速开始

### 1. 克隆主工程

```bash
git clone <TeamDev 仓库地址>
cd TeamDev
```

### 2. 配置功能包

编辑 `config/packages.yaml`，启用或禁用所需功能包：

```yaml
packages:
  - name: Utils
    git: https://github.com/Auzerx/Utils.git
    version: main
    enabled: true

  - name: Msg
    git: https://github.com/Auzerx/Msg.git
    version: main
    enabled: true
    depends_on: Utils
    target: Msg::msg
```

### 3. 构建

在 VS Code 状态栏选择 CMake 预设 **Debug** 或 **Debug + vcpkg**，保存 `CMakeLists.txt` 触发自动配置，然后构建目标 `teamdev`。

命令行示例：

```bash
cmake --preset debug
cmake --build build/Debug
```

### 4. 运行

```bash
./build/Debug/teamdev.exe
```

主程序将按 `packages.yaml` 中的配置，依次调度已启用功能包的 `run()` 入口。

---

## 新增功能包

### Git 自研包

1. 在 `config/packages.yaml` 添加条目（`name`、`git`、`version`、`target` 等）
2. 如需接入调度管线，在 `cmake/adapters/` 添加 `<Name>.hpp` 适配器
3. 重新 CMake Configure

### vcpkg 第三方库

1. 在 `vcpkg.json` 添加包名
2. 在 `cmake/VcpkgDeps.cmake` 中 `find_package` 并 `target_link_libraries`
3. 使用带 vcpkg 的 CMake 预设重新配置

---

## 协作规范

- **功能包独立仓库**：每个功能包维护独立 Git 仓库与 CMake 导出目标（如 `Msg::msg`）
- **主工程只编排**：TeamDev 负责拉取、排序、链接，不承载业务实现细节
- **命名约定**：包名简短清晰；远程拉取的包位于 `packages/<name>/`，不放入 `third_party/`
- **提交范围**：`packages/` 下 Git 拉取的目录默认不提交；仅提交本地自研包或清单变更

---

## 相关仓库

| 仓库 | 说明 |
|------|------|
| [Auzerx/Msg](https://github.com/Auzerx/Msg.git) | 消息与协议定义 |
| [Auzerx/Utils](https://github.com/Auzerx/Utils.git) | 通用工具库 |
| [Auzerx/RCS-Net](https://github.com/Auzerx/RCS-Net.git) | RCS 网络通信 |

---

## 许可证

各功能包仓库遵循各自许可证；主工程许可证以仓库根目录声明为准。

---

## 联系方式

凿岩台车项目组 — 如有问题或协作需求，请通过项目组内部渠道联系维护者。
