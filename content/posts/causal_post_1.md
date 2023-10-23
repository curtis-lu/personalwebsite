---
title: "如何用機器學習做因果推論？先來瞭解什麼是增益模型與平均因果效應"
date: 2023-08-03T23:59:34+08:00
weight: 3
draft: false
ShowToc: true
TocOpen: true
math: true
enableEmoji: true
cover:
    image: img/causal_ml_1.png
    alt: '增益模型與平均因果效益'
    caption: ''
tags: ['causal inference', 'machine learning']
categories: ['knowledge']
series: ['causal inference for the brave and true']
---

## 背景介紹：預測 vs 因果推論

近期機器學習的領域中，有關因果推論的討論與商業應用越來越多。一般而言，機器學習擅長的是預測問題（監督式機器學習）。

然而，許多商業問題其實是因果推論的問題，例如：如果給某些客戶一個促銷優惠，該些客戶會增加多少購買金額？提高售價會增加或減少銷售額？銀行該如何決定給客戶多少額度和利率來最大化利潤？這些問題不再是只要把所有特徵丟到模型裡就能得到解答的問題，而必須經過一些「實驗設計」，並理解「反事實推論」的概念。

用一個剛開始研究因果推論時，大家常常會舉的例子來說明：
若你要投放廣告，根據過去資料計算出兩個客戶群的廣告轉換率，你會決定向用戶群A還是用戶群B投放？


|         | 廣告轉換率 |
|:-------:|:----------:|
| 用戶群A |     2%     |
| 用戶群B |    0.8%    |


一般來說，若人數與客單價都差不多的情況下，選擇以廣告轉換率較高的用戶群A來投放廣告非常合理。

事實上，考慮自然轉換率（也就是不投放廣告的情況下，本來就會購買的機率）之後，會發現投放廣告的效益是在用戶群B比較高：

|         | 廣告轉換率 | 自然轉換率 | 增益 |
|:-------:|:----------:|:----------:|:----:|
| 用戶群A |     2%     |    1.7%    | 0.3% |
| 用戶群B |    0.8%    |    0.2%    | 0.6% |

也就是説，如果單純只看「廣告投放」後的「購買機率」並不能區分出以下4種客群：


|                    | **有廣告時購買** | **有廣告時不購買** |
|--------------------|------------------|--------------------|
| **無廣告時不購買** | Persuadable     | Lost Causes        |
| **無廣告時購買**   | Sure things      | Sleeping dogs      |


* **Persuadable**: 這些人沒看到廣告時不會購買，看到廣告時就會購買，也就是我們廣告最希望投放到的對象！
* **Sure things**: 這些人無論有沒有看廣告，本來就都會購買。所以廣告投給這些客群其實是一種資源的浪費。
* **Lost causes**: 這些人無論有沒有看廣告，都不會購買。可以說是所謂的靜止戶，若沒有其他更有效促動的方法，單純放廣告給他們也是一種浪費。
* **Sleeping dogs**: 這些人比較特別，沒有看到廣告時會購買，但看到廣告時反而不購買。所以廣告投放到這些人的話，反而會造成反效果。

從以上可知，用戶群A的客戶當中，自然轉換率比較高，可能屬於Sure thing的比較多，無論有沒有廣告其實都會購買，因此廣告投放的成本也就浪費掉了。而用戶群B的客戶當中，可能屬於Persuadable的客戶較多，所以廣告投放的確展現出促使客戶購買的效果。

用機器學習的語言來說，廣告投放稱作一種intervention或treatment，而購買率則稱作response。這種將4種客群混淆在一起，只看「有intervention」（＝有廣告投放）下的「response」（＝購買率），稱作response model。即只看有被廣告投放的客戶的購買率：

$$ P(response|intervention) $$

如果是以Uplift model （增益模型）的概念出發，我們關心的是，對同一個人「有intervention」與「沒有intervention」相比，「response」的變化是多少。這種情況下的response變化，就是intervention的因果效應。即有廣告投放與沒有廣告投放相比，到底對購買率影響多少：

$$ P(response|intervention) - P(response|no intervention) $$

上述的數學表示觸及了因果推論一個很重要的概念：「反事實推論」。之所以稱作「反事實」，是因為在某一個時間點，「同一個人」不是有被廣告投放，就是沒有被廣告投放，而我們只能觀察到其中一個。Uplift model 就是運用一些機器學習方法，搭配實驗設計的概念，來試圖做到反事實推論。

以上僅對uplift model 做概念上的說明，有關uplift model的實際應用會留待後續的文章再做介紹。
有興趣的朋友可以先參考以下資源，看看台灣的Line團隊、以及Uber如何使用增益模型：

*Line分享使用「增益模型」來找出對廣告較有反應的受眾：*

https://www.ithome.com.tw/news/149109

*uber的資料科學家介紹「增益模型」的基本概念與專案應用：*

{{< youtube 2J9j7peWQgI >}}

---

下一部分的內容主要參考的資源來自[Causal Inference for The Brave and True](https://matheusfacure.github.io/python-causality-handbook/landing-page.html)，由巴西Nubank的 Staff Data Scientist [Matheus Facure](https://www.linkedin.com/in/matheus-facure-7b0099117/)所寫，該書整理了許多學者的最新研究，敘述很淺顯易懂（迷因圖也超多），是一本很棒的入門書，推薦給大家。我的相關系列文可以說是擷取某些章節所做的筆記。

- 本篇：因果推論基本概念。
- 第二篇：隨機試驗、信賴區間、因果圖模型。
- 第三篇：傾向分數與Doubly Robust Estimation。
- 第四篇：Meta Learners: S-learner, T-learner, and X-learner。
- 第五篇：Debiased/Orthogonal Machine Learning or R-learner。


## 因果推論概念簡介
有學過regression model的朋友，相信常常聽到「相關不等於因果」這句話，本篇內容一言以蔽之就是在詳細說明這句話，並試圖「讓相關轉變為因果」。

### 數學符號
對unit i 的 treatment 或 intervention表示如下：


$$
T_{i} = \begin{cases}
   1 &\text{if unit i received the treatment} \\\\
   0 &\text{otherwise}
\end{cases}
$$

接下來有關potential outcome比較複雜一點，需要思考一下。同一個unit有可能會接收treatment，也有可能不會，在同一個時間點下，我們能觀察到的，只有這兩種可能的「其中一種」。而potential outcome指的就是，在邏輯上，該unit在「另一種情況下」的可能結果，而這是我們無法觀察的，因為實際上根本沒有發生：

\\( Y_{0i} \\) is the potential outcome for unit i without the treatment

\\( Y_{1i} \\) is the potential outcome for the same unit i with the treatment

舉個例子來說明，當你對以前做了某Ａ決定(\\(T_{i}\\) = 1)，得到了現在這個結果(\\(Y_{1i}\\))。若你對做了A決定感到後悔的話，可能會想，「若是」當初沒有做的話，那現在結果可能就會是怎樣怎樣，這個可能的結果就是potential outcome \\(Y_{0i}\\)。

反過來說，如果你當初放棄了機會Ｂ(\\(T_{i}\\) = 0)，所以現在的結果是(\\(Y_{0i}\\))。若你對沒有好好把握機會B感到後悔，可能會想，「若是」當初有把握的話，現在可能結果會是怎樣怎樣，這個可能的結果就是potential outcome \\(Y_{1i}\\)。

### 因果效應

因果效應可以分為以下幾種：

#### *個別因果效應(individual treatment effect)*

個別因果效應(individual treatment effect)就是利用potential outcome的概念來表示：

$$ Y_{1i} - Y_{0i} $$

從上一段的說明可以知道，我們實際可以觀察到的只有這兩個的其中一個，這個例子只是在概念上去表示因果效應。所以也就稱為「反事實（counterfactual ）」。

#### *平均因果效應(average treatment effect, ATE)*

當綜合某個群體，平均來看的因果效應，就是所謂的平均因果效應(average treatment effect, ATE)：

$$ ATE = \Epsilon[Y_{1} - Y_{0}] $$

#### *受測群體平均因果效應（average treatment effect on the treated, ATT）*

另一個跟ATE很像，可以說是只關注在受測群體的ATE：

$$ ATT =  \Epsilon[Y_{1} - Y_{0} | T=1] $$

#### *條件平均因果效應(Conditional Average Treatment Effect, CATE)*

這一種因果效應是指，當考慮各種特徵後，對具備相同特徵的群體中的平均因果效應。例如我們想要做到的是個人化的廣告投放，因此想要知道廣告對「哪類人」是比較有效果的（即具備哪些特徵的人，他們對廣告的CATE比較強），我們就可以把廣告資源做更有效的運用。CATE的數學表示式如下：

當treatment為二分類變數： \\( \Epsilon[Y_{1} - Y_{0} | X] \\)

當treatment為連續型變數： \\( \Epsilon[y^{\prime}(t)| X] \\)

這一種因果效應跟機器學習在因果推論領域的發展有很大關係，許多應用的目的即是為了利用機器學習的強大預測能力來估計CATE。

### 關聯、因果與偏差

關聯(association)可以理解為，平均來說，當T變化時Y跟著變化的程度。數學表示如下：

$$ \Epsilon[Y|T=1] - \Epsilon[Y|T=0] $$

注意到這邊的Y只有觀察到的部分，因此可以轉換如下（省略i）：

$$ = \Epsilon[Y_{1}|T=1] - \Epsilon[Y_{0}|T=0] $$

接著要使用一些小技巧，導入反事實的概念，加上\\(\Epsilon[Y_{0}|T=1]\\)再減去\\(\Epsilon[Y_{0}|T=1]\\)：

$$ = \Epsilon[Y_{1}|T=1] - \Epsilon[Y_{0}|T=0] + \Epsilon[Y_{0}|T=1] - \Epsilon[Y_{0}|T=1] $$

順序重整一下可得：

$$ = \Epsilon[Y_{1}|T=1] - \Epsilon[Y_{0}|T=1] + \\{ \Epsilon[Y_{0}|T=1] - \Epsilon[Y_{0}|T=0] \\} $$

最後，整併後得到如下：

$$ = \overbrace{\Epsilon[Y_{1} - Y_{0}|T=1]}^{ATT} + 
\overbrace{\\{ \Epsilon[Y_{0}|T=1] - \Epsilon[Y_{0}|T=0] \\}}^{BIAS} $$

由上述推論可知，「關聯」事實上等於「因果」加上「偏差」項。為什麼稱作偏差？需要先理解 \\( \Epsilon[Y_{0}|T=1] \\)，它是「反事實」的結果，代表受測群體「若是」沒有受測時的狀態，而 E\\( \Epsilon[Y_{0}|T=0] \\)代表非受測群體的狀態。這兩者相減，代表的意義是，受測群體跟非受測群體在施測之前，本來就有的差異。

用一個例子來說明，若我們觀察到：高警力的城市，犯罪較多。代表比較多的警察導致高犯罪嗎？先不論黑白兩道勾結的可能性，我們觀察到這個現象的原因很可能是，有高警力的城市，在高警力進駐之前，就已經有比較高的犯罪率（所以才加派警力！）。這導致我們用觀察到的關聯性來試圖做因果宣稱時，被偏差所影響，反而做出相反的結論。

反過來說，若我們知道偏差不存在時，即 \\( \Epsilon[Y_{0}|T=1] - \Epsilon[Y_{0}|T=0] = 0 \\)，那麽，我們可以得到：

$$ = \Epsilon[Y_{1}|T=1] - \Epsilon[Y_{0}|T=0] = \overbrace{\Epsilon[Y_{1} - Y_{0}|T=1]}^{ATT} $$

事實上，若偏差不存在，代表受測群體跟非受測群體是非常相似的，他們的區別只在於是否受測（即T本身），那麽因果效應在兩個群體也會是非常相似的。因此，若滿足此一條件，可以得到：\\( \Epsilon[Y_{1} - Y_{0}|T=1] = \Epsilon[Y_{1} - Y_{0}|T=0]\\)，換言之：

$$ = \Epsilon[Y_{1}|T=1] - \Epsilon[Y_{0}|T=0] = \overbrace{\Epsilon[Y_{1} - Y_{0}|T=1]}^{ATT} = \overbrace{\Epsilon[Y_{1} - Y_{0}|]}^{ATE} $$

也就是說，如果偏差不存在，則「關聯」等於「因果」！

## 結語
本篇首先簡述了response model跟uplift model的區別，帶到了因果效應必須考慮到反事實的概念。接著簡單介紹了各種因果效應的名詞及數學表示法，最後闡述了關連、因果以及偏差的關係。說明了在怎麼樣的條件下，關聯性是可以轉變成因果關係的。

