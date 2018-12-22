---
title: 如何书写Git Commit Message
date: 2018/12/22 21:00:00
tags:
  - Git
  - commit message
categories: Git
---

> 原作者已授权，原文链接：[https://chris.beams.io/posts/git-commit/](https://chris.beams.io/posts/git-commit/)
> 以下第一人称皆为原作者

[![image | left | 439x250](https://imgs.xkcd.com/comics/git_commit_2x.png "")](https://xkcd.com/1296/)

## 前言：为什么好的commit message 很重要
如果你随机浏览Git仓库中的日志，你可能会发现它的提交消息或多或少有些混乱。例如，看看我早期向[Spring commit](https://github.com/spring-projects/spring-framework/commits/e5f4b49?author=cbeams)的那些宝贵经验：
<!-- more -->

```plain
$ git log --oneline -5 --author cbeams --before "Fri Mar 26 2009"

e5f4b49 Re-adding ConfigurationPostProcessorTests after its brief removal in r814. @Ignore-ing the testCglibClassesAreLoadedJustInTimeForEnhancement() method as it turns out this was one of the culprits in the recent build breakage. The classloader hacking causes subtle downstream effects, breaking unrelated tests. The test method is still useful, but should only be run on a manual basis to ensure CGLIB is not prematurely classloaded, and should not be run as part of the automated build.
2db0f12 fixed two build-breaking issues: + reverted ClassMetadataReadingVisitor to revision 794 + eliminated ConfigurationPostProcessorTests until further investigation determines why it causes downstream tests to fail (such as the seemingly unrelated ClassPathXmlApplicationContextTests)
147709f Tweaks to package-info.java files
22b25e0 Consolidated Util and MutableAnnotationUtils classes into existing AsmUtils
7f96f57 polishing
```

再来看看最近向这个仓库的[提交](https://github.com/spring-projects/spring-framework/commits/5ba3db?author=philwebb)：

```plain
$ git log --oneline -5 --author pwebb --before "Sat Aug 30 2014"

5ba3db6 Fix failing CompositePropertySourceTests
84564a0 Rework @PropertySource early parsing logic
e142fd1 Add tests for ImportSelector meta-data
887815f Update docbook dependency and generate epub
ac8326d Polish mockito usage
```

你更愿意读哪一个？

* 前者在长度和形式上有所不同；而后者简洁而一致
* 前者是默认情况；后者绝不是偶然发生的

虽然许多Git 仓库的日志与前者类似，但也有例外。[Linux内核](https://github.com/torvalds/linux/commits/master)和[Git本身](https://github.com/torvalds/linux/commits/master)就是很好的例子。查看[Spring Boot](https://github.com/spring-projects/spring-boot/commits/master)或[Tim Pope](https://github.com/tpope/vim-pathogen/commits/master)管理的任何仓库。

这些仓库的贡献者知道，精心设计的Git commit 是向其他开发人员(和未来的自己)传递更改上下文的最佳方式。差异会告诉你修改了什么，但只有commit才能正确地告诉你原因。[Peter Hutterer说得很好](http://who-t.blogspot.co.at/2009/12/on-commit-messages.html)：
> 重建一段代码的上下文是一种浪费。我们不能完全避免它，所以我们应该努力尽可能地减少它。Commit message可以做到这一点，一个commit message可以显示开发人员是否是良好的合作者。

如果你没有过多地考虑如何制作一个优秀的Git commit message，那么你可能没有花太多时间使用 __git log__ 和相关工具。这就是一个恶性循环：由于commit 历史是无结构化和不一致的，不会有人花很多时间使用或处理它。然后由于它没有得到使用或处理，它仍然是无结构的和不一致的。

但一个精心照料的日志是一个好看且有用的东西。这样__git blame__, __revert__, __rebase__, __log__, __shortlog__ 以及其他的子命令也会焕发生命。审查其他人的commits和pr成为一件值得做的事情，并且突然之间可以独立完成。理解某些事情为什么发生在几个月或几年前不仅是可能的，而且是有效的。

项目的长期成功依赖于它的可维护性，而维护者几乎没有比项目日志更强大的工具了。花时间去学习如何正确地照顾是值得的。一开始可能是麻烦的，但很快就会变成习惯，最终成为所有人的骄傲和生产力的源泉。

在本文中，我只讨论保持健康的commit 历史最基本元素：如何编写单个 commit message。还有其他一些重要的实践，如commit squashing，我在这里不讨论。也许我会在后续的文章中这样做。

大多数编程语言都有了约定俗成的风格，如命名、格式化等等。当然，这些约定也有不同之处，但是大多数开发人员都同意，选择一种并坚持使用它，要比每个人都做自己的事情所带来的混乱要好得多。

团队对commit log的处理方法应该是一致的。为了创建一个有用的修订历史，团队首先应该同意一个提交消息约定，该约定至少定义了以下三件事：
* __Style__：标记语法，换行符，语法，大写，标点符号。把这些东西拼出来，移除猜测，让它尽可能的简单。最终的结果将是一个非常一致的日志，这不仅是一种阅读的乐趣，而且实际上是经常被阅读
* __Content__：提交消息的主体(如果有的话)应该包含什么样的信息？它不应该包含什么？
* __Metadata__：如何引用问题跟踪 id、pr number等?

幸运的是，对于如何生成惯用的Git commit message已经有了完善的约定。实际上，它们中的许多都是以某些Git命令的功能进行假设的。你不需要重新发明什么。只要遵循下面的七条规则，你就能像专业人士一样。

## Git commit message 的7条规则
> 注意：这些以前都说话

* 用空行把主题和主体分开
* 主题行不要超过50个字符
* 主题行大写
* 主题行不要以句号结束
* 在主题行使用祈使句
* 用72个字符包裹主体
* 用主体来解释什么、为什么以及怎样做

比如：

```plain
Summarize changes in around 50 characters or less

More detailed explanatory text, if necessary. Wrap it to about 72
characters or so. In some contexts, the first line is treated as the
subject of the commit and the rest of the text as the body. The
blank line separating the summary from the body is critical (unless
you omit the body entirely); various tools like `log`, `shortlog`
and `rebase` can get confused if you run the two together.

Explain the problem that this commit is solving. Focus on why you
are making this change as opposed to how (the code explains that).
Are there side effects or other unintuitive consequences of this
change? Here's the place to explain them.

Further paragraphs come after blank lines.

 - Bullet points are okay, too

 - Typically a hyphen or asterisk is used for the bullet, preceded
   by a single space, with blank lines in between, but conventions
   vary here

If you use an issue tracker, put references to them at the bottom,
like this:

Resolves: #123
See also: #456, #789
```

### 用空行把主题和主体分开
源自[git commit](https://mirrors.edge.kernel.org/pub/software/scm/git/docs/git-commit.html#_discussion) 手册：
> 虽然不是必需的，但是最好在commit message的开头使用一个简短的行(少于50个字符)来总结更改，然后使用空行，然后使用更全面的描述。commit message中第一个空行之前的文本被视为提交标题，该标题在整个Git中使用。例如，Git-format-patch(1)将提交转换为电子邮件，它使用主题行作为标题，commit剩下的正文中作为邮件正文。

首先，并非每次提交都需要主题和主体。有时候一行就可以了，特别是当更改非常简单，不需要进一步的上下文时。例如：

```plain
Fix typo in introduction to user guide
```

无需多言；如果读者想知道typo是什么，可以简单地看看更改本身，比如使用 git show、 git diff 或 git log -p。

如果在命令行提交类似的内容，使用__ git commit -m__ 即可：

```plain
$ git commit -m"Fix typo in introduction to user guide"
```

但是，当提交需要一些解释和上下文时，需要编写一个主体。例如：

```plain
Derezz the master control program

MCP turned out to be evil and had become intent on world domination.
This commit throws Tron's disc into MCP (causing its deresolution)
and turns it back into a chess game.
```

使用-m选项编写带有主体的commit message并不容易。你最好用一个合适的文本编辑器来写这条消息。如果你还没有在命令行中设置用于Git的编辑器，请阅读[Pro Git](https://git-scm.com/book/en/v2/Customizing-Git-Git-Configuration)的这部分。

无论如何，在浏览日志时，主题与主体的分离是值得的。下面是完整的日志条目：

```plain
$ git log
commit 42e769bdf4894310333942ffc5a15151222a87be
Author: Kevin Flynn <kevin@flynnsarcade.com>
Date:   Fri Jan 01 00:00:00 1982 -0200

 Derezz the master control program

 MCP turned out to be evil and had become intent on world domination.
 This commit throws Tron's disc into MCP (causing its deresolution)
 and turns it back into a chess game.
```

现在使用__git log --oneline__，只会打印出主题行：

```plain
$ git log --oneline
42e769 Derezz the master control program
```

或者使用__git shortlog__，哪些组由用户提交，为了简洁起见再次只显示主题行：

```plain
$ git shortlog
Kevin Flynn (1):
      Derezz the master control program

Alan Bradley (1):
      Introduce security program "Tron"

Ed Dillinger (3):
      Rename chess program to "MCP"
      Modify chess program
      Upgrade chess program

Walter Gibbs (1):
      Introduce protoype chess program
```

在Git中还有许多其他的语境，主题行和主体的区别是存在的——但是如果没有中间的空白行，它们都不能正常工作。

### 主题行不要超过50个字符
50个字符不是一个硬性限制，只是一个经验法则。保持主题行在这个长度可以确保它们是可读的，并迫使作者思考一下如何用最简洁的方式来解释正在发生的事情。

> 提示：如果你很难总结，你可能一次提交了太多的更改。争取[原子提交](https://www.freshconsulting.com/atomic-commits/)(一个提交一个主题)

GitHub的UI完全了解这些约定。如果你超过了50个字符的限制，它会警告你：



![gh1 | left](https://i.imgur.com/zyBU2l6.png "")


并以省略号截断任何超过72个字符的主题行：



![gh2 | left](https://i.imgur.com/27n9O8y.png "")


所以争取50个字符，但是72个字符是硬界限

### 主题行大写
这听起来很简单。所有的主题行以大写字母开始。

比如：
* Accelerate to 88 miles per hour

而不是：
* accelerate to 88 miles per hour

### 主题行不要以句号结束
在标题行中没有必要使用结尾标点。此外，当你试图将它们控制在50字符以内时，空间是宝贵的

比如：
* Open the pod bay doors

而不是：
* Open the pod bay doors.

### 在主题行使用祈使句
祈使句的意思是“说或写就像下达命令或指示一样”，比如：
* Clean your room
* Close the door
* Take out the trash
你现在读到的七条规则都是命令式的。

这个命令听起来有点粗鲁；这就是为什么我们不经常使用它。但是它非常适合Git commit主题行。原因之一是Git本身在为创建提交时使用了命令式。

比如，当使用 __git merge__ 时默认的message是：

```plain
Merge branch 'myfeature'
```

当使用 __git revert__ 时：

```plain
Revert "Add the thing with the stuff"

This reverts commit cc87791524aedd593cff5a74532befe7ab69ce9d.
```

或者在GitHub pull request上点击“Merge”按钮：
```plain
Merge pull request #123 from someuser/somebranch
```

因此，当你使用祈使句编写commit message时，你正在遵循Git的内置约定。比如：
* Refactor subsystem X for readability
* Update getting started documentation
* Remove deprecated methods
* Release version 1.0.0

一开始写可能会有点笨拙。我们更习惯用陈述性语气说话，也就是报道事实。这就是为什么commit message通常以这样的方式结束：
* Fixed bug with Y
* Changing behavior of X

有时commit message 被写成对其内容的描述：
* More fixes for broken stuff
* Sweet new API methods

为了消除任何困惑，有个简单的规则每次都能正确处理——一个格式正确的Git commit主题行应该始终能够完成以下句子（这里是指 commit message 可以替换下面加粗的部分）：
* If applied, this commit will ++__your subject line here__++

比如：
```plain
If applied, this commit will refactor subsystem X for readability
If applied, this commit will update getting started documentation
If applied, this commit will remove deprecated methods
If applied, this commit will release version 1.0.0
If applied, this commit will merge pull request #123 from user/branch
```

注意，这对其他非祈使句格式不起作用：
```plain
If applied, this commit will fixed bug with Y
If applied, this commit will changing behavior of X
If applied, this commit will more fixes for broken stuff
If applied, this commit will sweet new API methods
```

> 记住：祈使句的在主题行中是重要的。在写主题中可以放松这个限制


### 用72个字符包裹主体
Git从不自动换行。在编写commit message主体时，必须注意它的右边界，并手动换行。

建议将其设置为72个字符，这样Git就有足够的空间缩进文本，同时仍然将所有内容保持在80个字符以下。

一个好的文本编辑器可以在这里提供帮助。例如，在编写Git提交时，很容易配置Vim将文本包装为72个字符。然而，传统上，IDEs在为提交消息中的文本包装提供智能支持方面一直很糟糕（尽管在最近的版本中，IntelliJ IDEA终于在这方面做得更好了）

### 用主体来解释什么、为什么以及怎样做
这是一个[很好的例子](https://github.com/bitcoin/bitcoin/commit/eb0b56b19017ab5c16c745e6da39c53126924ed6)，说明了修改了什么以及为什么：

```plain
commit eb0b56b19017ab5c16c745e6da39c53126924ed6
Author: Pieter Wuille <pieter.wuille@gmail.com>
Date:   Fri Aug 1 22:57:55 2014 +0200

   Simplify serialize.h's exception handling

   Remove the 'state' and 'exceptmask' from serialize.h's stream
   implementations, as well as related methods.

   As exceptmask always included 'failbit', and setstate was always
   called with bits = failbit, all it did was immediately raise an
   exception. Get rid of those variables, and replace the setstate
   with direct exception throwing (which also removes some dead
   code).

   As a result, good() is never reached after a failure (there are
   only 2 calls, one of which is in tests), and can just be replaced
   by !eof().

   fail(), clear(n) and exceptions() are just never called. Delete
   them.
```

看看[完整的差异](https://github.com/bitcoin/bitcoin/commit/eb0b56b19017ab5c16c745e6da39c53126924ed6)，想一想作者花了多少时间来提供此时此地的上下文，从而为同行和未来的提交者节省了多少时间。如果他不这样做，它可能会永远消失。

在大多数情况下，你可以省略更改的细节。在这方面，因为代码通常是自解释的（如果代码非常复杂，需要用散文来解释，那么源注释就是用于此）。只需要清楚地说明为什么你一开始就做出了改变——改变之前的工作方式（以及它有什么问题），它们现在的工作方式，以及为什么你决定用这种方式解决。

未来的维护者可能会感谢你！

## Tips
### 学会爱上命令行，将IDE丢掉
原因和Git子命令一样多，使用命令行是明智的。Git非常强大；IDE也是，但是它们的方式不同。我每天都使用IDE (IntelliJ IDEA)，也广泛地使用过其他IDE (Eclipse)，但我从没有见过集成Git的IDE能与命令行的易用性和功能相匹配(一旦你了解它)

某些与git相关的IDE功能是非常宝贵的，比如删除文件时调用git rm，重命名文件时使用git执行正确的操作。当你开始尝试通过IDE提交、合并、重新建立基础或进行复杂的历史分析时，一切都崩溃了。

在使用Git的全部功能时，始终是命令行。

请记住无论使用Bash、Zsh还是Powershell，都有[tab](https://git-scm.com/book/en/v2/Appendix-A%3A-Git-in-Other-Environments-Git-in-Powershell)大大减轻记住子命令和开关的痛苦。

### 阅读Pro Git
[Pro Git](https://git-scm.com/book/en/v2)的在线图书是免费的，非常棒。

## 资料
* [https://chris.beams.io/posts/git-commit/](https://chris.beams.io/posts/git-commit/)
* [https://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html](https://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html)
* [https://www.git-scm.com/book/en/v2/Distributed-Git-Contributing-to-a-Project#\_commit\_guidelines](https://www.git-scm.com/book/en/v2/Distributed-Git-Contributing-to-a-Project#_commit_guidelines)
* [https://github.com/torvalds/subsurface-for-dirk/blob/master/README.md#contributing](https://github.com/torvalds/subsurface-for-dirk/blob/master/README.md#contributing)
* [http://who-t.blogspot.co.at/2009/12/on-commit-messages.html](http://who-t.blogspot.co.at/2009/12/on-commit-messages.html)
* [https://github.com/erlang/otp/wiki/writing-good-commit-messages](https://github.com/erlang/otp/wiki/writing-good-commit-messages)
* [https://github.com/spring-projects/spring-framework/blob/30bce7/CONTRIBUTING.md#format-commit-messages](https://github.com/spring-projects/spring-framework/blob/30bce7/CONTRIBUTING.md#format-commit-messages)

