# postfix for Docker

---

## 概要

安全な接続（SSL / TLS）を使ってメールを送信できます。
メールがスパムとしてマークされないようにするために、事前に DNS サーバーに SPF レコードを追加する必要があります。

## コンテナイメージ作成

```sh
$ docker build -t akazawa/postfix ./
```

## コンテナ起動

```sh
$ docker run -d --name postfix -e TIMEZONE=Asia/Tokyo \
     -e MESSAGE_SIZE_LIMIT=10240000 -e AUTH_USER=user \
     -e AUTH_PASSWORD=password \
     -e DISABLE_SMTP_AUTH_ON_PORT_25=true \
     -p 8587:587 -p 8465:465 --privileged \
     akazawa/postfix
```

## Port No.

通常は送信ポート 587 を使用できます。
メールクライアントが SMTPS（SMTP over SSL）を必要とする場合はポート 465 を使用し、表示された証明書の警告を無視します。

## Logging

このコンテナは、失敗して成功したすべての配信を「docker logs」に記録します。ログをリアルタイムで表示するには、次のコマンドを使用します。

```sh
$ docker logs -f postfix
```

## Usage

[sendjpmail.sh](https://github.com/nanaka-inside/C86/blob/master/richmikan/chap_sendmail.rst)のスクリプトを利用してメールを配信できます。
引数にメールのテンプレートファイルを用意してください。

```sh
$ /work/sendjpmail.sh　<ファイル名>
```

## メールテンプレートファイルの中身

```sh
From: XXXXXXX <XXXXXXXXXXX@XXXXXXXX-service.jp>
To: XXXXXXXXXXXX@XXXXX.com
Subject: テストメール
Content-Type: text/plain;charset="UTF-8"
Content-Transfer-Encoding: base64

テストメール本文

```

---

## POSTFIX コマンド一覧

キューの確認

```
# postqueue -p
```

特定のキューの削除

```
# postsuper -d <キューID>
```

全てのキューの削除

```
# postsuper -d ALL
```

強制再送

```
# postfix flush
```

キューのファイルを書き出す

```
# postqueue -p > /path/to/mailq.txt
```

キューの中身をキュー ID から確認する

```
# postcat -q [queue_id]
```

キューを強制配送する

```
# postqueue -f
```

特定のキューを削除する

```
# postsuper -d [queue_id]
```

キューを全て削除する

```
# postsuper -d ALL
```

キューを再送する

```
# postfix flush
```

Postfix のメールログ解析ツールのインストール

```
# yum --enablerepo=centosplus install postfix-perl-scripts
```

メールログ解析ツールを実行する

```
# pflogsumm /var/log/maillog
```

#### 参考：[メールキューの確認と削除・強制再送](https://qiita.com/hanko_pettanko/items/880d8d5bc8ed37ea88df)

#### 参考：[Postfix ログ解析 pflogsumm](https://blog.bungu-do.jp/archives/2034)
