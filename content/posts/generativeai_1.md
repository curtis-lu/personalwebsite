---
title: "如何開發專屬問答機器人？使用openAI, langchain與streamlit"
date: 2023-10-23T23:47:53+08:00
draft: false
weight: 1
ShowToc: true
TocOpen: true
math: true
enableEmoji: true
cover:
    image: 'img/generativeai_1.png'
    alt: 'generative ai with langchain'
    caption: ''
tags: ['generative ai', 'OpenAI', 'langchain', 'streamlit']
categories: ['projects']
series: ['generative ai']
keywords:
    - generative ai
    - chatbot
    - OpenAI
    - langchain
    - streamlit
---

# 前言

本文是我自己開發有關生成式AI應用程式的side project，目前有把大致的雛形開發出來，但還有許多部分值得再優化。

這個專案的目標是希望開發一個有關**信用卡回饋問題的問答型機器人**，針對想了解台新銀行信用卡的民眾，這個機器人可以回答相關問題。

ChatGPT是基於大型語言模型(LLM)而開發出來的應用，主要目的是想要讓程式可以像人類一樣地對談，提供有意義且上下文有邏輯的回應，**但若涉及一些細節和具體資訊就沒辦法回答得很好**。

例如我問ChatGPT:

![ChatGPT](../img/chatgpt231023.png)

所以就算有開源的大型語言模型可以使用，企業若想要做自家的應用，還是要做一些開發工作。通常有幾種方式：

1. **Pre-training:** 從零開始從頭訓練，根據訓練目標不同可以分為
    - Autoencoding model: 用填空題訓練的模型，適合做情感分析、命名實體標註、文字分類。例如BERT。
    - Autoregressive model: 預測下一個”字”的模型，適合做文字生成。例如GPT系列。
    - Sequence-to-Sequence model: 重組一串一串”字”的模型，適合做翻譯、文句摘要。例如BART。
2. **Fine-tuning：** 使用新任務的資料以及已經pre-train好的的LLM模型，來接著訓練整個模型。可能有災難性遺忘（catastrophic forgetting）的問題。
3. **Parameter-efficient tuning：** 大部分模型參數不動，只重新訓練部分參數，方式：
    - selective: 選擇部分參數重新訓練。
    - Reparameterization: 將部分參數，用一些代數方法做比較簡單地表示後重新訓練。
    - additive: 保留原本模型參數，但額外加上一些可以訓練的參數。
4. **Retrieval augmentation:** 利用抓取相關資訊來強化模型表現。例如可以連網做搜尋、執行python code做運算等等。與**Prompt engineering**一樣都不需要重新訓練模型。
5. **Prompt engineering:** 利用提示詞去引導模型的產出。

本專案即是利用 **Retrieval augmentation** 以及 **Prompt engineering** 的方法，讓LLM模型可以利用台新銀行信用卡相關的資訊，來回答民眾對台新信用卡的回饋問題。

目前的成果：

![langchain+streamlit1](../img/qa1231023.png)

![langchain+streamlit2](../img/qa2231023.png)

# 開發流程與架構說明

整個專案的開發流程如下：

1. **問題定位與規劃：** 希望模型能處理什麼問題？如何做到？
2. **文本處理：** 由於利用了Retrieval augmentation的技術，需要對文本做一些處理。
3. **Prompt engineering：** 運用提示工程，引導模型更好地利用文本來回答問題。

### 問題定位與規劃

在動手開發之前，需要**先釐清希望模型可以處理什麼問題**。本次專案聚焦在回答民眾對台新信用卡的回饋問題，思考民眾可能的提問後歸納為以下4種情境：

- 情境1：**特定消費類型**刷**特定卡片**會有什麼回饋？
- 情境2：**特定卡片**最高回饋的**特定消費類型**有哪些？
- 情境3：**特定消費類型**刷**哪張卡片**會有最高回饋？
- 情境4：**特定卡片**在各種**消費類型**有哪些回饋？

針對這些問題，模型要能夠擷取相關的資訊，舉例情境一的問題：「在便利商店＠gogo卡的回饋率有多少？」。模型要能夠解析這個問題，知道這是一個特定消費類型與特定卡片的問題，所以知道要抓取跟「便利商店」（特定消費類型）相關以及跟「@gogo卡」（特定卡片）相關的資訊。接著，統整抓出來的資訊，最後才是回答問題。

簡而言之，要模型解題的處理步驟大致如下：

1. **解析問題**
2. **理解問題後擷取相關資訊**
3. **統整擷取出來的資訊**
4. **根據問題與資訊回答問題**

### 文本處理

Retrieval augmentation的原理是透過詢問來檢索資料，利用檢索後的資料作為上下文，一起餵進語言模型來做最後的回答，確保語言模型生成的回答更接近正確的資訊。這樣的做法適合需要較精確回答問題的使用情境。

Retrieval augmentation 需要一些事前的準備工作，大致如下：

1. **讀入文本(load documents)。**
2. **文本切割(split documents)。**
3. **產出embedding vectors。**
4. **將文本Embedding存入向量資料庫(vector databases)。**

文本我認為是Retrieval augmentation最重要的部分，基本上可以把Retrieval augmentation理解為：根據提問，**把文本中的相關資訊抓出來後，成為提示詞的一部分**，一起拿來問語言模型。

如果文本本身雜亂無章，或太多不相干的資訊，導致擷取出來的資訊無助於回答問題的話，模型也就不可能正確回答問題。

此外，由於提示詞的長度有所限制，文本的長度也是很重要的一點，因此需要做**文本切割**。文本最好也很有結構性，才能更有效率的做分割。

切割後的文本就可以丟進Embedding模型，透過Embedding模型將一段一段的文句轉換成多維度的向量（embedding vector），所謂**embedding vector**可以理解為把**單詞**轉換為**數值**來代表。因為無論何種模型，都需要轉換成數值才能處理，所以需要把文字轉換為數值。

文本轉換成embedding vector後就可以丟入vector database中儲存，供後續使用。

### 提示工程

根據上述的規劃，提示工程分成幾個步驟如下：

1. 要求模型解析問題的提示
2. 要求模型統整資訊的提示
3. 要求模型根據問題與資訊回答問題的提示

接下來就直接看code吧！

# 實作語法展示

## 事前準備

### 環境建置

首先環境建置方面，我是使用pyenv管理python版本，加上poetry為每一個專案建立虛擬環境以及管理套件版本。

相關安裝在這邊不詳細說明，可以參考以下資源：

- pyenv 安裝：[pyenv/pyenv: Simple Python version management (github.com)](https://github.com/pyenv/pyenv)
- poetry安裝：[Introduction | Documentation | Poetry - Python dependency management and packaging made easy (python-poetry.org)](https://python-poetry.org/docs/)
- 相關參考資源：[Poetry + pyenv 實戰心得：常用指令與注意事項 - Code and Me (kyomind.tw)](https://blog.kyomind.tw/poetry-pyenv-practical-tips/)

若安裝好pyenv與poetry的話，簡單的操作流程如下：

```
# 建立專案資料夾
mkdir [專案名稱]
cd [專案名稱]

# 指定python版本
pyenv local 3.11

# 建立poetry專案資料夾與虛擬環境
poetry init
poetry env use 3.11

# 啟動poetry虛擬環境
poetry shell

# 安裝相關套件，也可以直接編輯pyproject.toml，然後使用poetry update指令
poetry add pandas
poetry add jupyter
poetry add openai
poetry add python_dotenv
poetry add langchain
poetry add markdown # for UnstructuredMarkdownLoader
poetry add tiktoken # for embedding
poetry add chromadb # for vector database
poetry add streamlit # for streamlit app
```

### 申請openAI API key

使用openAI來開發是要錢的，計費規則如網址說明：[Pricing (openai.com)](https://openai.com/pricing)。

首先先申請一個API Key，網址如下：

[OpenAI Platform](https://platform.openai.com/account/api-keys)

為了避免在程式中赤裸裸地打出API key，所以要把API key存到環境變數中，供系統抓取。以下是mac的做法：

```
# 存入環境變數
echo "export OPENAI_API_KEY='sk-xxxxxxxxxxx'" >> ~/.zshrc

# 套用shell環境變數
source ~/.zshrc

# 確認是否儲存成功
echo $OPENAI_API_KEY
```

接著就可開啟jupyter notebook，import需要的套件如下：

```python
import os
import openai

from langchain.text_splitter import MarkdownHeaderTextSplitter
from langchain.embeddings.openai import OpenAIEmbeddings
from langchain.vectorstores import Chroma
from langchain.prompts import PromptTemplate
from langchain.prompts import ChatPromptTemplate
from langchain.llms import OpenAI
from langchain.chat_models import ChatOpenAI
from langchain.chains import LLMChain
```

讀入剛存好的api key

```python
openai.api_key  = os.environ['OPENAI_API_KEY']
```


## 文本處理

文本的部分，本次實驗專案先聚焦@gogo卡、flygo卡、玫瑰giving卡、gogoro卡以及街口聯名卡，所以只有蒐集這5張卡的信用卡回饋資訊。

另外，文本蒐集的方式是最簡單暴力的複製貼上，並人工切割段落。會這樣處理的原因是希望用相對乾淨的文本來做試驗，盡可能去除文本品質對語言模型的影響。

整理好的文本儲存在檔案：tsb_creditcard.md，這邊我使用markdown的格式，透過h3標題將文本做分割，方便後續讀入與切割。

讀入事先編輯好的文本：

### 載入文本與文本切割

```python
with open('./documents/tsb_creditcard.md', 'r') as file:
    text = []
    for line in file:
        text.append(line)

markdown_document = ''.join(text)
```

因為提示詞有字數限制，避免擷取出來的段落太長，所以需要將文本分割：

```python
headers_to_split_on = [
    ("###", "Header 3"),
]

markdown_splitter = MarkdownHeaderTextSplitter(
    headers_to_split_on=headers_to_split_on
)

md_header_splits = markdown_splitter.split_text(markdown_document)

print(len(md_header_splits))
```

### embedding and vector database

切割好的文本就可以轉換為embedding vector並存入vector database，這邊使用的database是chroma。

```python
# 定義vector database儲存路徑
persist_directory = './docs/chroma/'

# 初始化一個embedding model
embedding = OpenAIEmbeddings()

# 將文本轉為embedding vector（注意會有費用產生不要一直重跑XD）
vectordb = Chroma.from_documents(
    documents=md_header_splits,
    embedding=embedding,
    persist_directory=persist_directory
)

vectordb.persist()

# 確認數量是否與切割後的段落數相同
print(vectordb._collection.count())

# 讀取vector database的方法
persist_directory = './docs/chroma/'
embedding = OpenAIEmbeddings()
vectordb_load = Chroma(persist_directory=persist_directory,
                       embedding_function=embedding)
```

## 提示詞工程

先說明一下大致的架構：

1.用LLM解析客戶詢問

2.根據客戶詢問抓出相關的文本段落

3.根據解析後的詢問以及資訊段落，用LLM統整資訊

4.最後將原始的客戶詢問以及統整後的資訊，拿來問ChatLLM，產出最後的答案。

畫成圖的話如下：

![miro](../img/miro231023.png)

### 解析客戶詢問

解析客戶詢問的提示詞：

```python
decomposition_template = '''

根據客戶詢問（<<<>>>符號之間的文句），整理出該問題的結構如下：
["提問類型","特定卡片","消費類型","資訊搜尋指令"]。

"提問類型"：判斷詢問屬於以下類型中的哪一種：
    A.[特定消費類型]刷[特定卡片]會有什麼回饋？
    B.[特定卡片]最高回饋的[消費類型]有哪些？
    C.[特定消費類型]刷[哪張卡片]會有最高回饋？
    D.[特定卡片]各種[消費類型]有哪些回饋？
    E.其他類型。
"特定卡片"：是否有指定某一張或某幾張信用卡？若有，請列出名稱，若有多張請以","分隔。
				  若無，請填「無指定卡片」。
"消費類型"：是否有指定特定消費類型？例如類別、地點，或方式。
					請列出3個該消費類型的同義詞，並以","分隔。
          若無，請填「無指定消費類型」。
"資訊搜尋指令"：根據上述客戶詢問及問題結構，想出最有可能找出有用資訊的資訊搜尋指令。
--------
範例1：
客戶詢問：gogoro卡有什麼回饋？
問題結構：("提問類型":"特定卡片各種消費類型有哪些回饋？",
         "特定卡片":"gogoro卡",
         "消費類型":"無指定消費類型",
         "資訊搜尋指令":"找出跟gogoro卡相關的消費回饋資訊")
範例2：
客戶詢問：街口卡在便利商店回饋幾％？
問題結構：("提問類型":"特定[消費類型]刷[特定卡片]會有什麼回饋",
         "特定卡片":"街口卡",
         "消費類型":"便利商店,超商,超市",
         "資訊搜尋指令":"找出街口卡在便利商店,超商,超市的回饋率")
範例3：
客戶詢問：飲料店消費用哪張卡回饋最多？
問題結構：("提問類型":"[特定消費類型]刷[哪張卡片]會有最高回饋？",
         "特定卡片":"無指定卡片",
         "消費類型":"餐飲消費,平日國內消費,一般國內消費",
         "資訊搜尋指令":"找出餐飲消費,平日國內消費,一般國內消費回饋率最高的卡片")
範例4：
客戶詢問：用以下哪一張卡在新光三越百貨消費最划算？玫瑰giving卡、@gogo卡、flygo卡、街口卡
問題結構：("提問類型":"[特定消費類型]刷[哪張卡片]會有最高回饋？",
         "特定卡片":"玫瑰giving卡,@gogo卡,flygo卡,街口卡",
         "消費類型":"新光三越百貨, 一般國內消費",
         "資訊搜尋指令":"找出玫瑰giving卡,@gogo卡,flygo卡,街口卡在新光三越百貨的回饋率")
--------
客戶詢問：<<<{question}>>>
問題結構：
'''
```

產出Prompt template，並使用langchain將這一小段流程串起來：

```python
decomposition_prompt = PromptTemplate.from_template(decomposition_template)
llm = OpenAI()
decomposition_chain = LLMChain(llm=llm,
                               prompt=decomposition_prompt,
                               output_key="decomposition")
```

### 擷取與統整資訊

初始化根據客戶詢問擷取資訊的retriever：

```python
retriever = vectordb_load.as_retriever()
```

統整資訊的提示詞（根據langchain提供的compressionchain改寫）：

```python
compression_template = '''

Given the following question and the key parts of the question,
extract the information from the following context,
and combine the extracted information into a new context.

Remember, *USE* the key parts to extract the information.
Remember, *DO NOT* edit the extracted information.
Remember, *KEEP* the new context relevant to answer the question.

Question: {question}
key parts of the question: {decomposition}
Context: {context}

new context:
'''
```

一樣把這一小段的流程串起來：

```python
compression_prompt = PromptTemplate.from_template(compression_template)
compression_llm = OpenAI(max_tokens=-1)
compression_chain = LLMChain(llm=compression_llm,
                             prompt=compression_prompt,
                             output_key="new_context")
```

### 問答模型

最後是回答問題的提示詞：

```python
chat_template = '''

你是台新銀行的信用卡客服人員，請根據客戶詢問，以及背景資訊回答問題。

客戶詢問：{question}
背景資訊：{new_context}

回答問題請根據以下結構來回覆客戶的詢問，但可以適時補充說明：
- 哪張卡片？
- 刷卡的方式或刷卡商店？
- 回饋率有多少？
- 是否有使用限制？或其他注意事項。
- 涉及多張卡片或多個情境的比較。

你的回覆：
'''
```

chat這段的流程串起來如下：

```python
chat_prompt = ChatPromptTemplate.from_template(chat_template)
chat_llm = ChatOpenAI(model_name="gpt-3.5-turbo-0301", temperature=0)
chat_chain = LLMChain(llm=chat_llm, prompt=chat_prompt)
```

統整整個流程如下（原本也想用langchain串起來，但debug了好久還是無法，所以用了看起來比較不優雅的方法）：

```python
question = """國外網站消費要用哪張卡回饋最高？"""

decomposition = decomposition_chain.invoke(question)

context = retriever.invoke(question)

new_context = compression_chain.invoke({'question':question,
                                        'decomposition':decomposition,
                                        'context':context})

answer = chat_chain.invoke({'question':question,
                            'new_context':new_context['new_context']})
```

### streamlit應用程式開發

最後，套用streamlit的模板，將整個程式塞進來後就可以部署。
在本地端部署的話，只要將下方語法存入`app.py`，然後command line中輸入`streamlit run app.py`就可以在本地端部署。

```python
import streamlit as st
from langchain.embeddings.openai import OpenAIEmbeddings
from langchain.vectorstores import Chroma
from langchain.prompts import PromptTemplate
from langchain.prompts import ChatPromptTemplate
from langchain.chains import LLMChain
from langchain.llms import OpenAI
from langchain.chat_models import ChatOpenAI

st.title('🦜🔗 Quickstart App')

openai_api_key = st.sidebar.text_input('OpenAI API Key')

## load vector store
@st.cache_resource
def load_vector_store():

    persist_directory = 'docs/chroma/'
    embedding = OpenAIEmbeddings()

    return Chroma(persist_directory=persist_directory, embedding_function=embedding)

@st.cache_data
def load_templates():

    templates = []
    for filename in ['decomposition_template', 'compression_template', 'chat_template']:
        with open(f'./templates/{filename}.txt', 'r') as file:
            templates.append(file.read())

    return templates

def generate_response(input_text, decomposition_chain, retriever,
                      compression_chain, chat_chain):

    question = input_text
    decomposition = decomposition_chain.invoke(question)
    context = retriever.invoke(question)
    new_context = compression_chain.invoke({'question':question,
                                            'decomposition':decomposition,
                                            'context':context})

    answer = chat_chain.invoke({'question':question,
                                'new_context':new_context['new_context']})

    st.info(answer['text'])

with st.form('my_form'):
    text = st.text_area('Enter text:', '國外網站消費要用哪張卡回饋最高？')
    submitted = st.form_submit_button('Submit')
    if not openai_api_key.startswith('sk-'):
        st.warning('Please enter your OpenAI API key!', icon='⚠')

    if submitted and openai_api_key.startswith('sk-'):

        vectordb_load = load_vector_store()
        retriever=vectordb_load.as_retriever()

        templates = load_templates()

        decomposition_template, compression_template, chat_template = templates

        decomposition_prompt = PromptTemplate.from_template(decomposition_template)
        llm = OpenAI(openai_api_key=openai_api_key)
        decomposition_chain = LLMChain(llm=llm, prompt=decomposition_prompt, output_key="decomposition")

        compression_prompt = PromptTemplate.from_template(compression_template)
        compression_llm = OpenAI(max_tokens=-1, openai_api_key=openai_api_key)
        compression_chain = LLMChain(llm=compression_llm, prompt=compression_prompt, output_key="new_context")

        chat_prompt = ChatPromptTemplate.from_template(chat_template)
        chat_llm = ChatOpenAI(model_name="gpt-3.5-turbo-0301", temperature=0, openai_api_key=openai_api_key)
        chat_chain = LLMChain(llm=chat_llm, prompt=chat_prompt)

        generate_response(text, decomposition_chain, retriever, compression_chain, chat_chain)
```

# 結論

本文嘗試使用**Retrieval augmentation**以及**Prompt engineering**的方法，來開發專屬的問答機器人。簡單測試起來，感覺好像還行。當然還有許多可以優化的部分：

1. 文本結構化，利用metadata來優化資訊擷取過程
2. 加入memory功能，更有對話感。
3. 加入tool agent，可以產出python code去執行一些運算。例如客戶提供消費金額試算回饋率。
4. 結構化的evaluation過程，確保修改的過程模型表現有越來越好。

對**Retrieval augmentation**以及**Prompt engineering**的方法來說，資料蒐集、文本切割，以及提示詞如何引導模型找尋或統整資訊都非常重要。

開發前若能夠清楚瞭解使用情境，並據以分類客戶意圖，對於要如何蒐集及組織文本，以及如何設計提示詞，相信會有很大的幫助。延伸來說，或許設計生成式AI應用程式時，如何”framing”客戶的使用情境可能會是影響客戶體驗的重要因素。

(p.s. 這個專案用下來大概花了3.8元美金，也是不便宜XD 但好像有消息指出要降價了？)
