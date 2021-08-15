# wsl2 セッティング

## 設定ガイド(2021/08 時点)

1. `wsl2`起動のための操作
1. `ubuntu 20.04`の導入
1. `fish` の導入
    - `oh-my-fish`
    - `dotfiles` から設定import
1. `neovim` の導入
    - `dotfiles` から設定import
1. `python`の導入
   - 2021/08 時点では`3.9`
   - `alias`の追加

     ```md
     # .zshrc に追加

     alias python="python3"
     alias pip="pip3"
     ```

1. `golang`の導入

   ```md
   # go の最新版を取得する

   バージョンは適宜置き換え

   wget https://dl.google.com/go/go1.11.5.linux-amd64.tar.gz

   # 解凍する

   sudo tar -C /usr/local -xzf go1.11.5.linux-amd64.tar.gz

   # .fish に追加

   export PATH=$PATH:/usr/local/go/bin
   ```

1. `npm`, `node.js`の導入

1. `workspace` ディレクトリを作成
