## 2時間でマスターしちゃう Docker＆Github ハンズオン 入門

### docker  
- スクラップビルドが容易　
- 軽い　
- 名称空間が独立
- プロジェクトの切り替えが簡単
- どのようなマシンでも環境に関係なく動く
- 共有

### dockerとVMの違い
- VM : pc全体をエミュレート
- docker : ミドルウェアだけエミュレート

### docker備考
- dockerをstopした状態でないと削除できない
- コンテナとイメージの違い
- runとbuildの違い
- 各ミドルウェアを管理する方法がある
- docker-compose down : コンテナを止めてコンテナを消す（imageは消えない）

### dockerコマンド
- docker pull ????? : ????? imageを落とす
- docker images :
- docker run -d -e MYSQL_ROOT_PASSWORD=password --name mysql5.6 mysql:5.6
- docker ps
- docker stop コンテナID
- docker ps -a
- docker start コンテナID
- docker rm コンテナID
- docker rmi イメージID
- docker run -d -p 80:80 --name php php:latest
- docker-compose stop
- docker-compose start
- docker-compose down
- docker-compose restart
