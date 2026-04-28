# Rocky Linux Notes

<details>
<summary><h2">👶基础环境</h2></summary>

### 🧰 使能 sudo

#### 执行

```bash
su # 切换到 root 用户
nano /etc/sudoers # 编辑 sudoers 文件
```

在 `root ALL=(ALL) ALL` 下面添加一行:

> `😀USERNAME😀`    ALL=(ALL)       ALL

保存并退出 nano: `Ctrl + X -> Y -> Enter`

#### 验证

```bash
su $(whoami) # 切换回普通用户
sudo whoami # 验证是否成功使能 sudo
```

### 🖥️更改主机名

#### 执行
```bash
 sudo hostnamectl set-hostname `🖥️HOSTNAME🖥️` # 设置主机名
 sudo nano /etc/hosts # 编辑 hosts 文件
```

在 `127.0.0.1 localhost` 下面添加一行:

> 127.0.0.1       `🖥️HOSTNAME🖥️`

保存并退出 nano: `Ctrl + X -> Y -> Enter`

#### 验证

```bash
 hostnamectl # 验证主机名是否成功更改
```

### 🌏dnf包管理器换源

#### 执行

参考[第三方镜像源列表](https://mirrors.rockylinux.org/mirrormanager/mirrors)，选择合适的cn镜像源，例如[USTC 中科大](https://mirrors.ustc.edu.cn/help/rocky.html#_5)

```bash
sudo sed -e 's|^mirrorlist=|#mirrorlist=|g' \
    -e 's|^#baseurl=http://dl.rockylinux.org/$contentdir|baseurl=https://mirrors.ustc.edu.cn/rocky|g' \
    -i.bak \
    /etc/yum.repos.d/rocky-extras.repo \
    /etc/yum.repos.d/rocky.repo
    
sudo dnf makecache # 清理并重建缓存
```

### 🔄更新

```bash
sudo dnf -y upgrade dnf # 更新 dnf 包管理器
sudo dnf -y update # 更新系统软件包
```

### ☁️Samba

#### 安装

```bash
sudo dnf -y install samba samba-common samba-client # 安装 samba
sudo mkdir -p /home/samba-share # 创建共享目录

sudo chmod -R 755 /home/samba-share # 设置共享目录权限
sudo chown -R  nobody:nobody /home/samba-share # 设置共享目录所有者
sudo chcon -t samba_share_t /home/samba-share # 设置共享目录标签

sudo nano /etc/samba/smb.conf # 编辑 smb 配置文件
```

在文件中添加以下内容：

> ```conf
> # See smb.conf.example for a more detailed config file or
> # read the smb.conf manpage.
> # Run 'testparm' to verify the config is correct after
> # you modified it.
> #
> # Note:
> # SMB1 is disabled by default. This means clients without support for SMB2 or
> # SMB3 are no longer able to connect to smbd (by default).
> 
> [global]
>         workgroup = WORKGROUP
>         server string = Samba Server %v
>         netbios name = rocky-9
>         security = user
>         map to guest = bad user
>         passdb backend = tdbsam
>         ntlm auth = true
>         printing = cups
>         printcap name = cups
>         load printers = yes
>         cups options = raw
>         unix chardeet = UTF-8
> 
> [homes]
>         comment = Home Directories
>         valid users = %S, %D%w%S
>         browseable = No
>         read only = No
>         inherit acls = Yes
> 
> [printers]
>         comment = All Printers
>         path = /var/tmp
>         printable = Yes
>         create mask = 0600
>         browseable = No
> 
> [print$]
>         comment = Printer Drivers
>         path = /var/lib/samba/drivers
>         write list = @printadmin root
>         force group = @printadmin
>         create mask = 0664
>         directory mask = 0775
> 
> [Shared]
>         path = /home/samba-share
>         browsable = yes
>         writable = yes
>         valid users = 😀USERNAME😀
>         guest ok = yes
>         guest only = yes
>         force create mode = 777
>         force directory mode = 777
> ```
>

```bash
sudo systemctl start smb nmb # 启动 samba 服务
sudo systemctl enable smb nmb # 开机自启 samba 服务

systemctl status smb nmb # 查看 samba 服务状态

sudo firewall-cmd --permanent --add-service=samba # 开放 samba 服务
sudo firewall-cmd --reload # 重载防火墙规则
sudo firewall-cmd --list-services # 查看开放的服务

sudo smbpasswd -a $(whoami) # 设置 samba 用户密码
sudo systemctl restart smb nmb --now # 重启 samba 服务
```

#### 远程访问共享文件夹

Win+R -> smb://`🖥️IP🖥️`/Shared -> 输入密码

### 💻RDP 远程桌面

#### 安装

```bash
sudo dnf -y install epel-release # 安装 epel 源
sudo dnf makecache # 清理并重建缓存
sudo dnf -y install xrdp # 安装 xrdp
sudo systemctl start xrdp # 启动 xrdp 服务
sudo systemctl enable xrdp # 开机自启 xrdp 服务

systemctl status xrdp # 查看 xrdp 服务状态

sudo firewall-cmd --permanent --add-port=3389/tcp # 开放 3389 端口
sudo firewall-cmd --reload # 重载防火墙规则

sudo firewall-cmd --list-ports # 查看开放的端口
```

#### 远程连接

Win+R -> mstsc -> `🖥️IP🖥️`

###  添加标题栏最小化最大化按钮

#### 安装

```bash
sudo dnf -y install gnome-tweaks

gnome-tweaks
```

在 gnome-tweaks 中启用 “Window Title Buttons” 选项。

### GitHub 加速

[Watt Toolkit / Steam++](https://steampp.net/download)是一个包含多种 Steam 工具功能的工具箱，其中包含 GitHub 加速器。

#### 安装

```bash
curl -sSL https://steampp.net/Install/Linux.sh | bash # 安装 Watt Toolkit

sudo cp /home/a/.local/share/Steam++/Plugins/Accelerator/SteamTools.Certificate.cer /etc/pki/ca-trust/source/anchors/ # 复制证书到系统信任库

sudo update-ca-trust extract # 刷新系统信任库
sudo systemctl restart NetworkManager # 重启网络管理器

openssl x509 -in /etc/pki/ca-trust/source/anchors/SteamTools.Certificate.cer -text -noout # 验证证书
```

#### Firefox 导入证书

设置->隐私与安全->安全->查看证书->导入
/home/`😀USERNAME😀`/.local/share/Steam++/Plugins/Accelerator

### Homebrew

[Homebrew](https://brew.sh/)是一个开源的包管理器，可以帮助你安装命令行软件。

#### 安装

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)" # 安装 Homebrew
(echo; echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"') >> /home/`😀USERNAME😀`/.zshrc # 配置环境变量
eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)" # 刷新环境变量
sudo dnf -y groupinstall 'Development Tools' # 安装开发工具
```

### 🐧fastfetch

[fastfetch](https://github.com/fastfetch-cli/fastfetch)是类似于neofetch，但更快，因为主要是用C编写的。

#### 安装

```bash
sudo dnf -y install fastfetch
```

#### 验证

```bash
fastfetch
```

### 💪oh-my-zsh ~~bash~~

[oh-my-zsh](https://ohmyz.sh/#install)是一个令人愉快的，开源的，社区驱动的框架，用于管理您的Zsh配置。它捆绑了数千个有用的功能，帮助程序，插件，主题和一些让你大喊的东西。

```bash
sh -c "$(wget -O- https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" # 安装 oh-my-zsh

nano ~/.zshrc # 编辑 zsh 配置文件
```

参考[Settings](https://github.com/ohmyzsh/ohmyzsh/wiki/Settings)配置，例如：

> ZSH_THEME="darkblood"
>
>plugins=(
> colorize
> command-not-found
> compleat
> copyfile
> cp
> dnf
> docker
> docker-compose
> docker-machine
> emoji
> encode64
> fzf
> git
> github
> gitignore
> git-lfs
> history
> history-substring-search
> npm
> pip
> python
> sudo
> )

保存并退出 nano: `Ctrl + X -> Y -> Enter`

重新连接ssh

Ctrl+R：fzf + 历史搜索命令
Ctrl+T：fzf + 补全路径名
Alt+C：fzf + 改变工作路径

### ✏micro ~~nano~~

[micro](https://github.com/zyedidia/micro)是一款基于终端的文本编辑器，旨在易于使用和直观，同时还利用了现代终端的功能。它是一个单一的、包含电池的静态二进制文件，没有依赖关系;您可以立即下载并使用它！

#### 安装

```bash
 cd /usr/bin

sudo curl https://getmic.ro | bash

/usr/bin/micro --version
```

#### zshrc 配置

```bash
nano ~/.zshrc # 编辑 zsh 配置文件
```

在文件中添加一行:

> alias nano='/usr/bin/micro'


```bash
source ~/.zshrc # 重新加载 zsh 配置文件
```

#### 验证


```bash
nano ~/.zshrc # 验证 nano 命令是否成功映射到 micro
```

### FNM

[FNM](https://github.com/Schniz/fnm)是快速简单的Node.js版本管理器，Rust编写。

#### 安装

```bash
brew install fnm
```

#### zshrc 配置

```bash
nano ~/.zshrc # 编辑 zsh 配置文件
```

在文件中添加一行:

> eval "$(fnm env --use-on-cd)"

```bash
source ~/.zshrc # 重新加载 zsh 配置文件
```

#### 验证

```bash
fnm --version
```

### 🫥Node.js

[Node.js](https://nodejs.org/docs/latest/api/)是一个免费的、开源的、跨平台的JavaScript运行时环境，允许开发人员创建服务器、Web应用程序、命令行工具和脚本。

#### 安装

```bash
fnm use --install-if-missing 22
```

#### 验证

```bash
node -v
npm -v
```

### pm2

[pm2](https://pm2.keymetrics.io/docs/usage/quick-start/)是一个守护进程管理器，可帮助您管理和保持应用程序全天候在线。


#### 安装

```bash
npm install pm2 -g
```

### 🐍Python

[miniconda](https://docs.conda.io/en/latest/miniconda.html)是一个免费的conda最小安装程序。它是Anaconda的一个小型引导版本，只包含conda、Python、它们所依赖的包，以及少量其他有用的包（如pip、zlib等）。如果你需要更多的软件包，使用 conda install 命令从Anaconda的公共仓库中默认提供的数千个软件包中安装，或者从其他渠道安装，如conda-forge或bioconda。

#### 安装

```bash
mkdir -p ~/miniconda3
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O ~/miniconda3/miniconda.sh # 下载安装脚本
bash ~/miniconda3/miniconda.sh -b -u -p ~/miniconda3 # 运行安装脚本
rm -rf ~/miniconda3/miniconda.sh

~/miniconda3/bin/conda init bash # 配置 conda 环境
~/miniconda3/bin/conda init zsh # 配置 conda 环境

```

#### 验证

重新连接ssh

```bash
conda info --envs
conda activate base
python --version
```

### 🤡thefuck

[thefuck](https://github.com/nvbn/thefuck)是一个用于自动纠正命令行的工具，它可以帮助你避免敲错命令，并帮助你修复它们。

#### 安装

```bash
brew install thefuck
```

#### zshrc 配置

```bash
nano ~/.zshrc # 编辑 zsh 配置文件
```

在文件中添加一行:

> eval "$(thefuck --alias)"

```bash
source ~/.zshrc # 重新加载 zsh 配置文件
```

#### 验证

```bash
git bach
fuck
```

### 🧐bottom ~~top~~

[bottom](https://github.com/ClementTsang/bottom)一个可定制的跨平台的图形化进程/系统监视器的终端。

#### 安装

```bash
sudo dnf copr enable atim/bottom -y && sudo dnf -y install bottom
```

#### 验证

```bash
bottom
```

### 📘tldr ~~man~~

[tldr](https://github.com/tldr-pages/tldr)是一个由社区维护的命令行工具帮助页面的集合，旨在成为传统手册页的一个更简单、更易于访问的补充。

```bash
brew install tlrc
```

#### zshrc 配置

```bash
nano ~/.zshrc # 编辑 zsh 配置文件
```

在文件中添加一行:

> alias man='tldr'

```bash
source ~/.zshrc # 重新加载 zsh 配置文件
```

#### 验证

```bash
man docker
```

### 📃eza ~~ls~~

[eza](https://github.com/eza-community/eza)是一个现代化的，维护的替代ls。

#### 安装

```bash
sudo dnf -y install eza
```

#### zshrc 配置

```bash
nano ~/.zshrc # 编辑 zsh 配置文件
```

在文件中添加一行:

> alias ls='eza --color=always --long --git --no-filesize --icons=always --notime --no-user --no-permissions'

```bash
source ~/.zshrc # 重新加载 zsh 配置文件
```

#### 验证

```bash
ls
```

### 🚶‍♂️zoxide ~~cd~~

[zoxide](https://github.com/ajeetdsouza/zoxide)是更聪明的cd命令，它能记住你最常用的目录，因此你只需敲几下键盘就能 "跳转 "到它们。

#### 安装

```bash
sudo dnf -y install zoxide
```

#### zshrc 配置

```bash
nano ~/.zshrc # 编辑 zsh 配置文件
```

在文件中添加一行:

> eval "$(zoxide init --cmd cd zsh)"

```bash
source ~/.zshrc # 重新加载 zsh 配置文件
```

### 🚮trash-cli ~~rm rmdir~~

[trash-cli](https://github.com/andreafrancia/trash-cli)是一个命令行工具，它可以从命令行中删除文件或目录，并将它们放入回收站，而不是直接删除。


#### 安装

```bash
sudo dnf -y install trash-cli
```

#### zshrc 配置

```bash
nano ~/.zshrc # 编辑 zsh 配置文件
```

在文件中添加:

> alias rm='trash-put'
> alias rmdir='trash-put'

```bash
source ~/.zshrc # 重新加载 zsh 配置文件
```

### 😽bat ~~cat~~

[bat](https://github.com/sharkdp/bat)是类似 cat，但带有 git 集成和语法高亮。

#### 安装


```bash
sudo dnf -y install bat
```

#### zshrc 配置

```bash
nano ~/.zshrc # 编辑 zsh 配置文件
```

在文件中添加一行:

> alias cat='bat'

```bash
source ~/.zshrc # 重新加载 zsh 配置文件
```

### toolong

[toolong](https://github.com/Textualize/toolong)是查看、跟踪、合并和搜索日志文件的终端应用程序（加上JSONL）。

#### 安装

```bash
pipx install toolong
```

#### 验证

```bash
tl --help
```

### 🔍fd ~~find~~

#### 安装

[fd](https://github.com/sharkdp/fd)是一个快速且用户友好的文件搜索工具。

```bash
sudo dnf -y install fd-find
```

#### zshrc 配置

```bash
nano ~/.zshrc # 编辑 zsh 配置文件
```

在文件中添加一行:

> alias find='fd -HI'

```bash
source ~/.zshrc # 重新加载 zsh 配置文件
```

#### 验证

```bash
find passwd
```

### 🫳ripgrep ~~grep~~

[ripgrep](https://github.com/BurntSushi/ripgrep)是递归地搜索目录中的正则表达式模式，同时尊重你的gitignore。

#### 安装

```bash
sudo dnf -y install ripgrep
```

#### zshrc 配置

```bash
nano ~/.zshrc # 编辑 zsh 配置文件
```

在文件中添加一行:

> alias grep='rg'

```bash
source ~/.zshrc # 重新加载 zsh 配置文件
```

#### 验证

```bash
grep passwd
```

### 🏓gping ~~ping~~

[gping](https://github.com/orf/gping)是图表化的ping工具。

#### 安装

```bash
sudo dnf copr enable atim/gping -y && sudo dnf install gping
```

#### zshrc 配置

```bash
nano ~/.zshrc # 编辑 zsh 配置文件
```


在文件中添加一行:

> alias ping='gping'

```bash
source ~/.zshrc # 重新加载 zsh 配置文件
```

#### 验证

```bash
ping 127.0.0.1
```

### 🐳 Docker

[中文手册](https://dockerdocs.cn/engine/install/)

#### 安装

```bash
sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo # 添加 Docker 仓库
sudo dnf -y install docker-ce docker-ce-cli containerd.io docker-compose-plugin # 安装 Docker
sudo systemctl --now enable docker # 启动 Docker 服务
sudo usermod -a -G docker $(whoami) # 允许普通用户使用 Docker
```

#### 验证

```bash
docker --version
docker run hello-world
docker ps -a
```

### LazyDocker

[LazyDocker](https://github.com/jesseduffield/lazydocker#installation)是一个用于管理 Docker 容器的终端 UI 工具。

#### 安装

```bash
docker run --rm -it -v \
/var/run/docker.sock:/var/run/docker.sock \
-v /etc/lazydocker:/.config/jesseduffield/lazydocker \
lazyteam/lazydocker
```

#### zshrc 配置

使用 lzdocker 命令代替 lazydocker 命令

```bash
nano ~/.zshrc # 编辑 zsh 配置文件
```

在文件中添加一行:

> alias lzdocker='docker run --rm -it -v /var/run/docker.sock:/var/run/docker.sock -v /etc/lazydocker:/.config/jesseduffield/lazydocker lazyteam/lazydocker'

```bash
source ~/.zshrc # 重新加载 zsh 配置文件
```

#### 验证

```bash
lzdocker
```

### Lazygit

[Lazygit](https://github.com/jesseduffield/lazygit#installation)是一个用于管理 Git 仓库的终端 UI 工具。

#### 安装


```bash
brew install jesseduffield/lazygit/lazygit
```

#### zshrc 配置

使用 lzgit 命令代替 lazygit 命令

```bash
nano ~/.zshrc # 编辑 zsh 配置文件
```

在文件中添加一行:

> alias lzgit='lazygit'

```bash
source ~/.zshrc # 重新加载 zsh 配置文件
```

#### 验证

```bash
lzgit -v
```

### ☁️Seafile

[Seafile](https://www.seafile.com/en/download/)是一个开源的网盘软件，支持多种存储方式，包括本地硬盘、NAS、S3、Ceph、OSS等。

#### 安装

```bash
sudo dnf -y install seafile-server
```

#### 配置

```bash
sudo seahub-setup
```

#### 验证

```bash
sudo systemctl start seafile
sudo systemctl enable seafile
```

### ☁️Alist ~~smb ftp~~

[Alist](https://alist.nn.ci/zh/guide/install/docker.html)是一个支持多种存储的文件列表程序，使用 Gin 和 Solidjs。

#### 安装

```bash
sudo docker run -d --restart=unless-stopped -v /etc/alist:/opt/alist/data -p 5244:5244 -e PUID=0 -e PGID=0 -e UMASK=022 --name="alist" xhofe/alist:latest # 安装 Alist
sudo docker exec -it alist ./alist admin set 🔑ALISTPASSWORD🔑 # 设置 Alist 密码

sudo mkdir /etc/alist/alist-share # 创建共享目录
```

#### 远程访问 Alist

http://`ℹ️IPADDRESSℹ️`:5244

</details>

---

<details>
<summary><h2">🕸️更多网页服务</h2></summary>

### sun-pannel

[sun-pannel](https://doc.sun-panel.top/zh_cn/usage/quick_deploy.html)是一个服务器、NAS导航面板、Homepage、浏览器首页。

#### 安装

```bash
sudo docker run -d --restart unless-stopped -p 3002:3002 \
-v ~/docker_data/sun-panel/conf:/app/conf \
-v /var/run/docker.sock:/var/run/docker.sock \
--name sun-panel \
hslr/sun-panel:beta
```

#### 使用

http://`ℹ️IPADDRESSℹ️`:3002/

### 1Panel

[1Panel](https://1panel.cn/docs/)是新一代的 Linux 服务器运维管理面板。

#### 安装

```bash
curl -sSL https://resource.fit2cloud.com/1panel/package/quick_start.sh -o quick_start.sh && sudo bash quick_start.sh
```

#### 使用

http://`ℹ️IPADDRESSℹ️`:`ℹ️PORTℹ️`

### ONLYOFFICE

[ONLYOFFICE](https://helpcenter.onlyoffice.com/installation/docs-community-install-docker.aspx)是一款基于文档的协作办公套件，支持多种文件格式、多种协作模式。

#### 安装

```bash
sudo docker run -i -t -d -p 8081:80 --restart unless-stopped -e JWT_SECRET=my_jwt_secret onlyoffice/documentserver
```

#### 使用

http://`ℹ️IPADDRESSℹ️`:8081/example/

### Etherpad

[Etherpad](https://github.com/ether/etherpad-lite)是一个实时的网络协作编辑器。

#### 安装

```bash
sudo mkdir -p /docker/etherpad
sudo chmod -R 777 /docker/etherpad
cd /docker/etherpad
code docker-compose.yaml
```

在 docker-compose.yaml 文件中添加以下内容:

> ```yaml
> services:
>   app:
>     user: "0:0"
>     image: etherpad/etherpad:latest
>     tty: true
>     stdin_open: true
>     volumes:
>       - plugins:/opt/etherpad-lite/src/plugin_packages
>       - etherpad-var:/opt/etherpad-lite/var
>     depends_on:
>       - postgres
>     environment:
>       NODE_ENV: production
>       ADMIN_PASSWORD: ${DOCKER_COMPOSE_APP_ADMIN_PASSWORD:-admin}
>       DB_CHARSET: ${DOCKER_COMPOSE_APP_DB_CHARSET:-utf8mb4}
>       DB_HOST: postgres
>       DB_NAME: ${DOCKER_COMPOSE_POSTGRES_DATABASE:-etherpad}
>       DB_PASS: ${DOCKER_COMPOSE_POSTGRES_PASSWORD:-admin}
>       DB_PORT: ${DOCKER_COMPOSE_POSTGRES_PORT:-5432}
>       DB_TYPE: "postgres"
>       DB_USER: ${DOCKER_COMPOSE_POSTGRES_USER:-admin}
>       # For now, the env var DEFAULT_PAD_TEXT cannot be unset or empty; it seems to be mandatory in the latest version of etherpad
>       DEFAULT_PAD_TEXT: ${DOCKER_COMPOSE_APP_DEFAULT_PAD_TEXT:- }
>       DISABLE_IP_LOGGING: ${DOCKER_COMPOSE_APP_DISABLE_IP_LOGGING:-false}
>       SOFFICE: ${DOCKER_COMPOSE_APP_SOFFICE:-null}
>       TRUST_PROXY: ${DOCKER_COMPOSE_APP_TRUST_PROXY:-true}
>     restart: always
>     ports:
>       - "${DOCKER_COMPOSE_APP_PORT_PUBLISHED:-9001}:${DOCKER_COMPOSE_APP_PORT_TARGET:-9001}"
> 
>   postgres:
>     image: postgres:15-alpine
>     environment:
>       POSTGRES_DB: ${DOCKER_COMPOSE_POSTGRES_DATABASE:-etherpad}
>       POSTGRES_PASSWORD: ${DOCKER_COMPOSE_POSTGRES_PASSWORD:-admin}
>       POSTGRES_PORT: ${DOCKER_COMPOSE_POSTGRES_PORT:-5432}
>       POSTGRES_USER: ${DOCKER_COMPOSE_POSTGRES_USER:-admin}
>       PGDATA: /var/lib/postgresql/data/pgdata
>     restart: always
>     # Exposing the port is not needed unless you want to access this database instance from the host.
>     # Be careful when other postgres docker container are running on the same port
>     # ports:
>     #   - "5432:5432"
>     volumes:
>       - postgres_data:/var/lib/postgresql/data/pgdata
> 
> volumes:
>   postgres_data:
>   plugins:
>   etherpad-var:
> ```

```bash
docker compose -f docker-compose.yaml up -d
```

#### 使用

http://`ℹ️IPADDRESSℹ️`:9001

### 🃏Docsify

[Docsify](https://docsify.js.org/#/quickstart)是将一个或多个Markdown文件转换为网站，无需构建过程。

#### 安装

```bash
npm i docsify-cli -g
sudo mkdir /usr/share/docsify-docs
sudo chmod -R 777 /usr/share/docsify-docs
docsify init /usr/share/docsify-docs

sudo firewall-cmd --permanent --add-port=3000/tcp
sudo firewall-cmd --reload
sudo firewall-cmd --list-ports

pm2 start `which docsify` -- serve /usr/share/docsify-docs
pm2 save
pm2 startup
```

http://`ℹ️IPADDRESSℹ️`:3000/

### Stirling-PDF

[Stirling-PDF](https://github.com/Stirling-Tools/Stirling-PDF)是一个强大的，本地托管的基于Web的PDF操作工具，使用Docker。它使您能够对PDF文件进行各种操作，包括拆分，合并，转换，重组，添加图像，旋转，压缩等。这个本地托管的Web应用程序已经发展到包含一套全面的功能，满足您的所有PDF要求。

#### 安装

```bash
sudo docker run -d \
  --restart=unless-stopped \
  -p 8080:8080 \
  -v /location/of/trainingData:/usr/share/tessdata \
  -v /location/of/extraConfigs:/configs \
  -v /location/of/logs:/logs \
  -e DOCKER_ENABLE_SECURITY=false \
  -e INSTALL_BOOK_AND_ADVANCED_HTML_OPS=false \
  -e LANGS=en_GB \
  --name stirling-pdf \
  frooodle/s-pdf:latest
```

#### 使用

http://`ℹ️IPADDRESSℹ️`:8080/

### draw.io

[draw.io](https://github.com/jgraph/docker-drawio)是一个白板/绘图软件应用程序，支持多种图表类型，包括流程图、UML图、组织结构图、ER图等。

#### 安装

```bash
sudo docker run -d -it --name="draw" --restart unless-stopped -p 8442:8080 jgraph/drawio
```

#### 使用

http://`ℹ️IPADDRESSℹ️`:8442/

### Excalidraw

[Excalidraw](https://docs.excalidraw.com/docs/introduction/development#self-hosting)是一个开源的虚拟手绘风格的白板。协作和端到端加密。

#### 安装

```bash
sudo docker run --restart unless-stopped -dit --name excalidraw -p 8443:80 excalidraw/excalidraw:latest
```

#### 使用

http://`ℹ️IPADDRESSℹ️`:8443/

### it-tools

[it-tools](https://github.com/CorentinTh/it-tools)是开发人员和IT工作人员的有用工具集合。

#### 安装

```bash
sudo docker run -d --name it-tools --restart unless-stopped -p 8765:80 corentinth/it-tools:latest
```

#### 使用

http://`ℹ️IPADDRESSℹ️`:8765/

### libretranslate

[libretranslate](https://github.com/LibreTranslate/LibreTranslate)是一个开源的机器翻译API，支持多种语言。

#### 安装

```bash
mkdir -p ~/libretranslate
wget --no-check-certificate https://raw.githubusercontent.com/LibreTranslate/LibreTranslate/main/run.sh -O ~/libretranslate/run.sh
cd ~/libretranslate
chmod +x run.sh
./run.sh # 测试运行

docker ps
docker stop `CONTAINER ID`

sudo docker run -d --restart=unless-stopped -ti -p 5000:5000 -v lt-local:/home/libretranslate/.local libretranslate/libretranslate
```

#### 使用

http://`ℹ️IPADDRESSℹ️`:5000/?source=auto&target=zh&q=

### memos

[memos](https://github.com/usememos/memos)是一个开源的、轻量级的笔记服务。轻松捕捉和分享您的好主意。

#### 安装

```bash
sudo docker run -d --restart=unless-stopped --name memos -p 5230:5230 -v ~/.memos/:/var/opt/memos neosmemo/memos:stable
```

#### 使用

http://`ℹ️IPADDRESSℹ️`:5230/

### wekan

[wekan](https://github.com/wekan/wekan)是一个基于Web的看板应用，支持多种看板类型，包括看板、任务列表、日历、Kanban、组织结构图等。

#### 安装

```bash
sudo docker run -d --restart=always --name wekan-db mongo:5

sudo docker run -d --restart=always --name wekan --link "wekan-db:db" -e "WITH_API=true" -e "MONGO_URL=mongodb://wekan-db:27017/wekan" -e "ROOT_URL=http://192.168.3.248:2000" -p 2000:8080 wekanteam/wekan:latest
```

#### 使用

http://`ℹ️IPADDRESSℹ️`:2000/

### snapdrop

[snapdrop](https://github.com/RobinLinus/snapdrop/blob/master/docs/local-dev.md)是一个基于Web的跨平台文件共享应用程序，支持多种文件格式。

#### 安装

```bash
docker run --restart unless-stopped -d -p 8444:80 linuxserver/snapdrop
```

#### 使用

http://`ℹ️IPADDRESSℹ️`:8444/

### next-terminal

[next-terminal](https://github.com/dushixiang/next-terminal)是一个简单好用安全的开源交互审计系统，支持RDP、SSH、VNC、Telnet、Kubernetes协议。

#### 安装

```bash
sudo mkdir -p /docker/next-terminal/data
sudo chmod -R 777 /docker/next-terminal/
cd /docker/next-terminal
code docker-compose.yaml
```

> ```yaml
> version: '3.3'
> services:
>     guacd:
>        image: dushixiang/guacd:latest
>        volumes:
>          - ./data:/usr/local/next-terminal/data
>        restart:
>              always
>      next-terminal:
>        image: dushixiang/next-terminal:latest
>     environment:
>       DB: sqlite
>       GUACD_HOSTNAME: guacd
>       GUACD_PORT: 4822
>     ports:
>       - "8088:8088"
>     volumes:
>       - /etc/localtime:/etc/localtime
>       - ./data:/usr/local/next-terminal/data
>     restart:
>       always
> ```
> 

```bash
docker compose -f docker-compose.yaml up -d
```

#### 使用

http://`ℹ️IPADDRESSℹ️`:8088/

</details>

---

<details>
<summary><h2">🖥️更多桌面应用程序</h2></summary>

### VSCode

[VSCode](https://code.visualstudio.com/docs/setup/linux)是微软推出的开源编辑器，支持多种编程语言。

#### 安装

```bash
sudo rpm --import https://packages.microsoft.com/keys/microsoft.asc
sudo sh -c 'echo -e "[code]\nname=Visual Studio Code\nbaseurl=https://packages.microsoft.com/yumrepos/vscode\nenabled=1\ngpgcheck=1\ngpgkey=https://packages.microsoft.com/keys/microsoft.asc" > /etc/yum.repos.d/vscode.repo'
sudo dnf check-update
sudo dnf -y install code
```

#### 使用

```bash
code .
```

### Firefox

[Firefox](https://www.mozilla.org/zh-CN/firefox/all/#product-desktop-developer)是一款开源的网页浏览器。

#### 安装

```bash
sudo dnf -y remove firefox # 卸载旧版本

cd ~/下载
wget -O firefox-latest.tar.bz2 "https://download.mozilla.org/?product=firefox-latest-ssl&os=linux64&lang=zh-CN" 
sudo tar xvjf firefox-latest.tar.bz2 -C /usr/local
sudo ln -s /usr/local/firefox/firefox /usr/bin/firefox

firefox &
```

#### 添加图标

```bash
nano ~/.local/share/applications/firefox.desktop
```

> ```
> [Desktop Entry]
> Name=Firefox
> Comment=Web Browser
> Exec=/usr/local/firefox/firefox %u
> Icon=/usr/local/firefox/browser/chrome/icons/default/default128.png
> Terminal=false
> Type=Application
> Categories=Network;WebBrowser;
> ```

#### 增强插件

[ShyFox](https://github.com/Naezr/ShyFox)是一个非常害羞的小主题，将整个浏览器界面隐藏在窗口边框中

### QQ

[QQ](https://im.qq.com/linuxqq/index.shtml)是一款跨平台的即时通信工具。

#### 安装

```bash
cd ~/下载
sudo dnf -y install ./QQ_x.x.x_xxxxxx_x86_64.rpm
```

</details>