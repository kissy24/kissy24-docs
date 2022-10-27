## はじめに

`pytest`を触っていて下記のような `Windows fatal exception`が大量に表示されたことはありませんか?

下記のようなエラーはOSSライブラリに依存するエラーでこちら側からどうしようもない場合があります。  
本稿はこのような大量に表示されるエラーの抑制方法について言及します。

## 環境

- python: 3.10.6
- pytest: 7.1.3

## 抑制方法

下記のオプションを追記することで無効化されます。

```sh
pytest -p no:faulthandler
```

## faulthandlerとは？

Pythonの標準ライブラリに含まれるモジュールです。  

> このモジュールは、例外発生時、タイムアウト時、ユーザシグナルの発生時などのタイミングでpython tracebackを明示的にダンプするための関数を含んでいます。これらのシグナル、SIGSEGV、SIGFPE、SIGABRT、SIGBUS、SIGILL に対するフォールトハンドラをインストールするには faulthandler.enable() を実行してください。python起動時に有効にするには環境変数 PYTHONFAULTHANDLER を設定するか、コマンドライン引数に -X faulthandler を指定してください。  
(https://docs.python.org/ja/3/library/faulthandler.html より抜粋)

何のことだかといった感じですが、簡潔に言うと「セグフォをスタックトレースにダンプするもの」になります。

## セグフォとは？

「セグメンテーション違反」というもので、

## pytest-faulthandlerというプラグインがあったけど…？

`pytest`のプラグインとして、`pytest-faulthandler`というものがありましたが、`pytest 5.0` から `pytest core` に統合されました。  
(詳細は[こちら](https://pypi.org/project/pytest-faulthandler/))

下記のオプションはレガシーなため、最初に紹介した方法で無効化してください。

```sh
pytest --no-faulthandler
```

## おわりに

筆者がこのエラーに遭遇したのは`pywinauto`や`comtypes`で、windowsアプリケーションの自動操作スクリプトをテストしていた時にでした。  
似たような境遇で困っている方の一助になると嬉しいです。