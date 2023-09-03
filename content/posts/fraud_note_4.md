---
title: "[閱讀筆記]實用詐欺預防：金融科技與電子商務的詐欺與洗錢分析，使用SQL與python(4)"
date: 2023-09-03T14:39:04+08:00
draft: false
ShowToc: true
TocOpen: true
cover:
    image:
    alt: '詐欺風險預防應用IP分析'
    caption: ''
tags: ['fraud', 'risk analysis']
categories: ['note']
series: ['practical fraud prevention']
keywords:
    - fraud risk
    - fraud prevention
    - fraud analysis
    - practical fraud prevention
    - 詐欺風險
    - 偽冒風險
    - 盜刷風險
    - 風險分析
    - 偽冒風險預防
    - IP 分析
---

# Chapter 6 被竊卡盜刷

- 即信用卡資訊（卡號、到期日、卡片上的名字、CVV安全碼）洩漏，詐欺者試圖用卡片來買東西或服務，目的是為了要變現。變現手法有：轉賣買來的球鞋、交易虛擬貨幣、從事直運電商（dropshipping）等等。
- 因為資料孤立個緣故，打擊被竊卡盜刷詐欺的能力有限。一方面，商家沒有客戶的信用卡交易紀錄，另一方面，銀行也沒有客戶消費項目，以及客戶的完整圖像。
- 這種詐欺方式通常是最好上手的，很多之前從小偷轉行過來，或是業餘的。因為盜刷手法及相關資訊蠻容易從暗網的論壇取得。
- 甚至出現「端到端的詐欺服務」：偷來的信用卡卡號＋符合的VPN服務＋符合的email。但本章只針對信用卡資訊洩漏的部分，前述打包好的詐欺情境請見第15章。
- email 分析常見定義：
    - aged email: 養帳號的概念，規避那些會阻擋新辦email的檢查機制。
    - spoofed/hacked email: 持卡人真實的email被盜用，或是購買另一個人的真實email，最好剛好也跟持卡人的名字很像的。

## 作案手法

- 作案的原理：越像真實的持卡人越好。作案前會盡量做好以下這些準備：
    1. 一個乾淨的裝置，沒有任何cookie可以追蹤到之前的攻擊，且沒有安裝任何跟盜刷相關的app。
    2. 裝置設定調整與持卡人的資訊相符，例如國別
    3. IP供應者與地理位置與被駭者相符
    4. 瀏覽行為根據被害者做調整，例如時區 語言 瀏覽紀錄 瀏覽速度
- 接著就可以開始shopping，並期望：
    1. 目標盜刷的商店不會跟被害者的消費歷史差異太多。
    2. 挑選一個被害者沒有已存在帳號的retailer。
    3. 這個卡號沒有被其他詐欺者在該retailer用過。
    4. 盜刷金額與商品數量，沒有觸發retailer的警鈴。
    5. 沒有超過信用卡額度。
- 聰明的盜刷者會在結帳頁面輸入詳細資訓：
    1. 持卡人姓名與帳單地址的拼法正常，而且不是用複製貼上的方式，輸入的速度也符合人類正常模式。
    2. 聯絡的email：有時會用被害者的真實email，但大部分是使用剛開的或是偷來的email。
    3. 聯絡電話：通常是用假的資訊，或是網路電話、拋棄式電話、公開的電話等等，而且會跟被害者的地理位置相符。

## IP分析

- 有關IP相關知識的資源：
    - IP address & IP header: [Hour 4. The Internet Layer - Sams Teach Yourself TCP/IP in 24 Hours, Fifth Edition [Book] (oreilly.com)](https://www.oreilly.com/library/view/sams-teach-yourself/9780132810814/ch04.html)
    - computer networks: [Computer Networks, Fifth Edition [Book] (oreilly.com)](https://www.oreilly.com/library/view/computer-networks-fifth/9780133485936/)
    - web bots and proxy server: [Types of Proxy Servers - Webbots, Spiders, and Screen Scrapers, 2nd Edition [Book] (oreilly.com)](https://www.oreilly.com/library/view/webbots-spiders-and/9781593273972/ch27s05.html)
    - internet core protocols: [Internet Core Protocols: The Definitive Guide [Book] (oreilly.com)](https://www.oreilly.com/library/view/internet-core-protocols/1565925726/)
    - network security: [4. IP Network Scanning - Network Security Assessment [Book] (oreilly.com)](https://www.oreilly.com/library/view/network-security-assessment/059600611X/ch04.html#networksa-CHP-4-SECT-1)

### 不相符的IP

- 即IP的註冊地與消費者不相符，有許多第三方工具可以查IP的註冊地。但由於以下原因，單純去看IP的地理位置與符合預期的位置之間的比較可能容易造成很多錯誤警報：
    1. 單純的絕對距離大小無法用來判斷是否盜刷：距離大小與IP的密度有關，偏遠地區通常沒有分到很多IP，所以距離通常差很大。IP代表的位置通常預設是區域的中心點，非精確的位置。
    2. 國別或城市的比較太過簡單：人們會旅行，歐洲人民常常穿梭在各個國家。
    3. 企業或軍隊等等組織可能管制IP，可能使用hosting service，讓所有員工共用。
- 另外IP本身常常會重新指派，不一定與地理位置有對應的關係，雖然實務上的確會有關聯，但不一定精確。

### repeated offender IP

- 跟犯罪者同一個IP也有可能是錯誤警報。因為IP可能重複使用：
    - ISP會收回家用IP重複利用
    - 手機IP可能是聚合到cell tower level
    - 公司利用NAT路由器管理員工的裝置

### 多人使用的IPs

- 犯罪者可能利用public Wi-Fi、別人家裡的Wi-Fi、學校的Wi-Fi來掩飾。
- 或是透過TOR瀏覽器，可以google IP + “TOR”關鍵字，可能會找到一些東西。

### 遮蔽的IP

- 利用proxy / VPN / hosting IP來遮蔽真實IP
- 可以利用traceroute工具來檢核網路封包是從哪個route跳到哪個route的軌跡，一直到發送封包的源頭。可以從route跟route之間的距離及相對位置來判斷合理性，如果距離差太多，很可能是hosting IP在故意混淆。

### IP分析的可靠性跟地點有關

- 相當依賴於第三方的服務，把IP對應到地理位置以及Whois/ASN(autonomous system number)資料（即註冊資料）。但這些資料並不是百分之百正確的。
- provider有MaxMInd提供的GeoIP2，或是Digital Envoy。該種服務通常是每月或每季更新，資料來源是他們會定期去全球各個區域的註冊服務，爬ASN records。例如APNIC, ARIN LACNIC, RIPE NCC, AFRINIC。但並非每個區域都會時常更新到最新資料。此外，更新的頻率也跟服務使用的熱門程度有關。

## 降低偽冒

- 盡量自動化才能達到規模化阻擋偽冒，但是利用外部的IP地理位置查詢服務很貴，可能Service-level agreement也未必能達到要求。可以批次下載清單，或是利用黑名單的方式，來避免即時要去呼叫第三方服務。
- 可以利用IP註冊名稱，例如“hotel”來識別異常。
- 可以利用reverse DNS的功能。python的gethostbyaddr功能可以利用。
- 記住IP來交易的次數，可以避免false positive。
- 可以搭配其他面向一起看，國家跟裝置語言不一致、裝置數量、週末或假日的活動等等。
- 可以透過mobile SDK or web SDK取得一些裝置資訊，例如service set identifier(SSID)或是Wi-Fi name等等。

## 本章小結

IP的資訊雖然很有價值，但只是個起點，需要搭配更多資訊來發揮最大效用。