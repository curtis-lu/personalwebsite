---
title: "如何使用 plotly + streamlit 製作資料視覺化網頁？"
date: 2023-10-09T17:56:46+08:00
draft: false
weight: 2
ShowToc: true
TocOpen: true
cover:
    image: img/streamlit_1.png
    alt: 'streamlit 資料視覺化應用'
    caption: ''
tags: ['data visualization', 'data analysis']
categories: ['projects']
series: ['streamlit app']
keywords:
    - streamlit
    - plotly
    - data visualization
    - data analysis
    - 房地產資料分析
    - 房地產資料視覺化
    - 不動產資料分析
    - 不動產資料視覺化
---

# 如何使用 plotly + streamlit 製作資料視覺化網頁？

## 什麼是 plotly？

plotly是python的繪圖工具，傳統python的繪圖工具有matplotlib, seaborn等等。

matplotlib雖然功能強大，但說實在語法結構頗複雜，要能上手有些難度。
seaborn基本上是搭建在matplotlib上，語法相比matplotlib更簡潔一些，預設的風格也比較好看一點。

plotly個人覺得比seaborn又更容易上手，語法結構非常清楚易懂，官方文件也寫得非常好，開發上可以更快速，

另外，plotly的圖表還有很強的互動性，例如游標移過去資料點上可以顯示資訊、可以選取資料範圍更細部查看資料點等等。這點也是seaborn所沒有的。

plotly的官方文件可以參考 {{< a href="https://plotly.com/python/" target="_blank" >}}這裡{{< /a >}}。


## 什麼是streamlit？

{{< a href="https://github.com/streamlit/streamlit" target="_blank" >}} streamlit {{< /a >}}
是一個建立互動式資料視覺化網頁應用的工具，通常主要目的是製作各種儀表板，功能類似 Power BI 以及 Tabeleau。

streamlit 特點是單純使用python就能製作資料視覺化網頁app的工具，不需要前端的程式技能如CSS和HTML也能快速實作。

相比Power BI以及Tabeleau需要許多拖拉點選的人工操作，streamlit單純使用python，對python使用者而言，私心覺得開發及維護成本比較低。

類似的工具還有
{{< a href="https://github.com/plotly/dash" target="_blank" >}} plotly Dash {{< /a >}}
以及
{{< a href="https://github.com/bokeh/bokeh" target="_blank" >}} bokeh {{< /a >}}
。

但其實streamlit已經跳脫 dashboarding 工具的範圍了，有各式各樣的網頁應用：

* {{< a href="https://cheat-sheet.streamlit.app/" target="_blank" >}} streamlit code cheat sheet {{< /a >}}

* {{< a href="https://bgremoval.streamlit.app/" target="_blank" >}} 為圖像去背的程式 {{< /a >}}

* {{< a href="https://tweets.streamlit.app/" target="_blank" >}} 推文產生器 {{< /a >}}

更多應用可以參考 {{< a href="https://streamlit.io/gallery" target="_blank" >}} 這裡。 {{< /a >}}


## plotly + streamlit 基本操作

直接先來看看streamlit的基本功能。

安裝好plotly 和 streamlit套件後，新建立一個名稱為：`app.py` 的檔案，輸入以下語法，然後存檔。

{{< highlight html >}}
import streamlit as st
import pandas as pd
import numpy as np
import plotly.express as px

st.write('This is a scatter plot: ')

chart_data = pd.DataFrame({'x': np.random.randn(20),
                           'y': np.random.randn(20),
                           'category': np.random.choice(['a','b', 'c'], 20),
                           'size': np.random.choice([1, 3, 5], 20)})

fig = px.scatter(chart_data, x='x', y='y', color='category', size='size')

st.plotly_chart(fig)
{{< /highlight >}}

接著在 command line 中輸入以下語法（注意工作資料夾與 app.py 在同一個資料夾）：
{{< highlight html >}}
streamlit run app.py
{{< /highlight >}}

可以看到跳轉出一個網頁，內容如下：

{{< figure align=center src="../img/streamlit_1_scatter.png">}}

簡單來說，繪圖的部分完全就是按照原本的做法，把plotly產生的圖表存在名為`fig`的變數裡面，再透過`st.plotly_chart(fig)`，就可以把plotly圖表顯示出來。

## plotly 學習資源

個人覺得直接從官方文件下手就可以了，建議的學習大方向如下，：

1. plotly express

https://plotly.com/python/plotly-express/

2. plotly graph object

https://plotly.com/python/graph-objects/

3. 各種基礎圖表實作

https://plotly.com/python/basic-charts/

4. plotly 圖表結構

https://plotly.com/python/creating-and-updating-figures/

5. 進階主題：
* subplots: https://plotly.com/python/subplots/
* hover text: https://plotly.com/python/hover-text-and-formatting/
* 控制器:
    - https://plotly.com/python/custom-buttons/
    - https://plotly.com/python/sliders/
    - https://plotly.com/python/dropdowns/



## streamlit 學習資源

推薦 streamlit 官方的 30 days challenge，可以直接動手實作。每個單元需要花的時間長度有點不均，有的可能5分鐘，有的可能要花1個小時，
所以可以照自己的步調來學：

{{< a href="https://30days.streamlit.app/" target="_blank" >}} 30 days challenge {{< /a >}}
<br></br>

比較有概念後可以參閱官方的介紹說明，可以對整體架構更瞭解一些：

{{< a href="https://docs.streamlit.io/library/get-started/main-concepts" target="_blank" >}}
main concepts of streamlit {{< /a >}}
<br></br>

後續若看到不錯的資源再補充。

## 我的範例

{{< figure align=center src="../img/streamlit_1_bubble.png">}}

我利用公開資料集做了一些不動產資料視覺化分析，分析了「房屋買賣屋齡」以及「購置住宅民眾的年齡」近10多年的變化。看到了一些有趣的現象：

{{< a href="https://appapp1-idi2rrzcvops6c43hnkcc3.streamlit.app/" target="_blank" >}}
網頁app連結在此 {{< /a >}}
<br></br>
語法可參考此專案的
{{< a href="https://github.com/curtis-lu/streamlitapp1" target="_blank" >}}
github repo {{< /a >}}。


