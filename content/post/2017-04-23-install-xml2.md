---
title: "[R] El Capitan に xml2 パッケージのインストール"
author: ~
date: '2017-04-23'
slug: 'xml2-on-el-capitan'
categories: []
tags: ["computer", "Rmarkdown"]
description: ''
---

OSX El Capitan と XCode に問題があって， **tidyverse** をインストールしようとすると **xml2** パッケージをコンパイルできず止まってしまう。一向に解決されない，OS のアップグレードもしたくない。学習さえしないので同じ問題に遭遇した回数はや3回。ようやく記録を取ろうと思う。

注意：Sierra にアップグレードできる人は，アップグレードした方がいいと思います。

参考にしたのは

* https://github.com/hadley/xml2/issues/127
* https://stat.ethz.ch/pipermail/r-sig-mac/2016-September/012046.html

要するに，こういうことだろう。

```
$ cp /usr/bin/xml2-config ~/bin
$ nano ~/bin/xml2-config
```

3行目を書き換える

```
#prefix=$(xcrun -show-sdk-path)/usr
prefix=/usr
```

一時的に PATH の優先順位を変えてやる。

```
$ export PATH=~/bin:$PATH
```

同じシェルのセッションから R を呼び出す。（RStudio などを使いたければから `open -a RStudio` などとすればよいかもしれない）

```
$ R
R version 3.4.0 (2017-04-21) -- "You Stupid Darkness"                                Copyright (C) 2017 The R Foundation for Statistical Computing
...

> install.packages("xml2")
```

CRAN ミラーを選ぶように促されるのでこだわりがなければ 0-Cloud を選ぶ。インストールがはじまる。上手くいったら `q()` で終了。

```
> q()
```
