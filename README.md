# `Kibana`-`Elasticsearch`-`Docker`

## 概要

Kibana + Elasticsearch を Docker で起動する。  
過去に取得していたログをグラフィカルに参照できることを目的にするため、Elasticsearch へのデータ投入は、 `Embulk` を使用する。  
Embulk は Fluentd のバッチ版とも呼ばれ、 Fluentd がリアルタイムにデータを収集することを得意とすることに対し、 Embluk は溜め込んだデータを取り込むことを得意とする。

## 環境

* Kibana 7.0
* Elasticsearch 7.0
* Docker x.x
* Docker Compose x.x  
* Embulk x.x 

## 前提

* macOS Mojave 10.14.4 
* Docker for mac がインストールされていること

## Kibana + Elasticsearch をインストール
docker-compose で起動する。

## Embulk を インストール

## サンプルログを取り込む

## 動作確認
