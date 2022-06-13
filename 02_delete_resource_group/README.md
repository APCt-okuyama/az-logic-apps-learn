# リソースグループを削除する定期処理

Logic Apps を使って、簡単なワークフローを作成してみる。

## 開発環境について (portal or vscode)

https://docs.microsoft.com/ja-jp/azure/logic-apps/quickstart-create-logic-apps-visual-studio-code

vs codeの拡張機能を利用して開発を行うことが可能ではあるが、編集はJSON定義を直接編集することになる。

vs codeの拡張機能はビューとして利用可能。ロジック アプリのワークフローを視覚的に確認できる。ただし編集はできない。

(注意) MSのサンプルのJSONは時々間違っていることがあります。(jsonのタグが足りないとか。)
https://docs.microsoft.com/ja-jp/azure/logic-apps/logic-apps-data-operations-code-samples#parse-json-action-example


## 背景・ワークフローの内容

開発や検証でAzureのリソースを作成・削除を毎日のように行いますが、Azureの料金を節約するために利用していない時間帯はリソースを削除しておきたいです。

Logic Apps に就業時間後に毎日自動でリソースを削除させます。


## 構成
Portal画面から以下のように作成することが可能。

![delete_resoucegroup_by_name](./delete_resoucegroup_by_name.PNG)

処理の内容
```
1. 繰り返し(決まった時間に実行するスケジュール)
2. リソースグループの一覧を取得
3. For each (取得したリソースグループ毎に繰り返す)
 3.1 条件の判定 
 3.2 条件に一致した場合は、リソースグループを削除する
```
フローが資格的に確認できるのは非常に分かりやすくて良い。

# まとめ
VS Codeの拡張機能でもデザイナーで編集可能になればさらに使いやすい。
バージョン管理はフローの定義ファイル(JSON)を管理することになる。
