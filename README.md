# postfix for Docker

---

## 概要

安全な接続（SSL / TLS）を使ってメールを送信できます。
メールがスパムとしてマークされないようにするために、事前に DNS サーバーに SPF レコードを追加する必要があります。

## コンテナイメージ作成

```sh
$ docker build -t akazawa_postfix ./
```

## コンテナ起動

```sh
$ docker run -d --name akazawa_postfix -e TIMEZONE=Asia/Tokyo -e MESSAGE_SIZE_LIMIT=10240000 -e AUTH_USER=user -e AUTH_PASSWORD=password -e DISABLE_SMTP_AUTH_ON_PORT_25=true -p 8587:587 -p 8465:465 -p 8082:8080 --privileged akazawa_postfix
```

## Port No.
通常はポート　25を利用しますが、送信ポート 587 も使用することができます。
サブミッションポートを使用する場合には「ID」と「パスワード」による認証（smtp auth）が必要になります。
メールクライアントが SMTPS（SMTP over SSL）を必要とする場合はポート 465 を使用し、表示された証明書の警告を無視します。

### postfix設定

| 設定項目 | 設定箇所 | 設定内容 | 説明 |
| :--- | :--- | :--- | :--- |
| ホスト名 |myhostname|mail.netbranch-service.net|ホスト名をFQDN形式（ホスト名+ドメイン名）で設定|
| ドメイン |mydomain|netbranch-service.net|ドメイン名を設定|
| メールアドレスの@以下ドメイン名 |myorigin|$mydomain|ローカルから送信されたメールのメールアドレス@以下の内容を指定|
| メール受付NIC |inet_interfaces|all|メールの送受信を行うインターフェイスを指定|
| ローカル配送するメール指定 |mydestination|「localhost.netbranch-service.net」「localhost」「netbranch-service.net」|ローカルユーザに配信するメールアドレスを設定|
| メール送信許可ネットワーク |mynetworks|「192.168.1.0/24」と「127.0.0.1/8(ローカルホスト)」ネットワークに属しているクラインアントからはメール送信を無条件許可|メール送信を許可するクライアントのIP情報を設定|
| ローカルユーザに配送したメールデータの保存形式 |home_mailbox|Maildir/ (Maildir形式)|受信メールのデータ形式を指定<br>mbox形式ならばMailbox Maildir形式ならばMaildir/を指定|

### リレー設定
コメントで無効化されている「relayhost」の項目を編集して、メールをリレー（転送）するsmtpサーバを設定します。
smtpサーバはプロバイダ等で指定されているサーバを設定し、ポート番号はサブミッションポートである「587」番を設定します。
```
relayhost = [smtpサーバ]:587
```

### SMTP認証と暗号化設定
リレー先のsmtpサーバのサブミッションポートを使用するに必要な、認証を行うための設定をします。
「main.cf」を下記のように編集します。
　※当Dockerファイルは既に設定済です。
 ```
 # SMTP認証設定
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:/etc/postfix/smtp_pass
smtp_sasl_tls_security_options = noanonymous
smtp_sasl_mechanism_filter = plain,login

# 暗号化設定
smtp_tls_security_level = may
smtp_tls_loglevel = 1
smtp_tls_CApath = /etc/pki/tls/certs/ca-bundle.crt
```

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
