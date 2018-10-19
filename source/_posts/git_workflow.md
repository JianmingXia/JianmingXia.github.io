---
title: Git Workflow
date: 2018/10/19 20:00:00
tags:
  - Git
categories: Git
---

> 虽然使用Git很久了，但是之前也只是在遵循团队的工作流，没怎么在意使用的工作流模型。这个问题在于朋友沟通时表现得很明显，没有一个官方的名称，沟通起来还是很费劲的。

## 集中式工作流
不改变已有的工作流，依然可以享受Git带来的收益：
* 每个开发者都有一份属于自己的整个工程的本地拷贝，隔离的环境让各个开发者的工作与项目其他部分修改隔离——自由提交到本地仓库（先忽略中央仓库，在合适的时候再push至中央仓库）
* Git的强壮分支与合并模型

### 工作方式
与SVN类似，集中式工作流以中央仓库作为项目所有修改的单点实体。相比SVN缺省开发分支——trunk，Git中是master，所有修改都提交到master分支。
<!-- more -->

#### 使用分支
* master

#### 使用模式
##### 开发者先克隆中央仓库

```plain
git clone ssh://user@host/path/to/repo.git
```

##### 在本地项目中进行编辑及提交修改（此时修改是存在本地的）

```plain
git status # View the state of the repo
git add <some-file> # Stage a file
git commit # Commit a file</some-file>
```

##### 同步至中央仓库（可以选择合适的时间）

```plain
git push origin master
```




![image.png | left | 238x294](https://cdn.nlark.com/yuque/0/2018/png/92822/1539777100567-cfe55a64-d901-47fe-8246-80d50744c53c.png "")


### 冲突解决
中央仓库代表了正式项目，所有的提交历史应当是被尊重且稳定不变的。
如果开发者本地的提交历史和中央仓库有冲突，Git会拒绝push操作



![image.png | left | 228x198](https://cdn.nlark.com/yuque/0/2018/png/92822/1539777118229-b8bafee9-24be-401d-93f9-b4aeb8b8642b.png "")


#### 操作流程
在push至中央仓库前：
* 先fetch在中央仓库中的新增提交
* rebase本地的commit在中央仓库提交历史之上
    如果本地修改与新增提交有冲突，Git会暂停rebase过程，进行手动解决冲突

### 示例
#### 初始化中央仓库



![image.png | left | 115x113](https://cdn.nlark.com/yuque/0/2018/png/92822/1539777136372-c6e974f8-1199-49ef-a8b0-782081afdac1.png "")


#### 开发者克隆中央仓库



![image.png | left | 227x259](https://cdn.nlark.com/yuque/0/2018/png/92822/1539777147690-a2db41c2-e599-46b6-ad21-962d526ca172.png "")


```plain
git clone repo.git
```

__git clone__时会自动添加远程别名origin指向中央仓库

#### 小明开发功能



![image.png | left | 216x258](https://cdn.nlark.com/yuque/0/2018/png/92822/1539777160699-c4cb04b7-0403-4a79-9d62-6a14dee09a64.png "")


```plain
git status # 查看本地仓库的修改状态
git add # 暂存文件
git commit # 提交文件
```

在开发的工作，不断在本地修改、暂存（stage）及提交

#### 小红开发功能



![image.png | left | 221x262](https://cdn.nlark.com/yuque/0/2018/png/92822/1539777171228-0cb9ac2d-744b-48f4-93d8-5b78c9558c1b.png "")


同样，小红此时也做着类似的事

#### 小明发布功能



![image.png | left | 216x261](https://cdn.nlark.com/yuque/0/2018/png/92822/1539777181749-7e697687-d716-4c1e-b6f9-57ef19ee33ea.png "")


当小明完成开发后，会将之前的commit push到中央仓库：

```plain
git push origin master
```

Note：
* origin是克隆仓库时远程中央仓库别名
* master是要推送的目标分支
* 此时中央仓库自克隆代码后还没有被更新过，push操作不会有冲突

#### 小红试着发布功能



![image.png | left | 229x265](https://cdn.nlark.com/yuque/0/2018/png/92822/1539777193897-0c6ff72f-8031-4493-9ad0-f7417039bfdd.png "")


小红同样使用__git push__操作，但由于之前小明已push过一次，此次会有冲突，小红的commit无法覆写中央仓库。小红需要先pull小明的更新到本地仓库，合并修改后，再重试

#### 小红在小明的提交上rebase



![image.png | left | 235x263](https://cdn.nlark.com/yuque/0/2018/png/92822/1539739895530-ae58a2f3-6e76-4f41-8809-dc6ef003e45f.png "")


小红使用__git pull__合并上游的修改到本地仓库，并尝试与本地修改合并：

```plain
git pull --rebase origin master
```

__--rebase__是将小红commit移至同步中央仓库修改后的master分支head：



![image.png | left | 590x307](https://cdn.nlark.com/yuque/0/2018/png/92822/1539739968462-27096eba-516b-4b80-972c-ef54051e0dee.png "")


如果没有__--rebase__选项，pull操作依然可以完成，但每次pull操作同步中央仓库他人修改时，commit历史会以一个多余的【合并提交】结尾。对于集中式工作流，最好是使用rebase。

#### 小红解决合并冲突



![image.png | left | 229x258](https://cdn.nlark.com/yuque/0/2018/png/92822/1539740097813-06f8a59f-690b-4c1e-a015-8518de8bbc1b.png "")


rebase操作过程是把本地提交一次一个地迁移到更新后的中央仓库master分支之上，这意味着要解决迁移每个提交时出现的合并冲突，而不是解决包含所有提交的大型合并时所出现的冲突。
这样的方式能够尽可能保持每个提交的聚焦和项目历史的整洁。

如果小红与小明的功能是不相关的，冲突发生的可能性不太大。如果有，Git在合并有冲突的提交处暂停rebase过程，输出下面的信息并带上相关的指令：

```plain
CONFLICT (content): Merge conflict in <some-file>
```



![image.png | left | 572x288](https://cdn.nlark.com/yuque/0/2018/png/92822/1539740396497-7bc8655c-f9da-4a73-a1f3-93e8b3e2a9c8.png "")


解决冲突文件后，继续使用__git rebase__：

```plain
git add <some-file> 
git rebase --continue
```

以这样的姿势不断的合并，最终就会合并成功。
如果碰到了冲突，而且还搞不定，可以使用__git rebase --abort__命令，回到__git pull --rebase__前的状态

#### 小红成功发布功能
合并完后，就可以使用__git push__发布到中央仓库中了。

### 小结
这个可以作为从SVN版本控制切换到Git时的一个缓冲过程——可以按照SVN的使用方式使用Git
但是没有发挥Git的分布式特性，只适合人数少项目单一的团队。

## 功能分支工作流
功能分支工作流以集中式工作流为基础，不同的是为各个新功能分配一个专门的分支来开发。可以在把新功能集成到正式项目前，用Pull Requests的方式讨论变更：



![image.png | left | 241x154](https://cdn.nlark.com/yuque/0/2018/png/92822/1539777219980-ee84605c-7dce-4ea8-b559-1303b8cdf6cd.png "")




![image.png | left | 589x495](https://cdn.nlark.com/yuque/0/2018/png/92822/1539777235794-4b8e5a86-c61a-498e-a2dd-5eaa30a66f7b.png "")


功能分支工作流的核心思路是所有的功能开发应该有一个专门的分支，而不是在master分支上。这样方便各个开发者在各自的功能上开发而不会弄乱主干代码。
同时，功能开发隔离也让Pull Requests工作流成为可能——为每个分支发起一个讨论，在分支合并正式项目之前，其它开发者有表示赞同的机会

### 工作方式
功能分支工作流仍然用中央仓库，并且master分支还是代表了正式仓库的历史。但不是直接提交至master分支，开发者在开始新功能前创建一个新分支（新分支应具有一个描述性的名字）

对于master分支和功能分支，Git上没有技术上的区别，可以使用之前一样的方式编辑、暂停及commit

#### 使用模式
##### 拉取中央仓库master分支最新代码

```plain
git checkout master
git fetch origin
git reset --hard origin/master
```

##### 创建新的功能分支

```plain
git checkout -b new-feature
```

##### 在本地进行开发

```plain
git status
git add <some-file>
git commit
```

##### 推送到中央仓库中的功能分支

```plain
git push -u origin new-feature
```

##### 发起Pull Request至master分支
如果有冲突先处理冲突

### Pull Requests
在之前也提到过，开发者开发完之后，将功能分支push到中央仓库（不是push到master分支）。一旦功能完成，不是立即合并到master，而是在功能分支上发起一个Pull Request请求，将修改合并至master——此时，其它的开发者可以去Review变更。
一旦Pull Request被接收了，发布功能就和集中式工作流很类似：
* 确定本地的master分支和上游的master分支是同步的
* 合并功能分支到本地master分支，并push本地master分支至中央仓库

### 示例
#### 小红开发一个新功能



![image.png | left | 238x118](https://cdn.nlark.com/yuque/0/2018/png/92822/1539777264290-1bc8d1d9-d826-4efc-9933-3abd671e4a77.png "")


新建一个功能分支：
```plain
git checkout -b marys-feature master
```

```
git status
git add <some-file>
git commit
```

#### 小红去吃个午饭



![image.png | left | 220x256](https://cdn.nlark.com/yuque/0/2018/png/92822/1539777285288-7caf6f76-b8a5-4ae7-a208-8d2f08369fae.png "")


此时小红可以将早上开发的新功能提交push到中央仓库（假如吃饭的时候硬盘进水了呢）：

```
git push -u origin marys-feature
```

-u选项设置本地分支跟踪中央仓库中对应的分支——设置好之后，可以使用git push命令进行push

#### 小红完成开发



![image.png | left | 217x250](https://cdn.nlark.com/yuque/0/2018/png/92822/1539777584276-8895801b-5444-452a-84c7-b3e327fdb95c.png "")


在合并到master之前：
* 确认功能开发代码已push到中央仓库
* 发起Pull Request，请求合并分支至master

#### 小黑收到Pull Request



![image.png | left | 219x260](https://cdn.nlark.com/yuque/0/2018/png/92822/1539777872181-5fa9caa2-e057-4bb3-b399-e5ee127ae0c2.png "")


查看分支的修改，决定是否需要进行修改（在这个过程中可以与小红进行讨论）

#### 小红完成修改



![image.png | left | 220x255](https://cdn.nlark.com/yuque/0/2018/png/92822/1539777862550-e2c6b61d-0267-4c42-b862-e0e86fbe21ce.png "")


根据讨论结果做对应的修改，并最终带代码push到中央仓库对应的分支——这些提交都会反映在Pull Request中，小黑可以继续讨论（甚至小黑有需要，可以将功能分支pull到本地，修改之后，再push到远程仓库的功能分支）

#### 小红发布功能



![image.png | left | 220x104](https://cdn.nlark.com/yuque/0/2018/png/92822/1539778018227-cb43cdda-156b-4469-b3a1-4321051137f3.png "")


```
git checkout master
git pull
git pull origin marys-feature
git push
```

在我的使用中，当Pull Request通过后，代码会自动合并到目标分支中；如果不可以，可以通过以上方式来合并代码

#### 与此同时，小明在做同样的事
当小红和小黑在她的功能分支讨论Pull Request时，小明可以在自己的功能分支做相同的事——这就是功能分支工作流带来的益处。

### 小结
较于集中式工作流
* 功能分支工作流可以完成多功能的开发，经过讨论的代码才会在master分支中
* 使用Pull Request，可以讨论某个提交

功能分支工作流是开发项目异常灵活的方式，但也带来了灵活的问题——对于大型团队，需要给不同分支分配一个更具体的角色。Gitflow工作流是管理功能开发、发布准备和维护的常用模式

## Gitflow工作流
Gitflow工作流通过为功能开发、发布准备和维护分配独立的分支，让发布迭代过程更流畅。严格的分支模型也为大型项目提供了一些非常必要的结构



![image.png | left | 561x323](https://cdn.nlark.com/yuque/0/2018/png/92822/1539825709642-97409c57-6c61-4d88-92e9-1559fe266cfd.png "")


Gitflow工作流定义了一个围绕项目发布的严格分支模型——虽然比功能分支工作流更加复杂，但提供了更加健壮的用于管理大型项目的框架——并没有超出功能分支工作流的概念和命令，而是通过为不同的分支分配一个明确的角色，并定义分支之间如何以及什么时候做交互。

### 工作方式
Gitflow工作流仍然用中央仓库作为所有开发者的交互中心，和其它工作流一样，开发者在本地工作并push分支到中央仓库中

### 历史分支
Gitflow工作流使用两个分支来记录项目的历史——master分支存储了正式版本的历史，而develop分支作为功能的集成分支：



![image.png | left | 578x150](https://cdn.nlark.com/yuque/0/2018/png/92822/1539826025030-33b1c29d-fcea-4443-83bd-d4cdc391959e.png "")


### 功能分支
每个新功能都有自己的Feature分支，可以Push到中央仓库做备份及写作。需要注意的是，Feature分支是从develop分支拉出来的，当功能完成后，同样也是合并到develop分支——功能分支应当从不与master分支交互



![image.png | left | 569x255](https://cdn.nlark.com/yuque/0/2018/png/92822/1539826175981-4dd205a9-26a2-4707-89e5-aa09337118db.png "")


Note：在这里能看到功能分支工作流的用法，但Gitflow并没有止步

### 发布分支



![image.png | left | 567x303](https://cdn.nlark.com/yuque/0/2018/png/92822/1539826233302-03fcb43b-e9a3-45c8-9593-b1167eabffd5.png "")


一旦develop分支的功能可以发布了，就从develop分支checkout一个发布分支——release：新分支用于发布循环，在这个时间点之后新的功能不能再加到这个分支上（这个分支只应该做Bug修复、文档生成和其它面向发布任务）。
一旦对外发布工作都完成了，将发布分支合并到master中并分配一个版本号打好Tag。有一个重要的点，在Release分支上做的修改需要合并到develop分支中。
使用一个用于发布准备的分支，使得一个团队在完善当前的发布版本的同时，另一个团队可以继续开发下个版本的功能。

常用的分支约定：

```
用于新建发布分支的分支: develop
用于合并的分支: master 及 develop
分支命名: release-* 或 release/*
```

### 维护分支



![image.png | left | 564x355](https://cdn.nlark.com/yuque/0/2018/png/92822/1539826542005-d33ac19a-eee0-4249-812e-95ab3fa259ee.png "")


维护分支或是热修复分支用于给产品发布版本快速生成补丁，这是唯一可以从master分支fork出来的分支，修复完成后——修改马上合并到master分支及develop分支

### 示例
#### 创建开发分支



![image.png | left | 226x147](https://cdn.nlark.com/yuque/0/2018/png/92822/1539826649170-f25b0eb6-53ca-41b5-a887-2971cdf49cbd.png "")


中央仓库的master分支及develop分支：

```
git branch develop
git push -u origin develop
```

开发者克隆中央仓库，建好develop分支的跟踪分支：

```
git clone ssh://user@host/path/to/repo.git
git checkout -b develop origin/develop
```

#### 小红和小明开发新功能



![image.png | left | 225x232](https://cdn.nlark.com/yuque/0/2018/png/92822/1539826768518-073e796f-2155-48ee-b502-5c78cda3003c.png "")


小红和小明为各自的功能创建相应的分支（基于develop分支）：

```
git checkout -b some-feature develop
```

按照之前的操作进行开发：编辑、暂存、提交：

```
git status
git add <some-file>
git commit
```

#### 小红完成功能开发



![image.png | left | 225x105](https://cdn.nlark.com/yuque/0/2018/png/92822/1539826878786-66038bd2-b516-42ce-83c1-9bb152c170e3.png "")


如果团队不使用Pull Requests，可以直接合并到本地的develop分支后push到中央仓库：

```
git pull origin develop
git checkout develop
git merge some-feature
git push
git branch -d some-feature
```

#### 小红准备发布



![image.png | left | 223x141](https://cdn.nlark.com/yuque/0/2018/png/92822/1539826996394-af5e83ae-960e-40e6-ac09-4284f8c65833.png "")


```
git checkout -b release-0.1 develop
```

这个分支是清理发布、执行所有测试、更新文档和其它为下个发布做准备操作的地方——可以理解为改善发布的功能分支

#### 小红完成发布



![image.png | left | 230x220](https://cdn.nlark.com/yuque/0/2018/png/92822/1539827097474-6907d938-9372-4cf7-be12-06df90e69bcf.png "")


一旦准备好了对外发布，小红合并修改到master分支及develop分支，然后删除发布分支——合并develop很重要，在发布分支提交的更新在后面的新功能也是要可用的——如果要求Code Review，也是一个发起Pull Request的理想时机

```
git checkout master
git merge release-0.1
git push
git checkout develop
git merge release-0.1
git push
git branch -d release-0.1
```

发布分支是作为功能开发（develop分支）和对外分布（master分支）间的缓冲，只要合并到master分支，应当打好Tag，方便追踪：

```
git tag -a 0.1 -m "Initial public release" master
git push --tags
```

#### 最终用户发布Bug



![image.png | left | 230x118](https://cdn.nlark.com/yuque/0/2018/png/92822/1539827321846-bfcd3138-2fa1-4284-95ad-c9fe0144d346.png "")


小明从master分支fork一个维护分支，提交修改以解决问题，然后合并回master分支：

```plain
git checkout -b issue-#001 master
# Fix the bug
git checkout master
git merge issue-#001
git push
```

与发布分支一样，维护分支中的重要修改也需要合并到develop分支中：

```
git checkout develop
git merge issue-#001
git push
git branch -d issue-#001
```

## Forking 工作流
Forking工作流是分布式工作流，充分利用了Git在分支和克隆上的优势。可以安全可靠地管理大团队的开发者，并能接受不信任贡献者的提交。
Forking工作流的一个主要优势就是，贡献的代码可以被集成，而不需要所有人都能push代码到仅有的中央仓库中。开发者可以push到个人仓库，虽然没有中央仓库的修改权限，但是可以通过Pull Request的方式，来请求合并代码。
这种模式一般活跃于各大开源社区，非常适合真正的大型、远程分布的团队开发工作，也非常适合开源项目，适合那些不限制贡献代码人员的场景。

### 工作方式
中央仓库仅作为公开仓库——其它开发者不允许push到这个仓库，但可以pull。开发者通过Fork的模式，然后clone个人仓库中的代码进行开发。完成开发后，代码也是提交到个人仓库，为了集成代码到公开仓库，通过Pull Request的方式。

## 我们的Git Workflow
* 使用fork模式，区分中央仓库与个人仓库——中央仓库用于持续构建与持续交付，记录重要版本信息
* 在中央仓库，只有两种分支：master及release
    * release分支数量视发布Minor版本数量而定
* 合并中央仓库的代码必须以PR形式来完成，不允许直接提交
* 以PR方式合并代码时，如果Commit数量较多（比如一次功能开发包含n个commit），推荐使用squash commits
* 中央仓库的master分支始终是最新的代码，并且可被构建、可被发布、可被运行，禁止未通过CI的代码合并
* 为了保持中央仓库的线性修改记录，中央仓库的merge必须是Fast-Forward形式
* 个人仓库的分支模型不做限制，建议使用master/feature/release三分支形式
* 中央仓库的release分支命名以__release/Semver语义化版本__形式命名，并且每个线上发布版本必须添加Tag，方便回滚
* 线上紧急bug和hotfix在对应版本release分支进行
    * 先在个人仓库release分支完成
    * 向中央仓库对应release分支提PR
    * CI通过、审阅完成后合入release分支
    * 由审阅人发起由release分支merge至master分支
    * 发布完成后在当前release branch打上递增的tag

## 小结
没有唯一最佳实践，只有最适合团队的实践！

## 资料
* [https://www.atlassian.com/git/tutorials/comparing-workflows](https://www.atlassian.com/git/tutorials/comparing-workflows)
* [https://github.com/xirong/my-git](https://github.com/xirong/my-git)
* [https://nvie.com/posts/a-successful-git-branching-model/](https://nvie.com/posts/a-successful-git-branching-model/)

