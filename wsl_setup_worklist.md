# wsl2 
## 設定したこと一覧(2021/02/13時点)
1. `wsl2`起動のための操作 
2. `ubuntu 20.04`の導入
3. `zsh`の導入
    - `Prezto`の導入
4. `python`の導入
    - 2021/02/13時点では`3.9`
    - `pip`の導入
    - `alias`の追加
      ```md
      # .zshrcに追加
      alias python="python3" 
      alias pip="pip3" 
      ```
5. `workspace`dirの追加
6. `golang`の導入
    ```md
    # goの最新版を取得する
    wget https://dl.google.com/go/go1.11.5.linux-amd64.tar.gz
    # 解凍する
    sudo tar -C /usr/local -xzf go1.11.5.linux-amd64.tar.gz
    # .zshrcに追加
    export PATH=$PATH:/usr/local/go/bin
    ```
7. `npm`, `node.js`の導入



# wsl
## 設定したこと一覧(2020年版)
- python3.8の導入
- pipの導入
- golangの導入
- zshの導入
- oh-my-zshの導入
  - zsh-syntax-highlightingを導入
  - export ZSH_DISABLE_COMPFIX=trueで警告を無視
  - python3, pip3をpython, pipで実行できるようにした
- workspaceフォルダの作成
- vscodeとの連携
- npm, node.js のインストール