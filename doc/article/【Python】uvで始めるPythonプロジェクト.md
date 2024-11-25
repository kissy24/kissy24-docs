## はじめに

**Rye で始める Python プロジェクト**を執筆してから半年近く経ちました。
(よければ、まず Rye の記事から読んでいただけると嬉しいです)

https://qiita.com/kissy24/items/37c881498dcb8a01f3bd

この半年で uv が大きく成長し、Rye と同等かそれ以上のプロジェクトおよびパッケージ管理ができるようになりました。そのため、今度は**uv**を使って開発を進めたい方向けの記事も上げてみようと思いました。
本稿は Rye の記事同様、ハンズオン形式で誰でも気軽にプロジェクトを作れるようになっています。また、Rye との違いについても簡単にですが、言及します。

### ハンズオン環境

- Ubuntu / MacOS
- uv : 0.4.24

## 0. uv とは？

Rust で書かれた非常に高速な Python パッケージとプロジェクト管理ツールです。

https://docs.astral.sh/uv/

以前に、Rye の記事を執筆した際には、「パッケージインストーラー(pip 等)の代替」ような位置づけで uv を利用していました。実際に Rye でプロジェクト作成する際は、pip か uv でパッケージインストーラーを選ぶ形になっていました。

現在の uv はというと、下記のような機能を代替できます。

- pipenv, poetry, virtualenv など(パッケージ、プロジェクト管理ツール)の代替
- pip の代替([10 倍から 100 倍](https://github.com/astral-sh/uv/blob/main/BENCHMARKS.md)速い)
- pyenv(バージョン管理ツール)の代替

Rye(0.41.0)と uv(0.4.24)を簡単に機能面で比較してみると、下記のような感じになります。

| 項目                 | uv                                   | Rye                             |
| -------------------- | ------------------------------------ | ------------------------------- |
| **パッケージ管理**   | pip に依存せず独自管理               | pip(uv も選択可)を利用して管理  |
| **プロジェクト管理** | `uv init`でプロジェクト作成          | `rye init`でプロジェクト作成    |
| **pip との互換性**   | uv 内の pip 互換で依存関係を管理可能 | pip(uv)と連携して依存関係を管理 |
| **依存関係の解決**   | 独自の依存関係解決システム           | pip(uv)を通じて依存関係を解決   |
| **バージョン管理**   | `uv python`で管理                    | `Toolchain`で管理               |
| **バージョン固定**   | pin でバージョン固定可能             | pin でバージョン固定可能        |
| **依存ファイル**     | `pyproject.toml`に記載               | `pyproject.toml`に記載          |
| **スクリプト実行**   | `uv run`コマンドで実行               | `rye run`コマンドで実行         |
| **ロックファイル**   | `uv.lock` で依存関係を固定           | `rye.lock` で依存関係を固定     |

大きく差異はないですが、現状、Rye から uv を使うのであれば**uv**だけで良いでしょう。もし、pip を今後も利用したい場合は、**Rye**を選ぶとよいでしょう。

:::note warn
**注意点**
ちなみに、uv から pip を操作することも出来ますが、「ベースとなっているツールのインターフェイスや動作を正確に実装しているわけではない」らしいので、Rye を使う方が無難です。
(参考 : https://docs.astral.sh/uv/getting-started/features/#the-pip-interface)

```bat
# 例
uv pip install
uv pip list
```

:::

## 1.インストール編

基本的には、公式の誘導に従います。
(Windows へのインストールは本稿では扱わないので公式見てね)

### 1.1. ダウンロードする

まず、curl コマンドを実行しましょう。適したバイナリファイルが落ちてきます。

```bat
curl -LsSf https://astral.sh/uv/install.sh | sh
```

pip からも落とすこともできますが、pip に依存する形を好まないので割愛します。Rye と異なり、 `$HOME/.cargo/env ($HOME/.cargo/env.fish)`に PATH が通っていれば良いようです。

```bat
# sh, bash, zsh
source $HOME/.cargo/env

# fish
source $HOME/.cargo/env.fish
```

### 1.2. Shell の補完スクリプトを入れる

rye と同様に shell 補完スクリプトが提供されているため、ついでに入れましょう。

```bat
# bash
echo 'eval "$(uv generate-shell-completion bash)"' >> ~/.bashrc

# zsh
echo 'eval "$(uv generate-shell-completion zsh)"' >> ~/.zshrc

# fish
echo 'uv generate-shell-completion fish | source' >> ~/.config/fish/config.fish
```

### 1.3. uv 自体をアップデートする

最後に uv 自体をアップデートしましょう。

```bat
uv self update
```

## 2. Python 管理編

uv では、pyenv のように Python のバージョン管理をすることが可能です。
uv は Rye 同様、必要に応じて自動的に Python のバージョンを取得するため、Python をインストールする必要はないですが、pyenv のように手動でインストールしたり、バージョン確認することが下記のように出来ます。

```bat
uv python install 3.13
```

一度に複数のバージョンをインストールすることも可能です。
(リビジョンまで指定可能です)

```bat
uv python install 3.11.7 3.12 3.13
```

```bat
# 結果
Searching for Python versions matching: Python 3.11.7
Searching for Python versions matching: Python 3.12
Searching for Python versions matching: Python 3.13
Installed 3 versions in 7.05s
   + cpython-3.11.7-macos-x86_64-none
   + cpython-3.12.7-macos-x86_64-none
   + cpython-3.13.0-macos-x86_64-none
```

インストールした Python は下記のように確認可能です。

```bat
uv python list
```

## 3. 新規プロジェクト作成編

### 3.1. プロジェクトを作成する

uv init を用いてプロジェクトを作成していきます。

```bat
uv init uv-sample-project
cd uv-sample-project
```

既にプロジェクトのフォルダがある場合は、下記で OK です。

```bat
cd uv-sample-project
uv init
```

そうすると、以下のようなフォルダ構成が出来上がります。

```bat
.
├── .python-version
├── README.md
├── hello.py
└── pyproject.toml
```

生成された`pyproject.toml`は下記のような感じになります。

```toml
[project]
name = "uv-sample-project"
version = "0.1.0"
description = "Add your description here"
readme = "README.md"
requires-python = ">=3.13"
dependencies = []

```

:::note info
**Rye との比較**
Rye の方が下記のように、デフォルトで色々と生成してくれます。簡単に動かすという観点では uv よりも優れているかもしれません。ただし、プロジェクトを作り込んでいく場合は、uv 位シンプルでいいかなと思います。

```bat
# フォルダ構成
.
├── .git
├── .gitignore
├── .python-version
├── README.md
├── pyproject.toml
└── src
    └── my_project
        └── __init__.py
```

````bat
# pyproject.toml
[project]
name = "my-project"
version = "0.1.0"
description = "Add your description here"
authors = [
    { name = "kissy24", email = "hogehoge@fuga.com" }
]
dependencies = []
readme = "README.md"
requires-python = ">= 3.8"

[project.scripts]
hello = "my_project:hello"

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[tool.rye]
managed = true
dev-dependencies = []

[tool.hatch.metadata]
allow-direct-references = true

[tool.hatch.build.targets.wheel]
packages = ["src/my_project"]
:::

### 2.2. hello worldする

まず何かパッケージを入れたいですね。とりあえずRuffを入れてみましょうか。

https://docs.astral.sh/ruff/


```bat
uv add ruff
````

ライブラリを追加したら必ず`uv sync`しましょう…と言いたいのですが、uv は`uv add`で`uv.lock`と`.venv`が**爆速**で更新される上に`uv run`を起動する前に、`uv.lock`と`pyproject.toml`が最新であること、`uv.lock`と`.venv`が最新であることを**爆速**で確認してくれます。そのため、Rye と異なり sync の出番はほぼありません。(コマンドとしては存在します)

```bat
>>> uv run hello.py
Hello from uv-sample-project!
```

uv で無事プロジェクトを作ることができました。Rye より考えることは少ないですね。

### 2.3. パッケージを開発環境にだけ入れる

おっと、Ruff は開発でしか必要の無いライブラリでしたね。
こういうときはいつも通り、dev パッケージとしてインストールしましょう。

```bat
# 一旦ruffを削除します
uv remove ruff
# devパッケージとしていれます
uv add --dev ruff
# 実行用途のみで使う場合
uv sync --no-dev
```

はい、ちょっと待ってください。さっき「sync の出番はほぼありませんといったそばから`uv sync`使ってません？」と思ったそこのあなた、センスがいいです。
uv では develop 環境で利用する tool を`uv tool`(`uvx`)というコマンド別管理することが出来ます。
(`.venv`に Ruff を含まず、Ruff が利用できます。保存先は`uv tool dir`で確認できます)

```bat
# 一旦devパッケージからruffを削除します
uv remove --dev ruff
# toolとしてruffをいれます
uv tool install ruff
```

では、実際に動くのか見てみましょうか。

```bat
>>> uv tool run ruff check hello.py
All checks passed!
```

コマンドによる Ruff のチェックが出来ました。ちなみにパッケージを削除・更新する場合は下記のコマンドより可能です。

```bat
# 更新
uv tool upgrade ruff
# 削除
uv tool uninstall ruff
```

更に面白いのは、ruff をインストールしなくても`uvx`コマンドでツール実行ができます。
(ちなみに `uvx` は `uv tool run` のエイリアスです)

```bat
>>> uvx ruff check hello.py
All checks passed!
```

開発時のパッケージの持ち方は、下記のように棲み分けすると良さそうですね。

- 開発時の環境に制約をかけたい場合は`uv add --dev`
- ローカルにパッケージを持っておきたい場合は`uv tool install`
- 特に制約のない場合は`uvx`

### 3.3. おまけ : uvx で Pytest を実行する

Ruff と同じ方法で Pytest も出来るのか実際にやってみましょう。
まず、ソースコードを下記のように変更します。

```py:hello.py
def hello() -> str:
    return "Hello from uv-sample-project!"


def main():
    print(hello())


if __name__ == "__main__":
    main()
```

次にテストコードを作成しましょう。(`hello.py`と同階層で OK です)

```py:test_hello.py
import hello


def test_hello():
    assert hello.hello() == "Hello from uv-sample-project!"
```

これで、準備ができたので Pytest してみましょう。

```bat
>>> uvx pytest

platform linux -- Python 3.13.0, pytest-8.3.3, pluggy-1.5.0
rootdir: /home/hoge/workspace/uv-sample-project
configfile: pyproject.toml
collected 1 item

test_hello.py .
```

無事に動作しましたね。

次に、`pytest-cov`が使えるか見ていきましょうか。

:::note info
※ 2024/11/23 追記

with オプションを利用することで依存関係を含めて実行することが出来ます。
(教えていただき、ありがとうございました！)

https://docs.astral.sh/uv/guides/tools/#commands-with-plugins

```bat
$ uvx --with pytest-cov pytest --cov

Installed 6 packages in 7ms
================================================================ test session starts =================================================================
platform linux -- Python 3.13.0, pytest-8.3.3, pluggy-1.5.0
rootdir: /home/kissy24/workspace/uv-sample-project
configfile: pyproject.toml
plugins: cov-6.0.0
collected 1 item

test_hello.py .                                                                                                                                [100%]

---------- coverage: platform linux, python 3.13.0-final-0 -----------
Name            Stmts   Miss  Cover
-----------------------------------
hello.py            6      2    67%
test_hello.py       3      0   100%
-----------------------------------
TOTAL               9      2    78%

```

`uv tool install`も問題なくできるようです。

```bat
$ uv tool install pytest --with pytest-cov

Resolved 6 packages in 78ms
Installed 2 packages in 5ms
 + coverage==7.6.7
 + pytest-cov==6.0.0
Installed 2 executables: py.test, pytest
```

pytest-mock 等の依存関係も同様になります。

:::

※ 以下は、with オプションを利用しない場合の手段になります。

```bat
$ uv add --dev pytest-cov

Resolved 8 packages in 80ms
Prepared 6 packages in 114ms
Installed 6 packages in 7ms
 + coverage==7.6.4
 + iniconfig==2.0.0
 + packaging==24.1
 + pluggy==1.5.0
 + pytest==8.3.3
 + pytest-cov==5.0.0

$ uv run pytest --cov -v

platform linux -- Python 3.13.0, pytest-8.3.3, pluggy-1.5.0 -- /home/hoge/workspace/uv-sample-project/.venv/bin/python3
cachedir: .pytest_cache
rootdir: /home/hoge/workspace/uv-sample-project
configfile: pyproject.toml
plugins: cov-5.0.0
collected 1 item

test_hello.py::test_hello PASSED   [100%]

---------- coverage: platform linux, python 3.13.0-final-0 -----------
Name            Stmts   Miss  Cover
-----------------------------------
hello.py            6      2    67%
test_hello.py       3      0   100%
-----------------------------------
TOTAL               9      2    78%
```

ここまでが、ハンズオンになります。下記に、ハンズオンのコードを置いてますのでよければ確認してみてください。

https://github.com/kissy24/uv-sample-project

## おわりに

いかがでしたでしょうか？個人的に uv を触っていて思ったのは、VSCode の Python 補完と uvx の相性が良いと感じました。例えば、VSCode Ruff であれば、

> Ruff v0.4.5 以降、Ruff には Rust で書かれた組み込み言語サーバーが搭載されています: ⚡ ruff server ⚡ このサーバーは Ruff v0.5.3 で安定版としてマークされ、利用可能であれば拡張機能で自動的に使用されます。 参照してください： Rust ベースの言語サーバーを有効にする
> (参考 : https://marketplace.visualstudio.com/items?itemName=charliermarsh.ruff)

となっており、Ruff に限らず開発者ツールはプロジェクト外(エディタや統合管理ツール)で管理されるような流れを感じます。おそらく、今後の Python プロジェクトは Rust や Go のように開発者が linter や formatter を気にしなくても、デファクト・スタンダードのようなものが勝手についてくるようになるのではと感じています。

uv は、コードに向き合う時間を増やしてくれるような可能性を感じさせてくれました。Rye、uv 問わずこの手の統合管理ツールがより盛り上がってくることを願っています。
