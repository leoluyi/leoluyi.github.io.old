---
layout: post
title: '[R] 字串操作：Regular Expression 及 stringr 套件'
date: 2016-06-15 13:28
comments: true
tags: 
---

## Functions for string manipulation in base funciton and `stringr` package

| _stringr_         | _base_                   | _Description_                                           |
|:------------------|:-------------------------|:--------------------------------------------------------|
| `str_match`       | `regmaches` + `regexpr`  | Extract matched groups from a string                    |
| `str_match_all`   | `regmaches` + `gregexpr` | Extract matched groups from a string (globally)         |
| `str_replace`     | `sub`                    | Replace first matched patterns in a string              |
| `str_replace_all` | `gsub`                   | Replace all matched patterns in a string                |
| `str_detect`      | `grepl`                  | Detect the presence or absence of a pattern in a string |
| `str_subset`      | `grep(value = TRUE)`     | x[str_detect(x, pattern)]                               |
| `str_split`       | `strsplit`               | Split up a string into pieces                           |
| `str_length`      | `nchar`                  | The length of a string                                  |
| `str_sub`         | `substr`                 | Extract and replace substrings from a character vector  |
