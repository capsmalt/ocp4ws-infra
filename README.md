# Red Hat OpenShift 4 ワークショップ <br> -インフラ編-

本ハンズオンは，OpenShift4(以降，OCPまたはOCP4)のインフラ編です。
Operatorによるミドルウェア運用について触れて頂きます。

## 諸連絡
**OCPクラスター接続情報など (Etherpad) ==>**

## タイムテーブル

|Time|Agenda|Content|
|:---:|:---|:---|
|13:00-13:30|受付||
|13:30-14:00|<講義><br>OCP4 概要 & Operator 概要|OCP基礎 <br> Controller/Resource基礎 <br> Operator基礎|
|14:00-14:20|OCP4クラスター接続確認||
|14:20-14:30|<講義><br>Prometheus Operator||
|14:30-15:30|<ハンズオン><br>アプリ と Operator 連携|Prometheus Operator|
|15:30-15:45|Break||
|15:45-16:30|<ハンズオン><br>Operatorの導入 と CLI活用|PostgreSQL Operator|
|16:30-17:00|QA, Closing||

## ハンズオン環境および前提
本ハンズオンは，Kubernetesクラスター(OpenShift)の動作環境としてAWSを使用します。今回は構築済です。

OCPクラスターに対するCLI操作をを行う際は，クライアントPCから，踏み台サーバー(Bastion Server)にSSH接続し，**ocコマンド** を使って制御します。  
`クライントPC <--SSH--> 踏み台サーバー <--oc--> OpenShift クラスター`

GUI操作は，クライアントPCのブラウザ(**Chrome/Firefox推奨**)を使用します。  
`クライアントPC <--Chrome/Firefox--> OpenShift Portal`

このため以下の準備を事前に済ませておいてください。
- SSH用ツール
- ブラウザ (Google Chrome or Firefox)

## ハンズオン
**Lab1: [アプリケーション と Operator 連携](Lab1)**

**Lab2: [Operator導入 と CLI活用](Lab2)**
