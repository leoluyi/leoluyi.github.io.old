---
layout: post
title: '[Git] 中文顯示亂碼'
date: 2017-03-22 15:41
comments: true
categories: 
---
使用 `git status` 或 `git ls-files` 時，Git 在顯示中文檔名會出現類似下面的亂碼：

`"\321\203\321\201\321\202\320\260\320\275\320\276\320\262"`


Git has always used octal utf8 display, and one way to show the actual name is by using printf in a bash shell.

Since Git 1.7.10 introduced the support of unicode, this [wiki page](https://github.com/msysgit/msysgit/wiki/Git-for-Windows-Unicode-Support) mentions:

> By default, git will print non-ASCII file names in quoted octal notation, i.e. "\nnn\nnn...". This can be disabled with:

```
git config [--global] core.quotepath off
```

Keep in mind that:

> The default console font does not support Unicode. Change the console font to a TrueType font such as Lucida Console or Consolas.
> The setup program can do this automatically, but only for the installing user.

## Reference

http://stackoverflow.com/a/22828826/3744499
