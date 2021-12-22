首先声明，这里讲的Workflow（其实就是开发模式），真的仅限于工作流，而非以工作流为名的各种工具，以及在Git之外的一些做法。 基于Git的工作流，主要是以分支策略来体现，解决分布式协作问题。 本文会严格地限定内容在分支策略中，通过Git的commit、branch、tag操作，来讲解三种基于代码托管平台的Workflow。

它们是：

- GitHub
- GitLab
- Gerrit

它们仨既是最流行的，也是最简洁的。

![A successful Git branching model by Vincent Driessen in 2010](https://about.gitlab.com/images/git_flow/gitdashflow.png)

这张图是2010年Vincent Driessen提出的一个Git工作流，对业界影响深远。 但它太复杂，不在今天的介绍内容中。 今天要介绍的，是更简洁、更基本、更核心的东西。

要注意的是，这些都是孤个人的理解，不是业界的共识。 孤希望这些能成为共识，以减轻上层无知给项目开发带来的痛苦。

## 相关概念 [¶](https://note.qidong.name/2019/04/git-workflow/#相关概念)

![Git基本概念](https://static.qidong.name/img/git-concept.jpg)

| 概念   | 翻译 | 解释                                                 | 特点                                                         |
| ------ | ---- | ---------------------------------------------------- | ------------------------------------------------------------ |
| Commit | 提交 | 每个commit包含文件改动若干及一些说明。               | 包含一个父commit（merge类有两个），和若干子commit。在Git历史中可以看做拓扑点，而Git历史可看做有向图。 |
| Branch | 分支 | 每个branch上可新增commit，或merge其它branch。        | Git历史也可看做河流，而branch就是分叉后的不同流向。未来也可能合流。 |
| Tag    |      | 有四种类型，可带描述或签名。可看做某个commit的别名。 | 它是对某个commit的特殊标定。与branch的HEAD不同的是，它不可变。 |
| Remote | 源   | `git fetch`的来源，`git push`的去路。                | 不同remote包含完整而独立的Git历史。                          |

其中，tag的常见翻译有标签和里程碑两种，但都不能很好地代表它的特殊作用。 如果当年是孤来翻译，孤会译为『版本』——这个翻译没有表达它的全部功能，却是一种带有提示性的限定。 另外，remote一般直译为『远程（仓库）』，孤译为『源』，字少、明确。

以上解释并未包含这些概念的全部内容，只是下面会用到的内容。 另外，再附上角色与权限的表格。

|   GitLab   | 权限（GitHub） |
| :--------: | :------------: |
|   Owner    |     Owner      |
| Maintainer |     Admin      |
| Developer  |     Write      |
|  Reporter  |      Read      |
|   Guest    |      None      |

## GitHub [¶](https://note.qidong.name/2019/04/git-workflow/#github)

GitHub模式是最接近Git分布式设计的模式。

有一个核心仓库，作为对外发布的门面。 核心仓库中只有一个有效分支——`master`。 每个Contributor，通过fork产生一个普通仓库，并且保持`master`与核心库的`master`同步。 普通仓库也可以被fork，产生若干层级。

Contributor在自己的仓库里是可以为所欲为的，但要更新核心仓库，只能向核心仓库的维护者发起请求。 这种更新请求，俗称PR（Pull Request）。

这样，每个开发者远程有两个仓库，本地有一个。 一般，本地仓库是从核心仓库（或某上游仓库）`clone`下来，并添加fork仓库为另一个remote； 当然，也可相反。 总之，本地仓库除了工作区（Working tree)外，还有核心仓库和fork仓库两个remote，以下简称origin和fork（本地工作区简称local）。 理想状态下，三个地方的Git历史大致相同，且仅有一个`master`分支。

```sh
git clone git@github.com/OTHER/repo.git
cd repo
git remote add fork git@github.com/YOU/repo.git
```

首次工作时，在local做出commit，并且push到fork中`git push fork`。 在Web界面上，从fork向origin发起PR。 在PR被接受或拒绝后，这次工作完成。 如果在review时，发现PR需要被修改，则可以继续创建新的commit，更新`fork/master`，PR会自动更新。

再次工作时，如果不用更新，那么和首次工作没有区别。 并非每次都需要更新，如果`origin/master`走得不是很远，或者commit涉及的文件相对独立。 如果需要更新，则要先执行`git pull --rebase origin`，更新local的`master`，再做commit。 这样一来，push到`fork/master`时，可能需要强推`git push fork -f`。 接下来还是在Web界面上，从fork向origin发起PR。

在核心仓库的那边，拥有Write权限的人可以接受PR。 从结果上看，在Git历史中产生了一次merge。

## GitLab [¶](https://note.qidong.name/2019/04/git-workflow/#gitlab)

在GitLab模式中，远程仓库只存在一个核心仓库。 所有人都有push权限，但只能push其它分支； `master`默认被守护，只有较高权限（Maintainer或Owner），才能push或merge，一般只用merge。

Developer在local的`master`做开发，完成后push到某个新的分支。 为了确保不会有人意外地推送到同一分支，一般有两种命名策略：按个人信息命名，或者按feature命名。 然后，Developer需要向`master`分支发起更新需求，俗称MR（Merge Request）。 如果Maintainer认可，接受了MR，就相当于在local执行了`git merge`并push到`master`分支。

```sh
git clone git@github.com/OTHER/repo.git
cd repo
```

首次工作时，只需要clone下来，直接修改`master`，完成后push到某个新的分支即可。

```sh
git push origin HEAD:feature_name
```

然后，在Web界面上发起MR。 如果MR需要更新，则继续向`origin/feature_name`推送commit即可。 无论MR被接受或拒绝，最终这个`origin/feature_name`分支都应该被删除。

再次工作时，和GitHub模式相同，可以选择是否用`git pull --rebase`更新。 更新后，操作和第一次相同。

## Gerrit [¶](https://note.qidong.name/2019/04/git-workflow/#gerrit)

![Gerrit as the Central Repository](https://gerrit-documentation.storage.googleapis.com/Documentation/2.16.7/images/intro-quick-central-gerrit.png)

Gerrit模式和GitLab非常相似，但更简化。

```sh
git push origin HEAD:refs/for/master
```

通过向魔法分支`refs/for/master`推送commit，可以生成新的patch set。 无论是PR还是MR，都可包含多个commit；而一个patch set，只能有一个commit。 一次push多个commit，会产生多个change。 一个change就是一次代码审查的内容，包含一个或多个patch set。 如果change需要更新patch set，则需要在local修改后执行`git commit --amend`，再进行push。

每一次push到`refs/for/master`，是更新某个change，还是产生一个新的change，由changeId决定。 如果changeId相同则更新，否则新建。 每个change都对应一个changeId，它写在commit的message中。

当change的某个（通常是最新的）patch set被submit之后，就会更新`master`分支。 如果commit的parent就是当时`master`的HEAD，则相当于直接push； 如果不是，则产生一次merge。 （这个似乎也可配置。）

## 比较 [¶](https://note.qidong.name/2019/04/git-workflow/#比较)

前面都是极简情况下的介绍，和实际情况往往有些差距。 但如果实际情况能简化成这样，对项目开发来说无疑是一个福音。

以下从模式、平台、复杂情况三个方面，对这三种模式进行比较。

### 模式 [¶](https://note.qidong.name/2019/04/git-workflow/#模式)

Git本身是一种分布式版本控制系统，而这三种都是利用Git的特性实现的中心化开发模式。 中心化的特点，是只有一个核心仓库。 如何更新这个核心仓库，是三种模式的最大区别。

PR和MR其实没有本质区别。 MR虽然更方便，但核心仓库中会有一大堆奇奇怪怪的分支； PR的核心仓库虽然干净，但local的操作复杂，超出了大部分开发者的能力范围。 （没错，虽然看上去简单，但必须深刻理解Git的核心概念和基本操作。 孤在现实中没见过第二个人能用出来，第一个则是镜子里那位。）

和PR/MR相比，patch set差别就大了。 看上去它是最简模式，也是最中心化的模式。 对核心仓库的守护最为严密，操作Git和SVN差不多。 但缺陷是需要把changeId写在commit中，在需要对外发布源码的场景下是难以接受的。 而且，一个patch set只能有一个commit，这对commit的粒度是一种过于严苛的要求——结果往往是粒度过大，代码劣化。

相比之下，PR/MR都可包含多个commit，粒度由开发者自行掌握。 在更新时，PR/MR可以选择新增一个commit，保留犯错历史；或者修改旧的commit，强推到待merge分支。 而patch set只能amend，犯错历史只在Gerrit网页可见。

Gerrit模式是操作最简单的模式，但可能是最难理解的。 像changeId这种东西的植入，几乎不可能靠手工来完成，效率过低；只能在Git Hook中下手，自动生成。 通常，Gerrit都是和`repo`混用。 很多人操作了几年，仍然对Git一知半解，甚至完全不知道自己在干什么——这种令人苦笑不得的效果，已经很难说是优点还是缺点了。

### 平台 [¶](https://note.qidong.name/2019/04/git-workflow/#平台)

谈到比较，往往会关乎选择。 对模式的选择，首先是对平台的选择。

与其说[Gerrit](https://www.gerritcodereview.com/)是一个代码托管平台，不如说是一个Review平台。 它的功能和[GitHub](https://github.com)、[GitLab](https://gitlab.com)相比，就是个子集，不值一提。

[GitHub](https://github.com)和[GitLab](https://gitlab.com)各有所长，但在核心的工作流上，反而是趋同了。 现在PR和MR的区别，就是叫法不一样。 它们都支持同一个仓库不同分支，和不同仓库的两个分支之间的比较和merge。 这里强行用『GitHub模式』和『GitLab模式』来区分两种开发方法，可能反而会让新人难以理解。

[GitHub](https://github.com)胜在外围生态丰富，[GitLab](https://gitlab.com)胜在自身免费。 如果是开源项目，通常优先选择[GitHub](https://github.com)（不知道微软收购对这件事的影响如何）。 而如果是[GitLab](https://gitlab.com)，至少搭建它本身是免费的，所以是企业级内部代码托管的优先选择。

[Gerrit](https://www.gerritcodereview.com/)嘛，除了Android平台项目，几乎没有必要使用它。 然而，它还是靠谷歌的金字招牌，和开源免费、零额外成本（其实也是因为生态几乎为零）的优势，成了很多中国程序员的Git启蒙平台。

### 复杂情况 [¶](https://note.qidong.name/2019/04/git-workflow/#复杂情况)

显然，以上谈的都是仅有`master`单分支的简单情况。 即使是GitLab模式，也只有一个被守护的`master`分支。 要阐述三种模式，帮助读者理解和运用，单分支就够了。 但在实际的项目中，往往会有复杂的需求，迫使人们使用更复杂的模式。

复杂的需求包括但不限于：

1. 发布并维护一个新版本
2. 基于一个项目做多线开发
3. 多个相关项目一起开发

最简单的[Gerrit](https://www.gerritcodereview.com/)，因为设定严格、功能不足，往往是最复杂的。 受Android影响，tag是保留不用的手段。 所以，一切需求意味着拉新的分支。 同样，多项目并行，则通过`repo`的`manifest.xml`组合起来。 一有需求，就集体拉分支。 以至于，有时候你只是安静地开发一个项目，但由于它被配置到了若干`manifest.xml`中，回过神来，已经被拉了一屏幕显示不完的分支。

[GitLab](https://gitlab.com)的一个官方推荐是这样的。

![Environment branches with GitLab flow](https://about.gitlab.com/images/git_flow/environment_branches.png)

孤对此不以为然。 但归根结底，它和[GitHub](https://github.com)都很灵活，怎么用还是可以自行选择。

无论发布新版本，还是基于一个项目做多线开发，GitLab模式如上图所示。 这种新分支，都和`master`一样，是被守护的。 所以，同时存在两种分支。 （Gerrit模式的分支，都相当于是被守护的。代码更新通过魔法分支`refs/for/*`来完成。）

对GitHub模式来说，项目多线开发是用fork。 永远不merge的两条分支，没有在一个仓库同时存在的必要，不如fork出去，改个仓名单干。 发布版本时，通常倾向于使用`master`分支。 与GitLab官方推荐相比，GitHub的`master`就相当于它最稳定的发布分支。 不稳定的开发，都在外围fork的开发仓库中进行。

对于多个项目一起开发，GitHub和GitLab都没有提供官方的解决方案。 Git自身倒是有两个粗浅的方案——[submodule](https://git-scm.com/docs/git-submodule)和[subtree](https://github.com/git/git/blob/master/contrib/subtree/git-subtree.txt)，不推荐尝试。 实际上，想通了会发现，这个问题不应该靠代码托管平台来解决，而是要借助于产物托管平台与包管理器。 这个问题上，切不可被Android蒙蔽了双眼，它只是用一种糟糕的方案，解决了它特有的问题。

## 版本与发布 [¶](https://note.qidong.name/2019/04/git-workflow/#版本与发布)

关于版本发布，其实还有一种更推荐的模式，值得单独介绍。

Git作为一个版本控制系统（VCS，Version Control System），其实自带了版本发布的功能。 如果要给Git的核心概念排个座次，commit无疑是第一位； 而第二位，应该给tag，而非branch。

**版本等价于tag，发布一个新版本就是创建一个tag。** 这是一个懂的人秒懂，不懂的人怎么都不懂的事实。（Gerrit接全锅。）

发布以后，如何维护？ 短期来看，需要基于tag，创建一个新分支。 所有的更新，都往这个branch去merge，然后再适时创建新的tag。

比如，发布一个版本`0.1.0`后，可以创建一个`b0.1.x`分支，继续维护。 如果bugfix进行了一个阶段，需要发布新的版本，则可在`b0.1.x`上创建tag，名为`0.1.1`。

```sh
git tag 0.1.0 master
git checkout 0.1.0 -b b0.1.x
git commit ...
git tag 0.1.1
```

当然，`b0.1.x`分支没有必要常在，甚至没有必要存在。 当`0.1.0`这个大版本不需要再维护后，即可永久删除`b0.1.x`分支。 甚至，从一开始就不需要创建分支，等真的有bugfix的需求了，再来创建、修改、发布、删除一个临时分支。

这是一个简洁的发布模型，在三种模式和平台中都能行得通。 但由于某些原因，在Gerrit平台上往往难以实践——管理员不允许使用tag。

## 总结 [¶](https://note.qidong.name/2019/04/git-workflow/#总结)

就开发模式相关的平台功能支持来说，GitHub和GitLab可以认为没有什么区别，PR和MR可以认为等效。

GitHub模式和GitLab模式可以混用。 核心开发者使用GitLab模式，而普通开发者使用GitHub模式，通常是一种比较方便的混合。 功能上，这两个平台都支持这种混合。 如果开发者对Git都比较熟，建议使用纯粹的GitHub模式，可避免分支约定。

Gerrit模式，看似简单，流毒无穷。 而且，面对复杂情况，GitHub和GitLab更加游刃有余，Gerrit则会让项目陷入泥沼。 有选择权的情况下，建议不要使用Gerrit。