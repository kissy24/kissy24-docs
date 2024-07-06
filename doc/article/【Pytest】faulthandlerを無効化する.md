## はじめに

`pytest`を触っていると、下記のように `Windows fatal exception`が大量に表示されたことはありませんか?

```sh
Windows fatal exception: code 0x80010108

Thread 0x0000625c (most recent call first):
  File "C:\Users\Hoge\AppData\Roaming\Python\Python38\site-packages\comtypes\__init__.py", line 185 in shutdown
Windows fatal exception: code 0x80010108

Thread 0x0000625c (most recent call first):
  File "C:\Users\Hoge\AppData\Roaming\Python\Python38\site-packages\comtypes\__init__.py", line 185 in shutdown
```

このエラーはOSSライブラリに依存するものでこちら側からはどうしようもない場合もあります。  
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
(<https://docs.python.org/ja/3/library/faulthandler.html> より抜粋)

何のことだかといった感じですが、簡潔に言うと「セグフォをスタックトレースにダンプするもの」になります。

## セグフォとは？

「セグメンテーション違反」というものです。

> ソフトウェアがアクセス禁止とされているメモリ上のエリアにアクセスしようとしたり、メモリ上の位置ごとに設定されているルールに違反してメモリにアクセスしようとするときに起こるものである。  
([wikipedia](https://ja.wikipedia.org/wiki/%E3%82%BB%E3%82%B0%E3%83%A1%E3%83%B3%E3%83%86%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E9%81%95%E5%8F%8D)より抜粋)

大量に出力される`Windows fatal exception`は、基本的にメモリへのアクセス違反が原因で起こるわけなんですね。

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
