---
title: Shiny for Python でタブの切替え
author: 'kenjisato'
date: '2023-02-08'
slug: shiny-for-python-1
categories:
  - 'python'
tags:
  - 'computer'
  - 'shiny'
  - 'python'
---

## 目的

Shiny for Python で複数アプリをタブで切り替えられるようにするとき，次のような機能があると便利。

1. URI のクエリパラメータに応じて開くタブを変更する。
1. UI を通してタブを移動するたびにアドレスバーが更新されるようにする。

ゴールは動画のような感じ。

{{< youtube 8kIAXZaM0O4 >}}

## 警告

この投稿の作成時点（2023/02/08）で [Shiny for Python](https://shiny.rstudio.com/py/) はα版です。ここで紹介したコードには，現時点では公式ドキュメントに記載されていない機能を使っているので，今後動かなくなる可能性があります。

## 環境

- Python 3.8.15
- shiny 0.2.9

## ファイルの準備

次のような構成にする。

```
.
├── app.py
└── www/
    └── js/
        └── msg.js
```

## 考え方

### 1. クエリパラメータを読み込んでアプリに反映させる

`UI` からHTTPリクエストの情報を取得するためには，UI を次のような関数として定義する。（[参考1](https://github.com/rstudio/py-shiny/issues/269), [参考2](https://github.com/rstudio/py-shiny/pull/299)）

```
def app_ui(request):
    request.query_params.get(...)
    _ui = ...
    return _ui
```

`request.query_params` が辞書ライクなオブジェクトとしてクエリパラメータを保有している。

### 2. URI を更新する

執筆時点では URI を更新させるためのインターフェースがなさそうだったので，JavaScript の関数を呼び出す `session.send_custom_message` を使う。

> Sends messages to the client which can be handled in JavaScript with
> ``Shiny.addCustomMessageHandler(type, function(message){...})``. 
> Once the message handler is added, it will be invoked each time  
> ``send_custom_message()`` is called on the server.　([ソース](https://github.com/rstudio/py-shiny/blob/main/shiny/session/_session.py#L598-L601))

- JavaScript 側では，`Shiny.addCustomMessageHandler('replaceState', fn)` のようなメッセージハンドラとコールバック関数 `fn` を用意する。`fn` は無名関数を引数リストの中にそのまま書いてもよい。
- Python 側で `session.send_custom_message("replaceState", msg)` が実行されると，JavaScript 側で `fn(msg)` が実行される。

具体的には，[GitHub 掲載の使用例](https://github.com/rstudio/py-shiny/blob/c100558cb8d79c4d364e42c1214292d7f59edf88/shiny/examples/send_custom_message/app.py#L22-L25) を参考にする。`async def`, `await` の使い方が重要。

JavaScript との連携は R でも使われている方法と同じなので，[Communicating with Shiny via JavaScript](https://shiny.rstudio.com/articles/communicating-with-js.html) も参考にするとよい。

## コード

### JavaScript コード

コールバック関数は [`History.replaceState`](https://developer.mozilla.org/en-US/docs/Web/API/History/replaceState) を使って URI を書き換えるようなものにする。

```
// www/js/msg.js

Shiny.addCustomMessageHandler('replaceState', function(message) {
    const theUrl = new URL(window.location);
    theUrl.search = new URLSearchParams(message);
    window.history.replaceState({}, '', theUrl);
});
```

### Python コード

まずは前半部分。

Panel A と Panel B をそれぞれ[モジュール](https://shiny.rstudio.com/py/docs/workflow-modules.html)として定義している。実際のアプリではこの部分が長くなるはずなので，ファイルを分けて管理するとよい。

```
# app.py

import pathlib
from shiny import App, module, ui, render, reactive

## Page A

@module.ui
def a_ui():

    return ui.TagList(
        ui.input_numeric("num", "Number for Panel A", value=0),
        ui.output_text_verbatim("x")
    )

@module.server
def a_server(input, output, session):

    @output
    @render.text
    def x():
        n = input.num()
        return f"{n} times 2 = { n * 2 }"

## Page B

@module.ui
def b_ui():

    return ui.TagList(
        ui.input_numeric("num", "Number for Panel B", value=0),
        ui.output_text_verbatim("x")
    )

@module.server
def b_server(input, output, session):
    
    @output
    @render.text
    def x():
        n = input.num()
        return f"{n} times 3 = {n * 3}"

```

Module A と Module B で同じ名前の input (`num`) と同じ名前の output (`x`) がある。モジュールの呼び出し側で設定する名前空間が名前の衝突を防いでくれる。


続いて後半部分，UI側から。

- (1) が JavaScript を読み込むコードを HTMLヘッダに埋め込む。
- (2a), (2b) の `value="a"` などによってタブ毎に ID をつけている。
  - (3) で設定した navset の `id` パラメータを使って，サーバー側で `input.page()` とすると，開かれているタブの ID を取得できる。
  - (4) の `selected` パラメータを "b" に設定すると，ID が "b" であるタブが開かれる。
- (4) の `selected=request.query_params.get("page", "a")` は，「クエリパラメータに `page` が含まれていれば，`selected` をその値で設定する，さもなくば "a" に設定する，という意味である」。例えば，http://localhost:4321/?page=b のようなアドレスでアプリにアクセスすると，Page B が開く。

```
## app.py 続き
##
## Put them together...


def app_ui(request):

    _ui = ui.page_fixed(
        ui.head_content(
            ui.tags.script(src="js/msg.js")                 # (1)
        ),
        ui.panel_title("Main Title"),
        ui.navset_tab_card(
            ui.nav("Page A", a_ui("pageA"), value="a"),     # (2a)
            ui.nav("Page B", b_ui("pageB"), value="b"),     # (2b)
            id="page",                                      # (3)
            selected=request.query_params.get("page", "a")  # (4)
        )
    )

    return _ui
```

最後に，サーバー側の設定。

- (5) は 「`input.page` が変更されたら以下を実行せよ」という意味。`input.page` は開いているタブを表しているので，「タブが移動したら以下を実行せよ」という意味になる。
- (6) は，クエリパラメータを作るための変数を辞書（JSONに変換される）として定義している。これがメッセージハンドラに渡される。
- (7) が JavaScript のコードを呼び出す。
- (8) は JavaScript ファイルの場所を設定して，これを `shiny.App()` に渡す。


```
## app.py 続き
##

def app_server(input, output, session):

    a_server("pageA")
    b_server("pageB")

    @reactive.Effect
    @reactive.event(input.page)                                # (5)
    async def _():
        msg = { "page": input.page() }                         # (6)
        await session.send_custom_message("replaceState", msg) # (7)


www_dir = pathlib.Path(__file__).parent / "www"                # (8)
app = App(app_ui, app_server, static_assets=www_dir)
```



## おわりに

将来的には [Bookmarking in _Mastering Shiny_](https://mastering-shiny.org/action-bookmark.html) のような機能が Shiny for Python にも提供されるかもしれない。さしあたり，JavaScript のコードを自前で用意して解決できた。


