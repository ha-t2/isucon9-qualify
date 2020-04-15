# isucon9-qualify

これは[isucon9-qualify](https://github.com/isucon/isucon9-qualify)を微妙に改造したものです。ご注意ください。  
詳しいルールは[manual](./docs/manual.md)を参考にしてください。


## ディレクトリ構成

```
├── bin          # 外部サービスやベンチマーカーがおいてある
├── docs         # 運営が用意した各種ドキュメント
├── initial-data # 初期データ作成
└── webapp       # 各言語の参考実装
```

## アプリケーションおよびベンチマーカーの起動方法

こちらのblogでも紹介しています。参考にしてください
http://isucon.net/archives/53805209.html

## 前準備

```

# 初期データ作成
cd initial-data
make

# 初期画像データダウンロード.　重たいのでVPNを切ってダウンロードしてください。
cd webapp/public
# https://github.com/isucon/isucon9-qualify/releases から initial.zip をダウンロード
unzip initial.zip
rm -rf upload
mv v3_initial_data upload

# ベンチマーク用画像データダウンロード。重たいのでVPNを切ってダウンロードしてください。
cd initial-data
# https://github.com/isucon/isucon9-qualify/releases から bench1.zip をダウンロード
unzip bench1.zip
rm -rf images
mv v3_bench1 images
```

## ベンチマーカー

### 実行オプション

```
$ ./bin/benchmarker -help
Usage of isucon9q:
  -allowed-ips string
        allowed ips (comma separated)
  -data-dir string
        data directory (default "initial-data")
  -payment-port int
        payment service port (default 5555)
  -payment-url string
        payment url (default "http://localhost:5555")
  -shipment-port int
        shipment service port (default 7000)
  -shipment-url string
        shipment url (default "http://localhost:7000")
  -static-dir string
        static file directory (default "webapp/public/static")
  -target-host string
        target host (default "isucon9.catatsuy.org")
  -target-url string
        target url (default "http://127.0.0.1:8000")
```

  * HTTPとHTTPSに両対応
    * 証明書を検証するのでHTTPSは面倒
  * 外部サービス2つを自前で起動するので、いい感じにするならnginxを立てている必要がある
  * nginxでいい感じにするなら以下の設定が必須
    * `proxy_set_header Host $http_host;`
      * shipmentのみ必須
    * `proxy_set_header X-Forwarded-Proto "https";`
      * HTTPSでないなら不要
    * `proxy_set_header True-Client-IP $remote_addr;`
    * cf: https://github.com/isucon/isucon9-qualify/tree/master/provisioning/roles/external.nginx/files/etc/nginx


## 外部サービス

決済サービス`payment`と配送サービス`shipment`がある。  
両者とも画面を動かすには必要だが、ベンチマークには不要。  
`./bin`以下にバイナリがある。

### 注意点

nginxでいい感じにするなら以下の設定が必須

  * `proxy_set_header Host $http_host;`
    * shipmentのみ必須
  * `proxy_set_header X-Forwarded-Proto "https";`
    * HTTPSでないなら不要

## webapp 起動方法

```shell-session
# 決済サービスと配送サービスを起動。以下２つは別ターミナルで
./bin/payment  # listen on port 5555
./bin/shipment # listen on port 7000

# mysqlを起動
cd webapp
docker-compose up -d

# ちょっと立ち上がるのを待つ
# データを流し込む
cd webapp/sql
./init.sh

cd webapp/go
make
./isucari
open http://localhost:8000/
```

### ベンチマークのとり方
```shell-session
# paymentとshipmentだけ落として
./bin/benchmarker
```


# その他
## 運営側のブログ

技術情報などについても記載されているので参考にしてください。

  * ISUCON9予選の出題と外部サービス・ベンチマーカーについて - catatsuy - Medium https://medium.com/@catatsuy/isucon9-qualify-969c3abdf011
  * ISUCONのベンチマーカーとGo https://gist.github.com/catatsuy/74cd66e9ff69d7da0ff3311e9dcd81fa
  * ISUCON9予選でフロントエンド周りの実装を担当した話 - はらへり日記 https://sota1235.hatenablog.com/entry/2019/10/07/110500

## サポートするMySQLのバージョン

MySQL 5.7および8.0にて動作確認しています。

ただし、nodejsでアプケーションを起動する場合、MySQL 8.0の認証方式によっては動作しないことがあります。
詳しくは、 https://github.com/isucon/isucon9-qualify/pull/316 を参考にしてください


## 使用データの取得元

- なんちゃって個人情報 http://kazina.com/dummy/
- 椅子画像提供 941-san https://twitter.com/941/status/1157193422127505412
