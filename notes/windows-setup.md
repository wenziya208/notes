终端设置中，如何避免路径冲突？
包管理器如何适配不同开发场景？
环境变量设置后如何验证生效？
Windows 开发环境配置备忘
终端选择
Windows Terminal 是主力，设置默认值：打开设置 → 启动 → 配置文件 → 默认配置文件 → 将“启动目录”设为当前工作区路径（如 C:\dev\myproject），默认 shell 选 PowerShell 或 WSL。
PowerShell 7 安装命令：winget install Microsoft.PowerShell，装完后在 Windows Terminal 中设为默认配置文件。
保留旧 cmd：在 Terminal 中单独添加“命令提示符”配置文件，路径填 C:\Windows\System32\cmd.exe，避免老脚本不兼容。
包管理器
winget：系统自带，常用命令：
搜索包：winget search <name>
安装包：winget install <PackageId>
更新全部：winget upgrade --all
Chocolatey（备用，适合批量装软件）：
安装脚本（管理员 PowerShell）：
powershell
Set-ExecutionPolicy Bypass -Scope Process -Force; `
[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; `
iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
安装包：choco install <package> -y
Python pip：确保 Python 安装时勾选“Add Python to PATH”，之后用 python -m pip install <package>。
环境变量
查看当前变量：PowerShell 中运行 Get-ChildItem Env:。
永久设置（用户级）：
系统设置 → 高级系统设置 → 环境变量 → 用户变量，新建如 MY_APP_HOME=C:\dev\myapp。
或在 PowerShell 中：[Environment]::SetEnvironmentVariable('MY_APP_HOME','C:\dev\myapp','User')。
JAVA_HOME 之类的开发必备变量，装完 JDK 后必须加，并在 Path 中加 %JAVA_HOME%\bin。
路径问题
长路径启用：Win10/11 默认限制 260 字符，需开启：
注册表 HKLM\SYSTEM\CurrentControlSet\Control\FileSystem\LongPathsEnabled 设为 1，或组策略（gpedit.msc）→ 计算机配置 → 管理模板 → 系统 → 文件系统 → 启用长路径。
或在 PowerShell（管理员）中：New-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1 -PropertyType DWORD。
避免路径空格：项目目录用反斜杠转义或直接避开空格，如 C:\dev\my_project 优于 C:\dev\My Project。
Git 换行符
全局配置（防止 CRLF 导致跨平台问题）：
bash
git config --global core.autocrlf false
git config --global core.safecrlf true
仓库级配置：进入项目目录，git config core.autocrlf false，再配合 .gitattributes 写明规则，如 * text=auto。
WSL 相关
安装 WSL2（管理员 PowerShell）：
powershell
wsl --install -d Ubuntu
装完重启，设置 Linux 用户名和密码。
设置默认版本：wsl --set-default-version 2（若已装旧版）。
访问 Windows 文件：在 WSL 中挂载点为 /mnt/c/，避免在 Linux 中直接编辑 Windows 文件，速度慢且可能引发 inotify 问题。
WSL 与 Windows 互转：wsl --export <Distro> <tar> 导出，wsl --import <Distro2> <InstallDir> <tar> 导入。
常见报错处理
PATH 不生效：装完工具后新开终端，或手动重开 Terminal 会话；若仍不生效，检查用户变量和系统变量是否重复，系统变量优先级高于用户变量。
WSL 启动失败：wsl --status 查看版本，若 Hyper-V 冲突，在 Windows 功能中启用“Virtual Machine Platform”和“适用于 Linux 的 Windows 子系统”，再重启。
Git 报错 fatal: detected dubious ownership：
bash
git config --global --add safe.directory "*"
Python 模块找不到：检查是否用对了解释器，在 VS Code 里选右下角的解释器路径，确认虚拟环境已激活。
npm install 慢或报错：切国内镜像，npm config set registry https://registry.npmmirror.com，或用 --registry 参数。
请帮我写一份 Markdown 笔记，主题是「Windows 开发环境配置备忘」，要求：
1. 面向在 Windows 上做开发的人
2. 覆盖：终端选择、包管理器、环境变量、路径问题、Git 换行符、
WSL 相关、常见报错处理
3. 每条给具体做法（命令或设置路径），不要泛泛而谈
4. 语气像自己的备忘，可以有"记一下免得又忘"这类话
5. 不要 AI 腔，不要 emoji，不要结尾总结
直接输出 Markdown 正文，从一级标题开始。