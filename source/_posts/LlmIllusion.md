---
title: 大模型幻觉
tags:
- 大模型
categories:
- 大模型相关
---
# 大模型幻觉

## 一、什么是 大模型幻觉问题？

### 1.1 大模型幻觉问题定义

- 定义：当模型生成的**文本不遵循原文（Faithfulness）或者不符合事实（Factualness）**，我们就可以认为模型出现了幻觉的问题。

### 1.2 何为 Faithfulness and Factualness？

- Faithfulness：是否遵循input content；
- Factualness：是否符合世界知识；

![](image.png)

（参考：Siren's Song in the AI Ocean: A Survey on Hallucination in Large Language Models，[https://arxiv.org/abs/2309.01219](https://arxiv.org/abs/2309.01219)）

![](image1.png)

1. **输入相冲突的幻觉。**输入相冲突的幻觉，即LLM生成的内容与用户提供的源输入相背离。当LLM生成的内容偏离用户输入时，就会产生这类幻觉。
2. **语境冲突性幻觉。**LLM生成的内容与之前生成的信息本身相冲突。LLMs在生成冗长或多轮回答时可能会表现出自我矛盾。这种类型的幻觉产生于LLMs在整个对话过程中失去对上下文的跟踪或无法保持一致性时，这可能是由于他们在保持长期记忆或识别相关上下文方面的局限性造成的。
3. **与事实相冲突的幻觉。**与事实相冲突的幻觉指的是LLM生成的内容不忠实于既定的世界知识。事实冲突型幻觉。当LLM生成的信息或文本与已有的世界知识相矛盾时，就会出现这种类型的幻觉。如图2所示，事实冲突幻觉的来源可能多种多样，并在LLM生命周期的不同阶段出现。

### 除幻觉外的常见问题:

![](image2.png)

1. **歧义性：**答案可能并不一定是不正确的，但是对于用户问题却没有给出一个有用的答案。
2. **不完整性：**不完整性问题发生在生成的响应不完整或碎片化的情况下，如上图例子中只给出了换轮胎4个步骤中的2步。
3. **偏见：**LLMs中的偏见是指生成文本中不公平或偏见态度的表现，这些偏见可能来源于训练数据。如上图例子中将教师描述为女性是一种偏见。
4. **信息不足：**指LLMs逃避回答某些问题的倾向，即使其有能力这样做。例如，由于奖励模型的不完善，RLHF可能会导致LLMs的过度优化，从而可能导致模型输出信息不足。

### 1.3 传统任务中的模型幻觉 vs LLMs 中模型幻觉

1. **在传统任务里，幻觉大都是指的是Faithfulness**：
   1. **Intrinsic Hallucination（信息冲突）**: LMs在生成回复时，与输入信息产生了冲突，例如摘要问题里，abstract和document的信息不一致；
   2. **Extrinsic Hallucination（无中生有）**: LMs在生成回复时，输出一些并没有体现在输入中的额外信息，比如邮箱地址、电话号码、住址，并且难以验证其真假。（PS: 按照此定义，Extrinsic Hallucination有可能是真的信息，只是需要外部信息源进行认证）
2. **而面向LLMs，我们通常考虑的幻觉则是Factualness**：
   1. 因为我们应用LLM的形式是open-domain Chat，而不是局限于特定任务，所以数据源可以看做任意的世界知识。LLMs如果生成了不在input source里的额外信息，但是符合事实的，这种情况也可能是对我们有帮助的。

## 二、大模型幻觉从何而来？

### 2.1 从 数据角度 进行分析

在 数据构建过程中，由于以下问题，导致 模型幻觉 的 发生：

1. 训练数据可信度问题。由于 大模型 的 训练数据 都是 通过 众包/爬虫检索 方式 收集得到的，这种数据构建方式的优点是量比较大，但是缺点是 包含 大量虚假信息。这种虚假信息 直接导致的问题就是使 模型出现错误认知；
2. 重复数据问题。过多的重复信息也可能导致模型的知识记忆出现bias，从而导致幻觉；
3. LLM偏向于肯定测试的样本，即人类生成的语料中也存在幻觉(可反映为过时的、双重的或捏造的表达)，LLMs很容易复制甚至放大这种幻觉行为。

> 引用至 [3] Deduplicating training data makes language models better

### 2.2 从 模型角度 进行分析

不止是 数据角度问题，大模型幻觉问题 出现的原因 还 表现在 模型角度。

- **模型结构**：如果是较弱的backbone（比如RNN）可能导致比较严重的幻觉问题，但在LLMs时代应该不太可能存在这一问题；
- **解码算法**：
  - 研究表明，**如果使用不确定性较高的采样算法（e.g.，top-p）会诱导LMs出现更严重的幻觉问题**。甚至可以故意在解码算法中加入一些随机性，进一步让LMs胡编乱造（可以用该方法生成一些negative samples）
  - 局部最优化(标记预测)并不一定能确保全局最优化(序列预测)，早期的局部预测可能会将LLMs带入难以形成正确反应的境地。

引用至 [4] Factuality enhanced language models for open-ended text generation

- **暴露偏差**：**训练和测试阶段不匹配的exposure bias问题可能导致LLMs出现幻觉，特别是生成long-form response的时候**。

> 引用至 [5] On exposure bias, hallucination and domain shift in neural machine translation

- **参数知识**：**LMs在预训练阶段记忆的错误的知识，将会严重导致幻觉问题**。

> 引用至 [6] Entity-based knowledge conflicts in question answering

## 三、如何 评估 大模型幻觉问题？

**现有的传统幻觉评估指标和人类结果的相关性往往较低**，同时大都是task-specific的 [7]。

### 3.1 Reference-based

Reference-based的指标有两类：

1. **基于Source Information和Target Reference**：利用一些统计学指标，比如**ROUGE、BLEU来评估输出结果和Source/Target信息的重叠度**;
2. **基于Source Information**：由于NLG任务里，Target输出往往是多种多样的，因此许多工作**只基于Source信息进行幻觉的评估**。比如Knowledge F1。

基于Reference的评价指标，**基本上只能评价Faithfulness，而无法评价Factualness，因此通常不适用于LLMs**。

### 3.2 Reference-Free

### 3.2.1 基于IE

- 介绍：**将知识限定于可以用三元组形式表示的关系和事件**，基于额外的IE模型进行抽取，接着使用额外模型进行验证；
- 缺点：
  - 可能存在IE模型的错误传播问题；
  - 知识被限定在三元组形式。

### 3.2.2 基于QA

- 介绍：

1. 第一步先**基于LM生成的回复**，使用一个QG(question generation)模型生成一系列QA pairs；
2. 第二步**给定Source Information，让QA模型对上一步生成的Question进行回复**；
3. 第三步则是**通过对比第一步的answers和第二步的answers，计算匹配指标，衡量模型的幻觉问题**；

- 缺点：可能存在IE模型的错误传播问题；难以评估Factualness，因为上述第二步里面，Source Information不可能包含全部的世界知识，因此对于一些问题难以生成可靠的回复。

> 引用至 [8] FEQA: A question answering evaluation framework for faithfulness assessment in abstractive summarization

### 3.2.3 基于NLI

- 介绍：基于NLI的方法**通过NLI模型评估是否Source Information可以蕴含Generated Text，从而评估是否出现了幻觉现象**。
  - 缺点：
    - Off-the-shelf NLI模型用于核查事实效果不是很好

> 引用至 [9] Evaluating groundedness in dialogue systems: The BEGIN benchmark.

- 无法评估需要世界知识的幻觉问题：
  - **仅能依赖于Source进行核查**；
  - 都是sentence-level的，**无法支撑更细粒度的幻觉检查；**

> 引用至 [10] Evaluating factuality in generation with dependency-level entailment.

- 幻觉问题和蕴含问题实际并不等价：

1. 例子：Putin is president. -> Putin is U.S. president (可以蕴含，但是是幻觉)

### 3.2.4 基于Factualness Classification Metric

- 介绍：标注/构造一批和幻觉/事实有关的数据，训练检测模型，利用该模型评估新生成文本的幻觉/事实问题。

> 引用至 [11] Knowledge-powered conversational agents

### 3.2.5 人工评估

- 介绍：目前为止最靠谱的，此外还可以依靠LLM打分（比如利用GPT4，但是GPT4也存在着严重的幻觉问题，即使经过retrival-augment，检索回来的信息也有可能是错误的）

## 四、如何 缓解 大模型幻觉问题？

### 4.1预训练阶段

LLM的知识大多是在预训练阶段获得的。在预训练语料库中存在诸如错误信息之类的噪声数据可能会破坏LLMs的参数知识，而这是导致误差的一个重要因素。有时，还可以将语言模型获得的事实知识追溯到其训练数据。**因此，减少幻觉的直观方法可以是人工或自动整理预训练语料库，尽可能减少无法验证或不可靠的数据。**

1. **人工消除噪声训练数据**
   1. 专注于"数据到文本"(data-to-text)任务，并邀请人工根据给定的知识库手动编写干净准确的回复。
   2. 在现有的表格到文本数据集中对文本进行人工提炼这一过程也大大减少了事实幻觉。
   3. 在构建表对文训练数据时，指导注释者修改维基百科中已验证的句子，而不是直接创建新句子。
2. **自动选择可靠数据或过滤掉噪声数据**
   1. **GPT-3**的预训练数据就是通过与一系列高质量参考数据的相似性进行清理。
   2. **Falcon**通过启发式规则从网络中仔细提取高质量数据，并证明了经过适当分级的相关语料库可以产生强大的LLM。为了减少幻觉，目前的LLM通常会从可靠的文本来源收集预训练数据。
   3. **Llama2**在构建预训练语料库时，从维基百科等高度事实性的来源中向上抽取数据
   4. 在事实性文档的句子前加上主题前缀，使每个句子在预训练时都能作为一个独立的事实，将文档名称作为主题前缀，并观察到这种方法提高了LLM在**TruthfulQA**上的性能**。**

### 4.2 SFT阶段

![](image3.png)

当前的LLMs一般都会经历一个被称为"监督微调"(SFT)的过程，以从预训练中获取所需的知识，并学习如何与用户互动。SFT通常包括首先注释或收集海量任务指令跟踪数据，然后使用极大似然法(MLE)在这些数据上对预先训练的基础LLM进行微调。

**目的**

- 学习如何与用户进行有效交互。
- 从预训练中获得的知识中抽取信息。

1. **加工整理数据**
   1. 与预训练类似，一种可能的方法是要减少SFT阶段的幻觉，可以对训练数据进行整理，SFT数据量相对较小。
   2. 手动和自动整理都是可行的选择。在未经编辑的数据上进行微调的LLM相比，在这些经过编辑的指令数据上进行微调的LLM表现出更高的真实性和事实性水平。
2. **加入拒答**
   1. 在推理过程中，如果被要求回答与未学知识相关的问题，他们很可能会自信地产生幻觉。
   2. 采用以诚实为导向的SFT，即在SFT数据中引入一些诚实样本。诚实样本指的是承认自己无能的回答，如"对不起，我不知道"。学会拒绝回答特定问题，从而帮助减少幻觉。

### 4.3 RLHF期间的缓解

![](image4.png)

1. 利用人类反馈不仅能缩小机器生成的内容与人类偏好之间的差距，还能帮助LLM符合预期的标准或目标。目前常用的一个标准是"3H"，即有益、诚实和无害。这里的"诚实"只是指在LLM反应中尽量减少幻觉。
2. 强化学习可以引导LLM探索其知识边界，使他们能够拒绝回答超出其能力范围的问题，而不是编造不真实的回答。但这种方法也带来了独特的挑战。例如，由于在乐于助人和诚实之间的权衡失衡，经过RL调整的LLM可能会表现出过度保守。
3. **GPT4使用合成幻觉数据来训练奖励模型并执行R**L，从而将TruthfulQA的准确率从约30%提高到60%。
4. 设计一种专门用于减轻幻觉的特殊奖励函数:"Unhedged/Hedged Correct/Wrong"，指的是LLM用肯定或犹豫的语气提供正确或错误的答案。

### 4.4 生成推理阶段-设计解码策略

在事实性方面，核采样(又称顶点采样)不如贪婪解码，可归因于top-p采样为提高多样性而引入的随机性，这可能会无意中导致幻觉，因为LLMs往往会编造信息以产生多样化的反应。

1. **引入事实核采样的解码算法**，旨在利用top-p和贪婪解码的优势，在多样性和事实性之间取得更有效的平衡。
2. "**验证链" (Chain-of-Verification, COVE )的解码框架**。该框架基于这样的观察：独立的验证问题通常比长形式答案中提出的问题产生更准确的事实。COVE框架首先规划验证问题，然后回答这些问题，最终产生一个强化的、修正的回应。在多种任务上的实验结果表明，COVE能够有效缓解幻觉。
3. **引入推理-时间干预(ITI)方法**，以提高LLM的真实性。该方法基于这样一个假设，即LLMs拥有与真实性相关的潜在的、可解释的子结构。ITI方法包括两个步骤:
   1. 在LLM的每个注意头之上拟合一个二元分类器，以确定一组在回答事实性问题时具有卓越线性探测准确性的注意头;
   2. 在推理过程中沿着这些与事实性相关的方向移动模型激活。
4. **检索增强设置**
   1. LLMs在处理下游任务时有时无法充分关注检索到的知识，尤其是当检索到的知识与LLMs的参数知识相冲突时。为了解决这个问题，可以采用直接的上下文感知解码(**CAD**)策略。
   2. **CAD**方法旨在迫使LLM更多地关注上下文信息，而不是过度依赖自身的参数知识来做出决策。实验结果表明，CAD能有效激发LLM利用检索知识的能力，从而减少下游任务中的事实幻觉
5. **修改模型结构**
   1. 多分支解码器和不确定性感知解码器。在构建LLM时采用双向自回归架构，从而实现从左到右和从右到左的语言建模，这种设计策略可以有效地利用双向自回归结构，有效地利用双向信息，有助于减少幻觉。

### 4.5 生成推理阶段-借助外部知识

使用外部知识作为补充证据来帮助LLMs提供真实的回复，是最近兴起的一种解决方案。这种方法通常包括两个步骤:第一步是准确获取与用户指令相关的知识。一旦获得了有用的知识，第二步就需要利用这些知识来指导应答的生成。

1. **知识获取**
   1. 外部知识库：大规模非结构化语料库、结构化数据库、维基百科等特定网站，甚至整个互联网
   2. 外部工具：FacTool针对特定的下游任务，利用不同的工具帮助检测LLM中的幻觉，如用于基于知识的质量保证的搜索引擎API、用于代码生成的代码执行器和用于科学文献审查的谷歌学术API。
   3. CRITIC让LLM与多个工具交互，并自动修改其响应，这已被证明能有效提高真实性。
2. **知识利用**
   1. **生成时补充**
      1. 利用检索到的知识或工具反馈的直接方法是，在提示LLMs之前直接将它们与用户查询串联起来，这种方法既有效又易于实施，这种知识也被称为内文知识。
   2. **事后纠正**
      1. 即在后处理阶段构建一个辅助固定器来纠正幻觉。固定器可以是另一个LLM，也可以是一个特定的小模型。这种固定器首先与外部知识源互动，收集足够的知识，然后纠正幻觉。

### 4.6 生成推理阶段-利用不确定性

不确定性是推理过程中保护和减少幻觉的重要指标。通常，它指的是模型结果的置信度。不确定性可以帮助用户确定何时信任LLM。只要能准确描述LLM响应的不确定性，用户就能过滤或纠正LLM的高不确定性声明，因为这类声明更容易是捏造的。

![](image5.png)

1. **基于Logit的估计**

   1. 这是一种基于对数的方法，它需要获取模型的logit，通常通过计算token级概率或熵来确定不确定性。

2. **其次是基于口头估计**

   1. 直接要求LLM表达其不确定度，例如使用以下提示:"请回答并提供您的置信度分数(从0到100)"。这种方法之所以有效，是因为当地语言学家的语言表达能力和服从指令的能力很强。也可以使用思维链提示来加强这种方法。

3. **基于一致性估计**

   1. 这种方法基于这样一个假设:当LLMs犹豫不决并对事实产生幻觉时，他们很可能会对同一问题做出逻辑上不一致的回答。
   2. 使用BERTScore、基于QA的指标和n-gram指标进行计算，并将这些方法结合起来能产生最佳结果。直接利用额外的LLM来判断两个LLM反应在相同语境下是否存在逻辑矛盾，可以采用另一种LLM来修正两个反应中这种自相矛盾的幻觉。

4. **多agent互动**

   ![](image6.png)

多个LLM(也称为代理)独立提出建议，并就各自的回应进行协作辩论，以达成单一共识。

### 4.3 个人总结

缓解大模型幻觉业务中相较可行的措施：

1. **构建高质量数据集：**
   1. 利用模型筛选出可能导致幻觉的数据并筛除。
   2. 预训练或SFT阶段给忠实度更高的数据加权，或者只使用可靠来源的数据（例如经过人工筛查的数据，医典、教科书等）。
2. **优化模型结构：**
   1. 检索增强被证明可以显著减少幻觉问题。
   2. 在解码时减少模型的随机性（例如，业务中已经采用的降低temperture的方法）。因为divisersity和faithfulness是一个trade-off的关系，减少diversity或randomness可以变相提高faithfulness/factuality。
   3. 设计更能充分编码以利用source information的方法。例如GNN网络，融入一些人类偏置。
3. **优化训练方式：**
   1. **多任务学习**，通过设计合适的额外任务，可以达到减轻幻觉的效果。
   2. **后处理**，设计一个小模型，专门处理幻觉的问题。
   3. **提前规划骨架**，再生成（sketch to content）。
   4. **~~可控文本生成~~**，将幻觉视为一种额外的属性，利用可控文本生成技术进行控制。
   5. **~~强化学习**，~~现有工作将减轻模型的幻觉作为reward函数，从而减轻幻觉现象。

## 参考文献

1. [Survey of Hallucination in Natural Language Generation](https://arxiv.org/abs/2202.03629)
2. [Reading list of hallucination in LLMs](https://github.com/HillZhang1999/llm-hallucination-survey)
3. Katherine Lee, Daphne Ippolito, Andrew Nystrom, Chiyuan Zhang, Douglas Eck, Chris Callison-Burch, and Nicholas Carlini. 2021. Deduplicating training data makes language models better. arXiv preprint arXiv:2107.06499 (2021).
4. Nayeon Lee, Wei Ping, Peng Xu, Mostofa Patwary, Mohammad Shoeybi, and Bryan Catanzaro. 2022. Factuality enhanced language models for open-ended text generation. arXiv preprint arXiv:2206.04624 (2022).
5. Chaojun Wang and Rico Sennrich. 2020. On exposure bias, hallucination and domain shift in neural machine translation. In Proceedings of the 2020 Annual Conference of the Association for Computational Linguistics. 3544–3552.
6. Shayne Longpre, Kartik Perisetla, Anthony Chen, Nikhil Ramesh, Chris DuBois, and Sameer Singh. 2021. Entity-based knowledge conflicts in question answering. In Proceedings of the 2021 Conference on Empirical Methods in Natural Language Processing (EMNLP’21). 7052–7063.
7. Artidoro Pagnoni, Vidhisha Balachandran, and Yulia Tsvetkov. 2021. Understanding factuality in abstractive summarization with FRANK: A benchmark for factuality metrics. In Proceedings of the 2021 Conference of the North American Chapter of the Association for Computational Linguistics: Human Language Technologies (NAACL-HLT’21). 4812–4829
8. Esin Durmus, He He, and Mona Diab. 2020. FEQA: A question answering evaluation framework for faithfulness assessment in abstractive summarization. In Proceedings of the 58th Annual Meeting of the ACL. 5055–5070.
9. Nouha Dziri, Hannah Rashkin, Tal Linzen, and David Reitter. 2021. Evaluating groundedness in dialogue systems: The BEGIN benchmark. In Findings of the Association for Computational Linguistics. Association for Computational Linguistics, 1–12.
10. Tanya Goyal and Greg Durrett. 2020. Evaluating factuality in generation with dependency-level entailment. In Proceedings of the 2020 Conference on Empirical Methods in Natural Language Processing: Findings. 3592–3603.
11. Emily Dinan, Stephen Roller, Kurt Shuster, Angela Fan, Michael Auli, and Jason Weston. 2018. Wizard of Wikipedia: Knowledge-powered conversational agents. In Proceedings of the International Conference on Learning Representations.
12. Saadia Gabriel, Asli Celikyilmaz, Rahul Jha, Yejin Choi, and Jianfeng Gao. 2021. GO FIGURE: A meta evaluation of factuality in summarization. In Findings of the Association for Computational Linguistics: ACL-IJCNLP 2021. Association for Computational Linguistics, 478–487.
13. Or Honovich, Leshem Choshen, Roee Aharoni, Ella Neeman, Idan Szpektor, and Omri Abend. 2021. Q2: Evaluating factual consistency in knowledge-grounded dialogues via question generation and question answering. In Proceedings of the 2021 Conference on Empirical Methods in Natural Language Processing. 7856–7870
14. Vectara：让你的LLM应用告别幻觉！：[https://zhuanlan.zhihu.com/p/626544154](https://zhuanlan.zhihu.com/p/626544154)
15. Nayeon Lee, Wei Ping, Peng Xu, Mostofa Patwary, Mohammad Shoeybi, and Bryan Catanzaro. 2022. Factuality enhanced language models for open-ended text generation. arXiv preprint arXiv:2206.04624 (2022).
16. Peng, Baolin, Michel Galley, Pengcheng He, Hao Cheng, Yujia Xie, Yu Hu, Qiuyuan Huang et al. "Check your facts and try again: Improving large language models with external knowledge and automated feedback."arXiv preprint arXiv:2302.12813(2023).
17. annah Rashkin, David Reitter, Gaurav Singh Tomar, and Dipanjan Das. 2021. Increasing faithfulness in knowledgegrounded dialogue with controllable features. In Proceedings of the 59th Annual Meeting of the Association for Computational Linguistics and the 11th International Joint Conference on Natural Language Processing. 704–718.
18. Zeqiu Wu, Michel Galley, Chris Brockett, Yizhe Zhang, Xiang Gao, Chris Quirk, Rik Koncel-Kedziorski, et al. 2021. A controllable model of grounded response generation. In Proceedings of the AAAI Conference on Artificial Intelligence. 14085–14093.
19. Ratish Puduppully, Li Dong, and Mirella Lapata. 2019. Data-to-text generation with content selection and planning. In Proceedings of the AAAI Conference on Artificial Intelligence
20. Yangming Li, Kaisheng Yao, Libo Qin, Wanxiang Che, Xiaolong Li, and Ting Liu. 2020. Slot-consistent NLG for task-oriented dialogue systems with iterative rectification network. In Proceedings of the 58th Annual Meeting of the Association for Computational Linguistics. 97–106.
21. Mohsen Mesgar, Edwin Simpson, and Iryna Gurevych. 2021. Improving factual consistency between a response and persona facts. In Proceedings of the 16th Conference of the European Chapter of the Association for Computational Linguistics. 549–562.
22. Sihao Chen, Fan Zhang, Kazoo Sone, and Dan Roth. 2021. Improving faithfulness in abstractive summarization with contrast candidate generation and selection. In Proceedings of the 2021 Conference of the North American Chapter of the Association for Computational Linguistics: Human Language Technologies (NAACL-HLT’21). 5935–5941.
23. 大模型的幻觉问题调研: LLM Hallucination Survey: [https://zhuanlan.zhihu.com/p/642648601](