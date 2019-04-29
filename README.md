# `Kibana`-`Elasticsearch`-`Docker` + `Embulk`

## 概要
`Embulk` を使ってデータを Elasticsearch に登録し、 Kibana で参照する。
Kibana + Elasticsearch は Docker で起動する。  
`Embulk` は Fluentd のバッチ版とも呼ばれ、 Fluentd がリアルタイムにデータを取り込むのが得意なのに対し、 `Embulk` はある一定量のデータをまとめて取り込むことのが得意。

## 環境
2019/05/01時点
* Kibana 7.0.0
* Elasticsearch 7.0.0
* Docker 18.09.2
* Docker Compose 1.23.2  
* Embulk 0.9.17

<img width="648" alt="スクリーンショット 2019-04-28 1 00 08" src="https://user-images.githubusercontent.com/8340629/56852071-5e7c9080-6951-11e9-98f9-17bd0333430e.png">

## 前提

* macOS Mojave 10.14.4
* Docker for mac がインストールされていること

## Kibana + Elasticsearch をインストール
docker-compose で起動する。  
Elasticsearch がメモリを食うため、 Docker のメモリ割り当てを増やしておくことを薦める。

```
$ git clone https://github.com/yu-dai/Kibana-Elasticsearch-Docker.git
$ cd Kibana-Elasticsearch-Docker
$ docker-compose up -d
```

docker-compose.yml は、以下のリファレンスをもとに作成。  
* Elasticsearch  
https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html  
* Kibana  
https://www.elastic.co/guide/en/kibana/current/docker.html  

Kibana が起動したら以下にアクセス。  
http://localhost:5601  

## Embulk を インストール
Embulk v0.9シリーズはJava 8上で動作するため、今回は Java8 をインストールした後、 Embulk をインストールする。
Java9 以降は当面の間どのバージョンでもサポートされないとのこと。

* Embulk  
https://github.com/embulk/embulk

```
# java をインストール
$ sudo yum install java-1.8.0-openjdk
$ sudo yum install java-1.8.0-openjdk-devel
$ java -version
openjdk version "1.8.0_201"
OpenJDK Runtime Environment (build 1.8.0_201-b09)
OpenJDK 64-Bit Server VM (build 25.201-b09, mixed mode)

# Embulk をインストール
curl --create-dirs -o ~/.embulk/bin/embulk -L "https://dl.embulk.org/embulk-latest.jar"
chmod +x ~/.embulk/bin/embulk
echo 'export PATH="$HOME/.embulk/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

$ embulk -version
embulk 0.9.17
```
### プラグインをインストール
* embulk-output-elasticsearch
  * Elasticsearch にロードする出力プラグイン  
  https://github.com/kakoni/embulk-parser-fixed
* embulk-parser-fixed
  * 固定長データをパースするプラグイン  
  https://www.rubydoc.info/gems/embulk-output-elasticsearch/

```
$ embulk gem install embulk-output-elasticsearch
$ embulk gem install embulk-parser-fixed
```

## 実行例
### 元ファイルを準備
1分間隔で毎時 vmstat の結果をファイルに出力
＜＜要見直し＞＞
```
$ crontab -e
*/1 * * * * (echo -n `date "+\%Y/\%m/\%d \%H:\%M:\%S "`; vmstat | tail -n 1) >> /home/XXXXXX/vmstat/vmstat-`date +\%Y\%m\%d\%H`.log
```
### embulk を実行
```
# Dry run
$ embulk preview config.yml

# Run
$ embulk run config.yml
```

## 動作確認
ちょっと汚いグラフだが、以下のような感じでグラフ化
<img width="1481" alt="スクリーンショット 2019-04-29 20 14 56" src="https://user-images.githubusercontent.com/8340629/56892689-60faf980-6abb-11e9-83ec-9c53e645ca58.png">


## 【参考】Linux に Docker をインストール
### docker インストール
```
$ sudo yum install -y docker
$ docker -v
Docker version 18.06.1-ce, build e68fc7a215d7133c34aa18e3b72b4a21fd0c6136
```
### docker-compose インストール
公式：https://docs.docker.com/compose/install/
```
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.24.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

$ sudo chmod +x /usr/local/bin/docker-compose
$ docker-compose -v
docker-compose version 1.24.0, build 0aa59064

# このままだと実行できないので、dockerグループに追加
$ sudo groupadd docker
$ sudo gpasswd -a $USER docker
$ sudo systemctl restart docker
$ exit
# ログインし直す
```
