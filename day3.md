# 我的 Git 学习笔记 (第三天)

**今日目标：** 精通变基 (Rebase)，特别是交互式变基，学会像艺术家一样雕琢提交历史。

---

## 第十一章精通：变基 (Rebase)——重塑历史的艺术

变基是 Git 中与 `merge` 并列的、最重要的分支集成策略之一。但它遵循着完全不同的哲学。

### 1. 变基 vs. 合并：两种不同的世界观

- **我遇到的问题：** “`merge` 也能合并代码，为什么还需要 `rebase`？它们到底有什么不同？”

- **你的解答 (绳结 vs. 嫁接)：**
  > **`merge` (合并) 是“系绳结”**：
  > 它会保留两条分支所有的原始提交，然后用一个全新的“合并提交”作为绳结，将两条历史线绑在一起。
  > - **优点：** 忠实地记录了所有发生过的事情，历史是完全真实的。
  > - **缺点：** 当分支众多时，`git log --graph` 会看起来像一碗意大利面，充满了各种分叉和绳结，难以追溯。
  >
  > **`rebase` (变基) 是“嫁接”**：
  > 它不会创造一个绳结。它会先把你当前分支上的所有提交“拔下来”，然后把你的分支起点移动到目标分支的最新末端，最后再把你那些提交**一个个重新种上去**。
  > - **优点：** 创造了一个**完全线性**的历史记录。整条历史是一条干净的直线，非常易于阅读和理解。
  > - **缺点：** 它**重写了历史**。你种上去的那些提交，虽然内容一样，但已经是拥有全新哈希值的“克隆体”了。

### 2. 变基的黄金法则：不可触碰的红线

变基非常强大，但也因此很危险。所有使用者都必须遵守这条用血泪换来的黄金法则：

> **永远不要在一个已经被推送到远程、并且可能被团队共享的分支上执行 `rebase`！**

- **为什么？**
  > 因为 `rebase` 会重写历史。如果你变基了一个共享分支，你的本地历史就和远程历史“分道扬镳”了。你的队友拉取了旧的历史，而你强行推送了新的历史，这会导致灾难性的后果：大量的重复提交、混乱的合并、以及队友对你无尽的抱怨。
  >
  > **结论：** `rebase` 只应该用在**你自己的、还未推送到远程共享的本地功能分支**上，用来在合并前“打扫房间”。

---

## 交互式变基 (`rebase -i`)：终极“时间机器”

这是 `rebase` 最强大的模式，也是你未来会最常用的功能。它让你能精细地控制一连串的提交。

### 核心任务：在发起“拉取请求”前，清理你凌乱的提交记录

想象一下，你开发一个功能，可能会有5、6次提交，比如：`"feat: add button"`, `"style: change color"`, `"fix: oops, fix typo"`, `"refactor: clean code"`。直接把这样凌乱的历史给同事审查是不专业的。交互式变基能让你把这6次提交“浓缩”成一次完美的提交。

### 实战演练：一次完整的交互式变基操作

**目标：** 我们来亲手制造一段凌乱的历史，然后用 `rebase -i` 将它清理干净。

**第一步：营造“犯罪现场”**

1.  确保你在 `learn-git` 项目的 `main` 分支上，并且它是干净的。
    ```bash
    git switch main
    git pull # 确保和远程同步
    ```
2.  创建一个新的功能分支，开始你的“凌乱”开发。
    ```bash
    git switch -c feature/newsletter-signup
    ```
3.  现在，我们模拟几次开发过程中的提交。
    ```bash
    # 第一次提交：添加基础表单
    echo "Newsletter Signup Form" > newsletter.html
    git add .
    git commit -m "feat: Add initial newsletter form"

    # 第二次提交：添加一个样式
    echo "Form Style" > style.css
    git add .
    git commit -m "style: Add basic form styling"

    # 第三次提交：修复一个拼写错误
    echo "Newsletter Sign-up Form" > newsletter.html
    git add .
    git commit -m "fix: Fix typo in newsletter form title"

    # 第四次提交：添加提交按钮
    echo "Submit Button" >> newsletter.html
    git add .
    git commit -m "feat: Add submit button"
    ```
4.  查看你凌乱的历史。
    ```bash
    git log --oneline
    ```
    你会看到类似这样的4次提交。

**第二步：启动“时间机器”**

我们想把这4次提交清理成一个。所以我们需要对从 `HEAD` 往前的4次提交进行操作。

```bash
git rebase -i HEAD~4
```

**第三步：编辑你的“任务清单”**

Git 会在 Neovim 中打开一个任务清单。你的目标是**保留第一次提交，然后把后续的所有提交都合并进去**。

**把文件从这样：**
```
pick 1a1b1c1 feat: Add initial newsletter form
pick 2d2e2f2 style: Add basic form styling
pick 3g3h3i3 fix: Fix typo in newsletter form title
pick 4j4k4l4 feat: Add submit button
```

**修改成这样：**
```
pick 1a1b1c1 feat: Add initial newsletter form
squash 2d2e2f2 style: Add basic form styling
squash 3g3h3i3 fix: Fix typo in newsletter form title
squash 4j4k4l4 feat: Add submit button
```
*   `pick` (p): 保留这个提交，它的提交信息将作为基础。
*   `squash` (s): 将这个提交“压扁”，合并到它前一个提交中。

**第四步：撰写最终的“历史决议”**

保存并关闭 (`:wq`) 任务清单后，Git 会开始工作。因为它遇到了 `squash`，所以它会再次打开 Neovim，让你为这次“四合一”的超级提交撰写一个全新的、最终的提交信息。

打开的文件会包含之前所有的提交信息，像这样：
```
# This is a combination of 4 commits.
# The first commit's message is:
feat: Add initial newsletter form

# This is the 2nd commit message:
style: Add basic form styling

# This is the 3rd commit message:
fix: Fix typo in newsletter form title

# This is the 4th commit message:
feat: Add submit button
```

你的任务是：**删除所有这些内容，只留下一行最能概括这次所有工作的、清晰的提交信息。**

**例如，把它修改成：**
```
feat: Add complete newsletter signup feature
```
现在，再次保存并退出 (`:wq`)。

**第五步：验收成果**

变基完成了！现在，再次运行 `git log --oneline`。

你会惊喜地发现，之前那4条凌乱的提交历史消失了，取而代之的是**一条干净、清晰、专业的提交记录**。

至此，你的 `feature/newsletter-signup` 分支已经打扫干净，可以随时发起一个高质量的“拉取请求”了。

---
---

## 第十二章精通：拉取请求 (Pull Request)——现代协作的灵魂

在你掌握了 `rebase` 这门“手艺”之后，`Pull Request` (PR) 就是你展示手艺、并与团队高效协作的“舞台”。它不是 Git 的一个命令，而是 GitHub/GitLab 等平台提供的核心功能。

### 1. PR 的真正目的：它远不止是合并代码

如果说 `git merge` 解决了技术上的合并问题，那么 PR 就是解决了**团队协作和质量保障**的问题。

- **我遇到的问题：** “既然我可以直接合并分支，为什么还需要费劲地创建一个 PR？”

- **你的解答 (代码的“质量安检门”)：**
  > 把 PR 想象成机场的安检流程。你的代码就是你的“行李”，`main` 分支就是待飞的“航班”。在你的行李被装上飞机之前，必须经过层层检查，确保它安全、合规。

  > - **代码审查 (Code Review) - “人工安检”**:
  >   你的同事会像安检员一样，逐行检查你的代码，发现潜在的 bug、不好的设计或拼写错误。这是保障代码质量最重要的一环。
  >
  > - **自动化检查 (CI/CD) - “X光机”**:
  >   当你提交 PR 时，会自动触发一系列“机器人”检查：代码风格是否统一？单元测试是否全部通过？项目是否能成功构建？这台“X光机”能过滤掉大部分低级错误。
  >
  > - **保护核心分支 - “登机口管制”**:
  >   在专业项目中，你无法直接把代码推送到 `main` 分支。`main` 分支就像一个有专人看守的登机口，只允许通过了所有安检流程的 PR 进入。
  >
  > - **讨论与知识共享 - “安检记录”**:
  >   PR 下的所有评论和讨论都会被记录下来。这不仅解决了当前的问题，更成为了后来者学习的宝贵资料，本身就是一份活的文档。

### 2. 实战演练：一次专业 PR 的完整生命周期

**目标：** 将我们之前用 `rebase -i` 清理干净的 `feature/newsletter-signup` 分支，通过一个完整的 PR 流程，最终安全地合并到 `main` 分支。

**第 0 步：【本地】打扫房间**
- 在你的 `feature/newsletter-signup` 分支上，使用 `git rebase -i` 将凌乱的提交历史清理成一次或几次逻辑清晰的提交。（我们在上一节已经完成了这一步！）

**第 1 步：【本地】推送你的功能分支**
- 将你打扫干净的本地分支推送到远程仓库 (`origin`)。
  ```bash
  git push origin feature/newsletter-signup
  ```

**第 2 步：【GitHub】创建 PR**
1.  在浏览器中打开你的 GitHub 仓库页面，通常会有一个黄色的提示条，引导你为刚推送的分支创建 PR。点击 **“Compare & pull request”** 按钮。
2.  进入 PR 创建页面，填写关键信息：
    - **标题 (Title):** 写一个清晰、概括性的标题，例如 `feat: Add newsletter signup feature`。
    - **描述 (Description):** **这是最重要的部分！** 详细描述你“做了什么”、“为什么这么做”、“如何测试”。一个好的描述能让审查者事半功倍。
3.  点击 **“Create pull request”** 按钮，你的 PR 就正式诞生了。

**第 3 步：【GitHub & 本地】审查与迭代的循环**
1.  **审查：** 你的同事会收到通知，在 GitHub 页面上逐行审查你的代码，并留下评论和修改建议。
2.  **修改：** 你收到了反馈。**你不需要关闭或新建 PR！**
3.  **更新：**
    - 在**本地**的 `feature/newsletter-signup` 分支上，根据反馈做出修改。
    - 完成修改后，正常地 `add` 和 `commit`。
      ```bash
      # 比如，根据反馈修改了表单的提示文字
      echo "Subscribe to our awesome newsletter" > newsletter.html
      git add newsletter.html
      git commit -m "refactor: Update form prompt based on feedback"
      ```
    - 然后，再次执行 `git push`。
      ```bash
      git push
      ```
    - 你刚刚的 `push` 会被**自动追加**到那个已经存在的 PR 中。这个“修改-推送-再审查”的循环会一直持续，直到所有人都满意。

**第 4 步：【GitHub】批准与合并**
- 当你的代码得到足够多的“Approve”(批准)，并且所有自动化检查都显示为绿色对勾时，“Merge pull request”按钮就会被点亮。
- 项目维护者点击此按钮，选择一个合并策略（例如 `Squash and merge` 来保持主干整洁），然后确认合并。

**第 5 步：【GitHub & 本地】清理战场**
- PR 合并后，`feature/newsletter-signup` 分支已经完成了它的历史使命。
1.  **在 GitHub 上：** 点击 PR 页面上出现的 **“Delete branch”** 按钮，删除**远程**的功能分支。
2.  **在本地终端：** 清理你**本地**的功能分支。
    ```bash
    # 1. 切换回主分支
    git switch main

    # 2. 从远程拉取最新的 main 分支代码（包含了你刚刚合并的 PR）
    git pull

    # 3. 安全地删除你本地的、已经完成使命的分支
    git branch -d feature/newsletter-signup
    ```

至此，一次专业、完整的 PR 流程就全部走完了。你不仅贡献了代码，还通过了团队的质量保证体系，并且清理了所有痕迹，让仓库保持整洁如新。 