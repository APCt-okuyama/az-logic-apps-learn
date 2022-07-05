[f:id:mountain1415:20220704182712p:plain]

# はじめに
こんにちは、ACS事業部の奥山です。
Azure Logic Apps(standard, single tenant) はどこでも動かすことが可能ということで Kubernetes環境 (AKS) で実行してみましたのでブログにしておきます。

※どこでも動かせるとは言え、Azure Storageが必要です。

(参考にしたサイト)  
https://techcommunity.microsoft.com/t5/integrations-on-azure-blog/run-logic-app-anywhere/ba-p/3118351
https://techcommunity.microsoft.com/t5/integrations-on-azure-blog/azure-logic-apps-running-anywhere-runtime-deep-dive/ba-p/1835564


## Logic App とは 

iPaaS(Integration Platform as a Service)と呼ばれるサービスの一種です。
簡単に言うと、システム間の連携を行うサービスになります。世の中にはいろいろな製品がありますが、MuleSoftなどが代表的な例になります。logic appsはAzureのiPaaSという位置づけになります。

( Microsoftの説明によると )  
Logic Apps とはアプリ、データ、サービス、およびシステムを統合する自動化された "ワークフロー" を最小限のコーディングまたはノーコードで作成および実行するためのクラウドベースのプラットフォームです。

* 最小限のコーディング または ノーコード で作成とういうのがとても魅力的です。

## Logic App の構成

Logic Appは下記のようにStorage QueueやTableを利用してworkflowを実現しています。

<figure class="figure-image figure-image-fotolife" title="image">[f:id:mountain1415:20220704172626p:plain]<figcaption>image</figcaption></figure>

QueueとTableをつかったイベントベースの処理という感じでしょうか、細かい仕様についてはまた機会があれば調査してみたいと思います。

## Logic Appの開発環境について

VS Code extension Azure Logic Apps (Standard)  を利用してローカル環境で workflowを定義し動作確認を行うことが可能です。  
※ VS Code extension は ConsumptionとStandardの２つがあるので注意。今回はシングルテナントなのでStandardを選択です。    
※ GUIベースでワークフローの定義が可能なので、視覚的にもとてもわかりやすいです。  

[f:id:mountain1415:20220704172633p:plain]

私は今回、wsl(ubuntu)環境にdocker, dotnet sdk(.NET Core 3.1 SDK)をインストールして検証を行いました。  

今回は以下のような単純なワークフローを vs code で作成しました。

内容としては２つ  
1. Http Requestを受信するトリガー  
2. ClientにResponseを返すアクション  

[f:id:mountain1415:20220704172622p:plain]

※ ローカルPC環境でのデバッグ（ブレークポイントの設定など）もできるようになっており効率よく開発作業ができるようになっています。

## Docker イメージ化

ベースイメージに mcr.microsoft.com/azure-functions/node:3.0 を指定してイメージを作成します。

```
cat Dockerfile
FROM mcr.microsoft.com/azure-functions/node:3.0
ENV AzureWebJobsStorage <BlobStorageConnectionString>
ENV AzureWebJobsScriptRoot=/home/site/wwwroot AzureFunctionsJobHost__Logging__Console__IsEnabled=true FUNCTIONS_V2_COMPATIBILITY_MODE=true
ENV WEBSITE_HOSTNAME localhost
ENV WEBSITE_SITE_NAME myexamplelogicapp
ENV AZURE_FUNCTIONS_ENVIRONMENT Development
COPY . /home/site/wwwroot
```
"BlobStorageConnectionString" の部分を利用するStorageアカウントの接続文字列に変更して build します。
```
docker build --tag my-logicapps:v1 .
docker images | grep my-logicapps
my-logicapps                     v1                     61326060ea61   11 minutes ago   1.59GB
```
※サイズが1.59GBもあります。

## コンテナをローカルPCで実行して確認

```
docker run --rm -p 8080:80 my-logicapps
```

curlコマンドで確認してみます。

1. Callback URLを取得  
codeはAzure Storageのコンテナ(azure-webjobs-secretsのhost.json)に格納されている。
```
curl -X POST -d "" -H "Content-Type: application/json" \
"http://localhost:8080/runtime/webhooks/workflow/api/management/workflows/MyStatufulWorkflow/triggers/manual/listCallbackUrl?api-version=2020-05-01-preview&code=PzhD2_UdggcGXctmR8r0GB6clZtEnPcAuRnEBU8Am8kaAzFuVrBjTw=="
```

2. Logic App URL(Callback URL)を呼び出し  
※プロトコルをhttpsからhttpにして、ポートを合わせて実行
```
curl -X POST -d @sample.json -H "Content-Type: application/json" \
"http://localhost:8080/api/MyStatufulWorkflow/triggers/manual/invoke?api-version=2020-05-01-preview&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig=W0Ue4MjE9HjizyPFQZueIwWC2IF6pNLa9UowbPFYlng"
im working fine
```
ワークフロー(logic apps)からの出力 "im working fine" が確認できます。

## レジストリー(ACR) へ Push して Azure Kubernates Service で動作確認
通常のコンテナイメージと同様にレジストリ(ACR) へ Push します。
```
az acr login -n acr001example
az acr build -t my-logicapps:v1 -r acr001example .
```

### Azure Kubernetes Service で実行
AKSリソースの準備
```
az aks create --resource-group $RG_NAME --name myAKSCluster --node-count 1 --enable-addons monitoring --generate-ssh-keys

# acrをアタッチしておく
az aks update -n myAKSCluster -g $RG_NAME --attach-acr acr001example
```

認証情報
```
az aks get-credentials --resource-group $RG_NAME --name myAKSCluster
kubectl get nodes
```

Deployment と Load Balancerをデプロイします
```
kubectl apply -f manifest_k8s/my-logicapps-dep.yml
```
※ my-logicapps-dep.yml の内容は Load Balancer と deployment

動作確認 ※ロードバランサーのIPアドレスに向けてAPIを発行する
```
curl -X POST -d "" -H "Content-Type: application/json" \
"http://<EXTERNAL_ID>/runtime/webhooks/workflow/api/management/workflows/MyStatufulWorkflow/triggers/manual/listCallbackUrl?api-version=2020-05-01-preview&code=PzhD2_UdggcGXctmR8r0GB6clZtEnPcAuRnEBU8Am8kaAzFuVrBjTw=="
```

※プロトコルはhttpにして、ip addressとport番号はLoad Balancerのモノを指定
```
curl -X POST -d @sample.json -H "Content-Type: application/json" \
"http://<EXTERNAL_ID>/api/MyStatufulWorkflow/triggers/manual/invoke?api-version=2020-05-01-preview&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig=nAVxG2J6V3_CVqPsW1t1FyYgGO_zMB_ivbdyvf62BbI"
im working fine
```
ワークフロー(logic apps)からの出力 "im working fine" が確認でき、AKS上でLogic Apps が動いていることが確認できました。

# まとめ
今回、Kubernetes(AKS)環境で Azure Logic Apps を動作させることができました。
図にすると下記のようになり、Logic Apps(workflow)を含んだPodがAKS内で実行されています。
[f:id:mountain1415:20220704172540p:plain]

Logic Apps の魅力は  
「多数のコネクターがすでに用意されておりそれらを利用することで開発工数を低減・品質も安定」  
「ローコード、ノーコードでの開発が可能でプログラミングのスキルに左右されない安定したシステム開発が行える」  
と認識していたのですが、さらにコンテナとしても利用可能で
「実行環境を選ばない」(オンプレでもどこでも動く)  
ということがわかりました。

検証に使ったソースコードは[こちら](https://github.com/APCt-okuyama/az-logic-apps-learn/tree/main/03_logicapp_in_docker)


