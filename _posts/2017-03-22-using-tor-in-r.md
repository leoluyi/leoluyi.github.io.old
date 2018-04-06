---
layout: post
title: '[R] Using TOR in R'
date: 2017-03-22 14:06
comments: true
categories: 
---
這篇要寫得比較隱晦一些。有時候需要 TOR 來隱藏自己的 IP，然而在 R 裡面要如何辦到呢？

## TOR 有洋蔥

[TOR installation guide](https://www.torproject.org/docs/installguide.html.en)

## Use TOR in R

TOR 是用 `SOCKS5` proxy server，所以在 R 常見的連線 `curl` 套件 (是 `httr` 底層連線的實作)，可以用幾個方式完成：

### 1. Set proxy in `curl`

在 handle 物件加入 proxy：

```r
library(curl)
h <- new_handle(proxy = "socks5://localhost:9050")
```

### 2. Set proxy in `httr`

```r
library(httr)
res <- GET("https://httpbin.org/get", 
           use_proxy("socks5://localhost:9050"))
```

或是用 global httr configuration：

```r
library(httr)
set_config(
  use_proxy(url="socks5://localhost", port=9050)
)

# 重設 global configuration
reset_config()
```

### 3. Set proxy globally (case sensitive!)

直接修改環境變數，可在該 Session 中的連線用到：

```r
Sys.setenv(HTTP_PROXY = "socks5://localhost:9050")
Sys.setenv(HTTPS_PROXY = "socks5://localhost:9050")

# 測試 Even works in base R
readLines(base::url("https://httpbin.org/get", method = "libcurl"))
readLines(curl::curl("https://httpbin.org/get"))
```

## Full sample code

<script src="https://gist.github.com/leoluyi/21fdf8c7eff74c63178046208806194e.js"></script>


## Reference

http://stackoverflow.com/questions/17783686/solution-how-to-install-github-when-there-is-a-proxy
