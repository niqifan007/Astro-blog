---
title: 零知识机器学习（ZKML）简介
published: 2023-09-17
description: '零知识机器学习（ZKML）简介'
image: 'https://s2.loli.net/2023/09/17/pT6MPR9FQADVx7G.png'
tags: [zk, 零知识, 区块链]
category: 'resources'
draft: false 
lang: ''
---
# 零知识机器学习（ZKML）简介

Zero-Knowledge 机器学习（ZKML）是最近正在密码学界引起轰动的一个研究和开发领域。但它是什么，有什么用处呢？首先，让我们把这个术语分解成它的两个组成部分，并解释一下它们是什么。

 ## 什么是 ZK？

零知识证明是一种密码协议，其中一方（证明者）可以向另一方（验证者）证明一个给定的陈述是真实的，而不泄露除该陈述为真以外的任何附加信息。这是一个正在各个方面取得巨大进展的研究领域，涵盖了从研究到协议实施和应用的所有方面。

ZK 提供的两个主要“原语”（或者说构建块）是能够为一组给定的计算创建具有计算完整性证明的能力，其中证明比执行计算本身要容易地多。（我们称这种属性为“简洁性”）。ZK 证明也提供了隐藏计算中某些部分同时保持计算正确性的选项。（我们称这种属性为“零知识性”）。

**生成零知识证明需要非常大的计算量，大约比原始计算贵 100 倍**。这意味着，在某些情况下由于最佳硬件上生成它们所需的时间使其不切实际，因此不能计算零知识证明。

然而，在近年来密码学、硬件和分布式系统领域的进步已经使零知识证明成为了越来越强大的计算可行的选择。这些进展已经为可以使用计算密集型证明的协议的创建提供了可能，从而扩大了新应用程序的设计空间。

**ZK 使用案例**

零知识密码学是 Web3 空间中最流行的技术之一，因为它允许开发人员构建可扩展和/或私有的应用程序。以下是一些实践中如何使用它的示例（尽管请注意，这些项目中许多都还在进行中）：

**1.通过 ZK rollups 扩展以太坊**

- Starknet
- Scroll
- Polygon Zero，Polygon Miden，Polygon zkEVM
- zkSync

**2.构建保护隐私的应用程序**

- Semaphore
- MACI
- Penumbra
- Aztec Network

**3.身份原语和数据来源**

- WorldID
- Sismo
- Clique
- Axiom

**4.第一层协议**

- Zcash
- Mina

随着 ZK 技术的成熟，我们相信将会出现新的应用程序的爆发，因为构建这些应用程序所使用的工具将需要更少的领域专业知识，对于开发人员来说将会更加容易使用。

## ZKML 的动机和当前的努力

我们生活在一个世界上，**AI/ML 生成的内容越来越难以与人类生成的内容区分开来**。零知识密码学将使我们能够做出这样的声明：“给定一段内容 C，它是由模型 M 应用于一些输入 X 生成的。”我们将能够验证某个输出是否是由大型语言模型（如 chatGPT）或文本到图像模型（如 DALL-E 2）等任何其他我们为其创建了零知识电路表示的模型所生成的。这些证明的零知识属性将使我们能够根据需要也隐藏输入或模型的某些部分。一个很好的例子是在一些敏感数据上应用机器学习模型，在不透露输入到第三方的情况下，用户可以知道他们的数据在模型推理后的结果（例如，在医疗行业）。

注：当我们谈论 ZKML 时，我们是指创建 ML 模型推理步骤的零知识证明，而不是关于 ML 模型训练（它本身已经非常计算密集）。目前，现有技术水平的零知识系统加上高性能硬件仍然相差几个数量级，无法证明当前可用的大型语言模型（LLMs）等庞大的模型，但是在创建较小模型的证明方面已经取得了一些进展。

我们对零知识密码学在为 ML 模型创建证明的上下文中的现有技术水平进行了一些研究，并创建了一个聚合相关研究、文章、应用程序和代码库的文章集。ZKML 的资源可以在 GitHub 上的 ZKML 社区的 [awesome-zkml](https://github.com/zkml-community/awesome-zkml) 存储库中找到。

[Modulus Labs](https://www.moduluslabs.xyz/) 团队最近发布了一篇名为“[智能的成本](https://medium.com/@ModulusLabs/chapter-5-the-cost-of-intelligence-da26dbf93307)”的论文，其中对现有的 ZK 证明系统进行了基准测试，并列举了不同大小的多个模型。目前，使用像 [plonky2 ](https://polygon.technology/blog/plonky2-a-deep-dive)这样的证明系统，在强大的 AWS 机器上运行 50 秒左右，可以为约 1800 万个参数的模型创建证明。以下是该论文中的一张图表：

另一个旨在改进 ZKML 系统技术水平的倡议是 Zkonduit 的 [ezkl](https://github.com/zkonduit/ezkl) 库，它允许您创建对使用 ONNX 导出的 ML 模型的 ZK 证明。这使得任何 ML 工程师都能够为他们的模型的推理步骤创建 ZK 证明，并向任何正确实现的验证器证明输出。

有几个团队正在改进 ZK 技术，为 ZK 证明内部发生的操作创建优化硬件，并针对特定用例构建这些协议的优化实现。随着技术的成熟，更大的模型将在较不强大的机器上短时间内进行 ZK 证明。我们希望这些进展将使新的 ZKML 应用程序和用例得以出现。

 

![](https://hx24-prod.mars-block.com/image/crawler/2023/04/06/1680766766002460.jpg)

## 潜在的使用案例

为了确定 ZKML 是否适用于特定的应用，我们可以考虑 ZK 密码学的特性将如何解决与机器学习相关的问题。这可以用一个 Venn 图来说明：

![](https://hx24-prod.mars-block.com/image/crawler/2023/04/06/1680766766002404.jpg)


定义：

**1.Heuristic optimization**[**启发式优化**](https://en.wikipedia.org/wiki/Heuristic_(computer_science))—— 一种问题解决方法，它使用经验法则或“启发式”来找到艰难的问题的好解决方案，而不是使用传统的优化方法。启发式优化方法旨在在相对的重要性和优化难度下，在合理的时间内找到好的或“足够好”的解决方案，而不是尝试找到最优解决方案。

**2.FHE ML**[全同态加密](https://en.wikipedia.org/wiki/Homomorphic_encryption)—— 完全同态加密ML允许开发人员以保护隐私的方式训练和评估模型；然而，与ZK证明不同，没有办法通过密码学方式证明所执行的计算的正确性。

- 像 [Zama.ai](https://www.zama.ai/)这样的团队正在从事这个领域的工作。

**3.ZK vs Validity** —— 在行业中，这些术语通常被互换使用，因为有效性证明是ZK证明，不会隐藏计算或其结果的某些部分。在ZKML的上下文中，大多数当前的应用程序都利用了ZK证明的有效性证明方面。

**4.Validity ML** —— ZK证明ML模型，在其中没有计算或结果被保密。它们证明计算的正确性。

以下是一些潜在的 ZKML 用例示例：

1.计算完整性（有效性 ML）

- Modulus Labs[模量实验室](https://www.moduluslabs.xyz/)
- 基于链上可验证的 ML 交易机器人 - [RockyBot](https://github.com/Modulus-Labs/RockyBot)

自我改进视觉区块链（示例）：

- 增强 Lyra 金融期权协议 AMM 的智能特性[Lyra 金融期权协议 AMM](https://www.lyra.finance/)
- 为 [Astraly](https://www.astraly.xyz/)  创建透明的基于 AI 的声誉系统（ZK oracle）
- 使用[ ML for Aztec Protocol](https://aztec.network/)（具有隐私功能的 zk-rollup）致力于合同级合规工具所需的技术突破。

2.[机器学习即服务](https://twitter.com/daniel_d_kang/status/1582519854405255168?s=20&t=16FXZixQvD5G_B9IFVzmaA)(MLaaS) 透明；

3.ZK 异常/欺诈检测：

- 这种应用场景使得可创建针对[允许为可利用性](https://blog.trailofbits.com/2020/05/21/reinventing-vulnerability-disclosure-using-zero-knowledge-proos/)/欺诈的 ZK 证明成为可能。异常检测模型可以在智能合约数据上进行训练，并由 DAOs 同意作为有趣的度量标准，以便能够自动化安全程序，如更主动、预防性地暂停合约。已有初创企业正在研究在智能合约环境中使用 ML 模型进行安全目的的方法，因此 ZK 异常检测证明似乎是自然的下一步。

4.ML 推理的通用有效性证明：能够轻松证明和验证输出是给定模型和输入对的乘积。

5.隐私 (ZKML)。

6.去中心化的 [Kaggle](https://www.kaggle.com/)：证明模型在某些测试数据上的准确率大于 x%，而不会显示权重。

7.隐私保护推理：将对私人患者数据的医疗诊断输入模型，并将敏感的推理（例如，癌症测试结果）发送给患者。（来源：[vCNN 论文](https://eprint.iacr.org/2020/584.pdf)，第 2/16 页）

8.Worldcoin：

- **IrisCode 的可升级性**：[World ID](https://id.worldcoin.org/) 用户将能够在他们的移动设备的加密存储中自我保管其生物特征，下载用于生成 IrisCode 的 ML 模型并在本地创建零知识证明，以证明其 IrisCode 已成功创建。这个 IrisCode 可以被无需许可地插入注册的 Worldcoin 用户之一，因为接收的智能合约可以验证零知识证明，从而验证 IrisCode 的创建。这意味着，如果 Worldcoin 将来升级机器学习模型以一种破坏与其之前版本兼容性的方式创建 IrisCode，用户就不必再次去 Orb，而可以在设备上本地创建这个零知识证明。
- **Orb 安全性**：目前，Orb 在其受信任的环境中执行几个欺诈和篡改检测机制。然而，我们可以创建一个零知识证明，表明这些机制在拍摄图像和生成 IrisCode 时是活动的，以便为 Worldcoin 协议提供更好的活体保证，因为我们可以完全确定这些机制在整个 IrisCode 生成过程中都将运行。

总之，ZKML 技术有着广泛的应用前景，并且正在快速发展。随着越来越多的团队和个人加入到这个领域，我们相信 ZKML 的应用场景将会更加多样化和广泛化。