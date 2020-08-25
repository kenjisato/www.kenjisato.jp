---
title: R/exams でオンラインテストの問題を作成する - その1
author: kenjisato
date: '2020-08-21'
slug: rexams-online-1
categories: ["teaching"]
tags: ["R", "exams"]
description: ''
---

## 目的

後期もオンライン授業という大学教員も多いと思うので，[R/exams](http://www.r-exams.org/) を使った問題作成の経験を共有したいと思います。色々な問題のパターンがあるので，何度かに分けて説明します。

## exams

[**{exams}**](http://www.r-exams.org/) という R パッケージが開発されています。その名の通り試験をするためのパッケージです。


- 紙ベースのマークシート問題（NOPS）を作成する機能（参考：[Written Exams](http://www.r-exams.org/intro/written/)）
- e-ラーニング用の LMS (learning management system) にインポートする問題を作成する機能

があります。

この投稿シリーズでは，LMS にインポートする方法を説明したいと思います。

ちなみに，exams パッケージの作者＆メイン開発者は AER パッケージの作者でもある [Achim Zeileis](https://eeecon.uibk.ac.at/~zeileis/) 先生です。[フォーラム](https://r-forge.r-project.org/forum/forum.php?forum_id=4377) や [Stack Overflow](https://stackoverflow.com/questions/tagged/r-exams) の質問にものすごい速さで答えてくれます。感謝。


## 環境構築

- R
- exams パッケージ
- knitr/rmarkdown まわりのパッケージ

をインストールしてください。最近，exams はユーザーが増加して，ユーザーからの要望に応える形で急速に機能を拡張しています。開発バージョンのインストールをおすすめします。


```
install.packages("exams", repos = "https://r-forge.r-project.org")
```

## Case 1: 択一問題を作成する

手始めに択一問題を作成してみましょう。複数の選択肢から正しい選択肢を1つ選ぶ問題です。

次のような Rmarkdown ファイルを作成します。名前は `mountain-japan.Rmd` としておきましょう。


```
Question
========
  
標高が最も高い山を選びなさい。

Answerlist
----------
  
* 富士山
* 北岳
* 奥穂高岳
* 間ノ岳


Solution
========

Answerlist
----------
  
* 正解。3776m 
* 誤り。3193m
* 誤り。3190m
* 誤り。3190m


Meta-information
================
  
extype: schoice
exsolution: 1000
exname: Highest mountain in Japan
exshuffle: TRUE
```


非常にシンプルですね。R のコードもありません。

本文部分は次のようになっています。

- `Question`, `Solution` という見出しがそれぞれ，問題文，解説を書く場所を示しています。
- 両見出しの下に，`Answerlist` という小見出し，さらにその下にリストが書かれています。これらは問題に出力される選択肢と，対応する解説です。
- `Meta-information` という見出しの下にメタ情報を記載します。
  - `extype`: schoice (択一問題), mchoice (複数選択問題), num (数値問題) などの情報を記述します。可能なオプションは[こちら](http://www.r-exams.org/intro/dynamic/)を御覧ください。
  - `exsolution`: 問題の解答です。前述の選択肢リストと順番が対応しています。1 = True, 0 = False に対応します。1000 は 1つ目の選択肢が True であることを表します。もちろん，schoice 問題では1が1つであることが要請されます。
  - `exname`: 問題を識別するための名前です。
  - `exshuffle`: TRUE の場合，選択肢（それに合わせて解説も）がシャッフルされます。シャッフルしたくない場合は FALSE を指定します。ここに数値，例えば 3 を指定すると，4つの選択肢のうちランダムに選ばれた 3 つだけが問題に記載されます（正解は必ず含まれます）。
  
  
どのような問題が生成されるかは， `exams2html()` という関数を使うとチェックできます。

```
library(exams)
exams2html("mountain-japan.Rmd")
```

下のようなHTMLファイルが生成されます。


<div style="width: 300px; border: 1px solid black; margin-bottom: 20px;">
<img src="/images/postimage/rexams1-html.png">
</div>


これを LMS に取り込むには，例えば Moodle の場合は

```
exams2moodle("mountain-japan.Rmd")
```

とします。Moodle にインポート可能な XML ファイルが生成されるので，ウェブインターフェースを使ってアップロードしてください。

インポートの方法はこちらを参考にどうぞ。→ <http://www.r-exams.org/tutorials/elearning/>

Moodle の他にも Blackboard や Canvas, OpenOLAT などのLMSが利用可能です。（が，使ったことがないのでよく分かりません）


## 今後の予定

夏休み中に色々なユースケースを紹介したいと思います。


