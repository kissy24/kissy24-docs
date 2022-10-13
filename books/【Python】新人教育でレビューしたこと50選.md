# 教育研修ナレッジ

研修課題の FB をケーススタディとしてナレッジ化しました。  
かなりの分量ですが、是非読んでください。

## 目次

### 命名

- [変数名が規約違反](#case1)
- [大文字小文字の区別](#case24)
- [クラス名とメソッド名の説明量](#case19)
- [する側かされる側か](#case10)
- [抽象的な命名](#case20)
- [名前が説明になってないパターン](#case27)
- [命名と型の相違](#case37)
- [命名(Write, Read)](#case23)
- [命名(Separate と Split の使い分け)](#case34)
- [命名(search と find の使い分け)](#case35)
- [定数](#case40)
- [無を表現する](#case42)
- [関数名がUpperCamelの理由](#case62)

### 変数

- [トップレベルでの宣言について](#case13)
- [同じ変数に 2 種類の状態が入る作りについて](#case52)
- [変数を説明するコメントについて](#case53)
- [f-string](#case3)
- [絶対値](#case15)
- [join メソッドをより良く扱う](#case46)

### Docstring

- [docstring での説明を前提にした名前について](#case25)
- [docstring の summary について](#case26)

### クラス

- [クラスとは？](#case29)
- [抽象的なクラス名](#case47)
- [抽象クラスとインターフェース](#case31)
- [dataclass を使う理由](#case9)
- [dataclass の初期化処理を追加でしたい場合](#case21)
- [dataclass の罠](#case30)
- [継承したクラスに dataclass を使う](#case54)
- [getter/setter について](#case55)
- [`__del__`について](#case59)

### マジックナンバー

- [マジックナンバー](#case28)
- [マジックナンバーではないパターン](#case51)

### ソート

- [sort と sorted](#case5)
- [sort, sorted 以外のソート](#case18)

### リスト

- [リスト内包表記](#case2)
- [\*演算子でリストを初期化する](#case6)
- [空のリストの評価](#case56)
- [リストオブジェクトの継承](#case57)
- [ジェネレーター式の使い道](#case8)

### 辞書

- [辞書の get メソッド](#case16)
- [キーが存在しない時のみキーバリューペアを追加する](#case49)
- [pop と del の使い分け](#case50)
- [`UserDict` について](#case60)

### 列挙型

- [Enum](#case12)
- [IntEnum を基本的に使うべきではない理由](#case22)

### 条件分岐

- [不等号](#case4)
- [シンプルな if 文](#case7)
- [三項演算子](#case38)
- [`isinstance`メソッドのユースケース](#case58)

### 型アノテーション

- [型アノテーション](#case11)
- [Optional](#case39)

### ライブラリ

- [collections ライブラリ](#case17)
- [numpy](#case44)

### I/O

- [入力待ち](#case41)
- [標準入力数が決まってないときの取得方法](#case48)
- [print 関数](#case43)
- [内包表記で print](#case32)

### map 関数

- [map 関数とラムダ式](#case14)
- [map 関数の仕組み](#case33)

### その他

- [エラーが握りつぶされる](#case36)
- [可変長引数をより分かりやすく使う](#case45)
- [`namedtuple` の新しい書き方](#case61)

---

<a id="case1"></a>

## 変数名が規約違反

```py
def main() -> None:
    str = input()
```

### FB. 組み込み関数 str と名前が同じ点

Python では衝突する名前を宣言しても特に警告もなくそのバインドを奪える  
(仕組みは公式ドキュメントを参考にしてください)

```py
str = 'hoge'
type(str) is str
```

意味不明なコードですがこれもエラーなく走ります。  
ただし 1 行目で str を'hoge'にバインドしているため評価は False です。  
今回は上書きした名前がクラスでしたがもし組み込み関数だったら…？と考えるとメンテナンス性に多大な不安を残すことが分かると思います。  
これが組み込みの名前を使ってはいけない理由です

---

<a id="case2"></a>

## リスト内包表記

```py
sales = []
for _ in range(n):
    sales.append(int(input()))
```

### FB. リスト内包表記を利用しましょう

```py
sales = [int(input()) for _ in range(n)]
```

---

<a id="case3"></a>

## f-string

```py
print('down {0}'.format(sales[i] - sales[i + 1]))
```

### FB. `format`はレガシーなので、`f-string`を利用しましょう

```py
 print(f'down {sales[i] - sales[i + 1]}')
```

---

<a id="case4"></a>

## 不等号

```py
# 前日より小さい
if sales[i] > sales[i + 1]:
    ...
```

### FB. 文章を左から読む特性上このコメントは直観に反するので以下のようにする

```py
# 前日より小さい
if sales[i + 1] < sales[i]:
    ...
```

---

<a id="case5"></a>

## sort と sorted

```py
nums.sort()
print(nums[-3])
```

### FB. 出来る限り副作用のあるメソッドを使いたくないため、以下の方が良い

```py
print(sorted(nums)[-3])
```

---

<a id="case6"></a>

## \*演算子でリストを初期化する

```py
nums = [0 for i in range(n)]
```

### FB. `*`演算子を使ってシンプルな書き換えが可能です

```py
[0] * n
```

リストリテラル([])は新しいリストオブジェクトを作成します。
ただし、以下のような場合は不可です。

```py
[["N"] * n] * n
```

この場合は内側のリストのポインタがすべて同一のもので作成されてしまうため、意図せぬ変更を生みます。
そのため以下のように実装してください。

```py
[["N"] * n for _ in range(n)]
```

---

<a id="case7"></a>

## シンプルな if 文

```py
# yが0より大きいか(0<=y)
if y > 0:
    ...
```

### FB. int は 0 が False、0 以外が True で評価されるため、以下で良い

あと、x,y のような意味のない変数名は NG

---

<a id="case8"></a>

## ジェネレーター式の使い道

```py
def main() -> None:
    # ユーザ数n, ログ行数qを取得
    strs = input().split(' ')
    n, q = [int(i) for i in strs]
```

### FB. 「リストが欲しい場合」と「処理結果が欲しい場合」どちらかを明確にして内包表記かジェネレーター式か決める

今回のように「処理結果が欲しい場合」はジェネレータ式を使いましょう

---

<a id="case9"></a>

## dataclass を使う理由

```py
class User:
    ...
```

### FB. 型を強制したいため、`__init__`をやめて`dataclass`を使いましょう

---

<a id="case10"></a>

## する側かされる側か

```py
def IsFollow(self, user) -> bool:
    ...
```

### FB. Following なのか Followed なのか文脈で判断するのが難しいので正確に命名する

この時は、Following が正確でした。

---

<a id="case11"></a>

## 型アノテーション

```py
input_str = input()
```

### FB. システムハンガリアンにするくらいなら型アノテーションを利用する

```py
input_ :str = input()
```

---

<a id="case12"></a>

## Enum

```py
def main():
    messages = {'MESSAGE_STAY': 'stay', 'MESSAGE_DOWN': 'down', 'MESSAGE_UP': 'up'}
```

### FB. typo が怖いのと関数とかがメッセージを受け取る場合に型が指定できるのでできればメッセージ集は列挙体で持ちたい

```py
class COMPARISON_MESSAGE(Enum):
    STAY = 'stay'
    DOWN = 'down'
    UP = 'up'

COMPARISON_MESSAGE.STAY.value
```

---

<a id="case13"></a>

## トップレベルでの宣言について

### FB. 可読性を損なわなければ、普通においても良い

他の言語だとトップレベルに色々置くのって嫌われますが、  
Python は 1 ファイルが 1 モジュール扱いなので割とガンガントップレベルにおいていいです。  
どうせトップに置いてもモジュールレベル変数なので。  
ただし、ユニットテスト時に副作用がいくつかあったりするのでケースバイケースで。

---

<a id="case14"></a>

## map 関数とラムダ式

```py
compared_to_previous_sales = []
data_length = len(sales_data)
i = 0
while i < data_length:
    i += 1
    if i == 1:
        continue
    compared_to_previous_sales.append(sales_data[i - 1] - sales_data[i - 2])
return compared_to_previous_sales
```

### FB. map 関数と無名関数(ラムダ式)の lambda で一行で表せます

```py
return list(map(lambda a, b: a - b, sales_data[1:], sales_data))
```

---

<a id="case15"></a>

## 絶対値

```py
amount = -1250
print(-amount)
```

### FB. 絶対値がほしい場合は、abs を利用する

```py
print(abs(amount))
```

---

<a id="case16"></a>

## 辞書の get メソッド

```py
messages = {'MESSAGE_STAY': 'stay', 'MESSAGE_DOWN': 'down', 'MESSAGE_UP': 'up'}
messages.get('MESSAGE_STAY')
```

### FB. 基本的には`hoge[key]`の形で value を取得します

`get`メソッドは存在しない`key`を指定した場合エラーではなく`None`を取得するので都合の悪い場面も多いです。

---

<a id="case17"></a>

## collections ライブラリ

```py
dupulicated_num = 0
for num in nums:
    if nums.count(num) != 1:
        dupulicated_num = num
        break
return dupulicated_num
```

### FB. Counter なんてめったに使いませんが一応こういう書き方も…

```py
from collections import Counter

def FindDupulicateNum(nums: List[int]) -> int:
    mc_num, mc_count = Counter(nums).most_common()[0]
    return mc_num if mc_count == 2 else 0
```

collections ライブラリにはよく使う deque が入ってるので存在だけ覚えといてください

---

<a id="case18"></a>

## sort, sorted 以外のソート

```py
def FindDeletedNum(nums: List[int]) -> int:
    non_duplicate_nums = list(map(int, set(nums)))
    sorted_nums = sorted(non_duplicate_nums)
```

### FB. リスト → セット → リストの処理を挟むと勝手にソートされます

```py
a = [4, 3, 6, 2, 1]
list(set(a))
```

---

<a id="case19"></a>

## クラス名とメソッド名の説明量

```py
@dataclass
class LogFecther:
    @classmethod
    def FetchLog(cls, log_counts: int) -> List[str]:
```

### FB. クラス名で説明しきってるのでメソッド名は`Fetch`だけで大丈夫です

例えばファクトリークラスとかも以下のようになります。

```py
class HogeFactory:
    def Make():
        return Hoge(fuga, piyo)
```

---

<a id="case20"></a>

## 抽象的な命名

```py
class FollowData:
```

### FB. `Data`って抽象的じゃないですか？なんのデータ？

例えばフォロー数だったり、フォローをしたユーザーの個人情報だったり  
`Data`や`Info`等は抽象度がとても高いため、具体的な名前に変更する。  
これらの名前は、該当する概念やグループがない場合の最終手段です。

---

<a id="case21"></a>

## dataclass の初期化処理を追加でしたい場合

```py
@dataclass
class FollowStates:
    user_counts: int
    statuses: List[List[bool]] = field(default_factory=List[List[bool]])

    def __init__(self, user_count: int):
        ...
```

### FB. `__init__`は dataclass が自動生成するため、追加で処理をしたい場合は`__post_init__`を利用します

```py
@dataclass
class FollowStates:
    user_count: int
    states: List[List[bool]] = field(init=False, default_factory=list)

    def __post_init__(self):
        for _ in range(self.user_count):
            self.states.append([False for _ in range(self.user_count)])
```

---

<a id="case22"></a>

## IntEnum を基本的に使うべきではない理由

### FB. 値ではなく型と名前でのみ管理したいので、値と比較できるのは本来マズい

ただし、数値が渡されることが制約で決まっている場合は例外的に良い。

---

<a id="case23"></a>

## 命名(Write, Read)

```py
class FollowDataWriter:
    """フォローデータを出力する"""
    ...
```

### FB. Write、Read はファイルへの入出力を指すのが一般的で、標準出力へ流す処理は Output です

組み込みが`print`なのでそのまま`Print`という命名でも良い

---

<a id="case24"></a>

## 大文字小文字の区別

```py
def Input_(cls) -> list[int]:
    ...
```

### FB. python は大文字小文字の区別があるので Input には\_つけなくて大丈夫です

---

<a id="case25"></a>

## docstring での説明を前提にした名前について

### FB. docstring を読まないと分からない名前は避けた方がいいです

docstring は、あくまでも補足なので

---

<a id="case26"></a>

## docstring の summary について

```py
    # case 1
    @classmethod
    def Double(cls, num: int) -> int:
        """2倍する"""
        ...

    # case 2
    @classmethod
    def CheckError(cls, string: str) -> str:
        """エラーかどうかのチェックを行う
```

### FB. 目的語や方法が分からないです。「何を」「どう」するのかはっきりさせましょう

「与えられた数値を」だったり「エラーかどうか」はどう判断するのかとか

---

<a id="case27"></a>

## 名前が説明になってないパターン

```py
class StringChecker:
    ...
```

### FB. string の「何を」check してますか？

---

<a id="case28"></a>

## マジックナンバー

```py
THIRD_NUM: int = 3
```

### FB. マジックナンバーについての理解を深めてください

マジックナンバーとは「意図の伝わらない数値」のことを指します。  
つまり名前がついていても意図が分からなければマジックナンバーということです。  
逆に言うと名前がついてなかろうが意図が伝わればマジックナンバーではありません。  
なぜ THIRD_NUM が必要なのかということを変数名にしてください。

---

<a id="case29"></a>

## クラスとは？

```py
class ElementPicker:
    """リストのn番目の要素を取得するクラス
    """

    @classmethod
    def GetElement(cls, list_: List[int], element_num: int) -> int:
        """n番目の値を取得する
        Args:
            list_ (List[int]): 整数のリスト
        Returns:
            int: リストのn番目の要素
        """
        return list_[element_num]
```

### FB. クラスは「データと手続きをひとまとめにして名前で隠蔽したもの」なんですが、その視点で言うと、このクラスは無意味です

データが無くて、関数と役割が同じになってしまっている  
クラスはグローバル変数を秘匿する仕組み

---

<a id="case30"></a>

## dataclass の罠

```py
@dataclass
class hoge:
    """フォロー状況を復元するクラス"""
    # 現在のフォロー状況 List[List[str]]
    fol_status = []
```

### FB. クラス変数が使いまわされるので NG

ミュータブル関連の罠です。デフォルト引数以外にもこのようなケースがあります。

---

<a id="case31"></a>

## 抽象クラスとインターフェース

```py
class Action(metaclass=ABCMeta):
    """何らかのアクションをする抽象クラス"""
    ...
```

### FB. 抽象クラスというよりはインターフェイスという概念です。また、`metaclass=ABCMeta`はレガシーです

```py
from abc import ABC

class MyABC(ABC):
    pass
```

python では、インターフェイスと抽象クラスの区別はないため、考えて実装する必要がある  
インターフェイスと抽象クラスの違い

- インターフェイスのみ多重継承ができる
- 抽象クラスのみ型だけでなく具体的な処理を持つことができる
- 抽象クラスはクラス変数以外にローカル変数、インスタンス変数を持つことができる
- 抽象クラスはアクセス修飾子として public だけでなく protected も持つことができる

この辺りは OOP の中で最も重要な概念なので是非いろいろ調べてみてください

---

<a id="case32"></a>

## 内包表記で print

```py
# 入力された売上高の日数を表示
print(day_count)
# 売上高の比較結果を表示
[print(result) for result in proceeds_results]
```

### FB. 裏技チックなのでやめましょう

print(result)が毎度評価されるので結果的に全て print されますが、「結果的にそうなる」系の書き方は全て × になります。

---

<a id="case33"></a>

## map 関数の仕組み

### FB. map は、平たく言うと第一引数のオブジェクトを第二以降の引数で呼び出し続ける関数です

第一引数は呼び出し可能なオブジェクトならなんでもいいんですよね(これをそのまま Callable オブジェクトと呼びます)  
で、Python において全てのクラスは type 型のインスタンスなのでその要件を満たすと。  
なので int は関数ではなくクラスです。クラスを渡しています。  
自分でもときどき何言ってるか分からなくなりますがこういう世界です。

---

<a id="case34"></a>

## 命名(Separate と Split の使い分け)

### FB. Separate は混ざりあったものを「分離する」の意味が強く,単純な「分割する」には Split を使ったほうが良いです

---

<a id="case35"></a>

## 命名(search と find の使い分け)

### FB. 「探す」に該当する単語はいくつかありますが Search は re の search と混同するので Find とかがいいです

- search の場合、「(文字列などを探して)ファイルの中を探す」という意味
- find の場合は普通に 「ファイルを探す」という意味
- また、同様に match も正規表現以外で使うのは避けたい

---

<a id="case36"></a>

## エラーが握りつぶされる

```py
if operation_command == 1:
    followed_by, follow_applies_to, *_ = log_details
```

### FB. エラーはエラーとして出るようにしましょう

```py
followed_by, follow_applies_to = log_details
```

これが最適です。こうすればおかしいコマンドが来た時に正しく止まります。

```py
followed_by, follow_applies_to, *_ = log_details
followed_by, follow_applies_to = log_details[0], log_details[1]
```

だと、握りつぶされてしまいます。

---

<a id="case37"></a>

## 命名と型の相違

```py
class Calculator:
    def __init__(self, number: str):
        self.input_number = number
```

### FB. input_number のような数値型っぽい名前が str 型であるのは誤読を生むので統一しましょう

---

<a id="case38"></a>

## 三項演算子

```py
if input_val.isdecimal():
    self.result = int(input_val) * 2
else:
    self.result = Exception('error')
```

### FB. 三項演算子を使いましょう

```py
self.result = int(input_val) * 2 if input_val.isdecimal() else Exception('error')
```

---

<a id="case39"></a>

## Optional

```py
def _FindDeplicatrionNumber(self) -> Union[int, None]:
```

### FB. Optional を使いましょう

`Union[int, None]`は`Optional[int]`と等価になります。

---

<a id="case40"></a>

## 定数

```py
N = int(input())
```

### FB. 入力によって値が変わる変数に N はダメです

Python は大文字の変数名を定数と見なすという不思議な習慣があります  
(実際には Python に定数という概念は存在しないのであくまでも見なすだけですが)

---

<a id="case41"></a>

## 入力待ち

```py
N = int(input())
privious_days_sales = int(input())
for i in range(N - 1):
    todays_sales = int(input())
```

### FB. 入力値を配列にするくだりと配列を計算するくだりで分かれていると good

今回の場合だと処理がかる～いので入力も一瞬で終わりますが、例えば入力値を使って激重な処理をすると考えると  
処理終わりました！→ 入力待ち → 処理終わりました！→ 入力待ち  
というプログラムは非効率です。

---

<a id="case42"></a>

## 無を表現する

```py
            isCorrect = False
            rewrite_from = target_list[i] + 1
if isCorrect:
    return -1, -1
```

### FB. -1 ではなく null と表現してしまって良い

無をどう表現するかは難しい問題ですが、こういうのは一律 null で良い。  
何故かと言うと意味のない値を探すのがはちゃめちゃに難しいため。  
例えばこの-1 って一見意味なさげですけど[1, 2, 3][-1]って 3 なんですよ。  
そのため、インデックスにおける-1 は意味のある数値です。  
そう考えると int 型という制約の中で「ないよ」を表す値を考えるのって大変じゃないですか？  
ちなみに python における null は別の名前です。  
あともう一つ言うと int 型は 0 が False 評価、0 以外は全て True 評価なので「ないよ」が True 評価になるというのも気になります

---

<a id="case43"></a>

## print 関数

```py
word_list = DivisionToWords(input())
word_list.sort(key = str.lower)
print("".join(word_list))
```

### FB. 同等の処理が `print(*word_list, sep="")` で可能です

print 関数の書式は`print(*objects, sep=' ', end='\n', file=sys.stdout, flush=False)`  
使いこなせると`print`デバッグがはかどります

---

<a id="case44"></a>

## numpy

```py
users_num, log_row = map(int, input().split())
operation_log = (tuple(map(int, input().split())) for _ in range(log_row))
# ユーザーの番号とインクリメントの数を合わせるためN+1*N+1の配列を作成
follow_status = [["N" for j in range(users_num + 1)] for i in range(users_num + 1)]
```

### FB. 込み入った配列の操作(主に多次元配列)をしたい場合 numpy というライブラリを用いるのが通例です

```py
import numpy as np

np.full([users_num] * 2, "N")
```

サードパーティ製だけどほぼ公式と化しているライブラリ  
`import numpy as np` というインポート文も慣例です。他に `import pandas as pd` というのもあります。  
ちなみに `ndarray` という型になるので気に食わない場合は `tolist` メソッドで `list` 型に戻すことも出来ます。

---

<a id="case45"></a>

## 可変長引数をより分かりやすく使う

```py
    def Borrow(self, *args) -> str:
        user_id = args[0]
        book_id_list = args[1:]
        ...
```

### FB. `def Borrow(self, user_id, *book_ids) -> str:` の方が分かりやすい

---

<a id="case46"></a>

## join メソッドをより良く扱う

```py
...
    sorted_book_id_list = sorted(lendable_book_id_list)
    joined_book_id = ' '.join(sorted_book_id_list)
    return f'can {joined_book_id}'
return 'can'
```

### FB. `' '.join(['can'] + sorted(borrowable_book_ids))` と 1 行で済む

---

<a id="case47"></a>

## 抽象的なクラス名

```py
@dataclass
class IDManager:
    """
    利用者IDと書籍IDの組み合わせを管理するクラス
```

### FB. `Manager`という語は抽象的な言葉なのでより具体的にすべき

- Docstring でもう少し補足してくれると読み手的にはありがたい
- または、クラス名自体をより具体的にしてくれるとありがたい

---

<a id="case48"></a>

## 標準入力数が決まってないときの取得方法

### FB. `sys.stdin`を利用する

```py
commands = [command.split() for command in sys.stdin]
```

---

<a id="case49"></a>

## キーが存在しない時のみキーバリューペアを追加する

```py
if book not in self.reserved_user:
    self.reserved_user[book] = []
```

### FB. `setdefault()`メソッドを利用する

`setdefault()`メソッドでは、第一引数にキー key、第二引数に値 value を指定する。  
第一引数に指定したキーが存在しない場合、新たな要素が追加される。  
第二引数のデフォルト値は`None`。省略すると値が`None`の要素が追加される。

```py
self.reserved_user.setdefault(book, [])
```

---

<a id="case50"></a>

## pop と del の使い分け

```py
def ReturnShelf(self, book: str) -> None:
    """書架に本を返す
    """
    self.place.pop(book)
```

### FB. pop は取り出した値を使いたい場合に使うので、使うつもりがないなら del の方が良い

---

<a id="case51"></a>

## マジックナンバーではないパターン

```py
TARGET = 0
hoge[TARGET]
...
```

### FB. 0 とか-1 は先頭/末尾であることが明らかなので定数にしなくてもオッケーみたいなところがあります

- ちなみにこれは、`TARGET`というより、「先頭」を指す語の方が適切です

---

<a id="case52"></a>

## 同じ変数に 2 種類の状態が入る作りについて

```py
# placeには"rack"か"{UserID}"が入る
self.place[book] = Info.PLACE_RACK.value
```

### FB. 同じ変数に'rack' or 'ユーザー ID'が入る作りは良くない

- 性質の違うものは別々に管理した方が保守性が高いです。

```py
lend_user: Dict[str, str] = {}
rack: List[str] = []
```

---

<a id="case53"></a>

## 変数を説明するコメントについて

```py
# 利用者が借りる/借りた本の合計
all_books_count: int = clac.sum(len(books), len(lend_books), len(reserve_books))
```

### FB. 変数名に対するコメントは、自分で変数名が悪いって言ってるようなものなので、名前を直しましょう

---

<a id="case54"></a>

## 継承したクラスに dataclass を使う

```py
class ReservationStatusError(Exception):
```

### FB. `@dataclass`は継承してても使えます

- 後、`class Book():`のような、継承しない際にも()をつける書き方は、2 系のレガシーな書き方
  - `class Book:`で宣言しましょう

---

<a id="case55"></a>

## getter/setter について

```py
    def GetUserID(self) -> str:
```

### FB. 現代における OOP は getter/setter を必要としません

- のっぴきならない事情でプロパティを読み取り専用にしたい場合は以下のようになりますが使ってその程度です。

  - これで Hoge(1).hoge = 2 とするとエラーになります
  - そもそも Python で言えばプライベートな属性って存在しないです

  ```py
  @dataclass
  class Hoge:
      _hoge: int

  @property
  def hoge(self) -> int:
      return self._hoge
  ```

- また、class の状態を構成する値を直接読み書きする(get/set する)のではなく、  
  その状態によって処理をした結果の値を返したり、class 全体の状態を変更する関数を実装するべき

---

<a id="case56"></a>

## 空のリストの評価

```py
if reserve_error_msgs == []:
```

### FB. 空の`list`は`False`として評価されます

---

<a id="case57"></a>

## リストオブジェクトの継承

```py
@dataclass
class Users:
    available_book_count: int
    users_table: List[User] = field(default_factory=list)
    ...
```

### FB. リストが本体のようななクラスは list を継承すると良い

- 属性を持ちつつただのリストとして振舞えるので便利

```py
class Users(List[User]):
    def __init__(self, available_book_count, iter_: Iterable = ()):
        super().__init__(iter_)
        self.available_book_count: int = available_book_count
```

---

<a id="case58"></a>

## `isinstance`メソッドのユースケース

```py
if isinstance(borrow_result, BookCountExceededError):
```

### FB. ここは返ってくる型が`BookCountExceededError`そのものなので`is`でいいと思います

- 何が来るかは分からん(とか変更の可能性がある)例外を補足したい場合だけ`isinstance(hoge, Exception)`ですね

---

<a id="case59"></a>

## `__del__`について

### FB. 怪しくて使いたくない

- インタプリタが終了したときに、残存しているオブジェクトの**del**()メソッドが呼び出される保証はありません  
  がこわい。プログラムが終了した後もプロセスやディレクトリが残ったままになる可能性が出てきます。  
  というわけで終了処理が必要なものについては出来る限りコンテキストマネージャを使うようにしましょう

---

<a id="case60"></a>

## `UserDict` について

```py
class MoneyDict(UserDict):
```

### FB. 今回の場合`dict`を継承した方が素直なように見えます

- `UserDict`は Python 2.2 より前のバージョンの「後方互換」モジュールです
- `dict`を継承する際には型も指定することができるので書いておきましょう

```py
class hogedict(dict[str, hoge]):
```

---

<a id="case61"></a>

## `namedtuple` の新しい書き方

```py
Property = namedtuple("Property", ["price", "quantity"])
```

### FB. namedtuple は新しい書き方があります

```py
class attributes(NamedTuple):
    price: int
    quantity: int
```

- あと`Property`はほぼ予約語といって差し支えないような単語なので自作のクラスには使わない方がいいです

---

<a id="case62"></a>

## プロダクト開発部の関数名はlower_snakeではなくUpperCamelなのか？

### FB. 関数はオブジェクトに対する「命令」であるということを強調させたいと考えているため

以下その全文。

> そういえばPEP8では関数名はlower_snakeで書けと言われているのに何故うちの規約はUpperCamelなのか？
> って話を今まで一度もしてなかった気がするのでここでポエムを残しておきます。
> そもそもpythonの関数がlower_snakeなのは恐らくですが関数が第一級オブジェクトであるからだと推測しています。つまり他のオブジェクトと同等であることを意識させたいがために変数と同じlower_snakeで書く、という話です。
> しかし今はpythonの生まれた1991年から29年後の2020年です。関数が第一級オブジェクトであることはもはやどの言語でも当たり前の世界になりました。ということでわざわざそれを意識させる必要はなく、むしろ関数はオブジェクトに対する「命令」であるということを強調させたいと考えUpperCamelを採用しています。