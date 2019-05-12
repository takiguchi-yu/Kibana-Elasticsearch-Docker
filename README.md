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
$ sudo yum install -y java-1.8.0-openjdk
$ sudo yum install -y java-1.8.0-openjdk-devel
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

macOSの場合は、上記のやり方ではなく、brew で簡単インストールしても良い
```
$ brew install embulk
```

### プラグインをインストール
* embulk-output-elasticsearch
  * Elasticsearch にデータをロードするプラグイン  
  https://github.com/kakoni/embulk-parser-fixed
* embulk-parser-fixed
  * 固定長データをパースするプラグイン  
  https://www.rubydoc.info/gems/embulk-output-elasticsearch/
* embulk-input-sftp
  * SFTPでデータを取得するプラグイン  
  https://www.rubydoc.info/gems/embulk-input-sftp/
* embulk-input-remote
  * scpで複数のホストから指定したパスにあるファイル読み込むプラグイン

```
$ embulk gem install embulk-output-elasticsearch
$ embulk gem install embulk-parser-fixed
$ embulk gem install embulk-input-sftp
$ embulk gem install embulk-input-remote
```
embulk のプラグイン関連は以下を参考
https://qiita.com/hiroysato/items/da45e52fb79c39547f69

## 実行例1
ローカルに置いてあるログを Elasticsearch にロードし、Kibana で参照する。
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

### 動作確認
ちょっと汚いグラフだが、以下のような感じでグラフ化
<img width="1481" alt="スクリーンショット 2019-04-29 20 14 56" src="https://user-images.githubusercontent.com/8340629/56892689-60faf980-6abb-11e9-83ec-9c53e645ca58.png">

## 実行例2
サーバ上のファイルを sftp で取得し、さらにまた別サーバの Elasticsearch にロードする。

<img width="745" alt="スクリーンショット 2019-04-30 11 17 09" src="https://user-images.githubusercontent.com/8340629/56937750-67c85180-6b39-11e9-901c-11c8ce7a7044.png">

config.yml を以下のように変更し、embulk を実行する。
```
config.yml
in:
  type: sftp
  host: XXXXX1.com # リモートサーバ1 のホスト名
  port: 22
  user: XXXX-user # リモートサーバ1 のユーザー名
  secret_key_file: /Users/hoge/XXXX.pem # リモートサーバ1 の鍵(ローカル上のパスを指定)
  path_prefix: /vmstat # ユーザーrootディレクトリ以降を指定（左記は/home/XXXX-user/vmstatと同意義）
  decoders:
  - {type: gzip}
  parser:
    type: fixed
    columns:
    - {name: time, type: string, pos: 0..19}
    - {name: second, type: long, pos: 32..39}
out:
  type: elasticsearch
  mode: replace
  nodes:
  - {host: XXXX2.com, port: 9200} # リモートサーバ2 のホスト名
  index: vmstat-log
  index_type: vmstat
```

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
$ sudo gpasswd -a $USER docker
$ sudo systemctl restart docker
$ exit
# ログインし直す
```

## 【参考】エラーケース
### VM 上で Elasticsearch が起動しない場合
#### FDエラーへの対応
```
max file descriptors [4096] for elasticsearch process is too low, increase to at least [65536]
```
docker-compose の ulimits を設定
```
ulimits:
  nofile:
    soft: 65536
    hard: 65536
```
参考：https://qiita.com/waytoa/items/6adbe4bdd5628419ecbf
#### メモリエラーへの対応
```
max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
```
カーネルパラメータを設定
```
$ sudo vi /etc/sysctl.d/99-sysctl.conf
# 以下を追記
vm.max_map_count = 262144

# 設定を適用
$ sudo sysctl --system
```

## 【参考】EC2 を再起動しても自動起動するようにする
### 自動起動シェルを作成
#### run-kibana.sh
```
#!bin/sh

# ホスト名を設定
sudo sed -i 's/^HOSTNAME=[a-zA-Z0-9\.\-]*$/HOSTNAME=kibana-ec2/g' /etc/sysconfig/network
sudo hostname 'kibana-ec2'

# docker グループ追加
sudo gpasswd -a $USER docker
sudo systemctl restart docker

# docker-compose 起動
cd /home/ec2-user/kibana
/usr/local/bin/docker-compose up -d

exit 0
```

### crontab 設定
```
$ crontab -e
@reboot /bin/sh /home/ec2-user/kibana/run-kibana.sh
```

## 【参考】input の Epoch を日時変換する
timezone 指定 且つ format を '%s' としておけばOK
```
in:
parser:
   default_timezone: 'Asia/Tokyo'
   columns:
   - {name: timestamp, type: timestamp, format: '%s'}
```
