# aws-getcost-zabbix-template

## 概要

 Zabbix(zabbix-agentとaws-cli)を利用し、AWSのコストをモニタリングする。


## 注意事項

 * AWSのCostExplorerを利用するためコストが発生します。
 * 現時点では、CostExplorerを実行するたびに、0.01USD発生。
 * 当機能を使用すると、1日に2回CostExplorerを呼び出すため、0.02USDの費用が発生します。

## 内容

作成物は以下の2ファイル 
 * zabbix-agentの設定ファイル
 * zabbixのテンプレート

実現内容は以下の2項目
* AWSのトータルコストと各サービスのコストをモニタリング(1日1回)
* トータルコストに対して、閾値(WARN/HIGH)のトリガーを設定

## 稼働条件

* zabbixサーバ(AWSCLIが動くこと)が存在すること
* 上記サーバでjqコマンドが使えること

## 導入手順

### AWSCLI導入(既に導入されていれば不要)

```
# pip3 install awscli
```

### zabbixユーザでのAWSコンフィグ設定(既に設定されていれば不要)

#### zabbixユーザのホームディレクトリのパスを調べる

```
# egrep zabbix /etc/passwd
zabbix:x:2000:2000:Zabbix:/var/lib/zabbix:/sbin/nologin
```

#### zabbixユーザのホームディレクトリ(上記例で/var/lib/zabbix)が無い場合は作成する

```
# mkdir /var/lib/zabbix
```

#### AWSコンフィグ用のファイルを設置する

```
# mkdir /var/lib/zabbix/.aws
# cat > /var/lib/zabbix/.aws/config 
[default]
output=json
^D
# cat > /var/lib/zabbix/.aws/credentials 
[default]
aws_access_key_id=XXXXXXXXXXXXXXXXXXXX
aws_secret_access_key=XXXXXXXXXXXXXXXXXXXX
^D
# chown -R zabbix:zabbix /var/lib/zabbix/.aws
```

### zabbix_angentファイルを/etc/zabbix/zabbix_agentd.d/配下に設置

```
# cp zabbix_template/zbx_AWSCost_templates.xml /etc/zabbix/zabbix_agentd.d/.
# systemctl restart zabbix-agent.service 
```

### zabbix_agentの設定ができていることを確認する
#### zabbix_agentその1: aws.getcost
```
# zabbix_get -s 127.0.0.1 -k "aws.getcost"
{"AWS Config":"0"
,"AWS Cost Explorer":"0"
,"AWS Elemental MediaStore":"0"
,"AWS Key Management Service":"0"
,"AWS Lambda":"0"
,"AWS Systems Manager":"0"
,"Amazon API Gateway":"0"
,"Amazon CloudFront":"0"
,"EC2 - Other":"0"
,"Amazon Elastic Compute Cloud - Compute":"0"
,"Amazon Elastic Load Balancing":"0"
,"Amazon Simple Notification Service":"0"
,"Amazon Simple Queue Service":"0"
,"Amazon Simple Storage Service":"0"
,"Amazon Virtual Private Cloud":"0"
,"AmazonCloudWatch":"0"
,"Tax":"0"
}
```
#### zabbix_agentその2: aws.getcosttotal
```
# zabbix_get -s 127.0.0.1 -k "aws.getcosttotal"
{"Total":"0"}
```

### zabbix_templateファイルをZABBIXにインポート

必要に応じて、template内のMacrosの内容を変更する

| Macro	| Value	| 備考 |
| ----- | ----- | ----- |
| {$BUDGET_HIGH} | 100 | コストの閾値 100USD超えればアラート（深刻度=HIGH) |
| {$BUDGET_WARNING} | 10 | コストの閾値 10USD超えればアラート（深刻度=WARNING) |
| {$COSTCHECK_DATETIME} | h11m00 | Cost  Explorerを呼び出す時間 (h11m00 = 11:00) |

> 注意： アイテム収集間隔(interval)を短くすると、コストが上がります。
>   テンプレートデフォルトでは、1日2回AWSCLI実行するので、0.02USD発生します。
