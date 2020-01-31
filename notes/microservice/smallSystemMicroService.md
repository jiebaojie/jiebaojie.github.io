---
layout: post
notes: true
subtitle: "【技术博客】小型系统如何“微服务”开发"
comments: false
author: "wc的一些事一些情"
date: 2018-10-20 00:00:00

---


原文：[https://www.cnblogs.com/wcd144140/p/9782823.html](https://www.cnblogs.com/wcd144140/p/9782823.html)

*   目录
{:toc }

# “微”只是一种正常思维逻辑

“微服务”的本质就是一种正常的思维，时一种基本解决问题的思路。

# “微服务”模式小案例

 在工作过程中，大项目毕竟少数，小项目才是考验技术人“接地气”的表现所在。有一天，我接到了一个小规模的“话费充值系统”需求，没有太多复杂功能和逻辑的描述，就是一个能让用户在上面自助充值的系统。剩下的理解，靠的就是自身工作经验的功力了。面对这样的需求，我首先想到的就是这个关键业务流程：

![](/img/notes/microservice/smallSystemMicroService/charge_flow.png)

这个流程说简单可以简单，说复杂可以“媲美”电商系统，例如“充值金额”相当于商品，“充值”相当于购物，“订单”跑不掉，“充值话费”类比物流。各种电商该有的“边界问题”几乎都要考虑，规模虽小，但五脏都得要有。至于“五脏”有哪些，这得根据业务的边界范围去划分“业务领域”了，先来根据自己的经验尝试一把：

![](/img/notes/microservice/smallSystemMicroService/charge_domain.png)

这种业务划分方式多少跟电商系统有点类似，直接呈现的是业务模型。根据“微服务”思维，每个领域都是一个独立的服务个体单元，每个服务“对象”又有自己的“属性”和“行为”：

![](/img/notes/microservice/smallSystemMicroService/charge_service.png)

每个服务的属性有“服务标识”、“服务名称”等，当然服务有自身的各种行为（更多以API体现），各种系统外部动作都是通过服务之间的“合作”来完成：

![](/img/notes/microservice/smallSystemMicroService/business_flow.png)

流程：

*	**用户查看充值金额：**接查询商品服务（“充值10元到账12元”，“充值100元到账106元”，......）；
*	**用户发起话费充值：**订单服务接收请求→订单服务查询商品信息（商品ID）→订单服务向支付服务下支付订单→订单服务向充值服务下充值订单→订单服务自身下商品订单；
*	**用户支付：**支付网关（外围）接受请求→回调支付服务通知支付结果→支付服务更新支付订单状态→支付服务向充值服务发起充值→充值服务向充值网关（外围）发起充值并更改充值订单状态；
*	**订单对账：**定时支付网关对账、定时充值订单对账：

![](/img/notes/microservice/smallSystemMicroService/account_check.png)

从以上流程可以看出，每个服务都有自己专注的“职能”，每个应用业务流程有需要1或N个服务的交互才能完成，每个服务都有自己独立的“数据源”，互不干扰。由于系统的初期规模预期比较小，可能每天就那么数百笔订单甚至可能更少，如果我们每个服务都需要“物理隔离”，未免有点大题小做。因此，项目初期我们按“单体”模式实施：

![](/img/notes/microservice/smallSystemMicroService/all_in_one.png)

所谓的“单体”，即把所有服务代码结合一个“项目”打包发布，也就是一个“普通”的项目并且共用一个数据库，但每个服务的表名都有服务的标识（约定），例如商品服务的相关表名以“KW_GOODS_XXX”命名，订单服务的相关表名以“KW_ORDER_XXX”命名，支付服务的相关表名以“KW_PAYMENT_XXX”命名，充值服务的相关表名以“KW_RECHARGE_XXX”命名，对账服务的相关表名以“KW_ACCOUNT_XXX”命名，服务之间决不能跨越服务操作数据库表，必须按照“业务流程设计”调用，所以“单体”只是体现在物理实施层面，逻辑层面始终保持着“微服务”的分布式特性，保留了各种不用修改一行代码即可灵活扩展的可能性：

![](/img/notes/microservice/smallSystemMicroService/service_collabation.png)

可能有人会问，“单体”模式的服务调用怎么调？“分布式”模式又是怎么调？怎么确保扩展时服务代码调用层面的不变？用的是什么技术？这篇随笔就先不谈太多的技术，服务的调用过程我大概通过伪代码图术一下：

![](/img/notes/microservice/smallSystemMicroService/service_proxy.png)

服务之间协作的“透明化”关键在于把“微服务”灵活特性所导致的“变化”打包封装起来，就是以上伪代码的DiscoveryClient调用代理。DiscoveryClient在技术框架内维护了所有服务的信息（Service Data Cache Container），而服务信息的加载方式是服务解耦的关键所在。首先，框架通过本地扫描的方式把所有本地服务信息扫描并加载至“服务容器”（本地扫描LocalService）。其次，框架会检测本地的“服务信息”配置文件并加载至容器（静态解析）。最后，框架会根据配置前两步所加载的服务信息判断是否存在“发现中心服务”并动态地周期性向“发现中心”更新服务信息（动态解析）。因此，无论是单体应用部署还是分布式应用部署，对服务调用是透明的，保留了整个系统的灵活扩展性。到这里，整个系统的设计基本完，完整的系统架构图如下所示：

![](/img/notes/microservice/smallSystemMicroService/single.png)

以上系统在无任何优惠的正常运行下，确实只能算得上小规模，一台服务器的单体部署模式足以支撑，但在每月会员日所推出“充100元送10元”商品的时候，单体应用就显得有点力力不从心了，商品服务访问量的增加（看得人多了）已经影响到了其它服务的稳定性，并且考虑对账的稳定性（以免充值不到账引起投诉），决定把商品服务和对账服务独立（进程）部署。通过加入网关（Nginx）进行服务分发（服务解耦）：

![](/img/notes/microservice/smallSystemMicroService/distribute_1.png)

在运营过程中，公司为了“流量经营”，不惜下血本推出“充100元送50元”的商品，系统再一次受到严峻的考验，为了业务质量，不得不把所有服务独立（进程）部署，增加支撑力度：

![](/img/notes/microservice/smallSystemMicroService/distribute_2.png)

不知道是不是老板喝多了还是运营短路了，竟然提个“充值100元送100元”的商品，并明天上线，系统峰值支撑并发量的预测已经超出了我的想象力，我能做的只有这样了：

![](/img/notes/microservice/smallSystemMicroService/distribute_3.png)

系统从业务规模来看确实存在大小之分，但从设计思想层面，系统是没有大小之分。以上“戏路”都是以服务为单元进行灵活扩展，其实业务的最小力度是服务的具体行为—API，每个API都是服务的一个独立行为，例如查询、变更等，完全符合“命令查询职责分离(CQRS)模式”的设计，按服务这种API粒度进行横向分解同样可行，例如“支付服务”存在支付订单查询API、支付订单下单API、支付结果通知接收API，我们可以通过读写特性把查询API和变更类API进行分离，同样可以以面向“消费对象”的角度进行分解：

![](/img/notes/microservice/smallSystemMicroService/service_extend.png)

如果某些业务存在服务链复杂的话（例如商品订单），还可以自定义“编排服务”解耦“基础服务”的复杂度：

![](/img/notes/microservice/smallSystemMicroService/service_arrangement.png)

# 学习总结

如果细心点可以从以上案例发现，我的整个项目开发过程跟传统的可能会有点区别，什么区别呢？这里没有突出太多的实体对象设计或者表结构设计，更没有突出所谓的“三层结构”设计，而是直接从业务角度触发划分“业务对象”，而我们的服务呈现的是根据业务领域划分的“对象”描述，与传统按“数据实体”划分的设计模式还是有一定的区别，从需求设计到软件设计和开发都是“业务模型”最原始和最直观的呈现，保证了业务准确性并减少了变更的风险成本，同还大大地降低了项目的沟通成本。这种思想有一个更加专业的术语叫“领域驱动设计”。我深知自己距离所谓的“微服务”或者“领域驱动设计”还有一大段距离，并且以上案例还可能存在诸多的细节问题，但这种类似的思想确实是我自身从业务中摸爬滚打并逐步思考和沉淀而形成的设计习惯。不是别人说什么就做什么，而是通过实践去思考和领悟并向真正的源头的“大师们”学习。