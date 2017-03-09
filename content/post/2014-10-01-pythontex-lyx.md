---
title: pythontex を lyx で使うときのメモ
date: '2014-10-01T13:13:00.002+09:00'
author: Kenji Sato
tags: ["computer", "lyx", "python"]
modified_time: '2014-10-01T13:13:37.405+09:00'
slug: pythontex-on-lyx
---

<p>PDFを作る話なのでPDFに書いた.  <a href="https://dl.dropboxusercontent.com/u/1028642/files/pythontex.pdf">pythontex.pdf</a></p> <p>要点としては, </p><ul><li>コンパイルに関して <ul>  <li>コンパイルはGUIに頼らず, lyx -> xelatex -> pythontex -> xelatex のコマンドを順に実行する</li>  <li>1回目のxelatex コンパイルはエラーを無視する</li></ul><li>画像挿入に関して <ul>  <li>フロート, キャプション, ラベルはGUIを使う</li>  <li> 画像本体の挿入はTeXコマンド( \includegraphics{hoge.pdf} )を書く</li></ul></li><li>Instant Preview は「No math」にしておくとよい</li></ul> 