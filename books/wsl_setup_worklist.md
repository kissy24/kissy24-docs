# wsl2

## 設定したこと一覧(2021/02/13 時点)

1. `wsl2`起動のための操作
2. `ubuntu 20.04`の導入
3. `zsh`の導入
   - `Prezto`の導入
4. `python`の導入

   - 2021/02/13 時点では`3.9`
   - `pip`の導入
   - `alias`の追加

     ```md
     # .zshrc に追加

     alias python="python3"
     alias pip="pip3"
     ```

5. `workspace`dir の追加
6. `golang`の導入

   ```md
   # go の最新版を取得する

   wget https://dl.google.com/go/go1.11.5.linux-amd64.tar.gz

   # 解凍する

   sudo tar -C /usr/local -xzf go1.11.5.linux-amd64.tar.gz

   # .zshrc に追加

   export PATH=$PATH:/usr/local/go/bin
   ```

7. `npm`, `node.js`の導入

# wsl

## 設定したこと一覧(2020 年版)

- python3.8 の導入
- pip の導入
- golang の導入
- zsh の導入
- oh-my-zsh の導入
  - zsh-syntax-highlighting を導入
  - export ZSH_DISABLE_COMPFIX=true で警告を無視
  - python3, pip3 を python, pip で実行できるようにした
- workspace フォルダの作成
- vscode との連携
- npm, node.js のインストール
