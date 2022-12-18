# Pythonユーザーのためのpytest解説

## はじめに

皆さんはPythonでユニットテストをちゃんとしていますか？なんだか小難しいし、コード自体は動くからやらなくても良いと思っていませんか？また、`pytest`でググってもまとまった記事に出会えないから`unittest`を使っているような状況だったりしませんか？そんな方々のためにお送りする記事になります。

本稿では、下記のような読者を想定しています。

- `pytest`が全然分からない・上手く使えない方
- `unittest`から`pytest`に移行したい方
- `pytest`の基礎的な使い方を理解したい方

動作環境は下記を想定しています。

Python: 3.10.6  
pytest: 7.1.3  

## 1. Hello pytest

まずは実際に`pytest`でテストを実施してみましょう。下記のように、`pip`で`pytest`をインストールします。

```sh
pip install pytest
```

次に、プロダクションコードとテストコードをそれぞれ任意のスクリプト内に記述します。

```py:hello_pytest.py
def hello() -> str:
    return "hello pytest"

def test_hello():
    assert hello() == "hello pytest"
```

最後に、`pytest`をコマンド実行します。

```sh
$ pytest .\hello_pytest.py
============================== test session starts ==============================
platform win32 -- Python 3.10.6, pytest-7.1.3, pluggy-1.0.0
rootdir: C:\Workspace\Scripts
collected 1 item                                                                  

hello_pytest.py .                                                          [100%]

=============================== 1 passed in 0.02s ===============================
```

導入からテストの流れはこのような感じになります。次項からそれぞれトピックごとに深堀して説明します。

### 1.1. pytest とは？

[pytest org](https://docs.pytest.org/en/stable/) 曰はく
> pytest is a mature full-featured Python testing tool that helps you write better programs.  
> (pytestは、より良いプログラムを書くための成熟した全機能を備えたPythonのテストツールです。)

という Python ユーザーならこれ使えと言わんばかりのテストツールです。以下のような特徴を持っています。

- 失敗したassert文の詳細情報の開示
  - unittestの`assert`ヘルパー関数を覚える必要はありません
- テストモジュールと関数の自動検出
  - test_hoge.py や hoge_test.py, test_hoge関数のように`test`と付くモジュールや関数を自動で検出してくれます。
- ビルトインで多様なfixtureが提供されている
  - 痒い所に手が届くパラメータや前(後)処理がたくさんあると思ってください
- プラグインが豊富(800以上あり、コミュニティにも活気がある)

### 1.2. unittest との違い

Pythonには標準ライブラリに`unittest`というテストライブラリを持っています。しかし、このライブラリは上記の`pytest`のような使いやすさは感じられない気がします。例えば下記ようなコードと実行結果になるのですが、pytestの方が分かりやすいのではないでしょうか？

#### pytest

```py
def hello() -> str:
    return "hello"


def test_hello_pytest():
    assert hello() == "hello pytest"
```

```sh
=============================== test session starts ===============================
platform win32 -- Python 3.10.6, pytest-7.1.3, pluggy-1.0.0
rootdir: C:\Workspace\Scripts\python
collected 1 item

hello_pytest.py F                                                            [100%]

==================================== FAILURES ===================================== 
________________________________ test_hello_pytest ________________________________ 

    def test_hello_pytest():
>       assert hello() == "hello pytest"
E       AssertionError: assert 'hello' == 'hello pytest'
E         - hello pytest
E         + hello

hello_pytest.py:6: AssertionError
============================= short test summary info ============================= 
FAILED hello_pytest.py::test_hello_pytest - AssertionError: assert 'hello' == 'he...================================ 1 failed in 0.19s ================================ 
```

#### unittest

```py
import unittest

def hello() -> str:
    return "hello"

class Hello_unittest(unittest.TestCase):
    def test_hello_unittest(self):
        self.assertEqual(hello(), "hello pytest")
```

```sh
F
======================================================================
FAIL: test_hello_unittest (hello_pytest.Hello_unittest)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "C:\Workspace\Scripts\python\hello_pytest.py", line 14, in test_hello_unittest
    self.assertEqual(hello(), "hello pytest")
AssertionError: 'hello' != 'hello pytest'
- hello
+ hello pytest


----------------------------------------------------------------------
Ran 1 test in 0.001s
-FAILED (failures=1)
```

デフォルトのスタックトレースだけでも`pytest`の方が見やすいと感じます。また、オプション等を駆使することでより理想的なスタックトレースを出力することが出来ます。テストコードに関しても`pytest`の方が簡潔で読みやすいように感じます。

ちなみに、`pytest`で`unittest`の機能を併用して使うことが可能ですが、`unittest`でできることは基本的に`pytest`でもできるため、併用はおすすめできません。

## 2. 実行時設定

`pytest`を実行する際に困るのが、多様なオプションや設定ファイルの多さだと思います。今回は初学者がpytestを実際に利用していく上で重要になりそうな点にフォーカスして説明します。

### 2.1. 設定ファイル

先にお伝えしておくと、「設定ファイル定義するのめんどくさい」と思った方は正直定義不要だと思います。なくても`pytest`は十分動いてくれます。そのため、不要だと感じた方は一旦この項はスキップしていただいても問題ないです。`pytest`をある程度使ってみて、オプション毎回付けるのが面倒くさいとか、特定のファイルをテストしたいといった思いが出てきてからで十分だと思います。

では、気を取り直して`pytest`の実行時のオプションや定義することのできる設定ファイルを紹介します。これらの設定ファイルは**リポジトリのルート**または**tests フォルダ**上に定義されます。

- `pytest.ini` : 最も優先で読まれるファイルだが個人的には後述する`pyproject.toml`を定義する事が多い。

  ```ini
  # 例
  [pytest]
  minversion = 6.0
  addopts = -ra -q
  testpaths =
      tests
      integration
  ```

- `pyproject.toml` : リンターやフォーマッターの設定ファイルの兼ね合いでこれをよく使います。`tool.pytest.ini_options`が定義されていると読まれます。

  ```toml
  # 例えばblack等を含めた設定ファイルを書くことが出来ます
  [tool.black]
  line-length = 120

  [tool.pytest.ini_options]
  minversion = "6.0"
  addopts = "-ra -q"
  testpaths = [
      "tests",
      "integration",
  ]
  ```

- `tox.ini` : `tox`のようなテストツールにも`pytest`の設定を記述することが可能です。

  ```ini
  # pytest.ini と記法は同じなので移行しやすいですね
  [pytest]
  minversion = 6.0
  addopts = -ra -q
  testpaths =
      tests
      integration
  ```

- `setup.cfg` : `setup.py`実行時のパラメータなどを記述する設定ファイルです。現在は`pyproject.toml`で事足りることが多いため、あまり定義する機会はないかな…。

  ```ini
  # pyproject.tomlと似た感じです。flake8とかmetadataなどが記載できます。
  [tool:pytest]
  minversion = 6.0
  addopts = -ra -q
  testpaths =
      tests
      integration
  ```

また、設定ファイルで定義できるオプションをいくつか紹介します。

- `minversion`: `pytest`のバージョンの下限値を指定できます。(そもそも`pipenv`とか`poetry`でバージョン管理したほうが良い)
- `addopts`: 常に実行したい`pytest`のオプション引数を指定できます。
- `testpaths`: テスト対象ディレクトリを指定できます。
- `python_files/python_classes/python_functions`: テスト対象の ファイル/クラス/関数 名を指定できます。(基本いじりません)
- `markers`: カスタムマーカーの登録ができます。(5. Marksで後述します。)

### 2.2. 実行時オプション

`pytest`を実行する際に、オプション引数を選択することができます。

まず、`pytest --help`(`pytest -h`)でオプションを確認してみましょう。

```sh
usage: pytest [options] [file_or_dir] [file_or_dir] [...]

positional arguments:
  file_or_dir

...
```

実行してみると分かりますが、多様なオプション引数があります。今回はその中でもよく使われるオプション引数をいくつか紹介します。

- -k オプション:「`pytest -k "{キーワード}"`」でモジュール名やクラス(関数)名に特定のワードが含まれているものだけをテストする。
  - テストケースが増えてくると活躍します。
  - `"{キーワード}"`は否定や論理和・積にも対応しています。

    ```sh
    # test_hoge 以外(キーワードを含まないもの)が実行されます
    pytest -k "not test_hoge"
    # test_hoge または test_fuga が実行されます(キーワードを含むもの)
    pytest -k "test_hoge or test_fuga"
    # hoge かつ fuga のキーワードを含むものが実行されます
    pytest -k "hoge and fuga"

    ```

- -s オプション:
- -v オプション:

## 3. ファイル構成

意外とハマりやすいのが、このファイル構成になります。このあたりはPythonのパッケージやインポートの理解度に依存します。  

### 3.0. ただ試したい人向けな構成

何も考えずに実行できるでしょう。拡張を予定していない、書き捨てコードのテスト検証などはこのような構成でも良いでしょう。

```txt
root/
├test_hoge.py
└hoge.py
```

### 3.1. テストディレクトリを含む簡単な構成

下記のような構成がよくある簡単な構成だと思います。

```txt
root/
└src/
  ├tests/
  │  ┝__init__.py
  │  └test_hoge.py
  └hoge.py
```

ここで気をつけたいのは、`tests`ディレクトリ以下に`__init__.py`を配置しなければならないということです。`__init__.py`を配置することにより、pytest実行時に`tests`パッケージを認識し実行できるようになります。ここで勘の良い方だと「Python3.3 以降は暗黙の名前空間パッケージがあるから`__init__.py`は不要では?」と思うかもしれません。実際どうなのかというと、`sys.path`で起点となる`src`を名前空間パッケージとして扱えば不要になります。

```py
import sys

sys.path.append("{プロジェクトの絶対パス}")
```

また、名前空間パッケージを設定する方法として`PYTHONPATH`を設定する方法もあります。

- Windows : 設定から環境変数を追加する。
- Mac,linux : `export PYTHONPATH="{プロジェクトの絶対パス}"`

個人的には、`sys.path`を触るのは個人やチームのPython習熟度によっては難しいと思うので、まず`__init__.py`の配置がおすすめです。(とは言いつつも、Pythonの名前空間の操作がmock等で重要になってくるので余裕があれば掘り下げて調べてみてください)

### 3.2. 複数のテストディレクトリを含む構成

規模が大きくなり、`src`と`tests`の分離や`conftest.py`を定義した構成です。

```txt
root/
├pyproject.toml
├src/...
└tests/
  ├conftest.py
  ├hoge_hoge/
  │  ┝__init__.py
  │  ├conftest.py
  │  ┝test_hoge1.py
  │  └test_hoge2.py
  └fuga_fuga/
    ┝__init__.py
    ├conftest.py
    ┝test_fuga1.py
    └test_fuga2.py 
```

ここで注意したほうが良いのは下記のようなことになります。

- `__init__.py`の配置には限界があるので、名前空間の操作が必要になる。
- `conftest.py`を利用して共通のフィクスチャを定義する。

`conftest.py`というのは、「6. Fixtures」で紹介するフィクスチャを特定のディレクトリ内で共有するためのファイルになります。`conftest.py`で定義された内容は、定義されたディレクトリ以下で有効になります。例えば、`hoge_hoge`ディレクトリ内の`conftest.py`は`hoge_hoge`内のみ有効で、`tests`ディレクトリ直下の`conftest.py`は`hoge_hoge`ディレクトリと`fuga_fuga`ディレクトリの両方で利用することが出来ます。

## 4. assert

### 4.1. assertとは？

テストコードを書く際に、頻出と言っても良いものがこの`assert`です。`assert`とは[IT用語辞典](https://e-words.jp/w/%E3%82%A2%E3%82%B5%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3.html#:~:text=%E3%82%A2%E3%82%B5%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E3%80%90assertion%E3%80%91%E3%82%A2%E3%82%B5%E3%83%BC%E3%83%88%20%2F%20assert%20%2F%20%E3%82%A2%E3%82%B5%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E3%83%81%E3%82%A7%E3%83%83%E3%82%AF&text=%E3%82%A2%E3%82%B5%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E3%81%A8%E3%81%AF%E3%80%81%E8%A1%A8%E6%98%8E%E3%80%81%E6%96%AD%E8%A8%80,%E3%81%99%E3%82%8B%E4%BB%95%E7%B5%84%E3%81%BF%E3%82%92%E3%82%A2%E3%82%B5%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E3%81%A8%E3%81%84%E3%81%86%E3%80%82)を引用すると、

> アサーションとは、表明、断言、主張などの意味を持つ英単語。プログラミングにおいて、あるコードが実行される時に満たされるべき条件を記述して実行時にチェックする仕組みをアサーションという。

ということであり、**条件の結果を表明する**仕組みになります。

Pythonには、[単純文](https://docs.python.org/ja/3/reference/simple_stmts.html#)として、「[assert文](https://docs.python.org/ja/3/reference/simple_stmts.html#the-assert-statement)」が存在しており、プログラムのデバッグ用に用意されています(`builtin`)。`assert`文の記法は

```py
assert {条件式}, {エラーメッセージ}
```

のようになっており、下記のような処理と等価になっています。

```py
if __debug__:
    if not {条件式}:
      raise AssertionError(エラーメッセージ)
```

要するに、条件が`False`だと`AssertionError`が`raise`されることになります。これが`Python`標準の`assert`文の機能になります。

では、`unittest`と`pytest`の`assert`はどうなっているのでしょうか？まず`unittest`から見ていきましょう。
`unittest`では下記のような`assert`ヘルパー関数が用意されています。

| 関数 ||
| ---- | ----|
| assertTrue | 真のときOK |  
| assertFalse | 偽のときOK |
| assertEqual | 一致するときOK |
| assertNotEqual | 一致しないときOK |
| assertLessEqual | 大小関係的に劣ってるときOK |
| assertIsNone | NoneのときOK |
| assertIsNotNone | NoneではないときOK |

基本的に、この`assert`ヘルパー関数は`assert`と同等の操作が可能です。

```py
# unittest
assertEqual(actual, expected)

# builtin
assert actual == expected
```

ただ、厳密に言えば同等ではないです。というのも`assert`ヘルパー関数は`assert`文を利用しておらず、独立した実装がなされているためです。`assertEqual`の元となっている`_baseAssertEqual`を見てみましょう。

```py
def _baseAssertEqual(self, first, second, msg=None):
    """The default assertEqual implementation, not type specific."""
    if not first == second:
        standardMsg = '%s != %s' % _common_shorten_repr(first, second)
        msg = self._formatMessage(msg, standardMsg)
        raise self.failureException(msg)
```

`builtin`の`assert`よりも例外やロギング周りがテコ入れされていることが分かります。ただ、この方式はヘルパー関数をすべて把握しておかなければならないし、何よりも文として用意されていた汎用性、可読性の高い`assert`を無視して実装されている点があまりイケてないように感じます。今度は否定形の`assertNotEqual`と`assert`の違いを見てみましょう。

```py
# unittest
assertNotEqual(actual, expected)

# builtin
assert actual != expected
```

ログや例外の違いはありますが、はっきり言って`builtin`の条件式で十分状態がわかるため、`unittest`のような書き方は可読性を下げている気がします…。このように感じている方が多かったのでしょうか、`pytest`では`builtin`の`assert`文そのものを書き換えします。`pytest`の`assertion`パッケージを除いてみると下記のようなフックスクリプトがあります。

```py::rewrite.py
# 分量が多いので処理は割愛しています
class AssertionRewritingHook(importlib.abc.MetaPathFinder, importlib.abc.Loader):
    """PEP302/PEP451 import hook which rewrites asserts."""

    def __init__(self, config: Config) -> None:
      ...

    def set_session(self, session: Optional[Session]) -> None:
      ...

    # Indirection so we can mock calls to find_spec originated from the hook during testing
    _find_spec = importlib.machinery.PathFinder.find_spec

    def find_spec(
        self,
        name: str,
        path: Optional[Sequence[Union[str, bytes]]] = None,
        target: Optional[types.ModuleType] = None,
    ) -> Optional[importlib.machinery.ModuleSpec]:
      ...
```

このような`assert`自体の書き換え処理が内部的に走っているため、`pytest`の`assert`は直感的な文を継承しつつリッチなスタックトレースが表示されることになります。

### 4.2. pytestでのassertの書き方について

`assert`は下記ような感じで活用します。

```py
def test_hoge():
  expected = "hoge"
  actual = hoge()
  assert actual == expected
```

また、一つのテストケースに複数の`assert`を記載することが出来ます。

```py
def test_hoge():
  expected = "hoge"
  actual = hoge()
  assert actual == expected
  assert len(expected) == 4
```

ただし、複数の`assert`を一つのテストケースに入れるのは、テストケースの可読性を損なう原因になるため多用しないことをおすすめします。基本的に、1テスト1アサートでどういう結果になるかを表せるような1つの事に着目できる設計を目指しましょう。

ただ、そうは言っても正しく検証するために、複数の処理、アサートをすべき場合もあります。そういう場合は、`unittest`のようにヘルパー関数を作成し、適用するという手もあります。

```py
def assert_equal_and_length(actual, expected, length):
  assert actual == expected
  assert len(expected) == length

def test_hoge():
  actual = hoge()
  assert_equal_and_length(actual, "hoge", 4)
```

例が雑で申し訳ありませんが、実際にはもっと複雑かつ再利用性のあるものに対しヘルパー関数を作成して使うことがあります。

## 5. marker

### 5.1. markerとは?

`pytest`にはマーカーと呼ばれるデコレータが存在しています。デコレータとは下記のような「関数を引数に取り、関数実行前後に追加処理を付与して返す機能」で、それらを簡単に呼び出す機構です。

```py
def hoge_decorator(func):
    def wrapper(*args):
        print("hoge")
        result = func(*args)
        print("hoge")
        return result
    return wrapper

@hoge_decorator
def hello_world():
  print("hello_world")

hello_world()
```

```sh
hoge
hello_world
hoge
```

`pytest`では`@pytest.mark.`で始まるデコレータを標準で提供しています。今回はその中でも重要なマーカーに着目して説明します。重要なマーカーは下記の4種類です。

1. skip
2. skipif
3. xfail
4. parametrize

順に紹介します。

### 5.2. skip(skipif)

### 5.3. xfail

### 5.4. parametrize

## 6. Fixtures

## 7. Mocks

## おわりに
