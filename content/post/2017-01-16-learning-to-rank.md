---
title: 假新闻引发的愤怒——非算法视角对自我学习的搜索排序算法和选择偏差的一些解读
date: '2017-01-16T14:59:16+00:00'
author: COS编辑部
categories:
  - 统计之都
slug: learning-to-rank
---

本文作者陈丽云，落园园主。

<span style="color: #808080;">声明：本文与作者工作单位及工作内容无关，完全出于个人兴趣爱好。</span>

<img class="aligncenter size-medium wp-image-13573" src="https://cos.name/wp-content/uploads/2017/01/17784079_1200x1000_0-300x168.jpg" alt="" width="300" height="168" srcset="https://cos.name/wp-content/uploads/2017/01/17784079_1200x1000_0-300x168.jpg 300w, https://cos.name/wp-content/uploads/2017/01/17784079_1200x1000_0-768x430.jpg 768w, https://cos.name/wp-content/uploads/2017/01/17784079_1200x1000_0-500x280.jpg 500w, https://cos.name/wp-content/uploads/2017/01/17784079_1200x1000_0.jpg 1200w" sizes="(max-width: 300px) 100vw, 300px" />

最近有条很火的新闻。美国大选刚刚落下帷幕，却余波不断。其中一条新闻就是，Google被指责利用搜索结果（假新闻）左右民意。可是事情到底是怎么回事呢？

<span style="color: #808080;">SAN, FRANCISCO/WASHINGTON – Google’s search engine is highlighting an inaccurate story claiming that President-elect Donald Trump won the popular vote in last week’s election, the latest example of bogus information spread by the internet’s gatekeepers.</span>

<span style="color: #808080;">The incorrect results are shown in a two-day-old story posted on the pro-Trump “70 News” site. On Monday, a link to the site appeared at or near the top of Google’s influential rankings of relevant news stories for searches on the final election results.</span>

原文不翻译了，大意是，在Google搜索大选相关信息的时候，“popularity vote”第一条结果是一个“洋葱新闻”网站70News。显然Google的算法认为这个网站是最相关的，结果无数的网民就天真地点击过去了，然后愤怒地发现这是一条假新闻（相似的例子可能还有百度医疗广告问题&#8230;）。可见人们潜意识里对搜索引擎有一种莫名的信任——排在前面的应该就是我想要的信息。可是，搜索引擎背后也只是一堆堆的机器学习模型，而模型也是需要不断改进的。要改进模型就要告诉模型什么时候判断错了，然后进行参数修正。

<img class="aligncenter size-medium wp-image-13577" src="https://cos.name/wp-content/uploads/2017/01/Google-e1457156368841-300x132.jpg" alt="" width="300" height="132" srcset="https://cos.name/wp-content/uploads/2017/01/Google-e1457156368841-300x132.jpg 300w, https://cos.name/wp-content/uploads/2017/01/Google-e1457156368841.jpg 480w" sizes="(max-width: 300px) 100vw, 300px" />

最近看到Google research放出来的一篇论文：Learning to Rank with Selection Bias in Personal Search（http://research.google.com/pubs/pub45286.html）。这篇论文是跟排序算法相关的，虽然跟上面的“假新闻”事件没啥直接关系，但殊途同归之处不少。正巧园主前些时日涉足了一些相关的问题，加之标题中的选择偏差（selection bias），一下子引起园主的好奇心，遂通读此文。读完之后感觉有些想法很新颖，只是术语习惯等等和园主习惯的方式有所区别，所以打算以一个非算法的视角来解读一下这篇文章，谈谈园主的一些理解。

<!--more-->

<span style="color: #808080;">（注：本文非直接翻译，技术细节建议阅读原文）</span>

&nbsp;<section class="Powered-by-XIUMI V5"> <section class=""> <section class=""> 

### 一些背景：网站是如何评价搜索结果的？</section> </section> </section> <section class="Powered-by-XIUMI V5"> <section class=""> <section class="">

<span style="color: #808080;">（下述内容是园主另加的，熟悉相关做法的读者可以直接跳过）</span></p> </section> <section class="">先说一下背景。Google众所周知，做的就是对各种内容的排序。Google search自然是对各种互联网上的内容的排序（称之为大众搜索，或网页搜索），而其他产品诸如gmail，Google drive等，排序的时候就是针对每个用户自己的内容，故称之为个人搜索（personal search）。这篇文章虽然着重的是后面一种情形下的排序，从园主的角度来看其实也不仅限于后者。互联网各大公司其实都有相同的问题：如何优化站内搜索，比如在淘宝搜索商品。其实，在现在的环境下，大家不仅仅要优化自然搜索算法（基于内容），也要优化付费搜索和自然搜索的关系（毕竟一个网页就那么多内容，多了付费搜索、自然搜索就看不到了）。这其中，很重要的一个业界的衡量指标就是：关联度(Relevance)，说白了就是返回的搜索结果是不是用户想要的。</p> 

关联度的评价办法有不少，其中大家广泛采用的就是——人工评价（human judgement)。人工评价的过程就是雇佣一群人，然后在电脑上给他们陈列搜索语句和对应的结果，让他们逐条打分是不是相关。这种方法虽然简洁有效，问题也是显而易见的：

1）每条数据的获取都是有人工成本的，因为往往受限于成本而只有有限的数据量，只能针对有限的搜索语句进行评价。

2）每个人对于同样的语句和评价标准的理解都是有偏差的，所以人工评价难逃因为评价人本身带来的误差。</section> <section class="">3）人工评价适用于大众搜索，如手机，明星等等，而对于专业领域则显得较为困难，不太可能雇佣到具有专业领域知识的人进行人工评价，因而评价范围受限。</p> 

然而在另外一个很相近的领域，互联网广告，对于广告的排序业界则有一套常见的基于点击率的算法模型。其核心就是，点击率越高的带来的利润越高，故而把此类内容排序提升。这样的算法潜在的一个假设就是，点击实际代表了用户对于内容的一个主观评价，类似于投票，所以点击数据本身就携带了对于算法的评价信息。从这个角度来说，放着大量的用户点击数据不用，即使是对自然搜索的评价都是浪费的。所以，近几年先后出现了很多基于用户点击数据的关联度评价模型。这篇文章也是在此大背景上，针对个人搜索的一些特殊性，开发出来的关联度评价模型。

<span style="color: #808080;">（注：原文表述为一个learning-to-rank的机器学习模型；而从非算法的视角，园主觉得最核心的就是对关联度的定性评价，故此处称之为评价模型，在后文中会详细解释）</span>

<img class="aligncenter size-medium wp-image-13576" src="https://cos.name/wp-content/uploads/2017/01/1-140Q4141610404-300x192.jpg" alt="" width="300" height="192" srcset="https://cos.name/wp-content/uploads/2017/01/1-140Q4141610404-300x192.jpg 300w, https://cos.name/wp-content/uploads/2017/01/1-140Q4141610404-500x320.jpg 500w, https://cos.name/wp-content/uploads/2017/01/1-140Q4141610404.jpg 600w" sizes="(max-width: 300px) 100vw, 300px" />

原文亦列出了个人搜索相比于网页搜索的特殊性：

1） 点击数据的稀疏化：网页搜索的时候，较热门的关键词和结果集往往会有成千上万的点击量数据；而相比而言，个人搜索的时候，搜索的文档对象往往是每个用户独有的（比如一封电子邮件仅属于少数用户），故而给定关键词和结果、对应点击数据就少的可怜了。因此，个人搜索的点击数据呈现高度稀疏化。

2）个人搜索难以进行人工评价：出于隐私保护，真实的个人搜索的数据是不能开放给第三方进行人工评价的，所以通过人工评价能获得结果有限（比如基于标准的文档库）。

&nbsp;</section> <section class=""> <section class="Powered-by-XIUMI V5"> <section class=""> <section class=""> 

### 利用点击数据来评价搜索结果

假如我们是用户，那么在我们搜索的时候面对一系列返回结果，是怎么给出自己的判断的呢？当然，点击行为由很多因素影响，比如用户的个人偏好、搜索的场景等等。但不可忽略的是，我们往往只看排名靠前的几条结果，然后判断哪个更相关值得点进去看看。在本文起始的“假新闻”事件中， 正是因为大量用户过于相信排序在前的结果，所以造成了假新闻被大量点击和阅读。可怕的是，由于用户长期对于搜索引擎的路径依赖，他们会不由自主地相信排在前面尤其是第一位第二位的结果。

用函数的形式表示，则每个返回结果的点击率会取决于如下几个因素：

**某条结果的点击率= function(排序位置，该结果的关联度，搜索语句特征，其他)**

前面说过，最简单的得到关联度的办法就是，利用前文提到的人工评价（human judgement）来直接进行评分。如果不考虑人工评价，我们是否可以确定上述函数里面各因素的系数，从而当数据中给定点击率、排序、搜索语句特征等，就可以直接计算出来关联度呢？

很多童鞋大概都有体会，做数学题的时候，很重要的一个技能就是，“猜”。先猜一个答案出来，然后看看能不能行得通。所以这里，最简单的想法就是我们先忽略其他因素，假设一个简单的函数形式，然后就可以计算求解关联度了。

<span style="color: #3366ff;">（园主问题1：园主不理解为什么通篇他们没有利用任何人工评价的信息来确定函数形式，而是如后文所述直接假设一个简单的相乘关系。难道是因为在personal search情境下，人工评价完全不可行？）</span>

假设我们数据库里有如下的数据（暂时不管其他用户或者搜索相关变量）：

<img class="aligncenter wp-image-13572" src="https://cos.name/wp-content/uploads/2017/01/捕获-1-300x80.png" alt="" width="458" height="122" srcset="https://cos.name/wp-content/uploads/2017/01/捕获-1-300x80.png 300w, https://cos.name/wp-content/uploads/2017/01/捕获-1-768x205.png 768w, https://cos.name/wp-content/uploads/2017/01/捕获-1-500x133.png 500w, https://cos.name/wp-content/uploads/2017/01/捕获-1.png 900w" sizes="(max-width: 458px) 100vw, 458px" /></section> </section> </section> <section class="Powered-by-XIUMI V5"> <section class=""> <section class="">

****某条结果的点击率 = 排序位置的影响*关联度 （对应原文4.1章）****</p> 

从而，我们倒推得到：

**关联度=某条结果的点击率/排序位置的影响**

所以，得到关联度的关键就是，我们知道排序位置的影响（原文称之为“排序位置系数”）。</section> <section class=""> </section> </section> </section> <section class="Powered-by-XIUMI V5"> <section class=""> <section class=""> <section> <section class="Powered-by-XIUMI V5"> <section class=""> <section class=""> 

### 排序位置系数的直接估计</section> </section> </section> <section class="Powered-by-XIUMI V5"> <section class=""> <section class="">排序位置系数其实很大程度上反映了用户的一些行为习惯，如他们潜意识里面对于搜索结果的信任，或者如园主一样就是没耐心、手快。可是换一个场景，比如园主前段时间突然对犹太教和穆斯林的历史感兴趣，然后逛书店的时候想买一本相关的书籍，园主只能一排排的从历史类的书架扫过去，然后通过标题和简介判断哪本书可能是最相关的。这个时候，园主知道书架的排序并不是按照我的嗜好来的，所以我只能一本本从左到右或者从右到左扫视。</p> </section> <section class="">

<img class="aligncenter size-medium wp-image-13574" src="https://cos.name/wp-content/uploads/2017/01/tooopen_sy_173446127669-300x200.jpg" alt="" width="300" height="200" srcset="https://cos.name/wp-content/uploads/2017/01/tooopen_sy_173446127669-300x200.jpg 300w, https://cos.name/wp-content/uploads/2017/01/tooopen_sy_173446127669-768x511.jpg 768w, https://cos.name/wp-content/uploads/2017/01/tooopen_sy_173446127669-500x333.jpg 500w, https://cos.name/wp-content/uploads/2017/01/tooopen_sy_173446127669.jpg 1024w" sizes="(max-width: 300px) 100vw, 300px" />
  
同理，如果为了估计用户的行为习惯的影响，最直接的办法就是做个“乱序”实验。想法很简单，我们只要固定一个因素来检测另一个因素的变化就好了嘛。乱序的情况下，返回的结果跟用户的搜索其实并没有什么关系（或者返回一些基本匹配的结果），所以无论结果本身跟索索语句的关联度如何，其出现在每个位置的概率都一样。这样，大量用户点击行为反映出来的就仅仅是排序位置的影响，从而我们得以排除关联度的干扰，直接估计排序位置系数。原文第5章即利用了这种想法来进行数据训练。</p> 

好了，如果通过“乱序”这个实验，我们测出来排在第一名的影响因素为0.5， 排在第二名的影响因素为0.3，排在第三名的影响因素为0.1，那么我们重新把用排序算法排过的数据呈现给用户，就可以计算出来每条结果的关联度了。不过这种方法显然用户体验非常糟糕，所以实践中仅用于验证，不用来直接做计算。后文会提到一宗巧妙的办法，来利用点击率数据修正排序算法。

&nbsp;</section> </section> </section> <section class="Powered-by-XIUMI V5"> <section class=""> <section class=""> 

<h3 style="text-align: left;">
  让机器自动学习起来：Learning-to-rank 模型
</h3></section> </section> </section> <section class="Powered-by-XIUMI V5"> <section class=""> <section class="">既然上面证实了通过点击数据来计算关联度的可行性，我们接下来就可以让机器自己变得越来越聪明：用户反馈关联度高的，下次往上排；用户反馈关联度低的，下次降下来。这样随着用户反馈越来越多，一定是最关联的结果在最上面。这个过程的实现便是下文要说的Learning-to-rank 模型。</p> 

<h5 style="text-align: center;">
  Learning-to-rank模型的损失函数
</h5>

与大部分机器学习模型一样，Learning-to-rank的核心就是定义一个损失函数。这里想法大概就是，如果一个排序算法把洋葱新闻排到第一位，而火眼金睛的用户点击的是第二位的真新闻，则损失1分。可见，这里并不直接计算关联度本身，而是利用与关联度有关的数据计算损失函数。原文中的损失函数定义为：（加权）加总各种可能的搜索结果从一个打分模型里面得出的分数的损失。具体实践中大家会利用一个特定的损失函数模型，比如原文中3.1章给出的配对型的损失函数(2)，惩罚如果后面的结果比前面的更加关联，其函数大致形如：

<img class="aligncenter wp-image-13575" src="https://cos.name/wp-content/uploads/2017/01/捕获-300x26.png" alt="" width="1742" height="151" srcset="https://cos.name/wp-content/uploads/2017/01/捕获-300x26.png 300w, https://cos.name/wp-content/uploads/2017/01/捕获-768x67.png 768w, https://cos.name/wp-content/uploads/2017/01/捕获-500x44.png 500w" sizes="(max-width: 1742px) 100vw, 1742px" />

<span style="color: #3366ff;">（园主问题2：用户不点击排名第一的结果和不点击排名第100位的结果，显然背后的考量是不同的。他可能看到了第一位的结果，心里嘀咕，八竿子打不着的结果你干嘛给我？而根本没看到第100位的结果。这种情况下，我们不应该惩罚排在第一位的结果吗？）</span>

可惜的是，损失函数(2)对第二种情形并没有进行直接的考量，所以造成了可用训练数据里面的选择性偏差（selection bias）现象，即能用的样本只有用户有一个点击的情况。然而一条搜索语句返回的结果能不能可以进入训练数据集（即是不是至少有一个点击），则取决于每条排序结果是不是关联。简单说来，如果第一位的结果是关联的（好的情况）、用户给予了点击，则搜索语句所有返回结果符合条件、进入训练集；如果第一位的结果是不关联的（坏的情况），就算第二位是关联的，那么用户可能忽略所有搜索结果，没有任何点击，所以不符合进入数据集的条件。而这条数据的价值我们前面已经说过了，明明应该惩罚第一位的结果啊。在作者原文中，特称这种因为排序位置导致可用的训练数据有选择性样本的情况为“position bias”。

这种选择性样本偏差存在的后果就是，Learning-to-rank模型随着时间并不会收敛到越关联的数据排序越高的理想情况，而很大程度上取决于初始化的时候排序打分模型如何。我们显然想得到收敛结果一个跟初始化无关的优化过程（即初始化只影响收敛速度，而不影响收敛结果）。

<span style="color: #3366ff;">（园主问题3：除了园主提到的可以直接惩罚没有任何点击的情况，其实另一种思路就是换一个可以利用聚合（aggregate）过的点击率数据的损失函数。即先计算每条返回结果在每个位置的点击率，然后基于不同结果的点击率计算分数损失，从而可以避免直接把一条条的原始点击数据放入模型训练。不知道原文是出于计算效率的考虑，还是想利用现成的配对型的损失函数(2)。不过后文的逆向加权好像也是类似的思路，只是处理细节上有所不同，不知道算不算异曲同工。）</span>

利用损失函数(2)的话，我们就要面临一个现实的问题：该函数直接利用原始的点击数据进行计算每两两配对之后惩罚不关联的结果。而这个函数很挑食，不是所有数据都可以利用：如果某条搜索返回所有结果都没有点击，那么所有记录都毫无利用价值，故而其影响被忽略。听起来那里怪怪的对不对？

<h5 style="text-align: center;">
  选择性偏差的修正：倾向度的逆向加权 （Inverse Propensity Weighting）
</h5>

为了修正配对型的损失函数(2)的选择性偏差问题，作者也有妙招。他们提出对0点击数据，其实可以利用海量数据的优势进行修正。比如用户a和用户b搜索同样的关键词，返回同样的结果排序。如果一个用户a是0点击，而b有至少一个点击，那么我们就把b的结果复制给a，这样我们就并没有忽略用户a的情况，该关键词依旧会就会被损失模型计算两遍。

实际操作中我们并不需要复制这么麻烦，直接加权即可（亦可以bootstrap重新抽样保证非0子集中关键词的分布和原始数据一致）。原文3.3 章的Inverse Propensity Weighting 正是基于这种思路，利用每个结果在每个位置出现的概率与0点击的概率(即倾向度，Propensity)，逆向加权。我们可以直接计算搜索算法中，给定搜索语句，各个结果出现的概率（作者原文称之为倾向 度 propensity，其实就是一个实证频率，这里和大部分统计中我们理解的propensity score并不是一个概念）。然后计算实际数据给出的非0点击概率，再用两者之比作为权重来加权计算损失函数。

这种思路听起来不错，可是为什么基于倾向度的逆向加权可以修正样本的选择性偏差问题呢？会不会加权过度？作者在第4章证明了，该方法的巧妙之处在于，在简单的点击率模型的函数形式假设下，通过数据计算出来的倾向度反映的正是前面提到的排序位置因素，且该倾向度和排序位置成反比。故而，我们只需要计算倾向度，就可以修正排序位置偏差了。就算我们考虑其他可能影响点击率的因素，将点击率模型进行拓展变复杂，该倾向度也可以进行相应调整。

故而沿用这种思路，作者提出既可以对所有的语句计算一个全局加权权重，亦可以细致到每一个类别。原文在第4章涵盖了三个模型：全局偏差模型 (Global Bias Model)、类别偏差模型 (Segment Bias Model)、广义偏差模型（Generalized Bias Model）。这三者的关系，大概类似于，在一个回归中，只包含我们感兴趣的自变量（关联度、排序），加一个额外的分类控制变量（搜索类别）及其交叉项 （即异质效应 heterogeneous treatment effect），加一堆额外的控制变量及其交叉项（原文5.1章加入的是搜索语句长度变量）。这三个模型给出的结果的区别就是，第一个对所有语句统一计算一个权重w，第二个对每一个搜索类别计算一个不同权重，第三个则进一步细分。

作者在第五章进行了各种线上和线下用户实验来验证各个模型并计算其效果提升，包括我们前面提到的当数据是乱序故而可以直接计算排序位置影响的情况。在此不再赘述。

&nbsp;<section class="Powered-by-XIUMI V5"> <section class=""> <section class=""> 

### 总结</section> </section> </section> 

通读文章几遍之后，园主的理解就是，点击率数据混合了两个因素的影响：排序位置、关联度。我们如果想办法剥离排序位置的影响，那么修正后的点击率数据就单纯的反映了关联度，故而可以直接用来改进排序算法。

回到选择性偏差的问题，园主的理解是其产生是由于损失函数的限制。原文作者给出的解决办法是基于倾向度的逆加权，不知道其他办法是不是亦可行。园主对此领域了解颇为有限，若有贻笑大方之处，还请大家不吝指正。</section> </section> </section> <section class="Powered-by-XIUMI V5"> <section class=""> <section class=""> </section> </section> </section> </section> 

<p style="text-align: right;">
  审稿 | 邱怡轩
</p>

<p style="text-align: right;">
  编辑 | 冯璟烁
</p></section> </section> </section> </section> </section> </section>