---
title: nbconvert のテンプレートの書き方が変わったらしい
author: 'kenjisato'
date: '2021-03-13'
slug: nbconvert-template
categories: []
tags: ["Jupyter"]
---

変化の早いオープンソース・プロジェクトを使って仕事をするのは大変だ。突然の仕様変更に頭を抱えることもしばしば。実際には突然ではなく議論があったんだろうけど，全部追いかけることはできない（……少なくとも僕には）。

という訳で気づいたら，普通に使われていたコードが deprecated の段階を颯爽と駆け抜けて使えなくなる。少し前に動いていたコードが動かなくなるということが起こる。

起こったのでメモ。

## 背景

講義ノートの作成のために次のようなことをやっている。

1. Jupyter で挿入用の Python コードを書く
2. nbconvert で Input セルのみからなる Python スクリプトに変換
3. knitr + reticulate の力を借りて LyX → Rnw -> LaTeX -> PDF

「Jupyter で最初から最後まで全部かけばいいじゃない」と思うかもしれないけど，数式が多いので LyX を使いたい。


## 現象

2020年度前期の授業用の講義ノートを作っていたので，2020年の夏頃まではちゃんと全行程動いていた。
しかし，半年寝かせた後，2021年度用に修正しようと思ったら2番の
ステップで make が失敗する。（※ 本当はもっと色んな所で全然上手くいかなくてすごく困ったんだけど，それは置いておこう）

動いていた `jupyter nbconvert` のコマンドが失敗するようになってしまった。nbconvert のバージョン 5.6.x から 6.0.7 に上がっている。メジャーバージョンアップに気づかない私が悪いのです。その間，テンプレートの指定の仕方に大幅（？）な変更があったそうな。

## 解決？

私は[5.6 のドキュメント](https://nbconvert.readthedocs.io/en/5.6.0/customizing.html) （注：古い）を参考に `simplepython.tpl` というファイルを置いて，jupyter nbconvert の --template パラメータに指定していた。これはもはや動かない。

今は [6.0 のドキュメント](https://nbconvert.readthedocs.io/en/6.0.7/customizing.html) を参考にしなければならない。こう書いている。

> Nbconvert templates are _directories_ containing resources for nbconvert template exporters such as jinja templates and associated assets.（大意：Nbconvert のテンプレートはテンプレート出力に必要なリソースがもろもろ入ったディレクトリだよ。リソースというのは例えばjinja テンプレートと関連ファイルみたいなやつ）

昔はファイルだったけど，今はディレクトリです。

という訳で，ディレクトリを作って `conf.json` と `simplepython.py.j2` みたいなファイルを入れるんでしょうな。


15分くらい奮闘したけど上手く In [30] とか，マークダウンセルを消すことができなかった。ので，諦めた。


## 回避策

頑張ってテンプレートの書き方を調べるという道もあったんだろうけど，実際にやりたいことは，高度なテンプレート機能が必要な訳ではない。

- Input プロンプトを出力しない
- Markdown セルを出力しない

ということだけを求めている。テンプレートを用意するのは大げさすぎる。

そこで次のようなスクリプトを書いて，

```
# cfg.py
c.TemplateExporter.exclude_input_prompt = True
c.TemplateExporter.exclude_markdown = True
```

`--config` オプションに渡してやる。

```
jupyter nbconvert notebook.ipynb --to script --config cfg.py
```

これでやりたいことはできた。


## 結論

いつまで経っても単純作業に落とし込むことができないので退屈することがない。



