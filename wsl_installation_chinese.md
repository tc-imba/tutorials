## WSL (Windows Subsystem for Linux) 安装教程

### 安装 WSL

使用管理员权限的 Shell 才能安装 WSL. 按 Win+X, 找到 `Windows PowerShell (管理员)`, 并复制

```powershell
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
```

以上命令会激活 WSL 服务，然后需要重启系统

重启后，打开 Windows 应用商店并搜索 `Ubuntu` 并选择版本号 (18.04/20.04)

如果遇到问题可以参考 https://docs.microsoft.com/en-us/windows/wsl/install-win10 if meeting any problems


### Apt 换源

使用 `apt` 或其他修改系统文件的指令都需要管理员权限，使用 `sudo` (superuser do) 以取得权限

Debian / Ubuntu 的官方源在国内访问很慢，可以参考清华 TUNA 镜像进行换源

https://mirror.tuna.tsinghua.edu.cn/help/ubuntu/

在上面的网页里选择正确的系统版本，并复制入 `/etc/apt/sources.list`

比如，在 Ubuntu 20.04 上，文件内容为

```
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security main restricted universe multiverse

# deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-proposed main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-proposed main restricted universe multiverse
```

首先，备份 `/etc/apt/sources.list`

```bash
$ sudo mv /etc/apt/sources.list /etc/apt/sources.list.backup
```

然后用 `vim` 或者 `nano` 来修改文件

使用 `vim`， 输入 `sudo vim /etc/apt/sources.list`. 然后按 `i`, 并用鼠标右键粘贴，最后按 `Esc`， `Shift+Z+Z` 保存文件并退出编辑器

使用 `nano`, 输入 `sudo nano /etc/apt/sources.list`. 然后用鼠标右键粘贴，最后按 `Ctrl+O` 和 `Enter` 保存文件，按 `Ctrl+X` 退出编辑器

换源后，安装库将会很快，比如可以用以下指令安装 `g++`

```bash
$ sudo apt update
$ sudo apt install g++
```


### 配置 SSH 服务器 (远程调试, CLion)

WSL 上的 SSH 服务器没有自动配置，需要手动重新安装，首先可以运行以下命令来检查

```bash
$ sudo service ssh stop
$ sudo /usr/sbin/sshd -d
```

如果你看到以下信息，则需要手动配置

```
debug1: sshd version OpenSSH_7.2, OpenSSL 1.0.2g  1 Mar 2016
debug1: key_load_private: incorrect passphrase supplied to decrypt private key
debug1: key_load_public: No such file or directory
Could not load host key: /etc/ssh/ssh_host_rsa_key
debug1: key_load_private: No such file or directory
debug1: key_load_public: No such file or directory
Could not load host key: /etc/ssh/ssh_host_dsa_key
debug1: key_load_private: No such file or directory
debug1: key_load_public: No such file or directory
Could not load host key: /etc/ssh/ssh_host_ecdsa_key
debug1: key_load_private: No such file or directory
debug1: key_load_public: No such file or directory
Could not load host key: /etc/ssh/ssh_host_ed25519_key
```

如果看到以上信息，重新安装 `openssh-server` 就可以解决问题

```bash
$ sudo apt purge openssh-server
$ sudo apt install openssh-server
```

然后需要配置 `/etc/ssh/sshd_config`，用 `sudo` 权限运行 `vi` 或者 `nano`

找到这两行
```
PermitRootLogin no/prohibit-password
PasswordAuthentication no
```

改为这两行
```
PermitRootLogin yes
PasswordAuthentication yes
```

记得删除 `#`，这两行允许了 `root` 账户和密码登录

然后运行
```bash
$ sudo service ssh restart
$ sudo service ssh status
```

如果需要用密码登录 `root` 账户，还需要设置密码
```bash
$ sudo passwd root
```

现在你可以尝试在本机直接连接 SSH 服务器
```bash
$ ssh root@localhost
```

#### CLion WSL Toolchain

CLion 可以使用 WSL 工具链. First, 你需要先安装 `g++`, `cmake`, `gdb`, `valgrind`

```bash
$ sudo apt install g++ cmake gdb valgrind
```

根据下图配置 Clion Toolchains

![clion_wsl.png](images/clion_wsl.png)

在 **Credentials**, 需要输入刚才配置的 SSH 服务器信息

![clion_wsl_ssh.png](images/clion_wsl_ssh.png)

输入你刚才设置的密码

如果你需要使用 `valgrind`，需要手动设置路径为 `/usr/bin/valgrind`

![clion_wsl_valgrind.png](images/clion_wsl_valgrind.png)



