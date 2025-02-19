# research_ideas
没空做的IDEA，放在这躺一躺
## 宽视野的图神经网络
GCN有一个非常坏的特性，就是视野的局限性，一个节点要与多跳外的节点产生信息交互，就必须经过多次GCN计算。当然CNN的计算核都是这副德行，GCN也免不了俗。

有没有具备全局视野，在图网络的问题里，也可以是具备所有可达节点视角的信息交互网络呢？

当然可以！一种解决思路是直接把attention搬到图网络里（不是GAT的那个，那玩意儿，难评），但是attention对所有节点都是一视同仁的，没有考虑到图上最关键的拓扑信息。另一种思路就是将远跳节点的信息按距离衰减再与当前节点进行信息交互，当然为了数据分布的稳定性，所有可达节点衰减相加后的权重需要乘以一个平衡系数，最好是个好算的平衡参数。

那么有没有这种衰减数列呢？还真有，而且好算，那就是矩阵指数。n跳邻居在邻接矩阵的表达式上就是`A^n`，可达节点可以通过`A^1+A^2+...+A^n`算出，而矩阵指数的定义如下：

![image](https://github.com/user-attachments/assets/35b29934-e8ea-44e6-8e96-e3364536ac2b)

因此，将`GCN=σ(D(A+I)D^-1HW)`改成`σ(D(exp(A)+I)D^-1HW)`,整个图神经网络就具备全局视野与局部聚焦的特性啦！而写exp(A)是一个常值，在网络拓扑固定之时就可以被计算出，因此并不会影响到网络的前向后向计算效率。

这个idea是22年想到的，应该会很快被人发成实验叭。

## 神经网络训练优化
人类有两种学习方式，短期学习和长期学习。反复的短期学习会逐渐固化为长期学习。包含记忆的神经网络是否可以通过长短期记忆学习的方式减轻训练负担呢？

当前的记忆神经元前向流程可以抽象为`σ(M(H+X)+B)=H'`，其中σ是激活函数，M和B是模型参数，H是代表记忆的状态向量，X是输入向量。显然M和B代表的是长期学习，而H是短期学习，问题在于H可以固化到M和B中吗？

结论是天然的，在传统的训练过程中，每一个H都首先被计算出来，而后通过δH/δM和δH/δB训练模型。问题在于，并不是所有的H都需要被固化到M和B中，人类大部分的短期状态都会被遗忘掉，而不是平等的固化为长期习惯或知识。换算到神经网络训练中，真的每一个token对应的H都值得被计算loss、梯度和训练吗？

反向计算梯度的耗时远大于前向计算，如果可以筛选出那些有用的H，有用的短时记忆，再只使用他们或者更大权重地使用他们训练模型，是不是可以缩减梯度计算的次数呢？

有用的H计算是简单的，对于一个文章，划分为句子，每一个句号结束的H都是一个阅读阶段的结束，他们的重要性大于句子中央的H，对于所有句号处的H，我们可以使用数据统计的方式、聚类的方式，计算出那些最常见的H，这些最常见的H就是最频繁出现的短时记忆，只用这些H训练模型就好了。其目的就在于，让模型更加容易地进入这些状态H，从人的角度类比，就是让人更快回想起当初的状态。

## 恶意软件基因
软件基因这个概念，是个为了创新而创新的生造词。这就导致很多人在说软件基因，但没有人知道软件基因到底是什么。从实用的角度出发，搞基因就是为了溯源，这个溯源不是说直接从某某字符串某某特征直接溯源到某一组织某一攻击者身上，而应该是溯源到父源样本。也就是说，搞基因真正有出路的，还是提取样本的一大堆特征，然后向前去搜索与他高重合度的旧样本，至于所谓的组织者归因，还是要通过父源样本的归类来进行推测。那么问题来了，这和以前的样本家族分类有什么区别？

区别就是对杂交的识别，家族分类在人类基因组里面也是一个很常见的课题，样本基因和样本家族不应该是两个独立的课题，从文字意义上来说就不应该，检测基因是检测家族的泛化。不同的地方在于，家族分类我们说这个样本属于什么家族，置信度多少多少，而软件基因溯源我们可以说这个样本与哪些家族一脉相承，他的这一片段与哪个家族高度重合，那个片段与哪个家族高度重合，这些高度重合的片段在整个恶意软件生态中是否有特异性，是否是某一类家族独有的，还是每个恶意样本都会这么做的。

我觉得这才是做恶意软件基因的意义。

## 大模型反汇编
当我们需要大模型做推理问题时，子任务拆分+reward构成了思维链的关键要素。通常我们的做法是在提问与微调回答中构建prompt让大模型遵照该子任务拆分进行回答，而后通过reward反馈大模型（进一步提问或微调训练）让其改进答案。问题在于，在反汇编领域子任务该如何拆分？

我们来看千问的回答：
```
Q：如果我要给你一段汇编代码，要求你将他翻译成伪C代码。你会按照什么思维链进行思考？
A：
1. 阅读并理解汇编代码
识别功能：首先，尝试理解这段汇编代码的整体功能是什么。这可能涉及到查看函数入口点、调用惯例、参数传递方式等。
识别数据结构：了解代码中使用的数据结构，比如数组、结构体、指针等。
理解控制流：识别代码中的条件分支、循环等控制结构。
2. 分析汇编指令
寄存器映射：确定哪些寄存器用于存储变量、地址或是临时值。
内存操作：分析加载（load）、存储（store）指令，理解它们如何与内存中的数据交互。
算术逻辑操作：识别加法、减法、位操作等指令，并考虑它们在C语言中的对应表达。
控制转移：理解跳转（jump）、调用（call）、返回（return）等指令的作用。
3. 构建C语言结构
变量声明：基于汇编代码中使用的寄存器和内存位置，决定在C语言中如何声明这些变量。
函数定义：根据汇编代码中的函数调用和返回点，定义相应的C函数。
逻辑重构：将汇编代码的逻辑重构成C语言中的if语句、while循环等。
4. 编写伪C代码
逐行翻译：将每一行汇编代码翻译成等效的C语言表达式或语句。
优化表达：利用C语言的高级特性简化代码，提高可读性。
注释解释：为复杂的转换添加注释，说明原始汇编代码的行为和翻译的理由。
5. 测试和验证
功能验证：确保翻译后的C代码实现了与原始汇编代码相同的功能。
边界条件检查：测试各种边界条件，确保C代码的健壮性。
性能对比：如果有必要，可以比较汇编代码和C代码在性能上的差异，特别是在循环和递归等密集计算场景下。
```

整理一下，反汇编的核心要素在于三点：数据结构识别、代码结构识别、一致性校验。
我们同样可以模拟传统反汇编器的操作原理，首先对每条汇编语句进行逐行或逐块翻译，而后通过代码优化技术不断使其贴近伪C代码。
实践中大模型在回答反汇编问题中可以返回伪C代码，但是总是存在遗漏、函数名修改等问题，逐行翻译可能是解决这一问题的有效途径。


## 大模型赋能搜索与广告的隐患
现在搜索工作都交给大模型搜索了，一大原因就是直接搜索的效率太低，且易受到广告的干扰，相较而言我们似乎更相信大模型的总结回答，甚至我们会期待大模型直接基于他搜索到的网页进行决策建议。

然而大模型搜索引擎同样可以接入广告，而且其广告成分更隐蔽、更难以察觉。

在使用deepseek联网搜索的过程中能够明确感受到搜索结果对于回答的影响，大模型对于搜索结果的信任度是极高的，这意味着一旦广告算法进入背后的搜索引擎，大模型会极度信任广告的宣传并主动以貌似客观的语言对用户进行决策建议。而因为鲜有用户真正打开搜索到的数十个搜索结果进行鉴别查看，这种决策建议比起以往的广告搜索更可能被用户所接受。这种商业模式似乎还未出现，但对我来说已有预见。

另一方面，即便没有广告算法的参杂，虚假搜索与类似回声谷的效应也不容忽视，我就见过以论坛内容作为回答依据，甚至论坛内容本身就是发帖者与大模型进行问答的记录分享的情况，这种不可信的搜索结果显然是不值得大模型进行完全信任地总结回答的。因此事实审查也将是未来大模型+搜索必不可少的一环，我们不能让大模型完全信任搜索结果从而误导用户，而要让大模型学会质疑。
