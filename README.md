# `Kibana`-`Elasticsearch`-`Docker`

## 概要

Kibana + Elasticsearch を Docker で起動する。  
過去に取得していたログをグラフィカルに参照できることを目的にするため、Elasticsearch へのデータ投入は、 `Embulk` を使用する。
Embulk は Fluentd のバッチ版とも呼ばれ、 Fluentd がリアルタイムにデータを収集することを得意とすることに対し、 Embluk は溜め込んだデータを取り込むことを得意とする。
