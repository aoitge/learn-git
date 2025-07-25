# 我的 Git 学习笔记 (第二天)

**今日目标：** 掌握 Git 的分支管理，并学会通过远程仓库与世界进行协作。

---

## 第五章精通：分支——Git 的超能力

分支是 Git 最强大的功能之一。它让并行开发和任务隔离变得轻而易举。

### 1. 核心哲学：为什么需要分支？

- **我遇到的问题：** “如果所有开发都在 `main` 分支上，当一个紧急 bug 出现时，我该如何一边修复 bug，一边不影响正在开发的新功能？”

- **你的解答 (书签比喻)：**
  > **分支不是副本，而是书签！** 创建一个新分支，并不是把整个项目复制一份，那太慢了。它更像是在你当前所在的提交（书页）上，**贴了一张新的、轻巧的“书签”**。
  >
  > - **隔离性：** 你可以在 `hotfix` 这张书签上进行修复工作，在 `feature` 这张书签上继续开发新功能。两边的历史记录沿着各自的书签向前走，互不干扰。
  > - **轻量高效：** 因为只是增删一个指针（书签），所以在 Git 中创建、切换、删除分支几乎是**瞬间完成**的。

- **`HEAD` 是什么？**
  > `HEAD` 是一个特殊的指针，它就像一个“您在这里”的箭头，永远指向你**当前所在的分支（书签）**。当你切换分支时，实际上就是移动了 `HEAD` 这个箭头，让它指向另一张书签。

### 2. 核心工具箱：如何管理分支？

- **创建与切换**
  - `git branch <分支名>`: 创建一个新的分支（书签），但你人还在当前分支。
  - `git switch <分支名>`: 切换到另一个已存在的分支（跳到另一张书签）。
  - **【效率神器】`git switch -c <分支名>`**: **同时完成创建和切换**两个动作。这是开始一项新任务时最常用的命令。(`-c` 是 create 的缩写)。

- **查看与删除**
  - `git branch`: 列出所有本地分支，带 `*` 号的是你当前所在的分支。
  - `git branch -v`: 查看更详细的信息，包括每个分支的最后一次提交。
  - **【安全删除】`git branch -d <分支名>`**: 删除一个**已经合并**的分支。如果 Git 发现这个分支的工作还没合并，它会拒绝删除以保护你的代码。
  - **【强制删除】`git branch -D <分支名>`**: 强制删除一个分支，不管它有没有被合并。用于丢弃不想要的实验性代码。

- **【全局视角】查看分支关系图**
  ```bash
  git log --all --graph --decorate --oneline
  ```
  > 这个命令能以图形化的方式展示所有分支的来龙去脉，非常直观！

### 3. 终极目标：如何合并分支？

当一个分支的任务完成后，我们需要将其成果合并回主干（例如 `main` 分支）。

1.  **第一步：** 切换到**接收**成果的分支：`git switch main`
2.  **第二步：** 执行合并命令：`git merge <功能分支名>`

- **快进式合并 (Fast-forward)**
  > 如果 `main` 分支在你开发期间没有任何新变化，那么它合并时只是简单地把自己的“书签”向前移动到和功能分支同一个位置。历史是一条直线。

- **三方合并 (Three-way Merge)**
  > 如果 `main` 分支在你开发期间**也前进了**，Git 就不能快进了。它会聪明地找到两个分支的“共同祖先”，然后创建一个全新的“合并提交”，这个提交像一个绳结，把两条分叉的历史线重新汇合在一起。

---

## 第六、七、八章精通：远程仓库——与世界同步

本地仓库是你自己的“私人图书馆”，而远程仓库（如 GitHub）是存放代码的“中央公共图书馆”。

### 1. 建立连接：如何让本地与远程“对话”？

- **远程仓库是什么？** 它是一个“裸仓库”，只包含 `.git` 历史，没有工作区。它的昵称通常是 `origin`。

- **【番外篇】身份验证：HTTPS vs. SSH**
  > 你的电脑如何向 GitHub 证明你是你？
  > - **HTTPS (账号密码/令牌)：** 像登录网站。每次操作都需要验证身份。设置简单，但在某些网络环境下可能受限。
  > - **SSH (密钥对)：** **(专业推荐)** 像给你的电脑配一把专属的“数字钥匙”。
  >   - **私钥 (Private Key):** 存在你电脑里，绝不外泄的钥匙。
  >   - **公钥 (Public Key):** 交给 GitHub 的“锁芯”。
  >   - **优点：** 一次配置，永久有效，无需反复输入密码，更安全、更高效。

- **实践命令**
  - `git remote add origin <SSH或HTTPS地址>`: 将一个远程仓库地址添加到你的本地配置中，并给它起个昵称叫 `origin`。
  - `git push -u origin main`: 将本地的 `main` 分支**首次**推送到 `origin`。`-u` 参数会建立跟踪关系，以后推送只需 `git push`。

### 2. 同步工作：如何与团队保持同步？

- **从零开始 (`git clone`)**
  > `git clone <地址>` 不仅仅是下载。它为你做了三件事：
  > 1.  完整下载所有历史 (`.git`)。
  2.  自动设置好了远程地址，并命名为 `origin`。
  3.  自动切换到默认分支（如 `main`）。

- **【番外篇】保持同步：`git fetch` vs. `git pull` (今日重点！)**
  > 这是 Git 协作中最核心的区别。
  > - **`git fetch` (获取)**
  >   - **它做什么：** 从远程仓库下载最新的历史记录，并更新你的**远程跟踪分支** (如 `origin/main`)。
  >   - **它不做什么：** **绝不**会自动修改你本地的 `main` 分支。
  >   - **比喻：** “老板，把最新的菜单给我看看。” (只看不点菜)
  >   - **评价：** **非常安全**。让你有机会在合并前，先查看一下远程更新的内容 (`git log main..origin/main`)。

  > - **`git pull` (拉取)**
  >   - **它做什么：** `git pull` = `git fetch` + `git merge`。
  >   - **比喻：** “老板，按最新的菜单给我直接上一份套餐！” (看了就直接吃)
  >   - **评价：** 方便，但有风险。如果远程更新和你本地修改有冲突，会立刻打断你的工作。

- **【专业工作流推荐】**
  1.  `git fetch origin`  (获取最新变动，更新“地图”)
  2.  `git log main..origin/main` (查看一下有什么新东西)
  3.  `git merge origin/main` (确认无误后，再合并到你的本地分支)

---

**今日总结：** 我已经掌握了分支的完整生命周期管理，并成功将本地项目推送到了 GitHub。我理解了 SSH 认证的原理，并完全弄清了 `git fetch` 和 `git pull` 的关键区别，建立起了安全、专业的远程协作工作流。

---

## 第九、十章精通：成为冲突解决专家

当简单的合并无法满足需求时，我们就进入了 Git 的“深水区”。在这里，我们将学会如何从容地处理最常见的多人协作难题——合并冲突。

### 1. 合并的幕后机制

- **我遇到的问题：** “Git 怎么那么聪明，总能找到两个分支的‘共同祖先’？”
- **你的解答 (寻根问祖)：**
  > Git 把你的所有提交看作一个巨大的、互相连接的“家谱”（技术上称为有向无环图）。当你命令它合并两个分支时，它会：
  > 1.  从 `main` 分支的最新提交出发，沿着“父指针”一路回溯，把所有祖先都列出来。
  > 2.  再从 `feature` 分支出发，做同样的事情。
  > 3.  这两条“寻根”路径上的**第一个交汇点**，就是它们最近的共同祖先。这个过程因为 Git 的巧妙设计而快如闪电。

- **三方合并提交的“指纹”**
  > 在 `git log --graph` 的输出中，一个三方合并产生的“合并提交”最显著的特征就是它有**两个父提交**，像一个绳结一样，将两条原本平行的历史线汇集到了一起。

### 2. 冲突解决实战手册

当 Git 无法自动合并文件时，它会暂停下来，把决策权交给你。这是 Git 最人性化的地方之一。

- **我遇到的问题：** “当终端提示我有冲突时，我的仓库到底处于什么状态？我该如何安全地处理或放弃？”

- **你的解答 (外科手术流程)：**
  > 把冲突解决想象成一台外科手术。

  > **第一步：分析现场 (`git status`)**
  > - 你的仓库正处于一个特殊的**“合并中”**的暂停状态。
  > - `git status` 会清晰地列出 `Unmerged paths` (未合并的路径)，这就是你的“手术清单”。

  > **第二步：查看“病灶” (冲突文件)**
  > - 打开冲突文件，你会看到 Git 帮你标注好的冲突标记：
  >   ```
  >   <<<<<<< HEAD
  >   这里是 HEAD (你当前所在分支) 的代码
  >   =======
  >   这里是另一个分支的代码
  >   >>>>>>> feature-branch-name
  >   ```
  > - 你的任务就是作为医生，决定最终保留哪部分代码，并**亲手移除所有这些标记**。

  > **第三步：【后悔药】安全退出手术 (`git merge --abort`)**
  > - **这是最重要的安全保障！** 如果你发现冲突太复杂，或者不确定如何修改，随时可以执行此命令。
  > - 它会立刻**中止**本次合并，丢弃所有改动，让你的仓库**瞬间恢复到合并之前的干净状态**。让你无后顾之忧。

  > **第四步：完成手术 (`git add` & `git commit`)**
  > 1.  **编辑文件：** 手动修改冲突文件，改成你最终想要的样子。
  > 2.  **`git add <文件名>`：** 在这个场景下，`add` 的意思是“**医生，我确认这个文件的冲突已经处理完毕，可以缝合了**”。它将文件从“未合并”状态标记为“已解决”。
  > 3.  **`git commit`：** 当所有冲突文件都 `add` 之后，执行提交。Git 会自动生成一条类似 `Merge branch '...'` 的提交信息。这个提交，就是那个拥有两个父提交的“合并提交”，标志着手术成功完成。

- **图形化工具 (`git mergetool`)**
  > 对于复杂的冲突，手动编辑容易出错。Git 支持使用图形化工具来解决冲突。这些工具通常会以三栏视图（你的版本、对方的版本、共同祖先的版本）来展示差异，让你更直观地进行合并。启动它的命令是 `git mergetool`。

---

### 【番外篇】精通 Mergetool (以 Neovim 为例)

当你真正遇到冲突时，一个好的合并工具远比手动修改冲突标记更高效、更安全。

- **我遇到的问题：** “我配置了 `mergetool`，但启动后只有一个窗口，还有冲突标记，并没有出现你说的三向对比图，这是为什么？”

- **你的解答 (Windows 环境下的配置陷阱)：**
  > 这个问题非常经典。在 Windows + MINGW 环境下，简单地用 `git config` 设置 `path` 可能会导致 Neovim 无法接收到开启 `diff` 模式的关键参数 (`-d`)。
  >
  > **正确的配置方式**是直接定义完整的启动命令，确保 `-d` 参数不会丢失。

  > **第一步：修正 Git 全局配置**
  > ```bash
  > # 首先，如果之前设置过不生效的 path，先移除它
  > git config --global --unset mergetool.vimdiff3.path
  >
  > # 然后，明确定义完整的启动命令
  > git config --global mergetool.vimdiff3.cmd 'nvim -d "$LOCAL" "$BASE" "$REMOTE"'
  >
  > # (推荐) 关闭合并后产生的备份文件
  > git config --global mergetool.keepBackup false
  > ```

  > **第二步：理解 Neovim 的三向合并“手术台”**
  > - 当你运行 `git mergetool` 后，Neovim 会开启一个三窗格视图：
  >   - **左侧 (LOCAL):** 你的版本 (`HEAD`)。**这里是你的主战场**，你最终的代码将在这里完成。
  >   - **中间 (BASE):** 共同祖先。只读的参考，告诉你文件最初的样子。
  >   - **右侧 (REMOTE):** 对方的版本。也是只读的参考。

  > **第三步：掌握 Neovim 的“手术刀”命令**
  > - 在左侧 `LOCAL` 窗口的冲突块上操作：
  >   - `:diffget RE`: **“用他的”**。从右侧 (`REMOTE`) 拉取版本。
  >   - `:diffget BA`: **“都不要了”**。从中间 (`BASE`) 恢复到最初版本。
  >   - **手动编辑：** 如果想融合两者，直接在左侧窗口编辑即可。
  >   - **保留自己的：** 无需操作，默认就是你的版本。
  > - **完成手术：**
  >   - `:wqa`: 当你确认左侧窗口的内容已经完美后，用此命令**保存并退出所有窗口**。

  > **最终工作流：**
  > 1.  遇到冲突，运行 `git mergetool`。
  > 2.  在 Neovim 的三向视图中，完成你的“手术”。
  > 3.  使用 `:wqa` 保存并退出。
  > 4.  回到终端，运行 `git commit`，完成合并。
