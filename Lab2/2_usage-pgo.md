
# 2. pgoの構成とPostgreSQLリソース制御  

## 2-1. 諸注意

### 2-1-1. pgoについて

*   pgoは，Postgres Operatorを操作・制御するためのクライアント用CLI。pgoはOperator Podである "apiserver" と通信することで，K8s上のPostgreSQLを制御する。    
    * PostgreSQLクラスター構築/削除
    * Posgresインスタンスのスケールアウト/スケールイン
    * Posgresインスタンスバックアップ/リストア
    * 上記のような制御を pgo CLIから行うことが可能

### 2-1-2. 事前準備
* Postgres OperatorをOCP上で動作させておく
* 踏み台サーバーで oc / kubectl / pgo が実行できること

## 2-2. pgoの構成

### 2-2-1. Operatorをサービス公開
まずはOperator Podに接続するためのServiceを作成してtype:LoadBalancerで公開。 
```
oc expose deployment -n pgo postgres-operator --type=LoadBalancer
```

```
oc get svc -n pgo

postgres-operator                LoadBalancer   172.30.77.27     a8a59cbf2b73d11e99d19066aaaf6145-115501653.ap-northeast-1.elb.amazonaws.com   8443:30861/TCP                                  9h
```

上記例の場合，
`a8a59cbf2b73d11e99d19066aaaf6145-115501653.ap-northeast-1.elb.amazonaws.com` をメモしておきます。

### 2-2-2. pgo(Client用CLI)の展開
Operator PodのapiserverのPod名を取得。

```
oc get po -n pgo

postgres-operator-9777dbc48-59kms                 3/3     Running     0          9h
```

上記結果のPod名を使用して，apiserverの証明書を取得。
```
oc cp pgo/postgres-operator-9777dbc48-59kms:/tmp/server.key /tmp/server.key -c apiserver

oc cp pgo/postgres-operator-9777dbc48-59kms:/tmp/server.crt /tmp/server.crt -c apiserver
```

pgo用の証明書のパスをexport。
```
export PGO_CA_CERT=/tmp/server.crt
export PGO_CLIENT_CERT=/tmp/server.crt
export PGO_CLIENT_KEY=/tmp/server.key
```

pgouserのユーザー名・パスワードを設定しexport。
```
echo username:password > $HOME/pgouser
chmod 700 ~/pgouser
export PGOUSER=$HOME/pgouser
```

Operatorのapiserverの公開リンクを指定。

```
oc get svc -n pgo

NAME                             TYPE           CLUSTER-IP       EXTERNAL-IP                                                                   PORT(S)                                         AGE
postgres-operator                LoadBalancer   172.30.77.27     a8a59cbf2b73d11e99d19066aaaf6145-115501653.ap-northeast-1.elb.amazonaws.com   8443:30861/TCP 
                                 9h
↓↓↓ PGO_APISERVER_URLに指定する

export PGO_APISERVER_URL=https://aeb4c262bb73211e9b7180eb47beb561-2052260300.ap-northeast-1.elb.amazonaws.com:8443
pgo version
```

### 2-2-3. pgo(Client用CLI)の動作確認
正常にpgo(Client)からapiserver(Operator Pod)に接続できるか確認します。
```
pgo version

pgo client version 4.0.1
pgo-apiserver version 4.0.1
```

上記のように出力されれば成功です。

## 2-3. PostgreSQLリソース制御
pgoから様々なリソースを制御してみよう。

### 2-3-1. PostgreSQLクラスターの作成

対象となるNamespaceを指定。
```
export PGO_OPERATOR_NAMESPACE=pgo
export NAMESPACE=pgo
```

PostgreSQLクラスターを作成。
```
pgo create cluster mycluster -n pgo
pgo show cluster mycluster -n pgo
```

Pgclusterリソースを確認。
```
oc get Pgclusters
NAME        AGE
mycluster   17m
```

Postgres関連のPodを確認。
```
oc get pods -n pgo
    mycluster-6c5b4ddc6-qq5zg                         1/1     Running     0          5m13s
    mycluster-backrest-shared-repo-668554dc6c-mvbjg   1/1     Running     0          5m13s
    mycluster-stanza-create-bh6lf                     0/1     Completed   0          4m6s
    postgres-operator-9777dbc48-59kms                 3/3     Running     0          25m
```

Postgresの動作確認。
```
pgo test mycluster -n pgo

    cluster : mycluster
	    psql -p 5432 -h 172.30.254.147 -U postgres postgres is Working
	    psql -p 5432 -h 172.30.254.147 -U postgres userdb is Working
	    psql -p 5432 -h 172.30.254.147 -U primaryuser postgres is Working
	    psql -p 5432 -h 172.30.254.147 -U primaryuser userdb is Working
	    psql -p 5432 -h 172.30.254.147 -U testuser postgres is Working
	    psql -p 5432 -h 172.30.254.147 -U testuser userdb is Working
```

ついでにServiceも確認。
```
oc get svc
NAME                             TYPE           CLUSTER-IP       EXTERNAL-IP                                                                   PORT(S)                                         AGE
mycluster                        ClusterIP      172.30.254.147   <none>                                                                        5432/TCP,9100/TCP,10000/TCP,2022/TCP,9187/TCP   7m49s
mycluster-backrest-shared-repo   ClusterIP      172.30.224.82    <none>                                                                        2022/TCP                                        7m49s
postgres-operator                LoadBalancer   172.30.77.27     a8a59cbf2b73d11e99d19066aaaf6145-115501653.ap-northeast-1.elb.amazonaws.com   8443:30861/TCP                                  26m
```

### 2-3-2. Postgresをスケーリング
PgreplicasリソースでPodを追加。
```
pgo scale mycluster -n pgo

WARNING: Are you sure? (yes/no): yes
created Pgreplica mycluster-hrbx
```

Pgreplicasリソースを確認。
```
oc get Pgreplicas

NAME             AGE
mycluster-hrbx   8m45s
```

PostgresのReplica Podを確認。
```
oc get pods
NAME                                              READY   STATUS      RESTARTS   AGE
mycluster-6c5b4ddc6-qq5zg                         1/1     Running     0          10m
mycluster-backrest-shared-repo-668554dc6c-mvbjg   1/1     Running     0          10m
mycluster-hrbx-7d9f8b569b-mvfb6                   1/1     Running     0          67s
mycluster-stanza-create-bh6lf                     0/1     Completed   0          9m46s
postgres-operator-9777dbc48-59kms                 3/3     Running     0          31m
```

### 2-3-3. その他

Postgresのバックアップを確認。
```
pgo backup mycluster -n pgo

created Pgtask backrest-backup-mycluster
```

```
oc get Pgtask

NAME                        AGE
backrest-backup-mycluster   9s
mycluster-createcluster     53m
mycluster-stanza-create     52m
```

```
pgo backup mycluster --backup-type=pgbasebackup -n pgo

created backup Job for mycluster
workflow id 2176b3ad-9666-41bd-91df-081f911493f0
```

```
oc get Pgtask

NAME                        AGE
backrest-backup-mycluster   91s
mycluster-backupworkflow    5s
mycluster-createcluster     54m
mycluster-stanza-create     53m
```

```
oc get pods

NAME                                              READY   STATUS              RESTARTS   AGE
backrest-backup-mycluster-6mmcg                   0/1     Completed           0          103s
backup-mycluster-cqdj-qwzl7                       0/1     ContainerCreating   0          14s
mycluster-6c5b4ddc6-qq5zg                         1/1     Running             0          54m
mycluster-backrest-shared-repo-668554dc6c-mvbjg   1/1     Running             0          54m
mycluster-hrbx-7d9f8b569b-mvfb6                   1/1     Running             0          45m
mycluster-stanza-create-bh6lf                     0/1     Completed           0          53m
postgres-operator-9777dbc48-59kms                 3/3     Running             0          75m
```

```
oc get pods

NAME                                              READY   STATUS      RESTARTS   AGE
backrest-backup-mycluster-6mmcg                   0/1     Completed   0          2m21s
backup-mycluster-cqdj-qwzl7                       0/1     Completed   0          52s
mycluster-6c5b4ddc6-qq5zg                         1/1     Running     0          55m
mycluster-backrest-shared-repo-668554dc6c-mvbjg   1/1     Running     0          55m
mycluster-hrbx-7d9f8b569b-mvfb6                   1/1     Running     0          45m
mycluster-stanza-create-bh6lf                     0/1     Completed   0          54m
postgres-operator-9777dbc48-59kms                 3/3     Running     0          75m
```

ログを確認。
```
pgo ls mycluster -n pgo /pgdata/mycluster/pg_log

total 60K
-rw-------. 1 postgres root 53K Aug  5 05:28 postgresql-Mon.log


pgo cat mycluster -n pgo /pgdata/mycluster/pg_log/postgresql-Mon.log | tail -3

2019-08-05 05:29:38 UTC [1022]: [3-1] user=postgres,db=postgres,app=psql,client=[local]LOG:  duration: 0.279 ms
2019-08-05 05:29:38 UTC [1022]: [4-1] user=postgres,db=postgres,app=psql,client=[local]LOG:  disconnection: session time: 0:00:00.002 user=postgres database=postgres host=[local]
```

PVCに対するPostgresの利用状況の確認
```
pgo df mycluster -n pgo

POD                       STATUS    PGSIZE    CAPACITY  PCTUSED

mycluster-6c5b4ddc6-qq5zg up        30 MB     1Gi       2
```

以下を参考に他にも色々試してみよう。  
↓  
https://access.crunchydata.com/documentation/postgres-operator/4.0.1/operatorcli/pgo-overview/


