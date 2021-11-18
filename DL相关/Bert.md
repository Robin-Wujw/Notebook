# BERT

【背景】ELMo相比word2vec会有这么大的提升，这说明预训练模型的潜力远不止为下游任务提供一份精准的词向量，所以可不可以直接预训练一个龙骨级的模型呢？如果它里面已经充分的描述了字符级、词级、句子级甚至句间关系的特征，那么在不同的NLP任务中，只需要去为任务定制一个非常轻量级的输出层（比如一个单层MLP）就好了。BERT正是做了这件事情。

BERT的全称是Bidirectional Encoder Representation from Transformers，即双向Transformer的Encoder，因为decoder是不能获得要预测的信息的。

模型的主要创新点都在pre-train方法上，即用了Masked LM和Next Sentence Prediction两种方法分别捕捉词语和句子级别的representation。

bert-large模型（24-layer, 1024-hidden, 16-heads），参数准确是340M。
bert-base模型（12-layer, 768-hidden, 12-heads），参数是110M,中文bert也是base版的110M的参https://zhuanlan.zhihu.com/p/91903871

## 1.BERT（Huggingface）代码
based-L-12_H-768_A-12：原生12层Transformer * 768维的中文预训练模型

（12层(深度)，768hidden_size（宽度），12 heads）

bert-base-uncased的模型，uncased和cased的区别在于uncased将全部样本变为小写，而cased则要区分大小写。

BertModel类，它就是BERT模型的基本代码。我们可以看到它的类定义中，由embedding，encoder，pooler组成，forward时顺序经过三个模块，输出output。

BertEmbeddings这个类中，embedding由三种embedding相加得到，经过layernorm 和 dropout后输出。
BertEncoder主要将embedding的输出，逐个经过每一层Bertlayer的处理，得到各层hidden_state，再根据config的参数，来决定最后是否所有的hidden_state都要输出。
Bertpooler 其实就是将BERT的[CLS]的hidden_state 取出，经过一层DNN和Tanh计算后输出。
【过程】读取一个预训练过的BERT模型，来encode我们指定的一个文本，对文本的每一个token生成768维的向量。如果是二分类任务，接下来就可以把第一个token也就是[CLS]的768维向量，接一个linear层，预测出分类的logits，或者根据标签进行训练。

https://zhuanlan.zhihu.com/p/120315111

vocab_size,
hidden_size=768,
num_hidden_layers=12,
num_attention_heads=12,
intermediate_size=3072,
hidden_act="gelu",
hidden_dropout_prob=0.1,
attention_probs_dropout_prob=0.1,
max_position_embeddings=512
{
  "architectures": [
    "BertForMaskedLM"
  ],
  "attention_probs_dropout_prob": 0.1,
  "hidden_act": "gelu",
  "hidden_dropout_prob": 0.1,
  "hidden_size": 768,
  "initializer_range": 0.02,
  "intermediate_size": 3072,
  "max_position_embeddings": 512,
  "num_attention_heads": 12,
  "num_hidden_layers": 12,
  "type_vocab_size": 2,
  "vocab_size": 30522
}

## 2.常考题

### 1、BERT时代，如何处理长文本分类？

由于显存占用和算力的限制，BERT等预训练语言模型的input一般来说最长512个token。某些场景下处理长文本分类，BERT可能还不如CNN效果好。为能让BERT等更适合处理长文本，从「文本处理」和「改进attention机制」两方面给出可尝试方法：

（1）文本处理（优先选择）

固定截断：一般来说，文本的开头和结尾信息量较大，可以按照一定比例对截取出文本的开头和结尾；
随机截断：如果固定截断信息损失较大，可以在DataLoader中每次以不同的随机概率进行截断，这种截断可以让模型看到更多形态的case；
截断&滑窗+预测平均：通过随机截断或者固定滑窗将一个样本切割成多个样本，在预测时对多个样本的结果进行平均；
截断+关键词提取：采取直接截断的方式可能会导致信息量损失，可以通过关键词提取补充信息。如：[CLS][截断文本][SEP][关键词1][SEP][关键词2]...
（2）改进attention机制

Transformer采取的attention机制，其时间复杂度为 O(n*n)，其中为文本长度。最近有paper聚焦于对attention机制的改进、降低计算复杂度，以更适合处理长文本序列。主要包括：Reformer、Linformer、Longformer、Longformer

https://zhuanlan.zhihu.com/p/183852900

### 2、讲讲multi-head attention的具体结构

BERT由12层transformer layer（encoder端）构成

首先word emb , pos emb（可能会被问到有哪几种position embedding的方式，bert是使用的哪种）, segment emb做加和作为网络输入
每层由一个multi-head attention, 一个feed forward 以及两层layerNorm构成
一般会被问到multi-head attention的结构，具体可以描述为，一个768（12*64）的hidden向量，被映射成query， key， value。 然后三个向量分别切分成12个小的64维的向量，每一组小向量之间做attention。
hidden(768) -> query(768) -> 12 x 64
hidden(768) -> key(768) -> 12 x 64
hidden(768) -> val(768) -> 12 x 64
然后query和key之间做attention，得到一个12乘以12的权重矩阵，然后根据这个权重矩阵加权val中切分好的12个64维向量，得到一个12 x 64的向量，拉平输出为768向量。

### 3、wordpiece的作用 https://zhuanlan.zhihu.com/p/198964217（BPE、WordPiece、ULM）

BERT对于英文模型，使用了Wordpiece模型来产生Subword从而减小词表规模；对于中文模型，直接训练基于字的模型。

wordpiece其核心思想是将单词打散为字符，然后根据片段的组合频率，最后单词切分成片段处理。

和原有的分词相比，能够极大的降低OOV的情况，
例如cosplayer, 使用分词的话如果出现频率较低则是UNK，但bpe可以把它切分吃cos play er,
模型可以词根以及前缀等信息，学习到这个词的大致信息，而不是一个OOV。
"Here is some text to encode"

Bert的tokenizer使用了wordpiece 算法，这句话在tokenize了以后是下面这样的
['here', 'is', 'some', 'text', 'to', 'en', '##code']，加上[CLS]和[SEP]就变成了9个token。
wordpiece与BPE(Byte Pair Encoding)算法类似，也是每次从词表中选出两个子词合并成新的子词。与BPE的最大区别在于，如何选择两个子词进行合并：BPE选择频数最高的相邻子词合并，而WordPiece选择能够提升语言模型概率最大的相邻子词加入词表。

https://zhuanlan.zhihu.com/p/151412524

## 3.BERT理论解读

https://mp.weixin.qq.com/s/OsfeAA_tbzAddh1eunwx2w（NewBeeNLP系列）

 BERT，其主要思想是：采用 Transformer 网络作为模型基本结构，在大规模无监督语料上通过掩蔽语言模型和下句预测两个预训练任务进行预训练（Pre-training），得到预训练模型。

再以预训练模型为基础，在下游相关 NLP 任务上进行模型微调（Fine-tuning）。BERT 预训练模型能够充分利用无监督预训练时学习到的语言先验知识，在微调时将其迁移到下游 NLP 任务上。
BERT 模型的模型结构主要由三部分构成：输入层、编码层和任务相关层。

输入层包括词嵌入（token embedding）、位置嵌入（position embedding）段嵌入（segment embedding），并将三者相加得到每个词的输入表示。
编码层直接使用了 Transformer 编码器来编码输入序列的表示。
任务相关层则根据下游任务不同而有所不同，如对于文本分类任务，任务相关层通常为带 softmax 的线性分类器。
通过这两个预训练任务，BERT 模型能够学习到先验的语言知识，并在后面迁移给下游任务。

【MLM】原理是：随机选取输入序列中的一定比例（15%）的词，用掩蔽标记 [MASK] 替换，然后根据双向上下文的词预测这些被掩蔽的词。【思想】如果两个词填在同一个地方没有违和感，那么它们就有类似的embedding。

MLM两个缺点：（1）会造成预训练和微调时的不一致，因为在微调时[MASK]总是不可见的；
（2）由于每个Batch中只有15%的词会被预测，因此模型的收敛速度比起单向的语言模型会慢，训练花费的时间会更长。
对于第一个缺点的解决办法是，把80%需要被替换成[MASK]的词进行替换，10%的随机替换为其他词，10%保留原词。由于Transformer Encoder并不知道哪个词需要被预测，哪个词是被随机替换的，这样就强迫每个词的表达需要参照上下文信息。
对于第二个缺点目前没有有效的解决办法，但是从提升收益的角度来看，付出的代价是值得的。
【NSP】主要目标是：根据输入的两个句子 A 和 B，预测出句子 B 是否是句子 A 的下一个句子。

【任务相关层】

经过预训练的 BERT 模型可以用于下游的自然语言处理任务。在使用时，主要是在预训练 BERT 模型的基础上加入任务相关层，再在特定任务上进行微调（fine-tuning）。
通常，我们取出 BERT 模型最后一层的向量表示，送入任务相关层中，就可以得到任务所要建模的目标概率。
文本分类取出最后一层 [CLS] 标记对应的向量表示，再进行线性变换和 softmax 归一化就可以得到分类概率。
在微调时，BERT 模型和任务相关层的所有参数都一起更新，最优化当前下游任务的损失函数。
【vs ELMo和GPT】





【优缺点】

优点：BERT是截至2018年10月的最新state of the art模型，通过预训练和精调横扫了11项NLP任务，这首先就是最大的优点了。而且它还用的是Transformer，也就是相对rnn更加高效、能捕捉更长距离的依赖。对比起之前的预训练模型，它捕捉到的是真正意义上的bidirectional context信息。

缺点：作者在文中主要提到的就是MLM预训练时的mask问题：

[MASK]标记在实际预测中不会出现，训练时用过多[MASK]影响模型表现
每个batch只有15%的token被预测，所以BERT收敛得比left-to-right模型要慢（它们会预测每个token）

### 3.2. Pre-training

#### 3.2.1 Task 1: Masked LM（真正双向，灵感源于完形填空）

第一步预训练的目标就是做语言模型，从上文模型结构中看到了这个模型的不同，即bidirectional。关于为什么要如此的bidirectional，意思就是如果使用预训练模型处理其他任务，那人们想要的肯定不止某个词左边的信息，而是左右两边的信息。而考虑到这点的模型ELMo只是将left-to-right和right-to-left分别训练拼接起来。

ELMo不足：虽然ELMo有用双向RNN来做encoding，但是这两个方向的RNN其实是分开训练的，只是在最后在loss层做了个简单相加。这样就导致对于每个方向上的单词来说，在被encoding的时候始终是看不到它另一侧的单词的。而显然句子中有的单词的语义会同时依赖于它左右两侧的某些词，仅仅从单方向做encoding是不能描述清楚的。

直觉上来讲我们其实想要一个deeply bidirectional的模型，但是普通的LM又无法做到，因为在训练时可能会“穿越”。这样显然会导致一些小问题。这样虽然可以放心的双向encoding了，但是这样在encoding时把这些盖住的标记也给encoding进去了，而这些mask标记在下游任务中是不存在的呀。。。那怎么办呢？对此，为了尽可能的把模型调教的忽略这些标记的影响，作者通过如下方式来告诉模型“这些是噪声是噪声！靠不住的！忽略它们吧！”。

所以作者用了一个加mask的trick。在训练过程中作者随机mask 15%的token，而不是把像cbow一样把每个词都预测一遍。最终的损失函数只计算被mask掉那个token！Only calculate loss on masked tokens！

Mask如何做也是有技巧的，如果一直用标记[MASK]代替（在实际预测时是碰不到这个标记的）会影响模型，所以随机mask的时候10%的单词会被替代成其他单词，10%的单词不替换，剩下80%才被替换为[MASK]。要注意的是Masked LM预训练阶段模型是不知道真正被mask的是哪个词，所以模型每个词都要关注。

随机mask 15%的token

有80%的概率用“[mask]”标记来替换
有10%的概率用随机采样的一个单词来替换
有10%的概率不做替换（虽然不做替换，但是还是要预测哈）
因为序列长度太大（512）会影响训练速度，所以90%的steps都用seq_len=128训练，余下的10%步数训练512长度的输入。

Encoder
在encoder的选择上，作者并没有用烂大街的bi-lstm，而是使用了可以做的更深、具有更好并行性的Transformer encoder来做。这样每个词位的词都可以无视方向和距离的直接把句子中的每个词都有机会encoding进来。另一方面我主观的感觉Transformer相比lstm更容易免受mask标记的影响，毕竟self-attention的过程完全可以把mask标记针对性的削弱匹配权重，但是lstm中的输入门是如何看待mask标记的那就不得而知了。

 

#### 3.2.2 Task 2: Next Sentence Prediction

因为涉及到QA和NLI之类的任务，增加了第二个预训练任务，目的是让模型理解两个句子之间的联系。Focus on the relationship between sentences. 训练的输入是句子A和B，B有一半的几率是A的下一句，输入这两个句子，模型预测B是不是A的下一句。预训练的时候可以达到97-98%的准确度。

注意：作者特意说了语料的选取很关键，要选用document-level的而不是sentence-level的，这样可以具备抽象连续长序列特征的能力。

学习句子与句对关系表示
像之前说的，在很多任务中，仅仅靠encoding是不足以完成任务的（这个只是学到了一堆token级的特征），还需要捕捉一些句子级的模式，来完成SLI、QA、dialogue等需要句子表示、句间交互与匹配的任务。对此，BERT又引入了另一个极其重要却又极其轻量级的任务，来试图把这种模式也学习到。

句子级负采样
word2vec的一个精髓是引入了一个优雅的负采样任务来学习词向量（word-level representation）嘛。那么如果我们把这个负采样的过程给generalize到sentence-level呢？这便是BERT学习sentence-level representation的关键啦。

BERT这里跟word2vec做法类似，不过构造的是一个句子级的分类任务。即首先给定的一个句子（相当于word2vec中给定context），它下一个句子即为正例（相当于word2vec中的正确词），随机采样一个句子作为负例（相当于word2vec中随机采样的词），然后在该sentence-level上来做二分类（即判断句子是当前句子的下一句还是噪声）。通过这个简单的句子级负采样任务，BERT就可以像word2vec学习词表示那样轻松学到句子表示啦。

句子级表示
BERT这里并没有像下游监督任务中的普遍做法一样，在encoding的基础上再搞个全局池化之类的，它首先在每个sequence（对于句子对任务来说是两个拼起来的句子，对于其他任务来说是一个句子）前面加了一个特殊的token，记为[CLS]，如图



ps：这里的[sep]是句子之间的分隔符，BERT同时支持学习句对的表示，这里是[SEP]便是为了区分句对的切割点。

然后让encoder对[CLS]进行深度encoding，深度encoding的最高隐层即为整个句子/句对的表示啦。这个做法乍一看有点费解，不过别忘了，Transformer是可以无视空间和距离的把全局信息encoding进每个位置的，而[CLS]作为句子/句对的表示是直接跟分类器的输出层连接的，因此其作为梯度反传路径上的“关卡”，当然会想办法学习到分类相关的上层特征啦。

另外，为了让模型能够区分里面的每个词是属于“左句子”还是“右句子”，作者这里引入了“segment embedding”的概念来区分句子。对于句对来说，就用embedding A和embedding B来分别代表左句子和右句子；而对于句子来说，就只有embedding A啦。这个embedding A和B也是随模型训练出来的。

Embedding
所以最终BERT每个token的表示由token原始的词向量token embedding、前文提到的position embedding和这里的segment embedding三部分（对应位）相加而成，如图：



Token Embeddings是词向量，第一个单词是CLS标志，可以用于之后的分类任务
Segment Embeddings用来区别两种句子，因为预训练不光做LM还要做以两个句子为输入的分类任务
Position Embeddings和之前文章中的Transformer不一样，不是三角函数而是学习出来的

### 3.3. Fine-tunning（简洁到过分的下游任务接口）




真正体现出BERT这个模型是龙骨级模型而不再是词向量的，就是其到各个下游任务的接口设计了，或者换个更洋气的词叫迁移策略。
首先，既然句子和句子对的上层表示都得到了，那么当然对于文本分类任务和文本匹配任务（文本匹配其实也是一种文本分类任务，只不过输入是文本对）来说，只需要用得到的表示（即encoder在[CLS]词位的顶层输出）加上一层MLP就好了呀～

既然文本都被深度双向encoding了，那么做序列标注任务就只需要加softmax输出层就好了呀，连CRF都不用了呀～

在span抽取式任务如SQuAD上，把深度encoding和深度attention这俩大礼包省掉就算了，甚至都敢直接把输出层的pointer net给丢掉了？直接像DrQA那样傲娇的用两个线性分类器分别输出span的起点和终点？不多说了，已跪

可以调整的参数和取值范围有：

Batch size: 16, 32
Learning rate (Adam): 5e-5, 3e-5, 2e-5
Number of epochs: 3, 4
因为大部分参数都和预训练时一样，精调会快一些，所以作者推荐多试一些参数。

## 参考资料

1.Google BERT详解：https://zhuanlan.zhihu.com/p/46652512

2.NLP的游戏规则从此改写？从word2vec, ELMo到BERT: https://zhuanlan.zhihu.com/p/47488095

3.从Word Embedding到Bert模型—自然语言处理中的预训练技术发展史：https://zhuanlan.zhihu.com/p/49271699

4.https://blog.csdn.net/AZRRR/article/details/90705953