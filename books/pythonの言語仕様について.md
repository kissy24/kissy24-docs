# Pythonの言語仕様について

## 目次

1. [Pythonはどういう言語か](#py_1)
2. [Pythonの良い所とは](#py_2)
3. [Pythonのダメな所とは](#py_3)

<a id="py_1"></a>

## Pythonはどういう言語か

### 1. 記述の自由度の高さ

- 「手続き」でも「オブジェクト指向」でも「関数型」でもコーディングできる
- 例外は投げても、返しても良い

```py
# example
def hoge(hoge: int):
    # print(hoge(1)) は error を返す
    if hoge == 1:
        return Exception("error")
    # hoge(2) は Exception: error を投げる
    if hoge == 2:
        raise Exception("error")
```

- 型付けしてもしなくても良い
  - `typing`モジュールを利用することによる型の指定ができる

- 大体なんでも`return`できる
  - `class`オブジェクトとかも返せる

### 2. インタプリタ言語

- 機械語に逐次翻訳される
- インタプリタであるがJITとかコンパイルとか抜け道が豊富

### 3. 少ないコード量

- if文のelse省略

```py
# 3だけ飛ばしたい
for i in range(10):
    if i == 3:
      continue
    ...
```

- リスト内包表記

```py
# for文を使ったlistへの追加
hoge = []
for i in range(10):
    hoge.append(i)

# リスト内包表記
hoge = [i for i in range(10)]
```

- ラムダ式

```py
# 加算の関数
def add(a: int, b: int) -> int:
    return a + b

# 加算のラムダ式
add = lambda a, b: a + b
```

- セイウチ演算子

```py
if (fuga := hoge[3]) == 1:
    print(fuga)
```

- map関数

```py
# 1~10をそれぞれ2倍する
mapped_list = map(lambda a: a*2, range(10))
```

<a id="py_2"></a>

## Pythonの良い所とは

### 1. インデントでブロックを構成できる

- {}がいらないという思想が素晴らしい
  - フォーマッタ等で勝手にインデントが調整される現代に{}をつけてブロックわけしないといけない理由はないでしょう
- スペース4つが推奨

### 2. 豊富なライブラリ

- pandas, numpy, matplotlib とか

### 3. 動くコードを作成するまでの敷居

- 直感でとりあえず動くものが作成できる 

<a id="py_3"></a>

## Pythonのダメな所とは

### 1. `class`オブジェクトの振る舞いの多さ

- 「データ型」も「複合型」も「インターフェース」も「列挙型」も全部class
  - javaとかCとかではありえない

### 2. 抽象クラスとインターフェースの区別が厳密にない

- Pythonではそういった区別がないため、抽象クラスとインターフェースを混ぜた書き方をすべきではない。

### 3. モジュールのimport

### 4. 自由度が高すぎて書き手によってはとんでもないコードになる

- 他の言語と違いいくらでも手抜き、自分本位に記述できる
  - pythonのコードは見やすいというサイトを見かけるがとんでもない。
  - しっかりとしたコーディングルールの上に成り立つ言語
