---
title: "在网格的边缘试探——企业服务行业如何试水 Istio"
date: 2019-01-11T11:10:19+08:00
draft: false
authors: ["崔秀龙"]
summary: "本文根据崔秀龙在 2019 广州 Service Mesh Meetup#5 分享整理。"
tags: ["service mesh"]
categories: ["service mesh"]
keywords: ["service mesh","服务网格"]
---

> 崔秀龙，HPE 软件分析师，Kubernetes 权威指南作者之一，Kubernetes、Istio 项目成员。
>
> 本文根据崔秀龙在 2019 广州 Service Mesh Meetup#5 分享整理，完整的分享 PPT 获取方式见文章底部。
>
> 本文内容收录在崔秀龙的新书：《深入浅出 Istio - Service Mesh 快速入门与实践》的第十章，该书将于近期由博文视点出版发行，敬请关注。

![img](1547173724138-2dd495ad-ea26-45eb-8d9b-6306df9c7855.jpeg)

Service Mesh 概念在 Linkerd 落地之后，让一直漂浮在空中的微服务治理方案有了一个明确的落地点，给微服务架构的具体实现指出了一个清晰的方向，围绕这一概念逐步开始形成新的技术生态，在业界造成不少震动。这种震动对于企业 IT 转型工作带来的影响，甚至比容器化的影响更加深远。对于承担企业 IT 转型工作的企业服务行业来说，也自然首当其冲感觉到新概念带来的压力。

企业服务行业和互联网行业相比，业务形态、技术积累和人员结构等方面都大相径庭，举几个常见的差异：

- 开发、运维、基础设施所属
- 人员结构、水平和年龄
- 资源使用率差别
- 架构和平台一致性
- 负载能力
- ...

目前进行 Service Mesh 布道的主力还是互联网行业的旗手们，一味追求跟进互联网同行们的进度和做法，颇有邯郸学步的风险。

本文中将会针对目前 Service Mesh 方面的一些普遍问题和关注热点发表一些个人意见。并尝试提供一种 Istio 的试用思路，给乙方同行们提供参考。

## Istio 的功能

无需赘述，多数用户都很清楚，Istio 使用和应用共享网络栈的方式，利用 Iptables 劫持应用的网络流量，从而在不修改业务源码的情况下，完成一系列的功能：

- 监控服务质量
- 控制服务间的访问路由
- 应对服务故障
- 在服务间通信之间进行加密
- 访问控制和频率限制

> 分布式跟踪和业务紧密相关，无法做到无侵入。

这其中最大的优势就是无侵入，这意味着给试用流程留下了全身而退的机会，如果没有回滚的能力，上述种种能力都是空中楼阁。

## Istio 的问题

- API 稳定性可能是最严重的一个问题。目前最成熟的功能组别应该是流量控制，其版本号也仅是 v1alpha3，一般来说，alpha 阶段的产品，代表着不提供向后兼容的承诺，流量控制 API 在从 v1alpha2 升级为 v1alpha3 的过程中，API 几乎全部改写，使得很多早期用户的精力投入付诸东流。核心功能尚且如此，遑论相对比较边缘的 Mixer、Citadel 以及 Galley 组件的相关内容。
- 发布节奏和发布质量的问题也相当严重。Istio并不算长的历史中，出现了多次版本撤回、大版本严重延期、发布质量低下无法使用以及 Bug 反复等状况，这无疑会让每次升级尝试都充满了不确定性，会很大的影响生产过程的连续性。
- Mixer 是一个问题焦点，其数据模型较为复杂，并且集中了所有应用的流量于一点，虽然其中加入了各种缓存等技术来降低延迟，但是其独特地位决定了 Mixer 始终处于一个高风险的位置。同时其 API 结构稍显混乱，重构风险较大。
- Pilot的性能方面也经常为人诟病，虽然经过几次升级，但是即使是 1.0 之后，还是出现了两次 Pilot 在集群中服务/Pod 过多的情况下会超量消耗资源的问题。
- 安全、物理机和虚拟机的支持以及网格边缘通信这三组功能，目前用户较少，质量尚不明确。
- 最后就是 Istio 的 Sidecar 注入模式，这种模式一定会增加服务间调用的网络延迟，在目前阶段这是一个痼疾，Sidecar 的固定延迟和 Mixer 的不确定行为相结合，有可能会产生严重后果。

这里提出的只是被反复提及，或者经常出现在 Issue 列表中的问题，由发布问题来看，面临的风险可能远不止这些。

## Istio 试用工作的理由和规划

**试用 Istio，首先应该确定，该技术的采用，是否能够在可控的风险和投入下，得到有效的产出。**

1. 微服务模式的推进，必须要有相应的管理能力，Service Mesh 目前看来，是一个确定有效的方案，如果不是 Istio，也会有其它替代产品出现。
2. 目前看来，Istio 是 Service Mesh 的标志性产品，有一定可能性成为事实标准。
3. 提供了众多开箱即用的丰富特性，能够迅速进入 Service mesh。
4. 最后是无侵入的优势：如果试用失败，可以退回，控制损失范围。

Istio 的多数功能，在无需对程序进行修改（分布式跟踪除外）的情况下，能对应用提供如此之多的功能支持，无疑是非常有吸引力的。Istio 的功能集，完全可以说是服务网格技术的典范。一旦确认现有环境有可能支持 Istio 的运行，并且在合理的投入下能够获得有效益的产出，那么这个试用就是有价值的。

结合 Istio 的现状，以及多数企业的运行状态，个人浅见，Istio 的应用在现阶段只能小范围试探性地进行，在进行过程中要严格定义试用范围，严控各个流程。 按照个人经验，笔者将试用过程分为如下 4 个阶段。

- 范围定义：选择进入试用的服务，确定受影响的范围，并根据 Istio 项目现 状决定预备使用的 Istio 相关功能。围绕这些需要，制定试用需求。
- 方案部署：根据范围定义的决策，制定和执行相关的部署工作。其中包含 Istio 自身的部署和业务服务、后备服务的部署工作。
- 测试验证：根据既有业务目标对部署结果进行测试。
- 切换演练：防御措施，用于在业务失败时切回到原有的稳定环境。

## Istio 的试用步骤

### 确定功能范围

在 Istio 中包含了非常多的功能点，从原则上来说，各种功能都是有其实际作用的。然而，Istio 作为一个新产品，本身也有很多不足，我们在 10.1 节中也提到了这些不足。

Istio 提供的众多功能对每个公司或者项目，都会有不同价值。我们在采用一个新系统时，首先要考虑的就是性价比问题，这里的“价”代表着 Istio 带来的风险、对业务应用的影响，还包括可能出现的服务停机等问题。

可以根据性价比，做出一个优先级别列表。在制定了优先级列表之后，就可以根据这一列表，结合项目的实际需求，按照效果明显、功能稳定、部署成本低、少改造或者不改造的标准来进行选择，最终确定待测试的功能点。

在选定功能点之后，应该遵循目前已有的 Istio 文档，对各个功能点进行单项测试和验证，以确保其有效性。并通过官方 GitHub 的 Issue 列表及讨论组内容，了解现有功能是否存在待解决的问题，以及相关的注意事项等。

### 选择试用业务

在试用功能点确定之后，就要选择用于试用的业务应用了。Istio 作为一个相对底层的系统，其部署和调试过程必然会对业务产生一定的影响，在运行阶段又有 Sidecar 和各个组件造成的损耗，如下所述：

- 所有网格之间的通信都要经过 Sidecar 的中转，会造成大约 10 毫秒的延迟。
- Pilot 对集群规模敏感，集群中的服务数量、Pod 数量都可能对 Pilot 造成较大影响，也会影响到 Istio 各种规则向 Pod 的传输过程。
- 所有流量都会经由 Mixer 处理，也有造成瓶颈的可能。
- 安全功能设置不当同样会造成服务中断。

如上所述还只是个概要，对业务来说，对这些风险都是必须正视并做好预案的。 为了避免引起过大损失，建议将如下标准作为选择试用服务的依据：

- 能够容忍一定的中断时间。
- 对延迟不敏感。
- 调用深度较浅。
- 能够方便地回滚和切换。
- 具备成熟完善的功能、性能和疲劳测试方案。

### 定义试用目标

按照现有业务的实际需要，对试用服务进行功能分析。和传统的需求功能分析类似，要在该过程中明确一些具体的需求内容。

- 环境需求：申请符合 Istio 运行以及业务要求的集群以及资源。
- 功能性需求：在 Istio 的功能中选择需要 Istio 为试用服务提供支撑的功能，应形成功能测试案例。
- 服务质量需求：根据现有业务的运行状况，对服务质量提出具体要求，例如并发数量、响应时间、成功率等，应形成性能测试案例。
- 故障处理需求：对于试点应用发生故障时，如何在网格和非网格版本的试用 服务之间进行切换以降低故障影响，应形成故障预案。

### Istio 部署

首先是基本环境的准备，按照前面提到的环境需求，复查集群环境。 如果是内网部署，应该部署内网可达的私有镜像库，推送全部所需的镜像，并利用 Helm 变量设置合理的镜像地址。 接下来根据试用需求，利用 Helm 对 Istio 部署进行调整，这方面的调整主要分为两类——资源分配和功能裁剪。

- 资源分配方面，在官方提供了资源分配建议可以参考，利用 Helm value 进行设置即可。
- 功能裁剪，同样需要对 Helm value 进行设置，关闭不需要的功能。

### 后备业务部署

在进行试用应用的注入之前，首先应该部署一组备份服务，这组服务需要和整体服务网格进行隔离。这一组备份服务应处于待机模式，以备网格版本的应用在出 现故障时，进行整体切换。基于这一点考虑，负载均衡等前端控制设施也应齐备。

### 试用业务部署

接下来要把试用服务部署到网格之中，同其他 Kubernetes 一样，网格应用的部署也是从 YAML 代码开始的。原有应用的部署代码需要根据 Istio 标准进行复核，检查其中的端口命名、标签设置。

除了待注入的应用清单文件，还应该为每个部署单元都提供默认的 VirtualService 和 DestinationRule，建立基本的路由关系，提供一个路由基准，方便在路由调整过程中进行对比。

然后根据在前面制定的具体网格需求列表，逐个编写所需的路由、规则等方面的配置内容。 在这些都完成之后，就可以按照顺序逐个提交部署了。

### 监控告警部署

在试用服务部署之后，就有更多的项目可以监测了，这里建议将其自带的 Prometheus 进行变更，连接到能够有效发出告警的 Alert manager 组件上，并根据试用业务的服务质量、Istio 组件进行告警设置。

### 验证测试

根据功能需求对试用服务进行功能测试，在测试通过之后进行性能和疲劳测试，观察各方面的性能指标是否符合，如果性能出现下滑，则可以尝试扩容，提高资源分配率。

关键组件的性能下降有可能是 Istio 自身的问题，应检查社区 Issue 或提出新的 Issue。

此处是一个关键步骤，如果测试方案不符合实际情况或者预期目标无法达到， 则强烈建议放弃试用。

### 切换演练

在功能和性能测试全部通过之后，就应该进行试用服务和后备服务之间的双向切换的演练，在双方切换之后都应该重复进行验证过程，防止故障反复。 切换演练是试点应用的最后一道保险，在网格严重故障之后能否迅速恢复业务，全靠这一步的支持，因此同样需要认真对待。

## 结论

虽然很多企业服务团队的研发能力不高，无法像一些高水平技术团队一样，对开源软件进行因地制宜的适应性修改。然而需要理解的重要一点是，不少软件项目其实并非为世界级的流量而生的，互联网 Say No 的时候，其它场景中未必无法接受。通过细致的调查研究，详尽的方案设计，谨慎的执行和验证之后，Service Mesh 或者其它的新技术的试用决策都是可以进行尝试的，甚至也是有可能因此获利的。

![img](1547173702228-fc2a6314-b23b-4f38-95c3-a23f29efc630.jpeg)

**PPT 下载和视频地址**

点击页面回顾按钮，回顾当天所有讲师分享：https://tech.antfin.com/activities/72

  **延伸阅读**

- [蚂蚁金服 Service Mesh 渐进式迁移方案|Service Mesh Meetup 实录](http://mp.weixin.qq.com/s?__biz=MzUzMzU5Mjc1Nw==&mid=2247484533&idx=1&sn=6e574fffc87c334aab896f79b6b03296&chksm=faa0ebafcdd762b91311ac5cebb3f85a695e0c03702278de937eed905f331f9cf9fff9b80a7c&scene=21#wechat_redirect)
- [蚂蚁金服 Service Mesh 新型网络代理的思考与实践 | GIAC 分享实录](http://mp.weixin.qq.com/s?__biz=MzUzMzU5Mjc1Nw==&mid=2247484541&idx=1&sn=81fefed2ab7f67d032a9f5b90db61890&chksm=faa0eba7cdd762b1498764303188a67e5b3c0628193256d8853fb6018873481c8ac816a09bac&scene=21#wechat_redirect)
- [蚂蚁金服 Service Mesh 实践探索](https://mp.weixin.qq.com/s?__biz=MjM5MDE0Mjc4MA==&mid=2651010202&idx=1&sn=742179879a25d526402a5b561b769ed1&scene=21#wechat_redirect)
