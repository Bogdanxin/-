# GIT 的分支提交

## 本地

### 1. commit 提交

* 对当前节点进行提交

  ```
  git commit
  ```

### 2. 创建分支、切换分支

* 在当前节点创建一个新的分支

  ```
  git branch <新分支名称>
  ```

  ![image-20200724103600989](https://raw.githubusercontent.com/Bogdanxin/cloudImage/master/20200724103601.png)

* 切换分支，创建分支后，控制的还是原来的另一个分支，所以要进行切换

  ```
  git checkout <新分支名称>
  ```

  ![image-20200724104000256](https://raw.githubusercontent.com/Bogdanxin/cloudImage/master/20200724104818.png)

* 可以既创建分支，还切换到新分支上

  ```
  git checkout -b <新分支名称>
  ```

### 分支合并  

* merge 两个分支进行合并，分支A合并到分支B

  ```
  git merge A
  ```

  先确定`checkout`的是合并的目标B，然后使用`merge`合并被合并的A，最后checkoutA，merge进行合并

  <img src="C:/Users/公维信/AppData/Roaming/Typora/typora-user-images/image-20200724104943532.png" alt="image-20200724104943532" style="zoom:50%;" /><img src="https://raw.githubusercontent.com/Bogdanxin/cloudImage/master/20200724105223.png" alt="image-20200724105223781" style="zoom:50%;" /><img src="https://raw.githubusercontent.com/Bogdanxin/cloudImage/master/20200724105129.png" alt="image-20200724105129820" style="zoom:50%;" />

* rebase 将一个分支的工作，复制一份到另一个分支

  ```
  git rebase <要移动到的分支> // 该方法是默认当前节点为父节点，要移动的节点为子节点，但是移动之后，活动节点为子节点
   
  git rebase <父节点> <子节点> // 从子节点到父节点的父辈节点路径上的所有节点都要依次添加到父节点下，分支还是子节点
  ```

### 分离HEAD

 HEAD 是一个对当前检出记录的符号引用 —— 也就是指向你正在其基础上进行工作的提交记录。

* 通过checkout 哈希值将HEAD指向节点，而不是指向分支

  ```
  git checkout <哈希值>
  ```

注意HEAD是既可以指向分支，也可以直接指向某个节点的

### 相对引用

* 不使用哈希值，直接通过树形结构的相对位置来获取之前的节点，注意，这是HEAD指向的节点

  ```
  git checkout <某个节点（可以是分支，也可以是HEAD，不能是哈希值）>^ 
  git checkout <某个节点（可以是分支，也可以是HEAD，不能是哈希值）>~[sum]
  ```

  `^`符号表示该节点向上一个的父节点，`~[sum]`表示该节点向上sum个节点

  `^`符号还可以作为当一个节点有多个父节点时候，``^[num]``移动到该节点第num个父节点的分支上，利用这两个符号和数字就能在分支树上随意移动了

* 使用强制修改引用，将某个节点强制修改到另个节点的某个父节点的位置，也可以将某个节点强制修改到另个节点（通过哈希值）

  ```
  git branch -f <要修改的分支> <基于HEAD>^/~[sum]
  git branch -f <要修改的分支> <要移动到的节点的哈希值>
  ```

### 撤销变更

* reset撤销本地变更

  对本地变更回溯到多少个版本，HEAD就是指的当前分支

  ```
  git reset HEAD~[sum]/^
  ```

* revert撤销变更，并分享到远程服务

  ```
  git revert HEAD~[sum]/^
  ```

注意，这两个变更都是基于当前分支HEAD来做的，而且HEAD是不能分离分支进行重置的

**所以HEAD必须是checkout当前的分支才能够重置**



### 整理提交记录

* 提交复制到当前位置（HEAD）的下面

  ` cherry-pick` 可以将提交树上任何地方的提交记录取过来追加到 HEAD 上（只要不是 HEAD 上游的提交就没问题）。而且会将HEAD调整到最新的版本，也就是当前树最底端

  ```
  git cherry-pick <节点1的哈希值> <节点2的哈希值>...
  ```

* 交互式rebase

  使用带有`-i`参数的`rebase`命令，打开一个UI从而通过UI控制版本位置

  ```
  git rebase -i HEAD~[num]
  ```

  num是从HEAD（包括HEAD）数往上多少个



### 创建标签

* 创建标签

  ```
  git tag <标签名> <节点哈希值>
  ```



## 远程

### 克隆

```
git clone
```

### 远程分支

远程克隆之后第一个事就是在本地仓库多了一个名为 `o/master` 的分支, 这种类型的分支就叫**远程**分支。由于远程分支的特性导致其拥有一些特殊属性。

远程分支反映了远程仓库(在你上次和它通信时)的**状态**。这会有助于理解本地的工作与公共工作的差别 

远程分支有一个特别的属性，在你检出时自动进入分离 HEAD 状态。Git 这么做是出于不能直接在这些分支上进行操作的原因, 你必须在别的地方完成你的工作, （更新了远程分支之后）再用远程分享你的工作成果。

远程分支有一个命名规范 —— 它们的格式是:`<remote name>/<branch name>`，远程仓库默认名为 `origin`，分支名默认为`master`

### 从远程仓库获取数据

当从远程仓库获取数据时, 远程分支也会更新以反映最新的远程仓库。

```
git fetch 
```

`git fetch` 完成了仅有的但是很重要的两步:

- 从远程仓库下载本地仓库中缺失的提交记录
- 更新远程分支指针(如 `origin/master`)

`git fetch` 实际上将本地仓库中的远程分支更新成了远程仓库相应分支最新的状态。

远程分支反映了远程仓库在**最后一次与它通信时**的状态，`git fetch` 就是与远程仓库通信的方式

但是注意：<font color=red>使用`git fetch`命令只是将远程分支的状态进行更新，不会影响到本地仓库的状态，不会更新本地master分支，不会改变磁盘的文件</font>

### 将远程分支合并到本地分支

由于都是分支，且都在本地，可以使用分支合并的方法将远程分支合并到本地分支如

* `git cherry-pick origin/master`
* `git rebase origin/master`
* `git merge origin/master`
* ...

git提供了更高效的方法，将远程仓库抓取到远程分支，再将远程分支合并到本地分支两个操作合为一个

```
git pull
```

`git pull`就等同于`git fetch` +  `git merge origin/master`

`git pull --rebase`等同于 `git fetch` + `git rebase origin/master`

### 提交代码

* 正常提交

  ```
  git push
  ```

  `git push` 负责将代码变更上传到指定的远程仓库，并在远程仓库上合并你的新提交记录。按照指定顺序进行提交

* 如果远程仓库已经有了其他的新版代码，本地的代码和远程仓库不同，就无法直接push了，需要强制将本地代码和远程代码合并成最新版本，然后在push

  ```
  git pull + git push
  或者
  git pull --rebase + git push
  ```


### 远程跟踪

`master` 和 `o/master` 的关联关系就是由分支的“remote tracking”属性决定的。`master` 被设定为跟踪 `o/master` —— 这意味着为 `master` 分支指定了推送的目的地以及拉取后合并的目标。当克隆仓库的时候, Git 就自动把这个属性设置好了。

Git 会为远程仓库中的每个分支在本地仓库中创建一个远程分支（比如 `o/master`）。然后再创建一个跟踪远程仓库中活动分支的本地分支，默认情况下这个本地分支会被命名为 `master`。

自己指定属性：

* 方法一

  ```
  git checkout -b <本地分支名称> <远程分支名称>
  ```

  创建一个新的本地分支，远程跟踪远程分支，这样操作的就是这个分支了，不再是master了

* 方法二：

  ```
  git branch -u <远程分支名> <本地分支名> # 如果已经在本地分支上了，就不用在写后面的了
  ```

### push 详解

```
git push <remote> <place>
```

例:

```
git push origin master
```

把这个命令：

*切到本地仓库中的“master”分支，获取所有的提交，再到远程仓库“origin”中找到“master”分支，将远程仓库中没有的提交记录都添加上去，搞定之后告诉我。*

我们通过“place”参数来告诉 Git 提交记录来自于 master, 要推送到远程仓库中的 master。它实际就是要同步的两个仓库的位置。

需要注意的是，因为我们通过指定参数告诉了 Git 所有它需要的信息, 所以它就忽略了我们所检出的分支的属性！

这个作用就是为了我们在提交给远程仓库时候，当前的分支处不用要提交，可以指定从place处提交。

### sourse

将本地分支A push 到远程仓库的分支B，是需要source的

要同时为源和目的地指定 `<place>` 的话，只需要用冒号 `:` 将二者连起来就可以了：

```
git push <remote> <source>:<destination>
```

这个参数实际的值是个 refspec，“refspec” 是一个自造的词，意思是 Git 能识别的位置（比如分支 `foo` 或者 `HEAD~1`）

如：`git push origin HEAD^:master`

`:`号左右都是分支，即使从本地已有分支提交到远程没有的分支，也是可以的，git会在本地和远程同时创建一个同名分支



也存在没有source 只有destination的指令

```
git push <remote> :<destination> # 删除远程仓库的destination分支
git pull <remote> :<destination> # 为本地添加一个destination分支
```



### 

### fetch

对于`<place>`参数，就是指定远程仓库的分支

```
git fetch <remote> <place>
```

这个指令就是从指定远程仓库origin的place分支拉取，如果本地不存在place分支，就更新到相关的远程分支上`origin/<place>`

```
git fetch <remote> <source>:<destination>
```

与push的不同，source指的是远程仓库中的分支名称，destination指本地名称，但是不能在检出分支上做



