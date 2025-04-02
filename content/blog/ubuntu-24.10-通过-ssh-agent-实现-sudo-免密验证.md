---
title: Ubuntu 24.10 通过 ssh-agent 透传实现 sudo 免密验证与 TouchID 指纹验证
date: 2025-03-26T06:54:16.439Z
---


## 前述

目前我的主要设备是 Macbook Air M1，macOS 上的应用 [Secretive](https://github.com/maxgoedjen/secretive) 可以利用 Apple Silicon 中的 Secure Enclave（类似 Android 的 Trustzone）生成、保存与管理 SSH 密钥对，并通过 ssh-agent 实现使用 Touch ID 认证。配合 GitHub 的 SSH 公钥托管设施[^1]，可以让 SSH 的免密登录流程变得较为舒适且符合人体工学。

## 动机

之前在我的 j18xx 上安装 Ubuntu 24.10 时，我注意到安装流程中提示可以从 GitHub 导入公钥[^2]。在这之前，我就已经发现并使用了 Secretive，于是第一次与这之后的登录流程都较为令人舒适。但后续我发现，我常需要输入密码使用 `sudo`，这与我在我本地的 macOS 环境中的配置：使用 Touch ID 认证 `sudo` 指令的使用体验不一致。在近日发现 Cloudflare 实现了类似的功能后[^3]，我便快速的从互联网上搜索，并得到了基于 PAM 模块的解决方案。

## 远程主机配置

### 配置 PAM 模块

通过指令安装 [`libpam-ssh-agent-auth`](https://launchpad.net/ubuntu/bionic/+package/libpam-ssh-agent-auth)。

```shell
sudo apt install libpam-ssh-agent-auth
```

打开 `/etc/pam.d/sudo` 文件，并根据该 diff 示例编辑。

```diff
#%PAM-1.0

+auth sufficient pam_ssh_agent_auth.so file=/etc/ssh/sudo_authorized_keys

# Set up user limits from /etc/security/limits.conf.
session    required   pam_limits.so

session    required   pam_env.so readenv=1 user_readenv=0
session    required   pam_env.so readenv=1 envfile=/etc/default/locale user_readenv=0

@include common-auth
@include common-account
@include common-session-noninteractive
```

这里我们配置 `sufficient` 使得如果基于 ssh-agent 的 PAM 验证通过，则不继续进行其他如 `common-auth` 等三项验证机制（一票通过[^4][^5]）；而若因某些原因无法继续，则将继续执行接下来的登录步骤，这使得你可以继续使用其他通用方法[^6]。

这里我们注意到加入的配置行中提到了 `/etc/ssh/sudo_authorized_keys` 文件，该文件存放了可被用于 `sudo` 指令通过 PAM 认证通过的 SSH 公钥。 

### 配置密钥
 
通过该指令将服务器上当前用户所信任的公钥复制到我们提到过的密钥文件中。

```
sudo cp ~/.ssh/authorized_keys /etc/ssh/sudo_authorized_keys
```

注意到该方法无法保证更新，所以如果你通过例如 `ssh-copy-id` 指令等方法更新过了 `~/.ssh/authorized_keys` 文件，需自行对该文件也进行更新。

顺便一提，如果你的主机没有被其他主机登录过（我想不太可能），你也可以手动的添加，这里给出示例。

```
cat ~/.ssh/id_*.pub | sudo tee /etc/ssh/sudo_authorized_keys
```

而鉴于 ssh-agent 较为泛用，这一节提到的方法可以被单独使用作一种实现 sudo 免密验证的方式，不过在这种情况下，我们不建议继续使用 `/etc/ssh/sudo_authorized_keys`，因其在这种情况下不符合原始语义，且违反单一职责原则。

脚注四[^4]所指向的博文中包含了更多对本节 PAM 配置的解释，如果有兴趣的可以自行了解。

### 为 sudo 启用 ssh-agent

目前互联网上的有关教程都提到直接编辑 `/etc/sudoers` 文件，但我们不推荐使用该方法，注意到可以使用 `/etc/sudoers.d` 实施更好的管理。
这里我们为 sudo 指令保留环境变量 `SSH_AUTH_SOCK`，使得 PAM 模块可以访问到我们透传的 ssh-agent。

```
echo "Defaults env_keep += SSH_AUTH_SOCK" | sudo tee /etc/sudoers.d/00_SSH_AUTH_OK
sudo chmod 0440 /etc/sudoers.d/00_SSH_AUTH_OK
```

### 配置 ssh-agent 随登录启动

在 `.profile` 中添加 `ssh-add` 指令以在登录时启动 ssh-agent。

```
echo ssh-add >> .profile
```

注意到 Ubuntu 24.10 默认使用 bash 作为 shell，如果你的 home 目录下存在 `.bash_profile` 或者 `.bash_login` 文件，则 `.profile` 不会被 bash 读取，且不会默认执行 `~/.local/bin/env`，这会导致一些问题（如重新登录时丢失基于 env 指令的个性化配置），因此不建议使用这两个文件。

## 本地配置

打开 `~/.ssh/config`，在对应的主机下配置对 ssh-agent 的透传。

```
Host elaina-j18xx
    # ...
    User elaina
    ForwardAgent true
```

当然也可以临时添加 `-A` 来实现。

```
ssh -A elaina-j18xx
```

## 结语

这样配置好了后，我们就能通过 ssh-agent 实现在目标远程设备上的 `sudo` 指令免密验证，进一步的方便了我们的管理实施与操作。
类似的方法对于 macOS 也可用，参见 [MacOS run sudo with Touch ID](https://gist.github.com/farhaduneci/10e30162eceb12a87bcee92b247b8808)。

[^1]: https://docs.github.com/en/enterprise-cloud@latest/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent

[^2]: 该功能是通过 `ssh-copy-id` 命令实现的，准确来说，是 `ssh-copy-id-gh`。

[^3]: https://blog.cloudflare.com/open-sourcing-openpubkey-ssh-opkssh-integrating-single-sign-on-with-ssh/

[^4]: https://www.cnblogs.com/zxl1024320609/p/16834625.html

[^5]: 值得一提的是，这对于 macOS 也有用，可以通过类似的方法配置使得 Touch ID 可以认证 sudo 指令。

[^6]: https://sfxpt.wordpress.com/package/libpam-ssh-agent-auth/
