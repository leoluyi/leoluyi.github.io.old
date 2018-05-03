---
layout: post
title: '[R] Fixing “Peer certificate cannot be authenticated”'
tags:
- R
- httr
- Web Crawler
date: 2017-03-09 15:52
comments: true
tags: 
---
On Windows machine:

```{r}
Error in curl::curl_fetch_memory(url, handle = handle) : 
  Peer certificate cannot be authenticated with given CA certificates
```
The machine in question is sitting behind a gnarly firewall and proxy, which I suspect are the source of the problem. I also need to use `--ignore-certificate-errors` when running chromium-browser, which points to the same issue.

Solution

```{r}
library(httr)
set_config(config(ssl_verifypeer = 0L))
```

https://www.r-bloggers.com/fixing-peer-certificate-cannot-be-authenticated/
https://github.com/jimhester/gmailr/issues/44
