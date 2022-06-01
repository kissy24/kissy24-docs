# Python: typing.Final で色々はまったので PEP591 を読みながら検証する

`typing.Final`を利用した際に、想定していた警告が出なかったり他言語のようなことが実現できなかったことはありませんか？  
本稿では、そういった問題の一助となるようにPEP591を確認しながら`typing.Final`の仕様を明らかにします。

:::note info
対象者: 型アノテーションで [typing.Final](https://docs.python.org/ja/3/library/typing.html#typing.Final) を利用したい方 | [PEP591](https://peps.python.org/pep-0591/) を一緒に読みたい方  
ゴール: 表題の通り  
環境: Python3.9, mypy, pyright
:::

## typing.Finalとは

[python.org](https://docs.python.org/ja/3/library/typing.html#typing.Final)では下記のように記載されています。

> 特別な型付けの構成要素で、名前の割り当て直しやサブクラスでのオーバーライドができないことを型チェッカーに示すためのものです。  
例えば:

```py
MAX_SIZE: Final = 9000
MAX_SIZE += 1  # Error reported by type checker

class Connection:
    TIMEOUT: Final[int] = 10

class FastConnector(Connection):
    TIMEOUT = 1  # Error reported by type checker
```

この説明と例を確認した限りでは、他の言語でもよくあるFinal型と相違ないように思えます。  
(もちろん型アノテーションなので静的型付け言語とは幾分か異なります)

## 個人で検証したこと

個人的に使えるかを確認した際に、以下の内容を簡単に検証しました。

### トップレベルに定数を宣言してみる

### クラス内のインスタンス変数を再代入できないようにしてみる

### Javaでいう`static final type`のクラス変数を宣言してみる

## はまったこと

1. `ClassVar[Final[int]]` あるいは `Final[ClassVar[int]]` のような型を宣言できない
2. クラスのインスタンス変数に再代入しようとした際に警告が出ない

## PEP591と照らし合わせる

## 静的解析チェッカーによる違い

## さいごに

PEPの型についての議論が外部ライブラリである`mypy`に依存していることは少し驚きでした。  
VSCode利用者は、`mypy`ではなく`pylance(pyright)`を使っている方も少なくはないと思いますが、型チェッカーでここまでばらつきが出てしまうのは如何なものか…。  
Python自体に型チェッカーは今後含まれたりしないのですかね、期待しています。
