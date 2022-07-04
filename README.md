# az-logic-apps-learn

Logic Apps を利用してみる。

リソースグループ作成
```
export RG_NAME=az-logic-apps-example-rg
export LOCATION=japaneast
az group create -n $RG_NAME -l $LOCATION
```

リソースグループ削除
```
az group delete -n $RG_NAME -y
```

# logic apps とは (簡単に)
iPaaS(Integration Platform as a Service)と呼ばれるサービスの一種です。
簡単に言うと、システム間の連携を行うサービスになります。世の中にはいろいろな製品がありますが、MuluSoftなどが代表的な例になる。logic appsはAzureのiPaaSという位置づけになります。


Microsoftの説明によると

## Logic Apps とはアプリ、データ、サービス、およびシステムを統合する自動化された "ワークフロー" を最小限のコーディングまたはノーコードで作成および実行するためのクラウドベースのプラットフォームです。

`最小限のコーディング`または`ノーコード`で作成とういうのが魅力的です。

# Logic App でリソースグループを定期的に削除する
内容
```
リソースグループの一覧を取得する (取得)
一覧から削除対象を抽出する (フィルター)
抽出した削除対象を削除する (実行)
```

# シングルテナント/マルチテナント

従量課金のリソースの種類は、"マルチテナント" Azure Logic Apps または "統合サービス環境" で実行

Standard のリソースの種類は "シングルテナント" Azure Logic Apps 環境で実行