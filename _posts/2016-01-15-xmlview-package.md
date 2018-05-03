---
layout: post
title: '在 RStudio 檢視 xml/html 的工具：xmlview Package'
date: 2016-01-15 16:31
comments: true
tags: 
---
![xml_view_ptt_xpath](https://lh3.googleusercontent.com/-jscbcVH-Yro/Vpiji5y2A5I/AAAAAAAAFUg/jjOgZ_EEthA/s0/xml_view_ptt_xpath.PNG "xml_view_ptt_xpath")

在寫爬蟲的過程中，常需要針對取得的 html 檢查內容，並用 XPath 或 CSS selector 擷取所需要的資料區塊。但在使用 IDE 撰寫腳本時，要做到這些事必須要把 html 的文本內容 print 出來，或是另存成 html file 再用瀏覽器檢視；若測試 XPath 時，因無法很清楚地直接在 console 瀏覽 xml 的樹狀結構，所以原本需搭配 Chrome 的 [XPath Helper](https://chrome.google.com/webstore/detail/xpath-helper/hgimnogjllphhhkhlmebbmlgjoejdpjl) 會比較方便。

`xmlview` package 提供了一個在 RStudio 上互動檢視 XML 以及測試 XPath 的方式，這裡用個簡單的 XML 當例子：

```r
# devtools::install_github("hrbrmstr/xmlview")
library(xml2)
library(xmlview)
library(magrittr)

## plain text XML
xml_view("<note><to>Dale</to><from>Chip</from><heading>Reminder</heading><body>Baby, don't forget tonight! xxxxx</body></note>")
```

利用 `xml_view` 這個函數吃進 XML string，即可得到 Parsed 後的顯示，

![xml_view_test](https://lh3.googleusercontent.com/-4r0jb3N9IUE/VpiRyinBotI/AAAAAAAAFTw/3FBZIM42gdM/s0/xml_view_test.PNG "xml_view_test")


### XPath 測試

用 PTT 的[隨便一篇文章](https://www.ptt.cc/bbs/Stock/M.1452818794.A.FEC.html)當範例，先把網頁的內容抓 下來，並用 `read_html` 做成 `xml_document` 物件：

```r
## read-in XML document
doc <- xml2::read_html("https://www.ptt.cc/bbs/Stock/M.1452818794.A.FEC.html",
                encoding = "UTF-8")
# xml_view(doc, add_filter = TRUE)
```

由於 read_html 會自動將內容轉換成 _unmarked UTF-8 encoding_，經測試吃進時`xml_view`無法顯示，所以必須先轉換成 _marked UTF-8 encoding_  或 system locale (e.g., Big5) 才能正確顯示，因此這裡先把 `xml_document` 直接轉成 `character` 後再調整 encoding，

```r
doc_string <- as.character(doc) %>% `Encoding<-`("UTF-8")
xml_view(doc_string, add_filter=TRUE)
```

![xml_view_ptt_result](https://lh3.googleusercontent.com/--Q_QnFITUI4/VpiglfZO9KI/AAAAAAAAFUI/PcKKYHY_DXs/s0/xml_view_ptt_result.PNG "xml_view_ptt_result")

吃進去 `xml_view` 後，在 RStudio 的 Viewer pane 顯示了剛才的網頁內容，因為加了`add_filter=TRUE` 這個參數，因此上方出現了 XPath 的輸入框，輸入想測試的 XPath expression 後直接按 <kbd>enter</kbd> 就會馬上跑出結果，還可以按下"R"的圖示自動產生 R code 可直接複製貼上。

![xml_view_ptt_xpath](https://lh3.googleusercontent.com/-jscbcVH-Yro/Vpiji5y2A5I/AAAAAAAAFUg/jjOgZ_EEthA/s0/xml_view_ptt_xpath.PNG "xml_view_ptt_xpath")

最後就得到想要的資料了！

```r
xml_find_all(doc, '//span[@class="f3 push-content"]', ns=xml2::xml_ns(doc))

## or you want to use rvest package
# doc %>% rvest::html_nodes(xpath = '//span[@class="f3 push-content"]')
```

只是在 Windows 的 encoding 問題還是要再處理一下，

```r
doc %>% 
  rvest::html_nodes(xpath = '//span[@class="f3 push-content"]') %>% 
  rvest::html_text() %>% 
  `Encoding<-`("UTF-8") %>% 
  gsub("^: ", "",.)

## [1] "他，還有隱形眼鏡"                               
## [2] "她,還有雙鏡頭"                                  
## [3] "蘋果怎麼可能容忍供應商EPS180但又不降價？"       
## [4] "它，還有眼鏡蛇"                                 
## [5] "不要看新聞做股票吧"                             
## [6] "外資要壓低吃貨囉"                               
## [7] "再爛股價還是會維持四位數啦"
```
