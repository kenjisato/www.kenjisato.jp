---
title: R/exams でオンラインテストの問題を作成する - その3 (expar の紹介)
author: kenjisato
date: '2020-09-08'
slug: rexams-online-3
categories: ["teaching"]
tags: ["R", "exams"]
description: ''
---

## 目的

[前回]({{< relref "2020-08-25-exams2moodle-2.md" >}}) に引き続き，[R/exams](http://www.r-exams.org/) を使ったMoodle 小テストの作成方法を紹介します。

前回は R のコードを組み合わせてランダムに択一問題を作る方法を紹介しました。今回は、exams 2.3.5 で追加された `expar()` 関数を使って、パラメータをRmd ファイルの外から設定して問題を生成します。

まずはユースケースを説明しましょう。例えば問題のRmdファイルに `p` というパラメータがあって、`p = 1, 2, ... , P` という有限のパラメータ範囲が定まっているとします。

Moodle で出題する問題を 1個だけランダムに生成するのであれば、次のようにすればよいです。

```
# Rmd ファイル内
p <- sample(1:P, 1)
```

ただ、こうしてしまうと本当に1問しか生成されないので、Moodle 上でランダム出題ができません。

簡単な解決方法としては、`exams2moodle()` 関数の引数 `n` を設定すれば、複数の問題が作成できます。

```
# 問題作成時のコマンド
R> exams2moodle(file, n = 100)
```

しかし、これにも多少のデメリットがあって、`n` の実引数をパラメータよりも十分大きくしないと、特定のパラメータに分布が偏ってしまう可能性があり、`p` について一様な分布になりません。生成される問題数がパラメータの数よりも大きいとファイルサイズが無駄に大きくなってしまいます。

結局は、`P` の数だけ Rmd ファイルを作るのが一番理想的なのですが、泥臭くコピーして修正をやっていくと、問題を微調整したいときに大変です。そこで、登場するのが `expar()` 関数です。


## exams のインストール

`expar()` は exams 2.3.5 で追加された比較的新しい関数です。R/exams の古くからのユーザーは最新版にアップデートをお願いします。

最近，exams は増加したユーザーからの要望に応える形で急速に機能を拡張しています。開発バージョンのインストールをおすすめします。

```
install.packages("exams", repos = "https://r-forge.r-project.org")
```

## Case 3: `expar()` 関数を使ってパラメータ範囲をすべて網羅した問題セットを作る

[前回]({{< relref "2020-08-25-exams2moodle-2.md" >}}) 作った数値問題（`mountain-japan-num.Rmd`）とほとんど同じものを使います。ファイル名も同じものにしてしまいましょう。最後の行だけ1つ修正を加えています。


### 前回使った問題


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
exsection: mountain

````

**唯一の修正点は最後の行に `exsection: mountain` という記述を追加したことです。これを書いておくと、これから生成する4つの問題がすべて同じ子カテゴリに含まれるようにできます。**

その他の内容は[前回]({{< relref "2020-08-25-exams2moodle-2.md" >}}) の記事で説明していますので、そちらをご覧ください。


### パラメータを固定した問題を1つ作ってみる

`expar()` 関数の仕事は **問題ファイルの1つ目のコードチャンクの変数定義を上書きした一時ファイルを生成すること** です。一時ファイルへのパスを返します。

やってみましょう。

```
R> library(exams)
R> expar("mountain-japan-num.Rmd", n = 1)
[1] "/var/folders/fg/sd85v8ms1g75pwydmrkwkwvc0000gn/T//Rtmpopkhiw/mountain-japan-num+5F56DE95+7D302.Rmd"
```

`n = 1` は Rmd ファイルで使われているパラメータを 1 に固定するという意味です。
出力されたファイルの中身を見てみましょう。

````
R> writeLines(readLines(.Last.value))

```{r, include=FALSE}
n <- 1 # sample(1:4, 1)

mnt <- data.frame(名称 = c("富士山", "北岳", "奥穂高岳", "間ノ岳"),
                  `標高 (m)` = c("3776", "3193", "3190", "3190"), 
                  check.names = FALSE)
mnt_q <- mnt                  
mnt_q[n, "標高 (m)"] <- "[ ? ]"
```
... 以下略 ...
````

期待したとおりに、 `n` を設定するコード（ファイルの2行目）が上書きされています。

### 全パラメータを網羅する

今、`n` は1から4の整数値であることが分かっているので、次のようにすればよいですね。

```
R> file <- sapply(1:4, function(p) expar("mountain-japan-num.Rmd", n = p))
R> file
[1] "/var/folders/fg/sd85v8ms1g75pwydmrkwkwvc0000gn/T//Rtmpopkhiw/mountain-japan-num+5F56E132+DB5F2.Rmd"
[2] "/var/folders/fg/sd85v8ms1g75pwydmrkwkwvc0000gn/T//Rtmpopkhiw/mountain-japan-num+5F56E132+DBE92.Rmd"
[3] "/var/folders/fg/sd85v8ms1g75pwydmrkwkwvc0000gn/T//Rtmpopkhiw/mountain-japan-num+5F56E132+DC273.Rmd"
[4] "/var/folders/fg/sd85v8ms1g75pwydmrkwkwvc0000gn/T//Rtmpopkhiw/mountain-japan-num+5F56E132+DC50C.Rmd"
```

あるいは `for` を使ったループの方がわかりやすい方は次のようにするとよいでしょう。

```
file <- character(4)
for (p in 1:4) {
  file[[p]] <- expar("mountain-japan-num.Rmd", n = p)
}
```


これを LMS に取り込むには，例えば Moodle の場合は `exams2moodle()` を使います。
`name` パラメータを設定して、トップレベル・カテゴリの名前を決めてしまいましょう。

```
exams2moodle(file, name = "final")
```


ここまで説明したコマンドを次のような Rスクリプトにまとめてしまいます。Rmd ファイルと同じディレクトリ（フォルダ）に保存してください。

```
# final.R

library(exams)

file <- character(4)
for (p in 1:4) {
  file[[p]] <- expar("mountain-japan-num.Rmd", n = p)
}

exams2moodle(file, name = "final")
```

とします。Moodle にインポート可能な XML ファイル `final.xml` が生成されるので，ウェブインターフェースを使ってアップロードします。

次のスクリーンショットで、4問同じカテゴリの中にインポートされている様子がわかります。
要求されている数値が問題ごとに変化していることも確認してください。

<div style="width: 600px; border: 1px solid black; margin-bottom: 20px;">
<img src="/images/postimage/rexams3-expar.png">
</div>


問題バンクから小テストを作るときに、「ランダム問題」で作った子カテゴリ（final/mountain）を指定してやると4つの問題がランダムに選ばれます。


## 次回の予定

次（いつになるやら・・・・）は穴埋め問題（Cloze）の作り方を説明しようと思います。


