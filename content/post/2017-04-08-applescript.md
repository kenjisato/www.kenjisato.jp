---
title: Keynote ファイルをエクスポートするスクリプト
author: Kenji Sato
date: '2017-04-08'
slug: 'export-keynote'
tags: ["computer", "Rmarkdown"]
description: 'Keynote to PDF/JPEG'
---

データに基いて描く必要のない概念図などは Keynote で描いている。Rmarkdown や LaTeX などで読み込むには，JPEG や PDF にエクスポートして利用する。

Rmarkdown で使う場合に利用する `knitr::include_graphics()` は出力がHTMLのときはJPEG画像を読み込んで，LaTeXのときはPDFを使うということをやってくれる。便利だけど手間は2倍。手作業を減らしたいので下のようなスクリプトを書いた。AppleScript も Bashスクリプトも初心者だけど一応動いている動いている。

使い方は Keynote ファイルを入れているフォルダを引数にしてBashスクリプトの方を実行する。例えば

```
bash keynote_export.sh folder -t pdf
bash keynote_export.sh folder -t jpeg
```

などとする。

ただし，本当に `knitr::include_graphics()` に渡すための PDF を作るためにはできたPDFファイル `filename.pdf` を `filename/filename.001.pdf` のように分割する必要があるので，もう少し処理が必要。

普通に LaTeX で `\includegraphics` を使う場合は pgfpages を使えばよい。



<script src="https://gist.github.com/kenjisato/de8435e64041d421605ea22af617038a.js"></script>

