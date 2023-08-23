# 1、版本控制分类介绍

> 本地版本控制

记录文件每次的更新，可以对每个版本做一个快照，或是记录补丁文件，适合个人用，如RCS；

<img src="./assets/image-20220323215151800.png" alt="image-20220323215151800" style="zoom:80%;" />

> 集中版本控制

所有的版本数据都保存在服务器上，协同开发者从服务器上同步更新或上传自己的修改。

所有的版本数据都存在服务器上，用户的本地只有自己以前所同步的版本。如果不联网的话，用户就看不到历史版本，也无法切换版本验证问题，或在不同分支工作。而且，所有数据都保存在单一的服务器上，有很大的风险这个服务器会损坏，这样就会丢失所有的数据。当然，也可以定期备份。代表产品：SVN、CVS、VSS；

<img src="./assets/image-20220323220328595.png" alt="image-20220323220328595" style="zoom:80%;" />

> 分布式版本控制

所有版本信息仓库全部同步到本地的每个用户，这样就可以在本地查看所有版本历史，可以离线在本地提交，只需在联网时push到相应的服务器或其它用户那里。由于每个用户那里保存的都是所有的版本数据，只要有一个用户的设备没有问题就可以恢复所有的数据，但这增加了本地存储空间的占用；

<img src="./assets/image-20220323220546212.png" alt="image-20220323220546212" style="zoom:80%;" />

# 2、Git基本理论

> 工作区域

Git本地有三个工作区域：工作目录（Working Directory）、暂存区（Stage/Index）、资源库（Repository或Git Directory）。如果再加上远程的git仓库（Remote Directory）就可以分为四个工作区域。文件在这四个区域之间的转换关系如下：

<img src="./assets/image-20220323220918049.png" alt="image-20220323220918049" style="zoom:80%;" />

- Workspace：工作区，就是你平时存放项目代码的地方；
- Index/Stage：暂存区，用于临时存放你的改动。事实上它只是一个文件，保存即将提交的文件列表信息；
- Repository：仓库区（或本地仓库），就是安全存放数据的位置，这里面有你提交到所有版本的数据。其中HEAD指向最新放入仓库的版本；
- Remote：远程仓库，托管代码的服务器。可以简单的认为是你项目组中的一台电脑用于远程数据交换；

<img src="./assets/image-20220323221306286.png" alt="image-20220323221306286" style="zoom:80%;" />

> 工作流程

1. 在工作目录中添加、修改文件；
2. 将需要进行版本管理的文件放入暂存区域；
3. 将暂存区域的文件提交到git仓库；

因此，git管理的文件有三种状态：已修改（modified）、已暂存（staged）、已提交（committed）；

<img src="./assets/image-20220323221518500.png" alt="image-20220323221518500" style="zoom:80%;" />

<img src="./assets/image-20220323223207843.png" alt="image-20220323223207843" style="zoom:80%;" />

# 3、Git环境配置

> 下载

- 官网：https://git-scm.com/

- 淘宝镜像：http://npm.taobao.org/mirrors/git-for-windows/

> Git程序（windows）

- Git Bash：Unix与Linux风格的命令行，推荐；
- Git CMD：Windows风格的命令行；
- Git GUI：图形界面；

> Git配置查看

`git config -l` 查看配置

`git config --system --list` 查看系统配置

`git config --global --list` 查看全局用户配置

# 4、Git操作

## 4.1、本地库初始化

`git init` 方式一：本地仓库搭建

`git clone url` 方式二：克隆远程仓库

<img src="./assets/image-20220323224241348.png" alt="image-20220323224241348" style="zoom:50%;" />

## 4.2、设置签名

- 区分不同开发人员的身份；
- ==**这里设置的签名和登录远程库的账号、密码没有任何关系；**==

> 项目级别/仓库级别：仅在当前本地库范围内有效

`git config user.name RyuAlvin`

`git config user.email ryualvin@aliyun.com`

<img src="./assets/image-20220323225021337.png" alt="image-20220323225021337" style="zoom:50%;" />

`git config --local --list` 也可用这个命令查看当前库用户配置

> 系统用户级别：对当前整个系统有效

`git config --global user.name QQ`

`git config --global user.email 306153957@qq.com`

<img src="./assets/image-20220323225821930.png" alt="image-20220323225821930" style="zoom:50%;" />

`git config --global --list` 也可用这个命令哈看当前系统用户配置

> 优先级

==**就近原则：项目级别优先于系统用户级别，二者都有时采用项目级别的签名。如果只有系统用户级别的签名，就以系统用户级别的签名为准。二者都没有则不允许；**==

## 4.3、工作区/暂存区/本地库间操作

> 工作区<=>暂存区

`git add .` 将所有文件从工作区添加到暂存区；

`git add test1.txt test2.txt` 将具体文件从工作区添加到暂存区；

`git rm --cached test1.txt` 将文件从暂存区移除，但仍保留在工作区；

`git rm -f test1.txt` 将文件从暂存区和工作区中强制删除；

`git restore --staged test1.txt` 将文件从暂存区撤出，退回工作区，但不更改文件内容；

`git restore test1.txt` 在工作区使用此命令，则是撤销更改；

> 暂存区=>本地库

`git commit -m "first commit" .` 将暂存区所有文件提交至本地库；

`git commit -m "first commit" test1.txt` 将暂存区具体文件提交至本地库；

> 删除untracked files（未监控）文件

```bash
# 在拉取的库的过程中，会产生一些中间文件，删除中间文件，用参数n先看看会删掉哪些文件，防止重要文件被误删
git clean -nf
# 连untracked目录一起删除
git clean -nfd
```

## 4.4、状态&日志查看

`git status [filename]` 状态查看；

`git log` 具体显示日志，[空格]/向下翻页，[b]/向上翻页，[q]/退出；

<img src="./assets/image-20220324224345107.png" alt="image-20220324224345107" style="zoom:50%;" />

==**上述Author中的签名在没有配置当前本地库签名的情况下，显示的系统配置.gitconfig中的全局签名；**==

`git log --pretty=oneline` 单行显示日志（完整版本号）；

<img src="./assets/image-20220324224919128.png" alt="image-20220324224919128" style="zoom:50%;" />

`git log --oneline` 单行显示日志（简略版本号）；

<img src="./assets/image-20220324225046324.png" alt="image-20220324225046324" style="zoom:50%;" />

`git log --graph` 图解；

`git log --graph --pretty=format:@%h %s"` 图解（简略版本号）；

`git reflog` 显示版本间移动需要多少步；

<img src="./assets/image-20220324225305616.png" alt="image-20220324225305616" style="zoom:50%;" />

## 4.5、回退

`git reset [--soft | --mixed | --hard] HEAD^` 回退所有内容到上一个版本；

`git reset [--soft | --mixed | --hard] HEAD^ test1.txt` 回退具体文件到上一个版本；

`git reset [--soft | --mixed | --hard] 版本号` 回退到具体版本；

<img src="./assets/image-20220327001816135.png" alt="image-20220327001816135" style="zoom:40%;" />

==**参考：https://baijiahao.baidu.com/s?id=1705605887130284508&wfr=spider&for=pc**==

## 4.6、比较差异

`git diff --cached test1.txt` 比较工作区和缓冲区的文件；

`git diff 版本号 test1.txt` 与历史版本比较；

`git diff --stat [版本号 | --cached]` 显示摘要而非整个diff；

## 4.7、分支操作

`git branch dev` 创建分支；

`git branch -v` 查看所有分支；

`git checkout dev` 切换至dev分支；

`git checkout -m oldName newName` 修改分支名；

==**切换时，若源分支在缓冲区有未提交的文件，也会一起切换至目标分支。在目标分支中将该文件提交后，该文件在源分支和目标分支中都存在，并且状态一致；**==

`git branch --delete dev` 删除分支；

`git branch -D dev` 强制删除分支；

> 合并分支

`git checkout 分支名` 首先切换至Base分支；

`git merge 分支名` 接着合并你想要的内容所在的分支；

<img src="./assets/image-20220327001729915.png" alt="image-20220327001729915" style="zoom:50%;" />

## 4.8、与远程库交互

### 4.8.1、clone和pull

> 本地没有初始化仓库的时候

==**当远程仓库为Public时：**==

`git clone url` 将远程仓库整个下载过来，远程仓库名即为文件夹名；

==**当远程仓库为Private时：**==

- HTTPS：

  <img src="./assets/image-20220326233044727.png" alt="image-20220326233044727" style="zoom:50%;" />

  - 上述消息说明现在用HTTPS方式克隆远程仓库时用户名密码的验证方式已被token取代；

    - 首先需要在服务器上生成一个token；

      <img src="./assets/image-20220326233606952.png" alt="image-20220326233606952" style="zoom:30%;" />

    - 接着通过token验证方式克隆远程仓库；

      `git clone https://token@github.com/RyuAlvin/Test.git`

- SSH：

  - `ssh-keygen -t rsa -f /Users/ryuu/.ssh/id_rsa_github_RyuAlvin -C "Key1"` 生成SSH Key；

    <img src="./assets/image-20220326235445357.png" alt="image-20220326235445357" style="zoom:50%;" />

    - -C：注释，可以是任何字符串，一般写邮箱；
    - -f：指定文件名，防止生成多个密钥的时候被覆盖；

  - 在.ssh目录下创建config文件；

    ```bash
    Host github_RyuAlvin
     HostName github.com
     User git
     PreferredAuthentications publickey
     IdentityFile /Users/ryuu/.ssh/id_rsa_github_RyuAlvin
    ```

    - Host：自己写的名字，可以是任何字符串；

    - HostName：用SSH方式克隆时，地址中的网站名字；

    - User：

      ![image-20220327000103737](./assets/image-20220327000103737.png)

    - IdentityFile：私钥文件。相当于将锁文件id_rsa_github_RyuAlvin.pub给了服务器，在用SSH访问的时候，用钥匙文件id_rsa_github_RyuAlvin去开锁；

  - 将pub文件里的内容拷贝至服务器上；

    <img src="./assets/image-20220327000615923.png" alt="image-20220327000615923" style="zoom:40%;" />

  - `git clone github_RyuAlvin:RyuAlvin/Test.git` 克隆远程仓库；

    <img src="./assets/image-20220327001526448.png" alt="image-20220327001526448" style="zoom:67%;" />

  - 多用户多种服务器情况，在.ssh目录下生成不同的SSH Key，以不同的名字区分。然后在config中配置；

    <img src="./assets/image-20220327095508318.png" alt="image-20220327095508318" style="zoom:50%;" />

    <img src="./assets/image-20220327095541192.png" alt="image-20220327095541192" style="zoom:50%;" />

> 本地已初始化仓库的时候

`git remote -v` 查看当前所有远程地址别名；

`git remote add origin url` 创建远程库地址（origin为别名，可任意）；

`git pull origin master` 将远程库内容下载至本地（master为分支名，须指定）；

- HTTPS：

  `git remote add origin https://token@github.com/RyuAlvin/Test.git` 创建远程库地址别名；

- SSH：

  `git remote add origin github_RyuAlvin:RyuAlvin/Test.git` 创建远程库地址别名；

> 本地已存在分支master，拉取其他分支到本地

```bash
# 前提条件，已设置过远程地址
git remote add origin url
# 拉取远程分支
git fetch origin dev-pc
# 创建分支并切换分支 -b 本地分支名称 远程分支名称
git checkout -b dev-pc origin/dev-pc
```

> 在idea中clone远程仓库

<img src="./assets/image-20220327151800849.png" alt="image-20220327151800849" style="zoom:30%;" />

<img src="./assets/image-20220327152012582.png" alt="image-20220327152012582" style="zoom:30%;" />

### 4.8.2、push

`git push origin master` 将本地代码推送至服务器；

- 在推送的时候必须保证，该分支名在本地仓库和远程仓库中都存在；

- 如果不是基于远程库的最新版所做的修改，不能推送，必须先拉取，解决冲突后再推送；

  <img src="./assets/image-20220327105354000.png" alt="image-20220327105354000" style="zoom:30%;" />

  <img src="./assets/image-20220327105449987.png" alt="image-20220327105449987" style="zoom:40%;" />

# 5、团队成员邀请

- ==RyuAlvin在自己的Test私有库向RyusBen发送成员邀请；==

<img src="./assets/image-20220327110847235.png" alt="image-20220327110847235" style="zoom:40%;" />

<img src="./assets/image-20220327111014022.png" alt="image-20220327111014022" style="zoom:50%;" />

- ==RyusBen接受邀请；==

<img src="./assets/image-20220327123544034.png" alt="image-20220327123544034" style="zoom:40%;" />

- ==RyusBen接受邀请后并没有在自己仓库中多了一个Test仓库。只是RyusBen在访问RyuAlvin主页的时候，可以查看Test私有库；==

# 6、跨团队协作

- ==RyusBen访问RyuAlvin主页，对ForkTest公开库进行Fork；==

<img src="./assets/image-20220327125839209.png" alt="image-20220327125839209" style="zoom:30%;" />

- ==Fork之后，在RyusBen就会增加一个新的ForkTest仓库，但是标有署名RyuAlvin；==

<img src="./assets/image-20220327130423756.png" alt="image-20220327130423756" style="zoom:30%;" />

- ==在RyusBen做出更改后，要发起pull request；==

<img src="./assets/image-20220327131101545.png" alt="image-20220327131101545" style="zoom:30%;" />

<img src="./assets/image-20220327131143019.png" alt="image-20220327131143019" style="zoom:30%;" />

<img src="./assets/image-20220327131233014.png" alt="image-20220327131233014" style="zoom:30%;" />

<img src="./assets/image-20220327131331941.png" alt="image-20220327131331941" style="zoom:30%;" />

- ==在RyusBen提交pull request后，在RyuAlvin界面就会出现有新的pull request；==

<img src="./assets/image-20220327131518545.png" alt="image-20220327131518545" style="zoom:30%;" />

<img src="/Users/ryuu/Library/Application Support/typora-user-images/image-20220327131548898.png" alt="image-20220327131548898" style="zoom:30%;" />

# 7、文件忽略

> .gitignore文件规则

```properties
#注释
#空行或注释被忽略
#可使用Linux通配符

#忽略所有txt文件
*.txt

#test1.txt文件除外
!test1.txt

#abc目录下的所有文件
abc/

#忽略abc目录
/abc

#忽略temp目录下的所有txt文件，但不包括temp下的的子目录中的txt文件
temp/*.txt
```

# 8、补充整理

## 8.1、Git工作流程

![image-20230824070602607](./assets/image-20230824070602607.png)

## 8.2、Git实操

设置用户，让别人知道是谁提交的代码：

```bash
#因为设置的用户名有空格，所以需要加双引号
git config --global user.name "Liu Sibin"
git config --global user.email lsben1014@gmail.com
```

初始化，告诉别人这个文件夹需要版本控制：

设置完了之后，文件夹里就会多了一个`.git`文件夹，初始化后，默认处于主分支`master`。

```bash
git init
```

查看文件状态：

```bash
git staus
```

将文件提交至暂存区（需要退回等操作，可通过git status提供的提示查看）：

```bash
git add
```

将文件提交至本地仓库（需要退回等操作，可通过git status提供的提示查看）：

```bash
git commit
```

同一个文件分别有不同状态在工作区和暂存区可一起提交至本地仓库：

```bash
git commit -m "message"
```

将文件从暂存区直接提交至本地仓库：

```bash
git commit -a -m "all"
git commit -am "all"
```

查看前面提交的版本：

```bash
git log
```

HEAD => master含义：当前处于哪个版本的哪个分支：

![image-20230824072216778](./assets/image-20230824072216778.png)

创建`.gitignore`配置文件，忽略不需要提交的文件：

接着在配置文件中`.gitignore`中设置需要忽略的文件信息。

```bash
touch .gitignore
```

创建新分支：

该命令不会让你马上切换至新分支。

```bash
git branch test
```

查看所有分支：

```bash
git branch
```

切换至分支：

切换后，本地文件夹里的文件是从主分支`master`复制过来的。

```bash
git checkout test
```

切换至主分支（或其他分支），删除`test`分支：

```bash
git branch -d test
```

但当`test`分支和主分支`master`存在差异的时候，会提示你`test`分支还未完全被合并，如果真的确定要删除该分支，则使用以下强制命令：

```bash
git branch -D test
```

创建新分支并马上切换至新分支：

```bash
git checkout -b temp
```

在`temp`分支上添加内容，接着切换至主分支`master`，然后将`temp`上新添加的内容合并过来：

```bash
git checkout master
git merge temp
```

解决冲突：

`HEAD`表示的是当前所在分支的内容，`master`表示的是想要合并的分支中的内容。

![image-20230824073559144](./assets/image-20230824073559144.png)

## 8.3、GitHub

克隆远程仓库至本地：

克隆完成后，就可以看到`.git`文件夹了，如果只是`Download`的话`zip`包里是不存在`.git`文件夹。

```bash
git clone url
```

查看远程仓库地址：

```bash
git remote -v
```

默认都会使用`origin`来表示远程仓库的名字，在操作时通过`origin`代替远程仓库地址：

![image-20230824074306555](./assets/image-20230824074306555.png)

提交至远程仓库：

```bash
git push
```

在2021年后就不支持通过账号密码的方式来提交代码了，需要我们生成个人访问token：

在GitHub上生成好访问token，再次提交代码，输入用户名，然后不要输入密码，粘贴生成好的token，这样就可以将代码提交至远程仓库了。

![image-20230824074424013](./assets/image-20230824074424013.png)

将远程仓库的新变动拉取到本地仓库：

也可以在后面指定远程仓库和分支名。

```bash
git fetch
```

查看本地仓库和远程仓库的区别：

```bash
git diff orgin/main
```

如果没有问题，则可将远程仓库的内容整合到工作区：

```bash
git pull
```
