---
title: Radicle Tour
date: 2025-04-14T05:43:25.835Z
---

本文将快速的介绍 Radicle，一种新式的开源、点对点（P2P）、去中心化 Git 协作平台。

Radicle 可以简单理解为 Bittorrent 式的 Git，通过将 Issue、Patch[^1] 等协作元素（[Collaborative Objects](https://radicle.xyz/guides/protocol/#collaborative-objects)）从 Git 中扩展，以此形成 Radicle Protocol，借此承载各种功能。通过实施 CRDT 这种分布式同步技术，Radicle 实现了在多个对等节点间贡献者的互相协作。

[^1]: 即 Pull/Merge Request

## 安装与配置

> [!tip]
>
> 本文假定读者已经拥有了良好的环境配置，即：
>
> - 运行了一个 Unix-like 的操作系统；
> - 已经配置好了 Git、OpenSSH 等基本工具。
>
> 如果没有这样的前提，本文无法保证接下来的流程能够顺利进行下去。
>
> 此外，Radicle 的各种实践极度依赖于本地运行的守护进程 Daemon，因此一定的用户权限是必须的，某些实践甚至需要系统管理员权限（如运行可以良好运作的做种 Seed 节点），而到时候会特别提及这点。

执行这段指令以安装 Radicle 的二进制文件。

```bash
curl -sSf https://radicle.xyz/install | sh
```

根据指示重启当前终端会话后，输入 `rad` 以检查安装是否成功，接下来所使用的所有 Radicle 的功能，均由该指令提供。

```
$ rad
rad 1.1.0 (70f0cc35)
Radicle command line interface

Usage: rad <command> [--help]
Common `rad` commands used in various situations:
...
```

如果安装过程中遇到错误，请根据终端输出解决相关错误。

在使用 Radicle 的各项功能之前，你首先需要执行 `rad auth` 生成你的密钥对，这类似 `ssh-genkey`。

```
$ rad auth

Initializing your radicle 👾 identity

✓ Enter your alias: paxel
✓ Enter a passphrase: ********
✓ Creating your Ed25519 keypair...
✓ Adding your radicle key to ssh-agent...
✓ Your Radicle DID is did:key:z6Mkhp7VUnuufpvuQ3PdysShAjL86VDRUpPpkesqiysDBGs9. This identifies your device. Run `rad self` to show it at all times.
✓ You're all set.
...
```

与 `ssh-genkey` 也非常相似的，你首先需要输入你的用户名或者说 `alias`，之后是需要重复输入确认，用于加解密密钥的密码。

执行完这一步骤后，我们可以运行 `rad self` 检查。

```
$ rad self
Alias           paxel
DID             did:key:z6Mkhp7VUnuufpvuQ3PdysShAjL86VDRUpPpkesqiysDBGs9
└╴Node ID (NID) z6Mkhp7VUnuufpvuQ3PdysShAjL86VDRUpPpkesqiysDBGs9
SSH             running (3817)
├╴Key (hash)    SHA256:YCmRe6BkDOp45lYg0m5DeYxgRcPKftQZb4RmQD1nkjQ
└╴Key (full)    ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIO9xo9DHlsZJeZWnZaaawsnKFjcQxN4LQ…
Home            /home/paxel/.radicle
├╴Config        /home/paxel/.radicle/config.json
├╴Storage       /home/paxel/.radicle/storage
├╴Keys          /home/paxel/.radicle/keys
└╴Node          /home/paxel/.radicle/node
```

很好，节点配置看上去很正常，现在我们运行 Radicle 的守护进程，也称*节点*。运行 `rad node start` 以启动它。

> [!note]
>
> 请注意这里的 DID，即去中心化 ID（Decentralized ID）和节点 ID（Node ID），在 [#搭建做种服务器] 和 [#私有仓库] 一节中会用到。

```
$ rad node start
✓ Node started (80277)
To stay in sync with the network, leave the node running in the background.
To learn more, run `rad node --help`
```

可以通过 `rad node status` 指令检查守护进程的运作情况。

```
$ rad node status
✓ Node is running.
╭───────────────────────────────────────────────────────────────────────╮
│ Peer                                               Address            │
├───────────────────────────────────────────────────────────────────────┤
│ z6Mkmqogy2qEM2ummccUthFEaaHvyYmYBYh3dbe9W4ebScxo   ash.radicle.garde… │
│ z6MkrLMMsiPWUcNPHcRajuMi9mDfYckSoJyPwwnknocNYPm7   seed.radicle.gard… │
╰───────────────────────────────────────────────────────────────────────╯

2025-04-14T00:04:40.237+08:00 INFO  service  Received command QueryState(..)
2025-04-14T00:04:40.491+08:00 INFO  service  Connected to z6Mkmqogy2qEM2ummccUthFEaaHvyYmYBYh3dbe9W4ebScxo (ash.radicle.garden:8776) (Outbound)
2025-04-14T00:04:40.502+08:00 INFO  service  Connected to z6MkrLMMsiPWUcNPHcRajuMi9mDfYckSoJyPwwnknocNYPm7 (seed.radicle.garden:8776) (Outbound)
```

可以看到，在启动守护进程后，他便开始尝试连接到 Radicle 去中心化网络中的对等节点（Peer），如由 Radicle 目前的维护团队开设并维护的节点 `ash.radicle.garden` 与 `seed.radicle.garden` 等。这将支撑我们后续的进一步操作与探索。

> [!tip]
>
> 你可以向 `rad` 指令传入 `--help` 或是用 `man` 指令查看指令帮助。

## 第一个提交与推送

需要注意的是，Radicle 整体都可以视作基于 Git 上的扩展，这也意味着你也可以将对 Radicle 的尝试视作你原有工作流之上的扩展，而非用 Radicle 的流程将其彻底推翻并取而代之。

由于我们假定你已经配置好了 Git，我们也将直接从一个足够简单的基础开始：一个足以开展简单工作的仓库文件夹。

```bash
$ pwd
/home/paxel/src/dark-star

$ git status
On branch master
Your branch is up to date with 'origin/master'.
nothing to commit, working tree clean
```

在仓库目录下，我们运行 `rad init` 来为仓库初始化 Radicle。

```
$ rad init

Initializing radicle 👾 project in .

✓ Name: dark-star
✓ Description: Decoding data from the dark star to establish a model of the universe
✓ Default branch: main
✓ Visibility: public
✓ Project dark-star created.

Your project's Repository ID (RID) is rad:z31hE1wco9132nedN3mm5qJjyotna.
You can show it any time by running `rad .` from this directory.

✓ Project successfully announced to the network.

Your project has been announced to the network and is now discoverable by peers.
You can check for any nodes that have replicated your project by running `rad sync status`.

To push changes, run `git push`.
```

看上去不错，这里有一个我们需要留意的信息：仓库 ID (Repository ID / RID)。这个 ID 是全局唯一的，可以用于 `rad clone` 等指令。

除此之外，`rad init` 指令还在本地仓库中创建了特殊的远程分支 `rad`。如果执行 `git remote show rad` 指令，我们可以了解其详情。

```
$ git remote show rad
* remote rad
  Fetch URL: rad://z31hE1wco9132nedN3mm5qJjyotna
  Push  URL: rad://z31hE1wco9132nedN3mm5qJjyotna/z6MkvZwzK64f3GuDcAs6dEcje89ddfHkBjS1v9Dkh7aCGq3C
  HEAD branch: (unknown)
  Remote branch:
    main tracked
  Local branch configured for 'git pull':
    main merges with remote main
  Local ref configured for 'git push':
    main pushes to main (up to date)
```

在 Fetch URL 中出现的显然是我们的仓库 ID，而 Push URL 的后半段是你的节点 ID，你可以用 `rad self --nid` 来确认这点。

```
$ rad self --nid
z6MkvZwzK64f3GuDcAs6dEcje89ddfHkBjS1v9Dkh7aCGq3C
```

然后我们随便写点代码，做出修改后，便是经典的 Git 三板斧。

```
$ git add README.md
$ git commit -m "Add instruction on cloning with Radicle"
$ git push rad master
✓ Synced with 1 node(s)
To rad://z31hE1wco9132nedN3mm5qJjyotna/z6MkvZwzK64f3GuDcAs6dEcje89ddfHkBjS1v9Dkh7aCGq3C
   ecb1bf0..74fb8d2  master -> master
```

自此，我们已经完成了一个非常简单的推送提交，只不过是推送到 Radicle 网络中。

> [!note]
>
> 在 Radicle 网络发展的早期，Radicle 维护团队的做种节点 `seed.radicle.garden` 将对所有*公开*仓库做种。

## 克隆与做种仓库

简单如 `git clone`，你只需要 `rad clone <repo>` 即可将仓库克隆到本地。在这里，我们以克隆 Radicle 的官方仓库 [`Heartwood`](https://app.radicle.xyz/nodes/seed.radicle.xyz/rad:z3gqcJUoA1n9HaHKufZs5FCSGazv5) 作为示范。

```
$ rad clone rad:z3gqcJUoA1n9HaHKufZs5FCSGazv5
✓ Seeding policy updated for rad:z3gqcJUoA1n9HaHKufZs5FCSGazv5 with scope 'all'
✓ Fetching rad:z3gqcJUoA1n9HaHKufZs5FCSGazv5 from z6Mkk4R…SBiyXVM..
✓ Fetching rad:z3gqcJUoA1n9HaHKufZs5FCSGazv5 from z6Mksmp…1DN6QSz..
✓ Fetching rad:z3gqcJUoA1n9HaHKufZs5FCSGazv5 from z6MkrLM…ocNYPm7..
✓ Creating checkout in ./heartwood..
✓ Remote cloudhead@z6MksFqXN3Yhqk8pTJdUGLwATkRfQvwZXPqR2qMEhbS9wzpT added
✓ Remote-tracking branch cloudhead@z6MksFqXN3Yhqk8pTJdUGLwATkRfQvwZXPqR2qMEhbS9wzpT/master created for z6MksFq…bS9wzpT
✓ Repository successfully cloned under /home/paxel/src/heartwood/
╭────────────────────────────────────╮
│ heartwood                          │
│ Radicle Heartwood Protocol & Stack │
│ 8 issues · 14 patches              │
╰────────────────────────────────────╯
Run `cd ./heartwood` to go to the project directory.
```

节点自身维护了对一系列仓库的做种策略（Seeding Policy），即对哪些仓库跟踪并复制其修改。当我们使用 `rad init` 或 `rad clone` 指令时，都会更新做种策略，使得我们可以与相应仓库保持同步。

> [!note]
>
> `rad clone` 指令等效于执行这些指令：
>
> | :-               | :-                                            |
> | ---------------- | --------------------------------------------- |
> | `rad seed`       | 更新节点的做种策略，并开始对目标仓库做种。    |
> | `rad sync -f`    | 从其他做种节点获取最新的仓库数据，即 `.git`。 |
> | `rad checkout`   | 检出仓库。                                    |
> | `rad remote add` | 将克隆源添加为仓库的远程分支。                |

如果只是想对某个仓库做种的话，只执行 `rad seed` 就可以了。顺便一提，这其实就是 Radicle 式的 Star，通过为仓库做种，帮助其传播，以此来表达对其所成就伟业的赞许。

```
$ rad seed rad:z3gqcJUoA1n9HaHKufZs5FCSGazv5
✓ Seeding policy updated for rad:z3gqcJUoA1n9HaHKufZs5FCSGazv5 with scope 'all'
✓ Fetching rad:z3gqcJUoA1n9HaHKufZs5FCSGazv5 from z6Mkk4R…SBiyXVM..
```

如果想取消做种，使用 `rad unseed` 指令。

```
rad unseed rad:z9DV738hJpCa6aQXqvQC4SjaZvsi
```

使用 `rad ls --seeded` 指令列出已经做种了的仓库。

```
$ rad ls --seeded
│ Name        RID        Visibility   Head      Description                        │
│ heartwood   rad:z3g..  public       .......   Radicle Heartwood Protocol & Stack │
```

> [!note]
>
> 默认情况下，使用 `rad clone` 或 `rad seed` 指令，本地节点会订阅围绕目标仓库的*所有*对等节点的内容。这一行为可以通过更新做种策略，限制订阅范围来定制。
>
> 举例说明，我们可以通过传入 `--scope followed` 参数限制做种内容，这会限制本地节点只订阅来自这些对等节点的内容：
>
> - 仓库的权威方（Delegate）；
> - 其他通过 `rad follow` 明确指定的对等节点；
>
> `--scope` 参数可以在 `rad seed` 与 `rad clone` 指令中使用。
>
> ```
> $ rad seed rad:z3gqcJUoA1n9HaHKufZs5FCSGazv5 --scope followed
> ```
>
> 如果传入 `--scope all`，这将更新做种策略，使之重新遵循默认行为。

## 以 Radicle 方式协作

Radicle 将 Issue、Patch 等统一封装为可扩展的协作元素（Collaborative Objects），这一扩展是基于 Git 的内部机制实现的。

### Issue

#### 创建 / Open

要为仓库创建 Issue，你需要执行 `rad issue open` 指令。

```
$ cd <REPO>
$ rad issue open
```

和 `git commit` 差不多，该指令会打开你事先在环境变量 `EDITOR` 中配置的文本编辑器。创建 Issue 需要一个标题和内容，和一些 Commit Message 规范差不多，标题和内容需要用空行隔开，包裹在 `<!--` 和 `-->` 之间的 HTML 风格注释会被忽视 —— 你会发现打开的文件名是 `RAD_COMMENT.markdown`，遵循 Markdown 格式，所以显而易见。

这里给出一个示例。

```markdown
Establish data standards for IUI Dark Star Data group

As we go out and recruit the Neboriens and other intelligences to join
the Intergalactic Union of Intelligences (IUI), we are going to have to
establish standards for Dark Star data submissions.

<!--
Please enter an issue title and description.

The first line is the issue title. The issue description
follows, and must be separated by a blank line, just
like a commit message. Markdown is supported in the title
and description.
-->
```

如果你没有填写标题或内容，指令会报错退出。

```
$ rad issue open
✗ Error: aborting issue creation due to empty title or description
```

填写完成，他会以美观的 TUI 打印出你提的 Issue 的摘要信息。

```
╭───────────────────────────────────────────────────────────────╮
│ Title   Establish data standards for IUI Dark Star Data group │
│ Issue   e4255cc2a0a65b543c2b5badac14bf9e0d9f409f              │
│ Author  paxel (you)                                           │
│ Status  open                                                  │
│                                                               │
│ As we go out and recruit the Neboriens and other              │
│ intelligences to join the Intergalactic Union of              │
│ Intelligences (IUI), we are going to have to establish        │
│ standards for Dark Star data submissions.                     │
╰───────────────────────────────────────────────────────────────╯
```

> [!tip]
>
> 如果你实在不想或没办法事先指定环境变量 `EDITOR`，也可以传入 `--title <TITLE>` 和 `--description <DESC>` 参数。
>
> ```
> # rad issue open --title "I have encountered problem HELP!!!" --description "I cannot find .exe file"
> ```

#### 列出 / List

你可以用 `rad issue` 浏览已经打开了的 Issue 列表。

```
$ rad issue
╭──────────────────────────────────────────────────────────────────────────────────────────────────╮
│ ●   ID        Title                                                    Author                    │
├──────────────────────────────────────────────────────────────────────────────────────────────────┤
│ ●   80464b3   Categorize initial dataset                               paxel    (you)            │
│ ●   e4255cc   Establish data standards for IUI Dark Star Data group    paxel    (you)            │
│ ●   3e2f653   Add anomalous data from ship 897AF                       calyx    z6Mkgom…unurCap  │
│ ●   badda04   Recruit Neboriens to join IUI and share dark star data   calyx    z6Mkgom…unurCap  │
╰──────────────────────────────────────────────────────────────────────────────────────────────────╯
```

要查看其中一个，使用 `rad issue show <ISSUE>`。

```
$ rad issue show 3e2f65
╭─────────────────────────────────────────────────────────────╮
│ Title   Add anomalous data from ship 897AF                  │
│ Issue   3e2f653383f0d2fe21ef4e859a25925c364c740a            │
│ Author  calyx                                               │
│ Status  open                                                │
│                                                             │
│ Before we onboard new IUI members like the Neborians, it is │
│ important we upload the data from ship 897AF. I just        │
│ reviewed the database and it seems this is missing.         │
╰─────────────────────────────────────────────────────────────╯
```

#### 回复 / Comment

使用 `rad issue comment <ISSUE>` 撰写对 Issue 的回复，不传入 `--comment <COMMENT>` 则会调用环境变量 `EDITOR` 指定的文本编辑器。

```
$ rad issue comment 3e2f65 --message "I can help."
╭─────────────────────────╮
│ paxel (you) now 74faf0e │
│ I can help.             │
╰─────────────────────────╯
```

#### 跟踪 / Track

当仓库的维护者准备好以一个足够糟糕的 Issue 毁掉自他早上起床开始便保持完美的一天后，他输入 `rad inbox` 指令，这将列出所有订阅仓库下的最新 Issue 活动。

```
$ rad inbox
╭────────────────────────────────────────────────────────────────────────────────────────────╮
│ dark-star                                                                                  │
├────────────────────────────────────────────────────────────────────────────────────────────┤
│ 004   ●   3e2f653   Add anomalous data from ship 897AF   issue   open   calyx   1 hour ago │
╰────────────────────────────────────────────────────────────────────────────────────────────╯
```

起码还不算糟，让我们结束这短暂的角色扮演。使用 `rad inbox show <NUMBER>` 展示活动。

```
$ rad inbox show 4
╭─────────────────────────────────────────────────────────────╮
│ Title   Add anomalous data from ship 897AF                  │
│ Issue   3e2f653383f0d2fe21ef4e859a25925c364c740a            │
│ Author  calyx                                               │
│ Status  open                                                │
│                                                             │
│ Before we onboard new IUI members like the Neborians, it is │
│ important we upload the data from ship 897AF. I just        │
│ reviewed the database and it seems this is missing.         │
├─────────────────────────────────────────────────────────────┤
│ paxel z6MkvZw…7aCGq3C 1 hour ago 74faf0e                    │
│ I can help.                                                 │
╰─────────────────────────────────────────────────────────────╯
```

> [!tip]
>
> 如果你在一个由 Radicle 跟踪的仓库目录中，则 `rad inbox` 仅会列出这一个仓库的最新活动。若不是这样，或是额外传入了 `--all` 参数，`rad inbox` 会列出本地节点跟踪的所有仓库的最新活动。

#### 分配 / Assign

居然有少见的好心人说他能解决这个问题，天底下还能有比这更好的好事吗！那就让我们用 `rad issue assign` 指令，将这个重责大任交给他吧。

```
$ rad issue assign 3e2f653 --add did:key:z6MkvZwzK64f3GuDcAs6dEcje89ddfHkBjS1v9Dkh7aCGq3C
```

#### 其他

- `rad issue edit <ISSUE>` 可以修改 Issue 的描述信息；

- `rad issue label <ISSUE>` 和 GitHub 的 Issue Label 差不多，可以给 Issue 添加圆形徽章状的小标签。

  `--add <LABEL>` 是添加；

  `--delete <LABEL>` 是删除。

- `rad issue archive <ISSUE>` 可以将 Issue 存档或列为只读，可以用 `--undo` 撤销。

### Patch

如果想对一个由 Radicle 管理的仓库贡献你做出的修改与提交，你需要提出 Patch。这类似 GitHub 的 Pull Request，只是用命令行操作会较为繁琐，可以跟踪后续有关易用性方面的更新。

#### 创建 / Open

在做出修改之前，你需要先创建并检出一个新分支……然后再做出修改。

```
$ git checkout -b anomalous-data-897af
$ git add .
$ git commit
```

> [!note]
>
> 如果你已经在其他分支上做出了修改，还可以使用 cherry-pick，或是 merge 过来。
>
> ```
> $ git cherry-pick A^...B
> ```

在完成这些繁重工作后，你需要将修改推送到 Radicle 的远程分支并发起 Patch，我们需要写成 `HEAD:refs/patches` 表示我们希望从 `HEAD` 发起 Patch。和 `rad issue open/comment` 之类的一样，会调用 `EDITOR`。

```
$ git push rad HEAD:refs/patches
✓ Patch e5f0a5a5adaa33c3b931235967e4930ece9bb617 opened
✓ Synced with 8 node(s)

To rad://z3cyotNHuasWowQ2h4yF9c3tFFdvc/z6MkvZwzK64f3GuDcAs6dEcje89ddfHkBjS1v9Dkh7aCGq3C
* [new reference]   HEAD -> refs/patches
```

输出结果中，`e5f0a5a5adaa33c3b931235967e4930ece9bb617` 是我们提交的 Patch 的 ID。

> [!tip]
>
> 这里也不一定非得是 `HEAD` —— 和其他 Git 操作一样，这里可以是任意的分支或 Commit ID。
>
> 忘记 `HEAD` 表示什么了？*HEAD 是一个指针，通常用来标识你当前在 Git 仓库中的位置，也就是当前所在分支的最新提交。*
>
> ```
> $ git push rad make-elaina-happy:refs/patches
> ```

这里，我们*无意中*使 `rad/patches/e5f0a5...` 成为了我们的上游分支，这使得我们可以继续以 `git push` 向创建出的 Patch 推送提交。可以使用 `git branch --remotes` 查看详情。

```
$ git branch --remotes
  calyx@z6Mkgom1bTxdh9fMFxFNXFMw3SbXnma6NARdsfcFuunurCap/main
  rad/main
  rad/patches/e5f0a5a5adaa33c3b931235967e4930ece9bb617
```

在这里，`calyx@.../main` 是仓库权威方的**主分支**，`rad/main` 是**我们**仓库的主分支，`rad/patches/...` 是我们提出的 Patch 的分支。我将这一句里比较重要的影响性质的词语标出来。这其实相当于你在 GitHub 里 fork 了目标仓库到自己的仓库，然后你又自己开了分支并发起 Pull Request 一样。

#### 更新 / Update

尽管会有些突兀，不过为了能更好的解释 Patch 的机制，这里我们以一个 Force Push 作为引子。

```
$ git commit --amend
$ git push --force
✓ Patch e5f0a5a updated to revision 4d0c1156f8ac7af2297d1314cd7556185cd16ae4
✓ Synced with 6 node(s)

To rad://z3cyotNHuasWowQ2h4yF9c3tFFdvc/z6MkvZwzK64f3GuDcAs6dEcje89ddfHkBjS1v9Dkh7aCGq3C
 + b766431...80b8d42 anomalous-data-897af -> patches/e5f0a5a5adaa33c3b931235967e4930ece9bb617 (forced update)
```

观察输出结果，这产生了一个新的 Patch ID： `4d0c1156f8ac7af2297d1314cd7556185cd16ae4`。不过这并不是创建了一个新的 Patch，注意到 `rad://` 指向的 URI 并没有改变：我们的提交实则创建了 Patch 的一个 Revision！

Revision 是不可变的，因此即使 `git push --force` 也不会抹除记录。

> [!warning]
>
> 请分清楚接下来我们所使用的提示性占位符 `<PATCH>` 和 `<REVISION>`！

#### 列出与展示 / List & Show

使用 `rad patch` 或 `rad patch list` 列出仓库下创建了的 Patch。

```
$ rad patch
╭───────────────────────────────────────────────────────────────────────────────────────────────────╮
│ ●  ID       Title                                Author           Head     +       -   Updated    │
├───────────────────────────────────────────────────────────────────────────────────────────────────┤
│ ●  e5f0a5a  Add anomalous data from ship 897AF   z6MkvZw…7aCGq3C  80b8d42  +50382  -0  1 hour ago │
╰───────────────────────────────────────────────────────────────────────────────────────────────────╯
```

使用 `rad patch show <PATCH>` 展示 Patch 信息。

```
$ rad patch show e5f0a5a
╭────────────────────────────────────────────────────────────────────────────────╮
│ Title     Add anomalous data from ship 897AF                                   │
│ Patch     e5f0a5a5adaa33c3b931235967e4930ece9bb617                             │
│ Author    paxel (you)                                                          │
│ Head      80b8d420658834afc444a13c03f0c3ff6875a71c                             │
│ Branches  anomalous-data-897af                                                 │
│ Commits   ahead 1, behind 0                                                    │
│ Status    open                                                                 │
│                                                                                │
│ Data from ship 897AF.                                                          │
├────────────────────────────────────────────────────────────────────────────────┤
│ 80b8d42 Add anomalous data from ship 897AF                                     │
├────────────────────────────────────────────────────────────────────────────────┤
│ ● opened by paxel (you) (b766431) 58 minutes ago                               │
│ ↑ updated to 4d0c1156f8ac7af2297d1314cd7556185cd16ae4 (80b8d42) 44 minutes ago │
╰────────────────────────────────────────────────────────────────────────────────╯
```

这里的 `b766431` 和  `80b8d42` 是 Commit Hash，而后者也就是 `80b8d42` 被视为 `HEAD`，我们将使用这个 revision 的 Patch 合入主分支……而在这之前我们还需要对其进行 Code Review。

#### 检出 / Checkout

使用 `rad patch checkout <PATCH>` 来检出 Patch 分支，这将检出 Patch 的 HEAD 指针指向的 Revision 上的提交。

```
$ rad patch checkout e5f0a5a
```

然后你就能用 `code .` 或是 `zed .` 什么的打开当前目录并开始审查了。

如果你只想在终端里随便看看变更，`rad patch diff <PATCH>` 会将 Patch 做出的修改以 diff 形式展示在终端里。

#### 审查 / Review

执行 `rad patch review <PATCH>`，将透过 `EDITOR` 引导你填写审查结果，当然，用 `--message <MESSAGE>` 也是可以的。

这相当于 GitHub 相关流程里的*提交一般性反馈意见*（*Submit general feedback without explicit approval*），如果你想要明确的表示 *Accepted* 或是 `Rejected`/`Request Changes`，你可以对应的传入 `--accept` 和 `--reject` 参数。

```
$ rad patch review --accept
```

或者，如果你只是想给 Patch 或一个 Revision 发一条评论。

```
$ rad patch comment <PATCH/REVISION>
```

#### 合并 / Merge

合并需要你预先检出（Checkout）目标分支，在这之后，使用 `git merge patch/<PATCH>` 合并更改。

> [!warning]
>
> 如果你只是开了个新分支和对应的 Patch，开发完想要合并回主分支，注意使用 `git push rad HEAD:refs/patches` 时，你的默认上游已经被修改。
>
> 此时你需要再手动指定一次目标远程分支，即 `rad`。

```
$ git checkout main
$ git merge patch/e5f0a5a
$ git push red
```

#### 其他

- `rad patch edit <PATCH>` 可以修改 Patch 的描述信息；

- `rad patch label <PATCH>` 和 GitHub 的 Label 差不多，可以给 Patch 添加圆形徽章状的小标签。

  `--add <LABEL>` 是添加；

  `--delete <LABEL>` 是删除。

- `rad patch archive <PATCH>` 可以将 Patch 存档或列为只读，可以用 `--undo` 撤销。

## 搭建做种服务器

做种服务器（Seeder）作为 Radicle 节点来说通常是相对高可用的 —— 如果仅依靠会经常下线的常规用户的节点，内容的获取会变得*非常有挑战性*，因此做种服务器须承担此种大任。

> [!warning]
>
> 做种服务器的运行需要一个新的特殊的 `seed` 用户与用户组，这意味着你需要系统管理员权限。

根据官方文档，你和你的服务器须满足以下标准：

- 安装有或可安装 `curl` 与 `git`；
- 系统管理员权限；
- `systemctl` 版本在 `232` 或以上；
- 具有公网 IP 与指向你服务器的域名解析记录；

鉴于 Radicle 并不承担 NAT Traverse —— 官方说把这活交给 Tor 就解决了，显然没法解决太多。所以对于一些助人为乐的事情，你可能得多考虑一下。嗯，所以我觉得应该可以用些 Tailscale 或是 ZeroTier 或是 EasyTier 或是 Netmaker 什么的，再加上 [traefik.me](https://traefik.me) 应该就行了。

### 安装

添加用户 `seed` 与其同名用户组。

```
$ sudo groupadd --system seed
$ sudo useradd --system --gid seed --create-home seed
$ sudo usermod -a -G seed $(whoami)
$ chsh -s /usr/sbin/nologin seed
$ sudo chown $(whoami): /usr/local/{bin,man,man/man1}
```

搞定了？从 [radicle.xyz/download](https://radicle.xyz/download) 获取对应架构的编译产物，这个页面上有可以帮助你的指南。我们这里将 `rad` 安装到 `/usr/local` 下。

```
$ tar -xvJf <archive> --strip-components=1 -C /usr/local/
```

然后我们 `su` 到 `seed` 用户。

```
$ sudo su seed
```

这里我们先确定一下你预先准备好的域名，他最好是 `seed.example.com` 这种格式，不是？也行。总之使用 `rad auth --alias <SEED>` 为我们的 `seed` 用户初始化 Radicle，步骤与之前的差不多。

```
$ rad auth --alias seed.example.com
```

使用 `rad node config --addresses` 查看 Radicle 节点的配置，如果不出意外，现在这条指令的输出是空的，也就是说还没有办法让其他 Radicle 节点连接上你的做种节点，我们还需要进一步配置。

### 配置节点

在 seed 用户下打开 `~/.radicle/config.json`，如果按照上面的步骤，现在节点配置 `$.node` 的内容应该类似这样：

```json
{
  "node": {
    "alias": "seed.example.com",
    "externalAddresses": [],
    "seedingPolicy": {
      "default": "allow",
      "scope": "all"
    }
  }
}
```

 首先，我们需要修改 `$.node.externalAddresses`，使得 Radicle 守护进程能够向外暴露服务。Radicle 通常使用 `8776` 端口，因此我们需要填 `seed.example.com:8776`。这里可以填多个，具体情况具体分析吧。

```json
{
  "node": {
    "externalAddresses": ["seed.example.com:8776"]
  }
}
```

然后我们得更改做种策略 `$.node.seedingPolicy` —— 如果就按照现在的设置不做修改，你的节点会尝试获取网络中所有的公开仓库的副本，显然，除非你是 Cloudflare 这种等级的赛博菩萨，只是想要不求回报的给网络做出贡献，否则你需要让你的节点有选择的做种。

```json
{
  "node": {
    "seedingPolicy": {
      "default": "block"
    }
  }
}
```

现在我们可以先把服务跑起来，开个 screen 或是 tmux，然后运行：

```
$ rad node start --foreground
```

这会把守护进程运行在前台，使得你可以通过输入 `exit` 来随时退出。这通常被用于调试用途。

如果没有发现问题，我们可以开始配置 `systemd` 服务了。首先，你得先切换回你的系统管理员用户，我们直接使用 `exit`。

```
seed $ exit
elaina $
```

获取服务单元文件（Unit File），将其保存到 `/etc/systemd/system/radicle-node.service`

```
$ curl -sS https://seed.radicle.xyz/raw/rad:z3gqcJUoA1n9HaHKufZs5FCSGazv5/570a7eb141b6ba001713c46345d79b6fead1ca15/systemd/radicle-node.service -o /etc/systemd/system/radicle-node.service
```

如果你并未完全按照上面的步骤进行配置，最好根据你的先验知识编辑一下这个单元文件。搞定了？我们可以启动我们的服务了。

> [!tip]
>
> 根据终端提示，你可能需要运行 `systemctl daemon-reload`。

```
$ sudo systemctl enable --now radicle-node
```

如果没有发生什么特殊情况，现在我们的守护进程，自建 Radicle 节点已经开始运作了。查看其运作情况。

```
$ systemctl status radicle-node
$ journalctl --unit radicle-node --follow
$ sudo -u seed -- rad node status
```

> [!note]
>
> 你需要切换到 `seed` 用户才能使用 `rad` 指令，接下来这条消息将不再重复。

### 设置做种

和上面的章节所提到一样，我们需要配置做种策略。使用 `rad seed <REPO>` 对特定仓库做种，`rad unseed <REPO>` 取消。

```
$ rad seed rad:z3gqcJUoA1n9HaHKufZs5FCSGazv5
$ rad unseed rad:z3gqcJUoA1n9HaHKufZs5FCSGazv5
```

如果你配置为 `$.node.seedingPolicy.default = allow`，你可能需要特别阻止某些特定的仓库被做种，使用 `rad block <REPO>`。

```
$ rad block rad:z9DV738hJpCa6aQXqvQC4SjaZvsi
```

使用 `rad inspect --policy` 检查对某个仓库的做种策略。

```
$ rad inspect rad:z3gqcJUoA1n9HaHKufZs5FCSGazv5 --policy
```

### 连接到节点

在配置好做种节点后，执行 `rad node connect <ADDRESS>` 以连接到做种节点。

```
$ rad node connect seed.example.com:8776
```

搞定，如果不出意外，`rad node status` 应该会展示出当前对我们的做种节点的连接情况。

### 配置 HTTP 访问

现在我们的 Radicle 做种节点还没有办法被 [Radicle Explorer](https://app.radicle.xyz) 检索，也就只能先使用 `rad clone` 等指令把仓库抓取下来才能查看其信息，为此我们需要配置 Radicle 的 HTTP 网关 `radicle-httpd`。

还是前往[下载页面](https://radicle.xyz/download)，根据其提示下载对应架构的 `radicle-httpd`，将其放置在 `/usr/local/` 下并 `chown` 给 `seed` 用户。然后我们简单的下载 `radicle-httpd` 的服务单元文件并启动服务。

```
$ curl -sS https://seed.radicle.xyz/raw/rad:z3gqcJUoA1n9HaHKufZs5FCSGazv5/570a7eb141b6ba001713c46345d79b6fead1ca15/systemd/radicle-httpd.service -o /etc/systemd/system/radicle-httpd.service
```

> [!note]
>
> 你有没有发现这里其实已经使用了 `/raw/rad:<REPO>/<COMMIT>` 路径？这其实就是 `radicle-httpd` 所提供的能力。

```
$ systemctl enable --now radicle-httpd
```

现在我们可以确认一下 HTTP 网关的运作情况。

```
$ systemctl status radicle-httpd
$ curl http://127.0.0.1:8080/api/v1
```

默认情况下，`radicle-httpd` 运行在本地的 `8080` 端口，你可以通过修改服务单元文件来更改。`radicle-httpd` 不提供 HTTPS 能力，你需要通过 `caddy`、`nginx`、`cloudflared` 等反向代理实现。

## 私有仓库

> [!warning]
>
> 私有仓库需要至少一个你私有或信任的做种服务器，而不能使用公开的例如 `seed.radicle.garden`！

要创建私有仓库，可以用 `rad init --private` 初始化 Radicle。

```
$ rad init --private

Initializing private radicle 👾 repository in /home/paxel/private-repo
...

Your Repository ID (RID) is rad:z3jEQE4VMzkR1UVeSLiN9E8AMtV6a.
You can show it any time by running `rad .` from this directory.

You have created a private repository.
This repository will only be visible to you, and to peers you explicitly allow.

To make it public, run `rad publish`.
To push changes, run `git push`.
```

使用 `rad ls --private` 列出私有仓库。

```
$ rad ls --private
╭────────────────────────────────────────────────────────────────────────╮
│ Name                   RID                                 Visibility  │
├────────────────────────────────────────────────────────────────────────┤
│ schrödingers-paradox   rad:z3jEQE4VMzkR1UVeSLiN9E8AMtV6a   private     │
╰────────────────────────────────────────────────────────────────────────╯
```

私有仓库意味着你需要特别的配置可以用于做种的节点，这一能力也是基于 Git 扩展得到的 —— 也就是说可以追溯其更改。

首先，你需要知道目标 Radicle 节点的*去中心化 ID（Decentralized ID abbr. DID）*或*节点 ID（Node ID abbr. NID）*。在做种服务器上以 `seed` 用户运行 `rad self` 即可得知。

```
$ sudo -u seed -- rad self
Alias           elaina
DID             did:key:z6MkthdS453JtGNt8mRCxjpTp4KdY8rXkno1ZHZXE2BtFrMr
└╴Node ID (NID) z6MkthdS453JtGNt8mRCxjpTp4KdY8rXkno1ZHZXE2BtFrMr
SSH             running (?)
├╴Key (hash)    SHA256:cREoz2isf1CyLpHs/bZIKExwMFte7ZYIEZKc+/nCgeE
└╴Key (full)    ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAINOx7lRluaOAExL6GT9XAZrw5Ypcb48mSz3ptemh0nX9
Home            /home/elaina/.radicle
├╴Config        /home/elaina/.radicle/config.json
├╴Storage       /home/elaina/.radicle/storage
├╴Keys          /home/elaina/.radicle/keys
└╴Node          /home/elaina/.radicle/node
```

这个 `did:key:` 开头的就是我们接下来需要用到的信息。

运行 `rad id update` 指令以修改仓库的访问许可，使得仓库允许被同步到对应节点。

```
$ rad id update \
    --title "Allow elaina's access & seed" \
    --allow did:key:z6MkthdS453JtGNt8mRCxjpTp4KdY8rXkno1ZHZXE2BtFrMr
```

> [!tip]
>
> 配置仓库的做种许可需要仓库的权威方（Delegate）权限。

除了需要允许做种节点的访问外，还需要允许其他人自己的 Radicle 节点访问，我们从别人那里问得他们的 DID，并使用 `rad id update` 指令更新许可即可。

```
$ rad id update \
    --title "Allow teague's access & seed" \
    --allow did:key:...
```

### 公开仓库

只需要运行 `rad publish` 即可将私有仓库重新公开。

```
$ rad publish
```

## 项目管理

### 权威方 / Delegate

类似 GitHub 的 owner 权限分级，Radicle 将能够修改仓库特殊信息的一类特权用户称为 Delegate 权威方。一般来说，如果是你自己执行的 `rad init`，那么你已经是仓库的权威方了。

使用 `rad id update --title <TITLE> --delegate <TARGET>` 设置某个节点具备权威方权限。

```
$ rad id update \
    --title "Upgrade elaina as delegate" \
    --delegate did:key:z6MkthdS453JtGNt8mRCxjpTp4KdY8rXkno1ZHZXE2BtFrMr
```

使用 `--rescind <TARGET>` 移除某个节点的权威方权限。

```
$ rad id update \
    --title "Remove delegate user elaina" \
    --rescind did:key:z6MkthdS453JtGNt8mRCxjpTp4KdY8rXkno1ZHZXE2BtFrMr
```

## 参考文献

本文基于以下文章提供的内容撰写。

- [Radicle Official Website](https://radicle.xyz/)
- [Radicle User Guide](https://radicle.xyz/guides/user)
- [Radicle Seeder's Guide](https://radicle.xyz/guides/seeder)
- [Radicle Protocol Guide](https://radicle.xyz/guides/protocol/)

