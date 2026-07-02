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
├── CMakePresets.json           # Debug / Release / vcpkg 预设（VS Code & VS 共用）
├── vcpkg.json                  # 第三方库清单
├── docs/
│   ├── TeamDev-vscode-config/        # VS Code 配置说明（可独立为 Git 仓库）
│   └── TeamDev-visualstudio-config/  # Visual Studio 配置说明（可独立为 Git 仓库）
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

### 通用工具链

开发与构建前，请先安装以下工具（Windows 10/11）：

| 工具 | 版本要求 | 说明 | 下载 |
|------|----------|------|------|
| [llvm-mingw](https://github.com/mstorsjo/llvm-mingw) | Clang 17+ | C/C++ 编译器（`x86_64`） | [Releases](https://github.com/mstorsjo/llvm-mingw/releases) |
| [CMake](https://cmake.org/) | ≥ 3.20 | 构建系统 | [下载页](https://cmake.org/download/) |
| [vcpkg](https://github.com/microsoft/vcpkg) | 最新 | 第三方库（Boost、Eigen3、fmt） | [安装文档](https://learn.microsoft.com/zh-cn/vcpkg/get_started/get-started) |
| [Git](https://git-scm.com/) | 最新 | 拉取功能包仓库 | [下载页](https://git-scm.com/download/win) |

**vcpkg 常用 triplet**：`x64-mingw-dynamic`（与 llvm-mingw 配套）

**环境变量（按本机路径修改）**：

```text
VCPKG_ROOT=E:/Windows1/vcpkg
VCPKG_DEFAULT_TRIPLET=x64-mingw-dynamic
PATH=%PATH%;<llvm-mingw>/bin;<CMake>/bin
```

llvm-mingw 路径也可通过 CMake 缓存变量 `LLVM_MINGW_ROOT` 指定（见 [`cmake/toolchain-mingw.cmake`](cmake/toolchain-mingw.cmake)）。

### CMake 预设

项目使用 [`CMakePresets.json`](CMakePresets.json) 统一管理配置，两种 IDE 共用同一套预设：

| 预设 | 用途 | 输出目录 |
|------|------|----------|
| `debug` | Debug + llvm-mingw 工具链 | `build/Debug` |
| `release` | Release + llvm-mingw 工具链 | `build/Release` |
| `debug-vcpkg` | Debug + vcpkg manifest 自动安装依赖 | `build/Debug-vcpkg` |
| `release-vcpkg` | Release + vcpkg | `build/Release-vcpkg` |

命令行示例：

```bash
cmake --preset debug
cmake --build --preset debug
```

### IDE 配置仓库

具体编辑器配置已拆分为两个独立说明仓库（可单独克隆，也可使用主工程 `docs/` 内副本）：

| IDE | 仓库 | 本地文档 |
|-----|------|----------|
| **VS Code / Cursor** | [Auzerx/TeamDev-vscode-config](https://github.com/Auzerx/TeamDev-vscode-config) | [docs/TeamDev-vscode-config/README.md](docs/TeamDev-vscode-config/README.md) |
| **Visual Studio 2022** | [Auzerx/TeamDev-visualstudio-config](https://github.com/Auzerx/TeamDev-visualstudio-config) | [docs/TeamDev-visualstudio-config/README.md](docs/TeamDev-visualstudio-config/README.md) |

**VS Code 快速接入**：克隆配置仓库 → 将 `.vscode/` 复制到 TeamDev 根目录 → 修改 `settings.json` 中的 CMake / vcpkg 路径。

**Visual Studio 快速接入**：设置系统环境变量 `VCPKG_ROOT` → 用 VS **打开文件夹** 选择 TeamDev → 工具栏选择 `debug` 预设 → 构建 `teamdev`。

---

### 方式一：VS Code / Cursor（摘要）

- 配置模板：[TeamDev-vscode-config](https://github.com/Auzerx/TeamDev-vscode-config)（含 `settings.json` / `tasks.json` / `launch.json`）
- 推荐扩展：CMake Tools、C/C++、clangd（可选）、CodeLLDB
- 详细步骤见 → **[VS Code 配置说明](docs/TeamDev-vscode-config/README.md)**

---

### 方式二：Visual Studio（摘要）

- 配置说明：[TeamDev-visualstudio-config](https://github.com/Auzerx/TeamDev-visualstudio-config)（含 `CMakeUserPresets.json` 本机覆盖模板）
- 工作负载：桌面 C++ 开发 + CMake 工具
- 详细步骤见 → **[Visual Studio 配置说明](docs/TeamDev-visualstudio-config/README.md)**

---

### 环境对照表

| 配置项 | VS Code / Cursor | Visual Studio |
|--------|------------------|---------------|
| 详细文档 | [vscode-config 仓库](https://github.com/Auzerx/TeamDev-vscode-config) | [visualstudio-config 仓库](https://github.com/Auzerx/TeamDev-visualstudio-config) |
| 预设文件 | [`CMakePresets.json`](CMakePresets.json) | 同左 |
| 本地 IDE 配置 | `.vscode/`（从配置仓库复制） | `.vs/`（本地缓存） |
| 本机路径覆盖 | `settings.json` 内环境变量 | `CMakeUserPresets.json`（可选） |
| 调试 | CodeLLDB + `launch.json` | 内置调试器，启动项 `teamdev.exe` |

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

### 3. 构建与运行

任选一种 IDE（详细配置见下方链接）：

| IDE | 配置仓库 |
|-----|----------|
| VS Code / Cursor | [TeamDev-vscode-config](https://github.com/Auzerx/TeamDev-vscode-config) · [本地文档](docs/TeamDev-vscode-config/README.md) |
| Visual Studio | [TeamDev-visualstudio-config](https://github.com/Auzerx/TeamDev-visualstudio-config) · [本地文档](docs/TeamDev-visualstudio-config/README.md) |

**VS Code / Cursor**

1. 状态栏选择预设 **Debug** 或 **Debug + vcpkg**
2. `Ctrl+Shift+B` 构建目标 `teamdev`
3. F5 启动调试

**Visual Studio**

1. 工具栏选择配置预设 **Debug (仅 MinGW)**
2. 启动项选择 `teamdev.exe` → **生成** → **开始调试**

**命令行**（不依赖 IDE）：

```bash
cmake --preset debug
cmake --build build/Debug
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
| [Auzerx/TeamDev-vscode-config](https://github.com/Auzerx/TeamDev-vscode-config) | VS Code / Cursor 工作区配置模板 |
| [Auzerx/TeamDev-visualstudio-config](https://github.com/Auzerx/TeamDev-visualstudio-config) | Visual Studio CMake 开发说明 |
| [Auzerx/Msg](https://github.com/Auzerx/Msg.git) | 消息与协议定义 |
| [Auzerx/Utils](https://github.com/Auzerx/Utils.git) | 通用工具库 |
| [Auzerx/RCS-Net](https://github.com/Auzerx/RCS-Net.git) | RCS 网络通信 |

---

## 许可证

各功能包仓库遵循各自许可证；主工程许可证以仓库根目录声明为准。

---

## 联系方式

凿岩台车项目组 — 如有问题或协作需求，请通过项目组内部渠道联系维护者。
