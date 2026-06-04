---
title: "我用 109GB 数据复盘了 Polymarket：为什么 99.99% 的人不适合做价差套利"
date: 2026-02-20T04:13:36+0000
tags:
  - Web3.0
source: "子琦的长期实验室"
url: https://mp.weixin.qq.com/s/6A9n7NFMutCnmEj2XQNNxg
description: "关于 Polymarket 价差套利的残酷真相 —— 你看得见，但你吃不到"
cover: "https://mmbiz.qpic.cn/sz_mmbiz_jpg/dNmAgvM5D09MSm6yVdReIjJhBPRGY23T2E9bLZJWQBurh3cwM9VGhP7lIoFxJib2cyLbsSd8Ud9iaHNrMusFmpE7v5SVbGk0E9Wtsvwkib11Bg/0?wx_fmt=jpeg"
---

# 我用 109GB 数据复盘了 Polymarket：为什么 99.99% 的人不适合做价差套利


# 关于 Polymarket 价差套利的残酷真相 —— 你看得见，但你吃不到

# 前言｜我为什么写这篇

结论先说：PM 的 Yes+No 价差套利，99.99% 的人不适合。

我用最近 3 个月 109GB 的交易数据做研究，自己从 0 搞定了：研究 → 开发 → 监控 → 实盘测试。

![](https://mmbiz.qpic.cn/mmbiz_jpg/dNmAgvM5D0ib6JeXjOiapLibVibAT8SWCyYIu5Ss2NmlLdTeDSl64xedqf8oQ60Z4hnzl8I9bNTturvUyUkeeClb3w1B3ZKP328puU9UpSQ6lVs/640?wx_fmt=jpeg&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=0)

监控端能稳定看到 Yes卖一+No卖一<1 / Yes买一+No买一>1 的机会，但实盘下来几乎 每一笔都亏。

原因很简单： 从「监控到机会」到「下单成交」我大概需要 2 秒+，而机会消失更快。 你逻辑没错，但你永远成交在机会已经没了的那一刻。

这篇文章不是成功案例复盘，而是我探索新领域的完整方法论： 我怎么找数据源、怎么筛盈利地址、怎么分类、为什么选价差套利、怎么做监控、怎么测延迟，以及为什么它更像 MEV/高频的基础设施战争。

# 一｜为什么要研究 Polymarket 的套利？

起因很简单：

1）我本来就在推进 PM 空投业务，手里有大量地址与交易数据 2）网上很多人分享“在 PM 赚钱”，我想自己验证它到底行不行 3）如果熊市能有一个现金流小业务，哪怕不大，也会更有安全感

# 二｜我是怎么开始研究的？

先说前提：我不懂程序。我用的开发工具有 Claude Code、Codex、Gemini，但基本只用到了 Claude。

我的思路非常朴素：

先找到赚钱的地址 → 再看看它们怎么赚 → 尝试抄作业（复现）

1）找到可以研究的数据源

为了研究 PM 的交易行为，我先搞定了一个能持续拉取 PM 交易数据的程序，可以不断更新新的成交数据。

缺点是非常占内存： 最近 3 个月的数据就大概 109GB 左右。我以这份数据作为研究起点。

2）找出盈利地址并分类

有了数据后，我开始筛选“盈利地址”。

盈利标准： 以最近 3 个月为窗口，按账户最后的「盈利 - 亏损」的最终结果为准。

我把筛出的地址数据交给 Claude 做辅助分析，最后大概得出几类：

套利

做市

交易

其他（可能有信息优势/内幕等）

但很快我发现一个问题：很难做严格分类。因为现实里策略会混在一起—— 套利者做的事看起来像做市；内幕的行为又像套利；同一个地址也可能同时跑多个策略。

所以最后我决定聚焦一个最“清晰、直观、看起来最简单”的方向： Yes + No 的价差套利。

# 三｜为什么选择价差套利？

原因有三点：

1）很多人说 PM 经常出现定价错误 2）这个逻辑最容易理解，适合先从简单的入手 3）我想先验证：这个市场到底有没有“低门槛现金流”的可能性

什么是价差套利？

正常情况下应该是：

买入成本最低： Yes 卖一 + No 卖一 = 1.0010（略大于 1）

卖出收入最高： Yes 买一 + No 买一 = 0.9990（略小于 1）

套利逻辑是：

如果 Yes 卖一 + No 卖一 < 1 那么理论利润： 利润 = 1 -（Yes + No）

如果 Yes 买一 + No 买一 > 1 那么可以先合并再卖出： 利润 =（Yes + No）- 1

锁定这个结构后，我开始开发“监控套利机会”的程序。

并且我也做过市场选择： 通过盈利地址的分布去反推——到底哪些市场更值得监控。

![](https://mmbiz.qpic.cn/mmbiz_png/dNmAgvM5D08oTlygL2tGqPOqw9juLTgMSJAs98kJKJia4soV9dy3QFEf6DaSmANicxBRy5d8DugA1eNKPMPrWeUHSicXD1gC6gVaLgFHibJ8t2c/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=1)

# 四｜先监控套利机会：看清“机会长什么样”

我自己也不太会配置什么机器人，但 AI 真的把门槛拉低了。 不会的就问，做不出来就拆任务继续问。

我认为第一关不是下单，而是：

先能稳定“看到”套利机会。

因为能看到机会，才算验证逻辑存在。 这一步我也踩坑很多——主要原因是跟 AI 沟通时，如果你描述不清楚，它并不一定能理解你到底要什么。

但最终，我确实监控到了 Yes + No 偏离 1 的瞬间：

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/dNmAgvM5D08Nbzx2vjkDBib6EYZ8uwGFaUKRlJKdEJIbCc2rVvPkKicjr743JW0FfHiaRzaib5ic2JWkDqrmUKeiacm4n9TsFtWlGriaGX3dcYX1OA/640?wx_fmt=jpeg&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=2)

这也验证了：网上说的这种结构确实存在。

# 五｜为什么价差套利不适合 99.99% 的人？

问题出在：从监控到下单的速度。

我发现： 虽然监控时看到的是 Yes + No < 1， 但当我真正下单成交时，买入的 Yes + No 往往已经 > 1 了。

也就是说： 机会从你看到，到你成交，就已经消失。

我做过一些优化：

从市价改单 → 限价单

单边成交 → 止损/撤单策略

尝试减少多余请求（比如余额查询）

后来我专门测量了两段时间：

1）从「监控到机会」→「我完成下单」的时间：约 2 秒多2）从「监控到机会」→「机会消失」的时间：更短

![](https://mmbiz.qpic.cn/mmbiz_png/dNmAgvM5D0icVJ8f1icsAKEFePfPnIl9DdWvGG3Bszh2iaSJ8eBEv11pnsKYibwvxUzoXd7GBLQCz0tj4ibhBH96SfNmFmQfKB61poEyPXGSjoxE/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=3)

结论是：即使你逻辑正确，也抓不住这种机会。

为什么抓不住？我总结是三点：

1）把延迟提上去的成本太高

你可能需要更好的服务器、更好的网络、更优化的架构、甚至更底层的工程能力。 每提升一点速度，都要花成本。

2）收益无法覆盖成本

单笔利润很薄，理论上靠“积少成多”能起来。 但问题是：当你为了那一点速度投入大量基础设施，成本很难被覆盖。

3）方向可能本身就不对

我后来越来越觉得：这种机会更像 MEV / 高频机器人战场。 它不是“普通人做个小现金流”的方向，而是“卷基础建设”的方向。

所以从：成本、技术门槛、收益三个角度看， 这条路大概率不适合大多数人。

# 六｜总结

虽然这是一次「失败」的探索，但我并不觉得亏。

我最大的收获有三点：

1）AI 真的解放生产力不会代码的人，也能靠 vibe coding 把一整套东西跑出来。 以前要么自己学很久，要么求助朋友；现在一个想法 + AI 能诞生很多可能性。

2）开发过程让我学到很多关键常识 比如：数据源的重要性、地址行为难分类、不同 API 的监控方式、下单与风控的细节等等。

3）分享不是为了装成功，而是呈现思考过程 我相信我的思考不一定最优，所以也希望抛砖引玉： 如果你有更好的视角、更成熟的做法，欢迎交流。说不定能认识一些新朋友。

最后祝大家新年快乐，马年大吉，马上有钱。🐎🧧