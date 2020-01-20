# 易观方舟Argo

![](imgs/pre-defined_dashboard.png)

我们最初推出 Argo 是希望有一种全新的产品体验——让大家在使用到商业级产品能力的时候尽量不用去考虑成本。能够免费、高效地搭建智能用户运营平台，分析用户行为特点、制定用户分群并完成用户触达。虽然现在难以做到完全开源，但我们也会努力将商业级的产品通过社区带给更多人。

我们觉得数据驱动是一种基础能力，大家都应该更自由地使用。数据驱动在未来会有更广阔的应用场景，而目前在企业中的使用率还达不到普及。从长远来看，降低使用门槛的做法对提升普及率有积极意义。也希望能在很快的未来，我们会和更多的开发者和社区爱好者一起，带动更多的朋友一起来做这件有意义的事情。

Argo为私有化部署产品，需要工程师参与。所以请业务同学将地址同步给工程师，助你快速上手！

**最新版下载地址**：

[4.5.1](http://arkinstall.analysys.cn/) 
 * [升级方法](https://github.com/analysys/argo-installer/blob/master/upgrade.md)

**What's New in 4.5**

新增

 - 智能监控，针对上报元事件的触发次数和触发人数的异动进行智能监控
 - 事件分析中增加触发时间作为默认筛选条件，可以查看关注的小时区间数据 
 - 新增企业设置功能，可以自由配置收数地址
 
 改进
 
 - 看板增加编辑说明功能，方便所有人员查看
 - 分析模块中各下载表格优化，增加显示结果类数据
 - 添加企业成员优化，通过禁用和启用管理成员状态
 - 分析模型中公共过滤条件和细分属性中的公共属性在切换指标时保留，方便任意切换指标进行查询
 - 支持方舟平台手动停止数据流
 - SDK预置小程序分享事件及属性，并默认放到埋点方案中
 - 添加了结果为无值的说明
 - 分析模块中维度选择着陆页时，可以选择按标题或按URL展示，细化查询维度
 - 新增分群管理API
 
更多更新内容在[这里](https://docs.analysys.cn/ark/release-notes)查看

## 为什么你需要 Argo ？

目前能做用户行为数据收集和分析的产品很多，有的例如 Google Analytics、百度统计和友盟能基础数据统计和分析（当然 GA 能做更多事情只不过也需要更多的学习成本）。使用这类工具可以满足你了解产品数据表现的基本需求，可当你需要数据做更多事情时他们就无法满足了。常见的场景有：

- 创建自定义的指标
- 将同一产品不同客户端的用户数据汇总分析
- 将数据明细导出到别的平台使用
- 基于实时数据驱动其它过程
- 管理用户分群并导出到其它平台使用
- 数据私有化而不是存在它们那里
- 更多他们不没有的分析模型

以上场景虽然国内也有其它厂商能满足，但收费不菲且开放性不够。

来自用户的数据是未来企业日常工作中会用到的基本元素，我们希望能帮助团队做到“**早收集，自己存，存明细，分析快，导出易**”。我们会致力于提供更开放、更低门槛、更易用的数据工具，希望能通过我们的努力让这种数据的使用能力成为团队初创伊始就具备的能力。如果你现在还在考虑用户行为相关的数据产品，可以先花几分钟部署 Argo 试一下。相信会给你的决策提供帮助。

## 他们已经在用

![](imgs/customers.png)

## Argo 提供的功能

### 分析类

* [渠道分析](https://docs.analysys.cn/ark/features/analytics/channel)
* [事件分析](https://docs.analysys.cn/ark/features/analytics/event)
* [会话分析](https://docs.analysys.cn/ark/features/analytics/session)
* [漏斗转化](https://docs.analysys.cn/ark/features/analytics/funnel)
* [留存分析](https://docs.analysys.cn/ark/features/analytics/retention)
* [智能路径](https://docs.analysys.cn/ark/features/analytics/pathfinder)
* [热图分析](https://docs.analysys.cn/ark/features/analytics/heatmap)
* [分布分析](https://docs.analysys.cn/ark/features/analytics/fen-bu-fen-xi)
* [间隔分析](https://docs.analysys.cn/ark/features/analytics/jian-ge-fen-xi)
* [属性分析](https://docs.analysys.cn/ark/features/analytics/shu-xing-fen-xi)
* [SQL查询](https://docs.analysys.cn/ark/features/analytics/sql)

### 用户类

* [用户群管理](https://docs.analysys.cn/ark/features/segmentation/profile)
* [用户行为轨迹](https://docs.analysys.cn/ark/features/segmentation/user-sequence)

### 动作类

* [发送系统通知](https://docs.analysys.cn/ark/features/operation/pushmessage)（极光、个推、百度、小米、华为）
* [发送电子邮件](https://docs.analysys.cn/ark/features/operation/email)（SendCloud）
* [发送短息](https://docs.analysys.cn/ark/features/operation/sms)（腾讯云短信、云集、领驭、乐信通）
* [广告活动管理](https://docs.analysys.cn/ark/features/operation/utm)（UTM、APP扫码）

# 安装说明

- [通过脚本安装](INSTALL_SCRIPT.md)（推荐！）
- [通过 Docker 安装](INSTALL_DOCKER.md) **NOTE**: Docker 版仅用于体验，不支持更新，不应用于生产环境；cpu需要支持SSE4.2指令集

#### License 

> D4833F05A784C925D17684A8A2114EFB8E1A431CCE5929B85F097A783BED24F247D07B1DD63414348BE4FB516DB18FD65A4167FCAE311022E71A223AE672ABEF10F721D949A6FECEA7272B1C3D0900FF33366F7AA30E5546818FD2521530F266287C891F802C5657C8742787919ACE2F7376D5D781C57509E5CBE4D12CCF254D46F32A84D0E887DA4DA5BF91F430F251

License 有效期为1个自然年，每年12月更新次年 License。用企业管理员账号登录平台，在企业概览页面的右上角点击“续期/扩容”，填入上面的 License 即可完成续期。

## 常见问题

部署中遇到问题可以看这里：

- [部署问题](https://github.com/analysys/argo-installer/issues?utf8=✓&q=label%3Adocs+)

完成了后续安装步骤就正式进入了易观方舟Argo的探索之旅，下面是一些快速开始的文档：

- [接入前准备](https://ark.analysys.cn/docs/integration-prepare.html)
- [SDK指南](https://ark.analysys.cn/docs/sdk.html)
- [功能介绍](https://ark.analysys.cn/docs/function.html)
- [其它问题](https://ark.analysys.cn/docs/faq.html)

## 提交反馈

* 提交[新功能需求](https://github.com/analysys/argo-installer/issues/new)
* 为[热门需求](https://github.com/analysys/argo-installer/issues?q=is%3Aopen+is%3Aissue+label%3A%22feature+request%22+sort%3Areactions-%2B1-desc)投票

# 热切地寻找志同道合的小伙伴

正如你所见，我们希望通过易观 Argo 能让更多的早期团队也具备商业级用户行为驱动的能力。我们通过社区的方式来运营此版本，就是希望 Argo 能是社会的，而不是私有的。如果你认同我们的做法，并且也想贡献一份力量，无论你是否在易观，都有可以成为 Argo 社区的一员。

我们现在急需以下小伙伴：
1. 社群运营（社区贡献者或易观工作机会）
2. 大前端架构师（易观工作机会）

我们长期欢迎下面的小伙伴勾搭：
1. 产品经理（易观工作机会）
2. 产品设计师（易观工作机会）

有想法的小伙伴可以给 zhangxiaoliang@analysys.com.cn 发送邮件，或者通过任何方式联系到 Argo 同学，并注明来意。

期待与你共建用户行为分析以及基于用户行为驱动更多业务的社区！

# 社区支持

希望我们的努力可以解放更多人的生产力，祝你使用顺利！

官方论坛：
https://geek.analysys.cn

微信群交流：
如有问题，请优先根据文档和论坛问答自研，若想进群交流请发送邮件至zhunan@analysys.com.cn ，邮件标题中注明你的微信号和进群目的。

