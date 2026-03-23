# Day1
## 规则和层级
```
git config --xxx
# xxx分三层,下级覆盖上级
# system<global<local
# /etc/gitconfig 系统上所有用户所有仓库，通常是底层网络或安全配置
# ~/.gitconfig 当前用户，常用
# .git/config 项目目录内


# 列出当前生效的规则
git config --list --show-origin
```

## 身份标识
- `user.name` 出现在git log 和GitHub提交历史，不是登录鉴权
- `user.email` 平台使用的验证邮箱，关联到头像和账户
### 配置名字和邮箱
```
# 配置你的名字（建议使用英文名或拼音，避免旧终端乱码）
git config --global user.name "YourName"

# 配置你的邮箱
git config --global user.email "your-email@gexample.com"

# 验证
git config --global --list
user.name=YourName
user.email=your-email@gexample.com
```
> 去掉`global`的话就是为某个项目使用不同身份

### 设置SSH密钥验证
- 生成密钥
```
# 进入到ssh目录
cd ~/.ssh
# 生成ssh public key
ssh-keygen -t rsa -b 4096 -C "your-email@example.com"
# 查看shh public key，这是GitLink平台配置时候需要输入的ssh key
cat id_ed25519.pub
ssh-ed25519 AAAAC3NzaC1lZDI1NAAAAAAAIAsaFu36H6AAAc+YUlSQnhqT1KAAAaXp8FFdNAAADRiE name@example.com
```
- 将密钥添加到Github账户
- 测试SSH连接
`ssh -T git@gitlink.org.cn`

### 配置个人访问令牌
- 获取OAuth2令牌
- 使用令牌认证
```
1. `git clone https://gitlink.com/your-repo.git`
2. `Username: your-email@example.com`
3. `Password: [your OAuth2 token]`
```

# Day2
## 仓库初始化
> `git init`在当前目录下会创建一下`.git`文件夹
- **`objects/`**：这是 Git 的**数据库**。你所有的文件内容、提交记录、目录结构，最终都会被压缩、加密并存储在这里。
- **`refs/`**：这是 Git 的**书签栏**。所有的分支（Branch）和标签（Tag）都存储在这里，指向数据库中的某个特定提交。
- **`HEAD`**：这是 Git 的**当前指针**。它告诉 Git：“我们现在位于哪个时间节点，基于哪个分支工作”。
- **`config`**：这是当前项目的**特有配置**。我们在第二章设置的 Local 级别配置就存储在这里。
- `description`
- `hooks/`
- `info/`

## 工作空间区域
![[Pasted image 20260319004107.png]]
- 工作空间working dir
	- 任意编辑的状态
- 暂存区staging area
	- 缓冲地带，由 `.git/index` 文件来维护
	- 想让指定的部分修改包含在下一次历史快照中，就把它们“放入”暂存区。
	- **隐喻**：它就像是发货前的**打包台**。你可以从仓库里拿十件商品出来，但只把其中三件放进打包箱。(暂存区更像购物车)
- 版本库repository
	- 在`.git`目录中，**永久的**、**不可变的**
	- Git 会将暂存区的内容永久存储到 `objects` 数据库中，并生成一个唯一的哈希值
	- 除非刻意使用高阶命令（如 Reset 或 Rebase）去修改历史，否则不可逆

## 文件状态status
> `git status`
![[Pasted image 20260319004632.png]]
- **Untracked**：新建或刚复制进来的文件，从未被`git add`,未被git管理和保管。`git status` 会显示它。
- **Unmodified**：已经入库，且当前工作区的内容与版本库中的记录（HEAD 指向的版本）完全一致。`git status` 通常不会显示这些文件。
-  **Modified**：文件已被 Git 追踪（Tracked），有修改但修改未到暂存区。在 `git status` 中通常显示为**红色**。既可以向前一步（被暂存`git add`），也可以向后一步（被撤销`git restore`）。
- **Staged**：Git 已记录文件的当前状态，存放在 `.git/index` 中。可执行 `git commit` 将其永久归档；或者使用 `git restore --staged` 将其踢出暂存区（变回已修改状态）

## 忽略的文件
> `.gitignore`用于指定哪些文件和文件夹在版本控制中应当被忽略。它帮助开发者避免将临时文件、编译文件、配置文件、敏感信息等不必要的文件添加到版本库中，从而保持代码仓库的整洁


## 提交commit编写
>业界标准
```
<Type>(<Scope>): <Subject>     # 一行概括
 <Body>                        # 选填，详细描述
 <Footer>                      # 选填，关联信息
```
- `<Type>`
	- `feat`: 新功能 (Feature)。
	- `fix`: 修复 Bug。
	- `docs`: 仅修改了文档 (Documentation)。
	- `style`: 格式调整（空格、分号等），不影响代码运行逻辑。
	- `refactor`: 代码重构（既不修复 bug 也不加新功能，旨在优化结构）。
	- `perf`: 性能优化 (Performance)。
	- `test`: 增加或修改测试用例。
	- `chore`: 构建过程或辅助工具的变动
- **Scope（范围，选填）**：说明改动影响的**模块**。例如 `(auth)`, `(ui)`, `(database)`
- **Subject** ： 主题。简短描述改动内容。用不加句号的祈使句，不超过50字

## Day3
### git log 看历史提交
- **Commit Hash (哈希值)**：那串长达 40 位的字符（SHA-1）。这次提交的**身份证号**，全宇宙唯一
- **HEAD -> master**：这不仅是位置标记，更是 Git 的灵魂。它表示当前你的工作区是基于 `master` 分支的最新提交构建的
- **Author vs Committer**：虽然这里只显示了 Author，但 Git 其实记录了两个名字。**Author** 是写代码的人，**Committer** 是执行提交命令的人（在开源项目中，当你合并别人的 Pull Request 时，这两个名字通常不同）
![[Pasted image 20260321183142.png|370]]

#### 精简冗长的git log
>`git log --oneline --graph --all --decorate
- `--oneline`：将每次提交压缩成一行（7位短哈希 + 标题）
- `--graph`：在左侧绘制 ASCII 字符图。直观展示分支的分叉（Branching）与合并（Merging）路径，让你看清团队协作的拓扑结构
- `--all`：显示**所有**分支的历史，而不仅仅是当前分支
- `--decorate`：显示分支名和标签名（如 `HEAD -> master`, `tag: v1.0`）
![[Pasted image 20260321183514.png|398]]

>git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"

命令起个别名,`git lg`

### 比较版本之间差异
> `git diff`
![[Pasted image 20260321183823.png|559]]
- `git diff`工作区与暂存区对比
	- 场景：在执行 `git add` 之前，最后检查一遍代码
- `git diff --staged` 暂存区与HEAD对比
	- 场景：在执行 `git commit` 之前，确认即将生成的快照内容是否符合预期
- `git diff <commit-hash-A> <commit-hash-B> `跨版本对比
```
	  git diff <commit-hash-A> <commit-hash-B>
	  # 例如：比较当前版本和上上个版本的差异`
		git diff HEAD~2 HEAD
```
#### 读懂diff
- **`--- a/README.md`**：变动前的文件（通常代表旧版本）。
- **`+++ b/README.md`**：变动后的文件（通常代表新版本）。
- **`-This is a draft.`**：红色减号开头，表示这行被**删除**了。
- **`+This is the final version.`**：绿色加号开头，表示这行被**新增**了。
> 在VSCode里可以用`git difftool`对比，有图形化界面

#### 结合日志看变更
>`git diff -p -2 ` 显示最近 **2** 次提交的日志，并同时展开显示它们的 **Diff** 补丁

### 指针HEAD
- HEAD指在GIt历史中的当前位置。是一个真实存在的文件，在.git/HEAD
- 本质是一个*指针的指针* 
	- 它通常**不**直接指向具体的 Commit 哈希值
	- 它指向的是**当前分支**（这里是 `refs/heads/master`）
	- 而 `refs/heads/master` 这个文件里，才真正记录了最新一次提交的哈希值
-   把 Git 仓库想象成一本书：                          
	- commit = 书的每一页                              
	- branch = 书签（夹在某一页上）
	- HEAD = 你的手指，指着你正在看的那个书签           
  commit1 ← commit2 ← commit3 ← commit4            
                                    ↑                
                                  main (书签)
                                    ↑                
                                  HEAD (你的手指)
- `git checkout feature`切换分支

#### 游离状态的指针
>`git checkout abc1234` HEAD 不再指向任何分支，而是直接指向一个 commit

这时候如果做新提交，没有书签跟着走，切走之后新提交 就容易"丢失
![[Pasted image 20260321192953.png]]

如果需要在旧版本上修改，应该创建新分支 `git switch -c fix-old-bug`

### 代码撤销和版本回退
- **Reset**：针对**提交 (Commit)**。通过移动指针，强制回到过去。适用于私有分支的后悔
	- 本质是把HEAD指针往回移动 `git rest HEAD~1`把 HEAD 指针从当前位置，强行拽回到上一个提交
	-  Soft 模式（轻度后悔）
		- **命令**：`git reset --soft HEAD~1`
		- **行为**：
		    - **版本库**：HEAD 指针回退一步。
		    - **暂存区**：不变（保留提交内容）。
		    - **工作区**：不变。
		- **场景**：
		    - **我刚才提交早了！**：提交完发现少文件，或者 Commit Message 写错了。
		    - **我想合并提交**：把最近的两次提交合并成一次。
	- Mixed 模式（中度后悔 - 默认）
		- **命令**：`git reset --mixed HEAD~1` (或简写为 `git reset HEAD~1`)
		- **行为**：
		    - **版本库**：HEAD 指针回退一步。
		    - **暂存区**：**被重置**（回退到上一版本，即清空了暂存状态）。
		    - **工作区**：**不变**（保留了修改，但变为 Unstaged 状态）。
		- **场景**：
		    - **add 了错误的文件**：测试数据不小心 add 进去了。想撤销提交并且重新add 
	- Hard 模式（重度后悔 - 危险）
		- **命令**：`git reset --hard HEAD~1`
		- **行为**：
		    - **版本库**：HEAD 指针回退一步。
		    - **暂存区**：**被重置**。
		    - **工作区**：**被强制重置**（所有未提交的修改瞬间消失，恢复到上一个版本的模样）。
		- **场景**：
		    - **我彻底搞砸了**：代码写得一团糟，想完全放弃最近的尝试，回到过去。
- **Revert**：针对**提交 (Commit)**。通过新增一个“反向提交”来抵消错误。适用于公共分支的修正。不删除历史，而是创建一个**新的提交**
	- `git revert <Commit-B-Hash>`Git 会自动生成一个新的提交 `D`。历史变成了 `A -> B -> C -> D`。 其中 `D` 的内容 = `B` 的逆运算
- **Restore**：针对**文件 (File)**。精准丢弃工作区或暂存区的修改，不涉及历史指针的移动。
	- `git restore config.js` 撤销工作区修改
	- `git restore --staged password.txt撤销暂存区修改
> Git 实际上很难真正删除东西。我们可以使用 git reflog（引用日志）来查找所有 HEAD 变动的轨迹

## Day4
### 内容寻址的底层定义
git储存快照`snapshot`的方式 `内容寻址` 。对于git来说，文件名=元数据，文件内容=唯一的hash值，通过后者来检索内容。每个k-v 对应 hash: 压缩后的文件内容
> **只要内容完全相同，无论文件名叫什么，在 Git 中都是同一个对象**

### 对象储存机制
git储存的是快照，每个快照对应一个“对象“实例（我猜），本质是基于内容寻址的文件系统
对象包括4个内容：
- **Blob Object** 最基本的存储单元，存储文件的**内容**，不包括文件名/路径/权限等
	- 使用zlib压缩算法
	- 内容相同即hash相同
	- 一旦创建不可变
- **Tree Object** 类似目录，记录Blob和Tree的组织结构，存储文件名和对象hash值的映射
	-  **存储目录快照**：记录某个时刻的目录结构
	- **包含元数据**：文件名、文件类型、权限、哈希引用
	- **递归结构**：Tree 可以包含其他 Tree（**Merkle Tree（默克尔树）** 结构，底层变动影响顶层Tree的hash变化）
	- **内容寻址**：Tree 的哈希由其包含的所有条目决定
```
# 根 Tree 对象
tree: d1b2c3...
├── blob: a1b2c3... "README.md" (文件内容)
├── tree: e1f2g3... "src/" (目录)
│ ├── blob: x1y2z3... "main.py" (文件内容)
│ └── blob: m1n2o3... "utils.py" (文件内容)
└── blob: p1q2r3... "config.json" (文件内容)
```

- **Commit Object** 将“目录树”（Tree）与“时间节点”和“作者信息”绑定在一起，形成具有完整上下文的**版本快照** 。用于定格快照，记录历史
	- **Top-level Tree**：指向项目根目录的 Tree 对象哈希。
	- **Parent**：指向父提交的哈希（第一次提交没有父节点，合并提交有多个父节点）。
	-  **Author/Committer**：完整记录了作者 (Author)、提交者 (Committer)、时间戳 (Timestamp) 以及提交信息 (Message)。
	-  **Message**：提交说明。
```
tree d1b2c3... (指向当前版本的顶层目录)
parent a1b2c3... (指向父提交，即上一个版本)
author Gitconomy User <user@gitconomy.org> 1705496400 +0800
committer Gitconomy User <user@gitconomy.org> 1705496400 +0800
feat: add README and source code
```
- **Tag Object** 本质是 Commit 的唯一标记。是一个**真正的对象**，存储在 `.git/objects` 。包含标签名、打标签的人、日期、标签信息，并包含一个指针指向目标 Commit。
    - **引用**：`.git/refs/tags/` 下的文件指向这个 Tag 对象，而 Tag 对象再指向 Commit。
    - **创建方式**：`git tag -a v1.0 -m "Release"`
```
object 8f3a1b2... (指向的目标 Commit 哈希)
type commit (目标对象的类型)
tag v1.0 (标签名称)
tagger Gitconomy User <user@gitconomy.org> 1705496400 +0800
Release version 1.0 (标签说明信息)
```

### 自动去重和增量打包
- **对象复用** 
	- 不同文件名相同内容：只有一个Blob对象，Tree 对象中指向同一个 Blob 哈希值。仅增加索引开支，不增加存储开支
	- 文件修改：不修改原Blob，创建新Blob
- **对象压缩和打包** `.git/objects`有两种文件形态
	- **松散存储 (Loose Objects)** 执行 `git add` 或 `git commit` 时， zlib 算法将每个对象单独压缩，并以哈希值的前两位为目录名，后 38 位为文件名，存储在 `.git/objects/` 
	- **打包存储 (Packfiles)** 松散对象越来越多导致磁盘碎片化。定期（或在执行 `git gc` 时）自动触发回收机制，将零散文件打包成**包文件**
		-  `git gc`（垃圾回收）or 推送到远程仓库时，Git 会启用**打包 (Packing)** 机制分析这些对象，对相似的内容（如同一个文件的 v1 和 v2 版本），使用 **Delta Compression (增量压缩)** 技术：
			- 存储最新版本的完整内容。
			- 存储旧版本相对于新版本的“差异”（Delta）。
			- 所有数据会被塞进一个 `.pack` 文件，并配对一个 `.idx` 索引文件以加快查找

### Merkle Chain Effect
Git 的数据结构是一个 **DAG (有向无环图)**
1. 修改了文件 -> Blob 哈希值变化
2. Blob 哈希变化 -> 包含它的 Tree 对象的哈希值变化
3. Tree 哈希变化 -> 根 Tree 的哈希变化
4. 根 Tree 哈希变化 -> Commit 对象指向的 Tree 变化，生成一个新的 Commit 哈希。

### 数据安全
- **防损坏**：磁盘故障导致的一位比特翻转导致hash变化不匹配
- **防篡改**：没有人能悄悄修改历史记录而不改变 Commit ID

```
# 1. 查看最新的 Commit 的哈希值 (HEAD)
git log -1 --pretty=format:%H
ca82a6d... (假设的哈希值)

# 2. 查看这个 Commit 对象里有什么
# -p 参数表示 pretty-print (漂亮打印)
git cat-file -p ca82a6d
tree 9c4d23...
parent 8f3a1b...
author Gitconomy User...
committer Gitconomy User...

	feat: add hello file

# 3. 顺藤摸瓜，查看那个 Tree 对象
git cat-file -p 9c4d23
3100644 blob e69de29bb2d1d6434b8b29ae775ad8c2e48c5391    hello.txt

# 4. 最后，查看那个 Blob 对象 (文件的真实内容)
git cat-file -p e69de29
Hello Git World!

```
