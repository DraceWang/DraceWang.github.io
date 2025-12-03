---
title: 在 Kali OS 上安装与配置 Strix 实践笔记    
categories: cybersecurity   
tags: 
    - kali 
    - Strix
    - security 
date: 2025-12-3 14:13:00
cover: https://i.loli.net/2020/10/27/9rtGQh87JIpNVbE.jpg
---

# 在 Kali OS 上安装与配置 Strix 实践笔记

Strix 是一款强大的应用安全测试工具，它能够与大型语言模型（LLM）结合，对目标应用进行自动化或半自动化的渗透测试。本笔记旨在记录在 Kali Linux 操作系统上安装和配置 `strix-agent` 的完整过程，并对每一步操作进行详细的解释，以便于学习和实践。

## 准备工作：安装kali，开启 SSH 远程登录

安装kali就不讲了，基本就是下一步的事，具体根据自己的情况调整即可。

在进行复杂的安装和配置前，开启 SSH 服务是一个好习惯。这能让你通过远程终端（如 PuTTY、Xshell 或 VSCode Remote）连接到你的 Kali 系统，从而方便地复制粘贴命令、传输文件和进行多任务操作。

### 1. 修改 SSH 配置文件

首先，我们需要修改 SSH 服务的核心配置文件 `sshd_config`。

```bash
sudo nano /etc/ssh/sshd_config
```

打开文件后，你需要找到并修改以下两个关键参数，以允许密码验证登录：

-   **`#PermitRootLogin prohibit-password`**
    -   **修改为**：`PermitRootLogin yes`
    -   **说明**：此项允许 root 用户通过 SSH 登录。在安全的生产环境中，通常不建议这样做，但在个人实验环境中可以临时开启以方便操作。

-   **`#PasswordAuthentication yes`**
    -   **取消注释**：确保该行前面没有 `#` 号，并设置为 `yes`。
    -   **说明**：此项开启了使用用户名和密码进行身份验证的登录方式。

修改完成后，按 `Ctrl+X`，然后按 `Y` 和回车键保存并退出 `nano` 编辑器。

### 2. 重启并设置 SSH 服务

修改配置后，需要重启 SSH 服务使其生效，并设置为开机自启动。

```bash
# 重启 SSH 服务
sudo systemctl restart ssh

# 设置 SSH 服务开机自启
sudo systemctl enable ssh

# 检查 SSH 服务状态
sudo systemctl status ssh
```

当看到状态显示为 `active (running)` 时，说明 SSH 服务已成功启动。

## 核心依赖：安装 Docker

根据 Strix 的设计，它可能依赖 Docker 容器来执行某些隔离的测试任务或运行其组件。因此，安装 Docker 是必不可少的一步。

我们将使用官方推荐的方法，并配置阿里云的镜像源以加速国内的下载速度。

### 1. 更新软件包列表与安装依赖

首先，更新 `apt` 包索引，并安装一些允许 `apt` 通过 HTTPS 使用仓库所必需的软件包。

```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common
```
- **说明**：
  - `apt-transport-https`：让 `apt` 支持 `https://` 协议的源。
  - `ca-certificates`：允许系统检查安全证书。
  - `curl`：用于从网络下载文件。
  - `software-properties-common`：提供了 `add-apt-repository` 命令。

### 2. 添加 Docker 的 GPG 密钥

为了确保我们下载的 Docker 软件包是官方且未经篡改的，需要添加 Docker 官方（此处使用阿里云镜像）的 GPG 密钥。

```bash
curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/debian/gpg | sudo apt-key add -
```

### 3. 添加 Docker 的软件源

接下来，将 Docker 的软件仓库地址添加到你的系统中。这样 `apt` 就能从中找到并安装 Docker。
**注意**：Kali 是基于 Debian 的，因此我们应使用 Debian 的仓库。`bookworm` 是 Debian 12 的代号，请根据你的 Kali 版本对应的 Debian 代号进行调整。

```bash
sudo add-apt-repository "deb [arch=amd64] https://mirrors.aliyun.com/docker-ce/linux/debian bookworm stable"
```

### 4. 安装 Docker 引擎

万事俱备，现在可以正式安装 Docker 了。

```bash
# 再次更新软件包列表，以包含刚刚添加的 Docker 仓库
sudo apt-get update

# 安装 Docker CE (Community Edition)
sudo apt-get install -y docker-ce docker-ce-cli containerd.io
```

### 5. 启动并配置 Docker 服务

与 SSH 类似，我们需要启动 Docker 并将其设置为开机自启。

```bash
sudo systemctl start docker
sudo systemctl enable docker
```

### 6. 处理 Docker 用户权限

默认情况下，你需要 `sudo` 权限才能运行 `docker` 命令。为了方便，可以将当前用户添加到 `docker` 用户组中，这样就可以直接执行 `docker` 命令了。

```bash
# 将当前用户（$USER）添加到 docker 组
sudo usermod -aG docker $USER
```

- **重要提示**：执行此命令后，你需要 **退出当前终端会话并重新登录**（或者重启系统），这样新的用户组权限才会生效。

## 安装 Strix Agent

现在，我们可以安装主角——`strix-agent` 了。推荐使用 `pipx` 进行安装，因为它可以将 Python 应用安装在独立隔离的环境中，避免与系统或其他项目的依赖产生冲突。

### 1. 安装 pipx

如果你的系统中还没有 `pipx`，可以通过 `pip` 安装：

```bash
python3 -m pip install --user pipx
python3 -m pipx ensurepath
```

### 2. 使用 pipx 安装 strix-agent

```bash
pipx install strix-agent
```

### 3. 配置 Strix

Strix 需要知道使用哪个大语言模型（LLM）以及对应的 API 密钥。这通过环境变量来配置。

```bash
# 设置要使用的 LLM 模型，例如 OpenAI 的 gpt-4
export STRIX_LLM="openai/gpt-4"

# 设置你的 API 密钥
export LLM_API_KEY="sk-YourActualApiKey"
```

- **说明**：
  - `STRIX_LLM`：定义了后端使用的模型。格式通常是 `provider/model_name`。请根据你拥有的 API 权限填写。
  - `LLM_API_KEY`：你的模型提供商的 API 密钥。
- **建议**：为了不用每次都手动设置，建议将这两行 `export` 命令添加到你的 shell 配置文件中，例如 `~/.bashrc` 或 `~/.zshrc`，然后执行 `source ~/.bashrc` 使其永久生效。

> [!IMPORTANT]
>
> #### 使用deepseek等国内大模型需要重复设置两个key
>
> ```
> export DEEPSEEK_API_KEY="sk-YourActualApiKey"
> ```
>
> 具体参考https://docs.litellm.ai/docs/providers中 :
>
> >
> >
> > # API KEY #
> >
> > ```
> > # env variable
> > os.environ['DEEPSEEK_API_KEY']
> > ```
>


### 4. 测试运行

一切就绪后，可以执行一个简单的命令来测试 Strix 是否正常工作。

```bash
strix --target https://public-firing-range.appspot.com
```
- **说明**：
  - `--target` 参数指定了你要测试的目标 URL。
  - `public-firing-range.appspot.com` 是 Google 提供的一个公开的、用于测试 Web 应用扫描器的网站，用它来做初次测试是安全且合适的。

如果一切顺利，你将看到 Strix 开始分析目标并输出测试信息。至此，整个安装和配置过程就完成了。
