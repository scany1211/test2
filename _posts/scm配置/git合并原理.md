相信刚开始使用git的时候大家都会遇到这些困惑：为什么我本地有些文件合并完远程代码后就消失了？我到底应该什么时候使用rebase什么时候使用merge？为什么我rebase代码时会有这么多冲突？

在我们搞清楚git的底层合并原理时，这些问题就迎刃而解了。

# Tree-Way Merge

 假设有两个同学在各自的分支上对同一个文件进行修改，如下图：

![img](https://pic4.zhimg.com/80/v2-28c7383a01dac8dea4584ce4d045dbdb_720w.jpg)

这个时候我们需要合并两个分支成一个分支，如果我们只对这两个文件进行对比，那么在代码合并时，只知道这两个文件在第30行有差异，却不知道应该采纳谁的版本。

如果我知道这个文件的“原件”，那么通过和“原件”代码的对比就能推算出应该采用谁的版本：

![img](https://pic1.zhimg.com/80/v2-f2e1ba69c71cbb485db2b35b7217c910_720w.jpg)

图示可以看出，Mine中的代码和Base一样，说明Mine中并没有对这行代码做修改，而Yours中的代码和Base不一样，说明Yours在Base的基础上对这行代码做了修改，那么Yours和Mine合并应该采用Yours中的内容。

当然还有一种情况是三个文件的代码都不相同，这就需要我们自己手动去解决冲突了：

![img](https://pic4.zhimg.com/80/v2-00942e0be3adef730ceb60e549be4e67_720w.jpg)

从上面的例子可以看出采用Tree-Way-Merge（也称为三向合并）原理来合并代码有个重要前提是可以找到两份代码的“原件”，而git因为记录了文件的提交历史，再通过自身的合并策略就可以找到两个commit的公共commit是哪个，从而通过比对代码来进行合并。

那么后面我们就来详细说一下**git是如何记录提交历史**和**git的合并策略是怎么推算出公共commit**的。

# git是如何记录提交历史的

我们在控制台执行`git init`，git会创建一个`.git`目录，这个目录包含了几乎所有git存储和操作的东西，结构如下：

```text
- config  //项目特有的配置选项
- description  //仓库描述信息，主要给gitweb等git托管系统使用
- HEAD  //文件，记录目前正在使用的分支
- hooks/  // 目录，包含git的钩子脚本
- info/   // 目录，包含一个全局性排除文件， 用以放置那些不希望被记录在 .gitignore 文件中的忽略模式
- objects/   //目录，存储所有的数据内容，
- refs/   //目录，存储指向数据的提交对象指针
```

其中HEAD文件，refs和objects目录是git能够记录提交历史的关键所在。

## blob object

接下来我们创建两个文件，并通过`git add .`将修改提交到git暂存区中:

```text
echo '111' > a.txt
echo '222' > b.txt
git add .
```

这个时候再去.git/object目录下，你会发现仓库里多了两个文件：

![img](https://pic3.zhimg.com/80/v2-d1ff475b3c2710472393d81db4ad801a_720w.jpg)

>  tree命令： 打印指定目录下的文件结构，mac需要自行安装：`brew install tree -g`
>  

好奇的我们再来打印一下这两个文件里的内容：

```js
cat c2/00906
cat d5/234eb
```

![img](https://pic3.zhimg.com/80/v2-07331bde6db791817c88e0082bd14752_720w.png)

 发现输出了一串乱码，这是因为git将信息压缩成二进制文件，不过没关系git提供`cat-file`命令来取回转码前的数据，`-t` 可以查看object文件的类型，`-p`可以查看object文件的具体内容。

>  git cat-file [-t] [-p]  object filename   object的名字不用输入全，能唯一区分就好（一般都是6位）
>  

```js
git cat-file -t c20090
git cat-file -p c20090
```

![img](https://pic4.zhimg.com/80/v2-0084458242b5681217de82859248403b_720w.png)

可以发现这个object文件是一个blob类型的节点，他的内容是”222“，也就是说这个object存储着`b.txt`文件的内容。

这是我们遇到的第一种git object：blob，它只存储一个文件的内容，不包含文件名等其他信息。该内容加上特定头部信息一起的 SHA-1  校验和为文件命名，作为这个object在git仓库中的唯一身份证。 校验和的前两个字符用于命名子目录，余下的 38 个字符则用作文件名。

此时，我们的git仓库长这样：

![img](https://pic1.zhimg.com/80/v2-ffe54a733b224a1f2b9fd1a825052da4_720w.jpg)

## tree object

我们继续探索，在控制台输入`git commit -m 'feat: 第一次commit'`，打印objects文件目录如下：

![img](https://pic1.zhimg.com/80/v2-5ee57bcfc8e331d4a16e7ebeeec9ff7c_720w.jpg)

继续`cat-file`查看新增的两个文件的类型和内容：

![img](https://pic3.zhimg.com/80/v2-3e6309146a7e6bc942878e94bb67a0e2_720w.png)

 这里我们遇到了第二个git object类型：tree，它将当前的目录结构打了个快照，从它的存储内容中可以发现它存储了一个目录结构，以及每个文件的模式编号，对应的blob object Hash值，文件名。

此刻我们的git仓库是这样的

![img](https://pic1.zhimg.com/80/v2-f66911b3c38f9da88520d1a2cd9b452c_720w.jpg)

## commit object

接着看另一个object文件的内容：

![img](https://pic4.zhimg.com/80/v2-40b4c6f948e6ea887cdca0c43114483f_720w.jpg)

至此我们得到了第三个git object类型：commit，它记录了当前项目tree object的Hash，作者/提交者的信息，以及这条commit的提交注释。

此刻我们的仓库是这样的:

![img](https://pic4.zhimg.com/80/v2-80e9daf485423c680e1abb1dbd7d3a2b_720w.jpg)

## HEAD and refs

我们再来看下HEAD和refs的内容：

![img](https://pic3.zhimg.com/80/v2-989990d40b87cac024d25ea5bb6c4dbe_720w.png)

refs下存储着你所有分支的当前commit object Hash，HEAD相当于一个指针，指向refs中你当前的分支。

**概括来说整个引用关系就是：HEAD里面的内容是当前分支的ref，而当前ref的内容是commit hash，commit object内容是 tree hash，tree object的内容是blob hash，blob存储着文件的具体内容**

![img](https://pic2.zhimg.com/80/v2-7944f6be18d50de767a9061a50c21615_720w.jpg)

为了更具象的看一下ref和HEAD指针代表值的含义，我们切换到一个新分支重复我们刚才的操作，控制台输入：

```text
git checkout -b dev
echo '333' > a.txt
git add .
git commit -m 'feat: 第二次dev提交'
```

可以看到object下又多了三个文件：

![img](https://pic3.zhimg.com/80/v2-5c845efe5c8c0e52ef26dc00bce0734a_720w.jpg)

我们依次输出一下三个文件的内容和类型：

![img](https://pic2.zhimg.com/80/v2-e6ef1a1a5695d597a9e57764b9c066cd_720w.jpg)

依次多了blob object，tree object和commit object，因为我们并没有修改b.text文件的内容，所以仓库里和b.text有关的blob object依旧是之前的那一个。

再看一下此刻的HEAD和refs下的内容：

![img](https://pic4.zhimg.com/80/v2-d2c139f6e913209ae091a108abdb2b6b_720w.jpg)

refs目录下增加了对应的heads/dev文件，记录的是我们新生成的commit object Hash，HEAD里的内容也变成了当前分支：dev。

此时的仓库为：

![img](https://pic4.zhimg.com/80/v2-6c3274095185730b0e7e60af31f92417_720w.jpg)

# commit 和 commit

到这一步，我们已经能弄清楚git是如何保存我们在代码仓库里写的代码，以及如何区分不同分支下的代码是什么样的。下面我们来看一下同一个分支的多个commit记录是怎么关联起来的？

实际上commit object除了记录当前项目tree object的Hash，还会记录前一次提价的commit object Hash，上面那个例子是因为我们是第一次提交，所以没有对应的记录值。

在控制台输入：

```text
git checkout master
echo '444' > a.txt
git add .
git commit -m 'feat: 第二次提交'
```

继续打印objects目录下新增的object类容和类型：

![img](https://pic4.zhimg.com/80/v2-f713116c02657e8c8410294cc7b70737_720w.jpg)

![img](https://pic3.zhimg.com/80/v2-b9f5f8befa3105f269fd8c250a2d1606_720w.jpg)

可以看到commit的内容里多了一个parent字段，记录了上一次提交的commit object Hash。

此刻的仓库长这样：

![img](https://pic3.zhimg.com/80/v2-4be5787538b68b0ac950651e31aa30ce_720w.jpg)

通过这种方法，一个分支下的所有提交就能像一个树一样串联起来了。

# git的合并策略

了解完git是怎么记录提交历史后，我们来了解一下git的合并策略。

git会有很多合并策略，最常见的几种是 **Fast-foward，Recursice，Ours，Octopus** 。默认git会帮你自动挑选合适的合并策略，也可以通过`git merge -s 策略名字`来强指定使用的策略类型。

## Fast-foward

![img](https://pic4.zhimg.com/80/v2-2645489d97a25cd331f0253f246d95af_720w.jpg)

Fast -foward是最简单的一种合并策略，如图将dev分支合并到master分支上，git只需要将master分支的ref指向最后一个commit节点上：

![img](https://pic2.zhimg.com/80/v2-bec8e3a9e4904552f6a018a7be535531_720w.jpg)

Fast-forward是git在合并两个没有分叉的分支时的默认行为，如果你想禁用掉这种行为，明确拥有一次合并的commit记录，可以使用`git merge --no-ff`命令来禁用掉。

## Recursive

Recursive是git中最重要也是最常用的合并策略，简单概述为：通过算法寻找两个分支的最近公共祖先节点，再将找到的公共祖先节点作为base节点使用三向合并的策略来进行合并。

举个例子：圆圈里的字母为当前commit中的内容，当我们要合并2，3两个分支时，先找到他们的公共祖先节点1，接着和节点1的内容进行对比，因为1的内容是A，所以2并没有修改内容，而3将内容改成B，所以最后的合并结果的内容也是B。

![img](https://pic3.zhimg.com/80/v2-ed9c9c5ddeb08c63876b2360cedc1f06_720w.jpg)

但实际的情况总是复杂的多的，会出现几个分支相互交叉的情况（Criss-Cross现象）

![img](https://pic2.zhimg.com/80/v2-e9a018d10f592da80c579c6bb303f245_720w.jpg)

如上图所示，当我们在寻找最近公共祖先时，可以找到两个节点：节点2和节点3。

如果我们以节点2作为base节点，如下图：

![img](https://pic4.zhimg.com/80/v2-0dcbec417e17c7d68c2c375abad0ddeb_720w.jpg)

此时通过三向合并策略合并（base节点的内容是A，两个待合并分支节点的内容是B和C）我们是无法得出应该使用哪个节点内容的，需要自己手动解决冲突。

而如果使用节点3作为base节点，那么通过三向合并策略合并（base节点的内容是B，两个待合并分支节点的内容是B和C）可以得出应该使用C来作为最终结果：

![img](https://pic3.zhimg.com/80/v2-9789ad32dacfc506cd18752b837c290e_720w.jpg)

>  查看两个分支的最近公共祖先可以是使用命令`git merge-base --all branch_1 branch_2`
>  

作为人类，其实我们很容易看出正确的合并结果应该是C，那么git要如何保证自己能找到正确的base节点，尽可能的减少代码的合并冲突呢？

实际上git在合并时，如果查找发现满足条件的祖先节点不唯一，那么git会首先合并满足条件的祖先节点们，将合并完的结果作为一个虚拟的base节点来参与接下来的合并。

如下图：git会首先合并节点2和节点3，找到他们的公共祖先节点1，在通过三项合并策略得到一个虚拟的节点8，内容是B，再将节点8作为base节点，和节点5，节点6合并，比较完后得出最终版本的内容应该是C。

![img](https://pic2.zhimg.com/80/v2-c589dd7e0359b750de2e91fe7870af15_720w.jpg)

## Ours & Theirs参数

在合并时我们可以带上`-Xours`， `-Xtheirs`参数，表明合并遇到冲突时全部使用其中一方的更改。如下图在master分支下执行`git merge -Xours dev`，最后产生的节点内容将自动采取master分支上的内容而不需要你再手动解决冲突。

![img](https://pic3.zhimg.com/80/v2-461fce9cc2bc062e2f6538c133489e1e_720w.jpg)

`-Xtheirs`参数和`-Xours`完全相反，遇到冲突时他会自动采取dev上的内容。**注意这两个参数只有遇到冲突时才会生效，这和我们下面提到的Ours策略不一样**

## Ours

Ours 策略和上文提到的`-Xours`参数非常相像，不同的是`-Xours`参数是只有合并遇到冲突时，git会自动丢弃被合并分支的更改保留原有分支上的内容，如果没有冲突，git还是会帮我们自动合并的。

而Ours策略是无论有没有冲突，git会完全丢弃被合并分支上的内容，只保留合并分支的上的修改，只是在commit的记录上会保留另一个分支的记录。

如下图在master分支下执行`git merge -s ours dev`，最后产生的合并节点其内容和master分支上一个节点完全一样。

![img](https://pic1.zhimg.com/80/v2-86688b70a00d49e231c0c55f53a81324_720w.jpg)

这种策略的应用场景一般是为了实现某个功能，同时尝试了两种方案，最终决定选择其中一个方案，而又希望把另一个方案的commit记录合进主分支里方便日后的查看。

## 为什么没有Theirs策略

既然合并的时候即有`-Xtheirs`参数又有`-Xours`参数，所以下意识的觉得git即有 Ours 策略也会有 Theirs 策略，实际上git曾经有过这个策略后来舍弃了，因为Theirs会完全丢掉当前分支的更改，是一个十分危险的操作，如果你真的想丢弃掉自己的修改，可以使用reset命令来代替它。

## Octopus

Octopus  策略可以让我们优雅的合并多个分支。前面我们介绍的策略都是针对两个分支的，如果现在有多个分支需要合并，使用Recursive策略进行两两合并会产生大量的合并记录：每合并其中两个分支就会产生一个新的记录，过多的合并提交出现在提交历史里会成为一种“杂音“，对提交历史造成不必要的”污染“。

Octopus在合并多个分支时只会生成一个合并记录，这也是git合并多个分支的默认策略。如下图：在master分支下执行`git merge dev1 dev2`：

![img](https://pic2.zhimg.com/80/v2-e9128b3d4ceee79ea34ddf26d8fcb261_720w.jpg)

# git rebase

看完git merge 的策略后，再看看另一个合并代码时常用的命令git rebase。git rebase和merge最大的不同是它会改变提交的历史。

如下图：在dev上rebase master时，git会以master分支对应的commit节点作为起点，将dev上commit节点”平移“至master commit的后面，并且会创建全新的commit节点来替代之前commit：

![img](https://pic4.zhimg.com/80/v2-5549fe1980fc8cf9ddd67982543d5f0f_720w.jpg)

接下来我们再来看一下”平移“的过程中git需要做的事情：首先我们需要以commit5作为base节点，commit1和commit6进行合并生成新的commit3，然后再将commit3的parent指向commit6。commit2到commit4转变进行了同样的步骤。因为相比较之前的commit，新的commit的parent变了，对应的hash值自然也变了。

所以我们在rebase的时候，你当前分支有几个commit记录那么git就需要进行合并几次。如果你当前分支比较”干净“，只有一个commit记录的话，那么你rebase需要解的冲突其实和merge是一样的，区别就是rebase不会单独生成一个新的commit来记录这次合并。

关于`git pull master --rebase`和`git rebase master`的区别，git pull --rebase相当于git fetch + git rebase，正常的git pull相当于git fetch + git merge。

至于什么时候用rebase什么时候用merge，我的理解是：开发只属于自己的分支时尽量使用rebase，减少无用的commit合到主分支里，多人合作时尽量使用merge，一方面减少冲突，另一个方面也让每个人的提交有迹可循。

git rebase还有一个功能是可以合并commit记录：`git rbase -i HEAD~n`。合并分支还有一个办法是`git merge --squash`，区别是merge --squash会将你之前所有的记录压缩成一个新的commit，而rebase具体要怎么压缩可操作性比较高，这里就不多展开论述了。

### 总结

最后我们来看一个新手合并代码经常会遇到的问题：

小明在他的开发分支上完成了一个开发功能并且合上了master分支，后来发现新开发的代码有点问题，于是小明执行了revert操作：将master需要回退到没有合并时的版本，并继续在之前的开发分支上修复了问题：

![img](https://pic3.zhimg.com/80/v2-c1f253d0cf90d71c95b9831ff1475aae_720w.jpg)

 这个时候他再试图把dev分支往master上合并时，会发现B节点上新增的内容莫名其妙的就丢失了。根据git的合并策略我们就很容里理解这个问题：

在合并两个有分叉的分支（上图中的D和A^），git会默认选择Recursive策略来进行合并，对于D和A^他们的最近父节点是B，以B为base节点，对D和A^做三项合并，B中拥有“B”的内容，D中也拥有“B”的内容，A^中将“B”的内容丢弃，所以合并的结果就是将“B”的内容丢弃。

根据原理解决的方案也有很多，最简单的一种在节点D合并A^前，先revert一下生成A^^（revert的revert），再继续合并就没问题了，或者修复问题时从A^节点单独拉一个新分支修复，而不是在之前dev分支上继续开发。

这个时候他再试图把dev分支往master上合并时，会发现B节点上新增的内容莫名其妙的就丢失了。根  复，而不是在之前dev分支上继续开。  