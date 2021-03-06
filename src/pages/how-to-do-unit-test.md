---
title: JavaScript怎么做单元测试
date: '2019-09-24'
spoiler: 单元测试，Jest
---

### 测试分类

首先对于`JavaScript`来讲，什么是测试（我们这里的说的都是自动化测试）？测试其实分为三类：

> - 单元测试(Unit testing) , 关注应用中每个零部件(function or procedure)的正常运转，防止后续修改影响之前的组件。
> - 功能测试(Functional testing), 确保其整体表现符合预期，关注能否让用户正常使用。
> - 整合测试(Integration testing), 确保单独运行正常的零部件整合到一起之后依然能正常运行。

其中整合测试很大程度上和单元测试是类似的，他们用的测试工具是一样的，但是他们有个很大的不同：单测的每个组件是相互独立的，而整合测试则恰恰相反，比如，单测里面访问 DB 的代码，不会与真实数据库对话。

使用整合测试的场景一般出现在单测无法满足需求的情况下，有时候你必须验证两个独立的系统之间的交互通信是否正常，比如你的应用和数据库之间。那么就需要用到整合测试。

因为涉及多个独立系统或者组件的之间交互，所以整合测试往往要比单测要慢，而且需要一些设定和配置，比如设置一个用来测试的 DB 环境，这使得它的开发和维护成本也变高了。所以我们应该把焦点放在单测上面。除非你确实需要它。

至于功能测试，他还有其他名字，就是 e2e 测试，浏览器测试。这个之前在架构组提供的移动端脚手架集成过（cypress，当下很流行的端对端测试框架，文档超级友好），不过因为咱们使用率较低，而且 cypress 体积比较大，影响依赖安装速度。所以被干掉了。功能测试针对的是应用的各个完整功能模块，通过一些工具，模拟浏览器事件也就是用户的操作行为，从而做到 web 自动化测试。我们可能用单测去测试一个独立的函数方法，或者用整合测试去测两个系统是否交互正常。但是功能测试它完全是另一个层面的东西。我们可以为一个应用去写上百个单测，但是通常只需要写少数的功能测试。主要原因是功能测试有较高的复杂度，而且运行很慢，因为你需要去模拟真实用户的各种行为，包括页面加载时间也是一个测试点。功能测试实际上和我们测试同学的工作是重合的

功能测试的应用场景是，如果你有比较多重复的手动操作，比如登陆注册这种功能。但是要特别注意注意测试粒度不宜过大，那将会带来沉重的维护负担。
如果大家对功能测试有兴趣的话，后续咱们可以单独开个专题讲这个。今天我们主要讲单测。

<table>
  <tr><th>测试类型</th><th>工具</th></tr>
  <tr>
    <td>Unit Testing</td>
    <td>Mocha, Jasmine, Jest, Cucumber, Tape
    </td>
  </tr>
  <tr><td>Unit Integration</td><td>Mocha, Jasmine, Jest, Cucumber, Tape</td></tr>
  <tr><td>Functional Testing</td><td>Selenium, Cypress, Nightwatch, Testcafe</td></tr>
</table>

---

### 为什么要做单测？（或者说为啥要做自动化测试）

首先，自动化测试是持续集成的必要基石，是一种能让你用更短的时间将更改发布到生产的开发方法。它能在用户接触到应用之前让你发现更多的错误，从而让应用更加的稳定。这使得你开发起来更加安心，不用害怕使用中出现一些未知的错误。

总的来讲，写测试用例可能会占用一些你的开发时间，但是缺少它，你将会失去更多，例如：

> - bugs 会影响用户体验，进而影响你的产品销售额和使用指标，更严重的可能会永久失去用户。
> - 复现一个 bug 可能会花费 QA 或者开发者 20 分钟时间。
> - 开发人员断点调试定位问题代码，每次断点分析又得花 20 分钟，这还没算上 bug 调试时间
> - 容易出现正常功能开发之外的 bug，比如有不熟悉代码的其他同事修改了代码，但是遗留的另一个 bug
> -
