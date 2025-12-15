---
title: Windows下的终极AI环境：WSL2+Docker+Python 
categories: AI  
tags: 
    - windows 
    - docker
    - wsl2
    - python 
date: 2025-12-15 14:39:00
cover: https://s2.loli.net/2025/12/15/p2CAFnvzDYdQraS.webp
---

# Windows下的终极AI环境：WSL2+Docker+Python

### 写在前面

最近，AI大火，我也想凑凑热闹，搞些AI的项目，比如本地知识库，n8nAI自动工作流等等，于是我就需要这么一个AI环境。最终选择了 **WSL2 (Windows Subsystem for Linux 2)** 与 **Docker Desktop** 相结合的方案，它不仅解决了环境冲突等各方面不兼容的痛点，更带来了容器化“一次构建，到处运行”的巨大优势。

### 第一章：奠定基石 - 启用并安装 WSL2（可选，Windows自带）

一切的起点，是在 Windows 系统中嵌入一个真正的 Linux 内核。WSL2 通过轻量级虚拟化实现了这一点，性能远超第一代。

**1. 启用必要的 Windows 功能**

首先，必须以**管理员身份**打开 PowerShell 或 CMD，执行两条命令，为 WSL2 和 Docker 启用系统级的虚拟化支持。

```powershell
# 启用 Linux 子系统功能
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart

# 启用虚拟机平台（WSL2 和 Docker 的基石）
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

执行完毕后，**务必重启电脑**，让配置生效。

***目前的windows10和windows11系统的电脑都已经集成了wsl的启动，只是没有安装linux发行版。***

**2. 安装 Linux 发行版**

重启后，我们从 Microsoft Store 安装一个 Linux 发行版，我个人推荐 **Ubuntu 22.04 LTS**，因为它社区庞大、生态稳定。

1.  打开 **Microsoft Store**。
2.  搜索 **Ubuntu 22.04 LTS** 并安装。
3.  首次启动后，系统会提示你创建一个用于 Linux 环境的用户名和密码，请务必记住，后续 `sudo` 操作全靠它。

**3. 将 WSL2 设置为默认版本**(默认安装的就是wsl2)

为确保新安装的系统运行在 WSL2 模式下，需要再次打开**管理员 PowerShell**，执行以下命令：

```powershell
# 将未来所有安装的子系统默认设置为 WSL2 版本
wsl --set-default-version 2

# 检查当前已安装的发行版及其版本
wsl -l -v
```

执行 `wsl -l -v` 后，你应该能看到你的 Ubuntu 版本号（VERSION）显示为 `2`。

### 第二章：容器之力 - 安装Docker Desktop 并集成 WSL2

由于现在多数AI项目都用到了docker，因此在linux子系统中安装docker就必不可少。Docker Desktop 是我们与容器世界交互的窗口，它与 WSL2 的深度集成是整个工作流的核心。

**1. 下载与安装**

前往 [Docker 官网](https://www.docker.com/products/docker-desktop/) 下载 Windows 版的安装包。安装过程中，请确保勾选了 **"Use WSL 2 instead of Hyper-V"** 选项。

**2. 授权 Docker 访问 WSL**

这是至关重要的一步。如果配置不当，你在 WSL 的终端里将无法使用 `docker` 命令，因为它找不到 Docker 守护进程。

1.  启动 Docker Desktop。
2.  点击界面右上角的**齿轮图标 (⚙️ Settings)** 进入设置。
3.  在左侧菜单栏选择 **Resources** -> **WSL Integration**。
4.  确保 **"Enable integration with my default WSL distro"** 已被勾选。
5.  在下方的发行版列表中，**务必确保你的 Ubuntu 发行版开关是打开状态**。
6.  点击 **Apply & Restart** 应用并重启 Docker 服务。

![image-20251215141746390](https://s2.loli.net/2025/12/15/wR8JYWXAjiUmeQg.png)

**验证集成**：
配置完成后，打开你的 Ubuntu 终端，输入 `docker --version`。如果命令成功返回版本号，例如 `Docker version 26.0.0`，则说明 Docker 与 WSL2 的“握手”成功了。

### 第三章：解决顽疾 - 在 WSL 中配置纯净的 Python 环境

很多教程在这一步只提到了 `sudo apt install python3`，但这往往是后续一系列问题的开端。Ubuntu 默认安装的 Python 环境并不“完整”，会导致 `pip` 缺失或在安装某些库时编译失败。

**1. 更新系统源**
这是一个好习惯，确保我们安装的都是最新的软件包。
```bash
sudo apt update && sudo apt upgrade -y
```

**2. 安装 Python “全家桶”**
为了避免后续麻烦，我们一次性把所有必需的组件都安装好。
```bash
sudo apt install python3 python3-pip python3-venv python3-full build-essential -y
```
*   `python3-pip`: Python 的包管理器，必须有。
*   `python3-venv`: 虚拟环境支持库，现代 Python 开发的基石。
*   `python3-full`: 包含了完整的 Python 标准库，可以避免很多意想不到的编译时依赖缺失问题。
*   `build-essential`: 包含了 `gcc` 等编译工具，当 `pip` 安装需要编译 C 扩展的库（如 `numpy`, `pandas`）时，它是必需的。

**3. 使用虚拟环境：告别权限报错**
为了彻底解决 `externally-managed-environment` 这类系统级环境管理报错，最佳实践是**永远在虚拟环境中进行开发**。

```bash
# 1. 创建并进入你的项目目录
mkdir ~/myproject && cd ~/myproject

# 2. 在项目内创建一个名为 venv 的虚拟环境
python3 -m venv venv

# 3. 激活虚拟环境
source venv/bin/activate
```
激活后，你的命令行提示符会带有 `(venv)` 前缀，表明你已处于一个隔离、安全的环境中。在此环境下，`pip` 可以自由使用，不会与系统环境产生任何冲突。

### 第四章：实战演练 - Dockerfile 最佳实践

现在，我们拥有了完美的 Python 开发环境，可以结合 Docker 来打包和部署应用了。

在项目目录（例如 `~/myproject`）中，创建一个 `app.py` 和 `requirements.txt`，然后编写一个 `Dockerfile`：

```dockerfile
# Dockerfile

# 使用官方的轻量级 Python 镜像
FROM python:3.11-slim

# 设置容器内的工作目录
WORKDIR /app

# 将依赖文件复制到容器中
COPY requirements.txt .

# 安装依赖，--no-cache-dir 减小镜像体积
RUN pip install --no-cache-dir -r requirements.txt

# 将项目所有文件复制到容器中
COPY . .

# 容器启动时执行的命令
CMD ["python", "app.py"]
```

在 WSL 终端中，于项目目录下执行构建和运行命令：
```bash
# 构建镜像
docker build -t my-python-app .

# 运行容器
docker run -it --rm my-python-app
```

###  第五章：“偷梁换柱” - Docker换源

由于某些我们众所周知的原因，国内下载docker官方的镜像源经常会遇到失败的情况，因此我们可以使用阿里的镜像加速。

可以访问阿里云官方的[镜像加速器](https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors)，登录你自己阿里云账号，会获取到一个专属的加速器地址。

打开deocker desktop，在系统右下角托盘图标内右键菜单选择 Settings，打开配置窗口后左侧导航菜单选择 Docker Daemon。编辑窗口内的JSON串，填写下方加速器地址：

```
{
  "registry-mirrors": ["https://xxxxxxxxxx.mirror.aliyuncs.com"]
}
```

编辑完成后点击 Apply 保存按钮，等待Docker重启并应用配置的镜像加速器。

### 总结

至此，我们已经成功搭建了一套健壮且高效的本地开发环境。回顾一下，我们获得了：

1.  **一个高性能的 WSL2 Linux 环境**：享受原生 Linux 的开发体验，与 Windows 文件系统无缝互通。
2.  **一个无缝集成的 Docker 环境**：可以直接在 Linux 终端中调用 Docker，进行容器的构建、运行和管理。
3.  **一个纯净且完整的 Python 开发环境**：通过安装“完整”组件和坚持使用 `venv` 虚拟环境，彻底根除了环境污染和权限问题。

这套工作流不仅适用于 Web 后端开发或数据科学，对于需要进行 AI 模型微调（如之前的 MiniCPM 实验）等复杂任务也同样游刃有余。它将“环境配置”这个一次性的阵痛，转化为未来高效开发的长期收益。