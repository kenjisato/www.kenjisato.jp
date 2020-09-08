---
title: R/exams でオンラインテストの問題を作成する - その2
author: kenjisato
date: '2020-08-25'
slug: rexams-online-2
categories: ["teaching"]
tags: ["R", "exams"]
description: ''
---

## 目的

[前回]({{< relref "2020-08-21-exams2moodle.md" >}}) に引き続き，[R/exams](http://www.r-exams.org/) を使った問題作成の方法を紹介します。

前回は R のコードを使わない基本的な択一問題を作りました。今回は簡単な R のコードを使って問題をコントロールする方法を紹介します。

## exams

最近，exams は増加したユーザーからの要望に応える形で急速に機能を拡張しています。開発バージョンのインストールをおすすめします。

```
install.packages("exams", repos = "https://r-forge.r-project.org")
```

## Case 2: R の変数を活用した数値問題を作る

[前回]({{< relref "2020-08-21-exams2moodle.md" >}}) 作った択一問題を数値問題に変形してみます。4つの山の標高を1つだけ抜いて、その高さを答えてもらいましょう。

ただし，空欄補充問題にする方が自然な感じもしますが、それは cloze タイプの問題を作るときにあらためて説明します。


### Rmd ファイル

次のような Rmarkdown ファイルを作成します。名前は `mountain-japan-num.Rmd` としておきましょう。


````

```{r, include=FALSE}
n <- sample(1:4, 1)

mnt <- data.frame(名称 = c("富士山", "北岳", "奥穂高岳", "間ノ岳"),
                  `標高 (m)` = c("3776", "3193", "3190", "3190"), 
                  check.names = FALSE)
mnt_q <- mnt                  
mnt_q[n, "標高 (m)"] <- "[ ? ]"
```


Question
========
  
[ ? ] に入る整数値を答えなさい。

```{r, echo=FALSE}
knitr::kable(mnt_q, align = "lr", format = "markdown")
```

Solution
========

`r mnt[n, "名称"]` の標高は `r mnt[n, "標高 (m)"]` [m]。


Meta-information
================
  
extype: num
exsolution: `r mnt[n, "標高 (m)"]`
exname: Highest mountain in Japan
extol: 0

````

本文でやっていることは次のようなことです。

- 1〜4 の数字を1つランダムに選んで `n` という変数名をつける。
- 山の名称と標高をデータフレームに保存する。
- データフレームの `n` 行目の「標高」データのみを消したデータフレーム `mnt_q` を作る。
- Question セクションで `mnt_q` を表示して，空欄に入る数字を解答するように指示する。
- `Solution` セクションでは，`n` 行目のデータに関する情報を書きます。`` `r mnt[n, "名称"]` `` というコードは R の変数の値を文章に埋め込むためのものです。

Meta-information は次のように読みます。

- extype: num
  - → この問題は1つの数値を解答する問題です。
- exsolution: `` `r mnt[n, "標高 (m)"]` ``
  - → この問題の解答は `mnt` データフレームの `n` 行目，"標高 (m)" 列のデータです。
- exname: → 問題を識別するための名前
- extol: 0
  - → 許容誤差を指定する。0 は誤差を許さないという意味。



どのような問題が生成されるかは， `exams2html()` という関数を使うとチェックできます。

```
library(exams)
exams2html("mountain-japan-num.Rmd")
```


これを LMS に取り込むには，例えば Moodle の場合は

```
exams2moodle("mountain-japan-num.Rmd")
```

とします。Moodle にインポート可能な XML ファイルが生成されるので，ウェブインターフェースを使ってアップロードしてください。

結果として次のような問題が生成されました。

<div style="width: 300px; border: 1px solid black; margin-bottom: 20px;">
<img src="/images/postimage/rexams2-moodle.png">
</div>


## 次回の予定

この記事では，問題をランダムに1つ生成し，Moodle に取り込みました。4つある山のそれぞれについて，同じ構造の問題を作れますから，4通り全部について問題を作って Moodle にアップロードできると便利そうです。どのパターンをランダムに選ぶかは Moodle の方に任せることができます。


