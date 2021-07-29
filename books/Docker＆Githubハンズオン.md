## 2 時間でマスターしちゃう Docker＆Github ハンズオン 入門

### docker

- スクラップビルドが容易
- 軽い
- 名称空間が独立
- プロジェクトの切り替えが簡単
- どのようなマシンでも環境に関係なく動く
- 共有

### docker と VM の違い

- VM : pc 全体をエミュレート
- docker : ミドルウェアだけエミュレート

### docker 備考

- docker を stop した状態でないと削除できない
- コンテナとイメージの違い
- run と build の違い
- 各ミドルウェアを管理する方法がある
- docker-compose down : コンテナを止めてコンテナを消す（image は消えない）

### docker コマンド

- docker pull ????? : ????? image を落とす
- docker images :
- docker run -d -e MYSQL_ROOT_PASSWORD=password --name mysql5.6 mysql:5.6
- docker ps
- docker stop コンテナ ID
- docker ps -a
- docker start コンテナ ID
- docker rm コンテナ ID
- docker rmi イメージ ID
- docker run -d -p 80:80 --name php php:latest
- docker-compose stop
- docker-compose start
- docker-compose down
- docker-compose restart
