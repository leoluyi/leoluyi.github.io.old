---
layout: post
title: '[R crawler] 公開資訊觀測站 (實作篇)'
date: 2016-01-05 13:33
featured-img: Jietu_20151230231209.png
comments: true
tags: 
---

在前一篇 [[R crawler] 公開資訊觀測站 (觀察篇) ](http://leoluyi.logdown.com/posts/400371-public-observatory)中，我們已經找到需要的資料在哪裡了，接下來就是用 R 將所需的資料抓回來。

![mops_post](https://lh3.googleusercontent.com/40U5s2kaifs8YEL9kjPIW5sLkSrMnklcnqoDhptJAxztVia9e_ERXE9W3OarLjI28WcH=s0 "mops_post")

---

爬蟲的流程可分為 **Connection** 和 **Parsing** 兩階段，這裡用到的套件是[`httr`](https://cran.r-project.org/web/packages/httr/vignettes/quickstart.html)作為 Connection 的工具，以及[`rvest`](https://github.com/hadley/rvest), `XML`作為 Parser。


![image](https://lh3.googleusercontent.com/3gQc0KwZV-07F3stcRQJjSLVPs1jnAPg70JqM_St2zBtAYMfRlv0GN4_i5PI0vtWcmFnI2POpyw-WJ0Gm4khYp-jJPOY5sDlK-IPY5WqWsx3W_dyaaIPDDdQKkLhXX2DmyKrenyRSpJj90pHSqXWkLQqNI5Z0-uZNLLLJCYielp-OmJqUcrrrxLVoOv-PgXV51I_GOEo4gpl4rezW_249ksDUv_EuLorONOd5ZcXCyjzL8sA4so2_wb-oIGwhrUa9CXEfJ6UGOvs9We4dOJgZqkJL8dmSf0nF2PB7a3dkGUHgalmKpfZaEqVi4u7AcxUxPdPv6AaJ6apRfEmlm_ffpHxAgmHhfGLx7jcKzikyONqurZPzU8nwbeP2R2mN--rJbo76j8pYDuCM7EM8goGofPny9WQJvuG-0mGR7IVHq58qG7BqFs_nlMjU4fBojEV-lg89gq9zpPORkUxScSkA2uVYn3a9hnMDmuHMHYR6bVR3834q-1W0Ol2TTlZoIskaYLr7KZmMZuH1o0HRQC-zG253jcFvYcrbpmmtpPixOj9zWIt2gqf4lkQvN6GUtc8UlfElxdsHQ4QNG5yeiOHtZ3Lv4RxRR0=w903-h618-no)

圖片來源: [data-sci.info](http://data-sci.info)

在開始之前先確認所需套件是否已安裝。

```r
library(magrittr)
library(httr)
library(rvest)
library(XML)  # readHTMLTable
library(dplyr) # data manipulation & pipe line
library(stringr)
```


## Connection

我們先試查尋「上市」、「水泥工業」的結果，可以直接將這個 POST 寫成這樣：

```r
res <- POST(
  "http://mops.twse.com.tw/mops/web/ajax_t51sb01",
  body = "encodeURIComponent=1&step=1&firstin=1&TYPEK=sii&code=01",
  encode = "form")
```

> 參數 `body` 裡面的東西就是 **form data** 

如前所述 **form data** 可從 Chrome 開發人員開發人員工具中看到：

```
encodeURIComponent: 1
step: 1
firstin: 1
TYPEK: sii
code:
```

但為了後面方便置換參數，這裡統一將 `body` 用 `list` 的方式呈現，再由 `POST` 函數直接幫我 encode。這裡有一點要注意的是，預設的 encode 方法是 `"multipart"`可解析檔案上傳或字串，但是在這個例子會產生錯誤；所以這裡設定為 `"form"`，因為這裡的 form data 是單純的 escaped string。

```r
res <- POST(
  "http://mops.twse.com.tw/mops/web/ajax_t51sb01",
  body = list(
    encodeURIComponent = 1,
    step = 1,
    firstin = 1,
    TYPEK = "sii",
    code = "01"
  ),
  encode = "form"
)
# ?httr::POST
```

看看連線是否成功：

```r
print(res)
```

```
# Response [http://mops.twse.com.tw/mops/web/ajax_t51sb01]
#   Date: 2016-01-06 13:34
#   Status: 200
#   Content-Type: text/html
#   Size: 18.2 kB   
```


## Parsing

取得連線後的資料 (**response**) ，先確定我們要的東西在裡面，用 `content` 這個函數來看 **response content**，如果沒錯的話應該會看到和 Chrome 開發人員開發人員工具，在 response 標籤的相同內容。

```r
res_text <- content(res, "text", encoding = "UTF-8") %>%
  `Encoding<-`("UTF-8")  # Windows encodind issue
res_text
```
```
[1] "\r\n\r\n<html>\r\n<head>\r\n\t<title>公開資訊觀測站</title>\r\n<!--\t<link href=\"css/css1.css\" rel=\"stylesheet\" type=\"text/css\" Media=\"Screen\"/> -->\r\n<!--\t<script type=\"text/javascript\" src=\"js/mops1.js\"></script> -->\r\n</head>\r\n\r\n<body>\r\n<table class='noBorder'>\n<tr><td align='right'>\n<form action='/server-java/t56ques' method='post'>\n<input type='hidden' name='step' value='0'>\n<input type='hidden' name='Market' value='sii'>\n<input type='hidden' name='SysName' value='公司彙總報表'>\n<input type='hidden' name='reportName' value='公司基本資料查詢彙總表'>\n<input type='hidden' name='colorchg' value=''>\n<input type='hidden' name='QNum' value='1'>\n<input type='hidden' name='Q1N' value='公司類別'>\n<input type='hidden' name='Q1V' value='???d?u·~                '>\n</form></td></tr></table>...
```

### Parsing html table

在 `rvest` 套件中的 `html_table()` 可輕鬆幫我們擷取表格。首先先用 `read_html()` 將 html 字串轉成 `xml_document`，再用 `html_nodes()` 選到我們要的表格，最後再將表格擷取出來：

```r
res_text <- content(res, as = "text", encoding = "UTF-8")
dt <- res_text %>%
  read_html(encoding = "UTF-8") %>%
  html_nodes(xpath = "//table[2]") %>%
  html_table(header=TRUE) %>%
  .[[1]]
```

但如果很不幸的是用 Windows 系統，因爲編碼問題 `html_table` 會出現問題，所以可以改用 `XML` 套件裡的 `readHTMLTable()` 替代：

```r
## Windows
dt <- res_text %>%
  read_html(encoding = "UTF-8") %>%
  html_nodes(xpath = "//table[2]") %>%
  as.character %>%
  XML::readHTMLTable(encoding = "UTF-8") %>%
  .[[1]]
```

可以看一下資料長什麼樣子：
```r
View(dt)
```




## Refactor

如果想要進一步取得不同市場別和產業別的資料，就要先取得各查詢的代號，以便在 POST 的 form data 置換。

![mops_xpat](https://lh3.googleusercontent.com/-ufi0pm7AwXw/VotQl8OUw-I/AAAAAAAAFKM/V2ZzW6RMxK8/s0/Jietu_20160102150708.png "mops_xpath")

先來做出市場別的 key-value vector：

```r
# different post data values

res_doc <- GET("http://mops.twse.com.tw/mops/web/t51sb01") %>% 
  content(type="text", encoding = "UTF-8") %>% 
  read_html(encoding = "UTF-8")

market_type <- setNames(
  res_doc %>%
    html_node(xpath = "//select[@name='TYPEK']") %>%
    html_children %>%
    html_attr("value"),
  res_doc %>%
    html_node(xpath = "//select[@name='TYPEK']") %>%
    html_children %>%
    html_text()
)

market_type
#    上市     上櫃     興櫃 公開發行 
#   "sii"    "otc"   "rotc"    "pub" 
```

```r
industry_type <- setNames(
  res_doc %>%
    html_node(xpath = "//select[@name='code']") %>%
    html_children %>%
    html_attr("value"),
  res_doc %>%
    html_node(xpath = "//select[@name='code']") %>%
    html_children %>%
    html_text()
)[-1]

industry_type
# 水泥工業         食品工業         塑膠工業         紡織纖維 
# "01"             "02"             "03"             "04" 
# 電機機械         電器電纜         化學工業       生技醫療業 
# "05"             "06"             "21"             "22" 
# 化學生技醫療         玻璃陶瓷         造紙工業         鋼鐵工業 
# "07"             "08"             "09"             "10" 
# 橡膠工業         汽車工業         半導體業 電腦及週邊設備業 
# "11"             "12"             "24"             "25" 
# 光電業       通信網路業     電子零組件業       電子通路業 
# "26"             "27"             "28"             "29" 
# 資訊服務業       其他電子業         電子工業       油電燃氣業 
# "30"             "31"             "13"             "23" 
# 建材營造           航運業         觀光事業       金融保險業 
# "14"             "15"             "16"             "17" 
# 貿易百貨         綜合企業             其他         存託憑證 
# "18"             "19"             "20"             "91"

```

試試取得「上櫃」的「半導體業」資料：

```r
# get different data
res <- POST(
  "http://mops.twse.com.tw/mops/web/ajax_t51sb01",
  body = list(
    encodeURIComponent = "1",
    step = "1",
    firstin = "1",
    TYPEK = market_type["上櫃"],
    code = industry_type["半導體業"]
  ),
  encode = "form"
)


res_text <- content(res, "text", encoding = "UTF-8")
dt2 <- res_text %>% 
  read_html(encoding = "UTF-8") %>%
  html_nodes(xpath = "//table[2]") %>%  
  html_table(header=TRUE) %>%
  .[[1]]
```

## Data Cleansing

檢查一下資料 `dt2` 發現，每隔 15 列就會出現一次表頭，這是一個非常爛的資料寫法，我們只好把它清掉，順便清掉資料中的多餘空白 [Non-breaking space](https://en.wikipedia.org/wiki/Non-breaking_space)：

```r
## data cleansing
dt2 <- dt2 %>% 
  filter(`公司代號` != "公司代號") %>% 
  sapply(str_trim)
```

![enter image description here](https://lh3.googleusercontent.com/-TtdQsxgfA8Y/V1wfbqWixGI/AAAAAAAAGvA/ltpIqRns3wA9pVypaTo6-F_3gf0w8dsNACKgB/s0/mops_result_table.png "mops_result_table.png")

