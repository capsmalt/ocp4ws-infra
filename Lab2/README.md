

# OpenShift Infra Lab02
　OpenShiftクラスターに付随しているOperatorHubから，PostgreSQLを活用して，Community版のOperatorのインストール手順，付属CLIの利用方法を学ぶ。

## 概要
　
GitHubで公開されているOSSの [CrunchyData/postgres-operator](https://github.com/CrunchyData/postgres-operator) を使用して，OCP上でPostgreSQLを制御する。具体的には，OCP上のPostgreSQL OperatorのPodに対して，クライアント環境からpgo(PostgreSQL Operatorを制御するCLI)で接続して制御を行う。Operatorのインストール手順と，その制御例について学ぶ。

![Lab02のワークショップ概要](画像のURL "ワークショップ概要")

## 前提条件
- OpenShift Container Platform 4.Xのデプロイメント
- oc，kubectlコマンドのセットアップ  
- pgo コマンドのセットアップ(パスが通っていればOK)
- system:adminの権限利用

### 実装手順
作業は以下の手順どおりに進める。

1. [Crunchy PostgreSQL Operatorのインストール](1_installtion-postgres-operator-pgo.md)  
2. [pgoの構成とPostgreSQLリソース制御](2_usage-pgo.md)  

## References

* [GitHub] - CrunchyData/postgres-operator  
https://github.com/CrunchyData/postgres-operator
* [Docs] - Crunchy Data PostgreSQL Operator
https://access.crunchydata.com/documentation/postgres-operator/4.0.1/
* [Operator Hub] - Crunchy PostgreSQL Enterprise  
https://operatorhub.io/operator/postgresql
