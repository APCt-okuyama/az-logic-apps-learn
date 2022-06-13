# az-logic-apps-learn

Logic Apps を利用してみる。

リソースグループ作成
```
az group create -n az-logic-apps-example-rg -l japaneast
```

リソースグループ削除
```
az group delete -n az-logic-apps-example-rg -y
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