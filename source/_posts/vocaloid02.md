---
title: 音乐合成：VOCALOID 原理以及更多
top: false
cover: /img/miku02.jpg
toc: true
mathjax: true
tags:
  - VOCALOID
categories: 音乐合成
abbrlink: 132c8286
date:
password:
summary:
---
写在前面

我认为音乐合成，或者说语音合成，需要一定的语音学基础，属于类似声音研究院这样的机构的研究范畴。最近在读《简明语言哲学》，我也对“语言”这一概念有更多的认识，但是想要形成自己对于音乐研究的基石，还需要大量的声学研究、语音学研究作为支撑。语音合成的方法论，有一些原子论、语音分析的样子，不知是否合理。

同时，读了一些关于 VOCALOID 原理的综述，以及相关的论文之后，觉得自己之前的想法太过天真，也过于“重复造轮子”了。之前思考的隐马尔可夫模型已经被用在 CeVIO 上。将 AI 技术用于语音合成也不是这几年才兴起的。同时，所谓的“语音合成软件”也远不止 VOCALOID、utau。Synth，CeVIO，也不是这两年才突然冒出来的，国内也在很久以前就有“袅袅”这样的尝试，另外，微软小冰等等非歌唱类，而偏向日常说话的语音合成软件也有很多了。

在我 2012 年左右刚知晓 VOCALOID 的时候，已经有国内大神开始制作自己的歌声合成软件了。不如说 2022 当下这样的应用类的软件多到让我有点迷失方向。我认为应该把我对声音合成研究“理想”放在基础声音合成建模以“模仿→模拟→创造”音色，以及通过人工智能学习唱法这两点上。也许目前也有对应的深入研究了。应该先尽可能多地了解现有应用的原理，争取站在巨人的肩膀上。

> 前文：[关于音乐合成的朴素分析](https://left-fdu.github.io/article/e645166c.html)

---

## High->Mid->Low

西班牙庞贝法布拉大学的研究组 MTG（Music Technology Group），在 90 年代末和 Yamaha 合作开发 VOCALOID。在 Yamaha 的投资下进行了一系列有关歌声频谱建模的研究并发表若干论文。

> MTG 把音频的表示分为三个方面，Low Level-声音底层参数的层面, Mid Level-语音学的层面, High Level-乐谱、歌词等更接近演奏者（说话人）的层面。大致上VOCALOID就是这么一个 High->Mid->Low 一层层下来转换合成的应用。
> VOCALOID 的引擎基于拼接合成（Concatenative Synthesis），即说话人的采样（经过处理）成为音源库的成分。拼接合成发源于上世纪 80 年代，优点是还原性好，合成质量高；缺点是数据库往往比较庞大。

### Low level 建模
VOCALOID 使用 SMS（Special Modelling Synthesis）技术对语音进行底层建模，合成则使用了 VPM（Voice Pulse Model）。

SMS 是 MTG 在 1989 年提出的技术，它在正弦模型的基础上增加了随机（Stochastic）成分，在语音分析中把语音拆分成若干不同频率和幅度的正弦波和气音，气音相当于通过声道滤波器的噪音。合成时生成若干正弦波并和气音叠加。SMS 也被称作 HNM（Hamonic and Noise Model）、HpN（Harmonic plus Noise）。语音分析阶段以傅里叶变换为基础，所以真要从头开始做语音合成，恐怕要从数学开始恶补了。

VPM 比 SMS 合成的速度更快，它直接在频域生成语音的短时频谱，且能够对语音的声门脉冲建模，直接控制相位。WBVPM（Wide-Band Voice Pulse Model）技术将短时频谱叠加生成最终的语音，并可在同一频谱中表示出语音的正弦成分（正弦波）和随机成分（气音）。

### Mid level 建模

通过 SMS 和 VPM，只要我们知道每一时刻语音各个谐波的频率和幅度，以及气音的频谱形状，就能完美合成出语音。在 Mid level，这些参数由 EpR（Excitation plus Resonance）语音模型产生。它把语音的频谱包络视作一条激励曲线和一条共振曲线的和。其中共振曲线是由好几个单独的共振峰曲线叠加起来的，每个共振峰的频谱形状由幅度、中心频率和带宽三个参数决定。

VOCALOID 的音源库包含了大量的 EpR 参数，通过在合成中修改这些参数可以实现时间缩放、音高变换、发音过渡、音色修改等效果。

### High level 建模

High level 考虑的是如何把用户输入的乐谱转化为 EpR 参数。VOCALOID 使用了一个名为 Sonic Space 的参数生成器，它是一个包含了High level 和 Mid level 样本的数据库，能够通过某种算法在两个层次之间进行匹配。

Jordi Bonada博士（MTG 的研究负责人）2008 年的论文里展望到，(VOCALOID)未来可能会使用 SVM（支持向量机）, ANN（人工神经网络）, GMM（高斯混合模型）, HMM（隐马尔科夫模型）等模型进行高阶建模。Jordi 还说，他们认为 HMM 模型可以直接架起从 Low Level 到 High Level 的桥梁。（可惜这提早被 HTS 实现了，现在被 CeVIO 使用）

## CeVIO、HMM 以及 Synthesizer V

上文提到 CeVIO，它使用的是名古屋工业大学开发的 HTS Engine（HMM-based Speech Synthesis System）。它可以直接跳过 Mid Level，合成的语音会有更好的韵律、节奏和真人发声习惯。相比之下 VOCALOID 是另一个极端：把一切能建模的建模了，通过精确的参数求得高质量的语音。从目前的软件操作上来看， 用户在 CeVIO 输入旋律和歌词后，可以得到听感尚可的歌声，而若是使用 VOCALOID，合成效果会更好，但需要细心地调整参数，才能实现更惊艳的演唱。

Synthesizer V 是由 SleepWalking 开发的中文歌声合成软件（开头提到的国内大神就是他）。SV 既有效仿 VOCALOID 模式的专业声库，也有自研的 AI 声库，可以说占据国内歌声合成市场的半壁江山，希望它能集众家所长，既能借助机器学习生成无参情况下就很优秀的歌声，又能通过调整参数达到创作者的特殊要求。

另外，跳出歌声合成，站在更广的语音合成的视角来看，TTS 语音朗读已经是较为成熟的应用了。还有微软小冰这样以假乱真的存在。以上提到的歌声合成只是语音合成在特定情境下的一种思路，希望未来歌声合成的门槛能够进一步降低，希望音乐创作可以更加自由。

## 参考文献
- Serra, X. 1989. "A System for Sound Analysis/Transformation/Synthesis based on a Deterministic plus Stochastic Decomposition" Ph.D. Thesis. Stanford University.
- Bonada, Jordi, et al. "Singing voice synthesis combining excitation plus resonance and sinusoidal plus residual models." Proceedings of International Computer Music Conference. 2001.
- Sanjaume, Jordi Bonada. Voice processing and synthesis by performance sampling and spectral models. Diss. Universitat Pompeu Fabra, 2008.
- Klatt, Dennis H. "Software for a cascade/parallel formant synthesizer." the Journal of the Acoustical Society of America 67.3 (1980): 971-995.
- McAulay, Robert, and Thomas F. Quatieri. "Speech analysis/synthesis based on a sinusoidal representation." Acoustics, Speech and Signal Processing, IEEE Transactions on 34.4 (1986): 744-754.
- Quatieri, Thomas F., and R. McAulay. "Phase coherence in speech reconstruction for enhancement and coding applications." Acoustics, Speech, and Signal Processing, 1989. ICASSP-89., 1989 International Conference on. IEEE, 1989.
- SleepWalking . 2019. “从初音ミク到歌声合成” https://github.com/Sleepwalking/prometheus-spark/blob/master/writings/from-miku-to-svs/from_miku_to_svs.md