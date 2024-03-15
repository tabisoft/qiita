---
title: 【OCI Functions】事前構成済みファンクション Document Generator 試してみた
tags:
  - oci
  - Functions
  - ファンクション
private: true
updated_at: '2024-03-01T21:56:02+09:00'
id: ea880796bce04ba824dc
organization_url_name: null
slide: false
ignorePublish: false
---
## はじめに
1か月ほど前に新たに追加された事前構成済みファンクションの「Document Generator」をマニュアルをなぞりながら使ってみました。  
事前構成済みなので自分でコードを書くことなく機能を使うことができます。

- [Release Notes-Document Generator pre-built function now available][OCI releaseNote]
- [ドキュメントジェネレータ関数][docGen]


## 概要
OfficeテンプレートおよびJSONデータに基づいてPDFドキュメントを生成するFunctionsです。  
オブジェクト・ストレージに保存した.docx形式のテンプレートファイル中の変数(タグで指定します。後述)をJSONファイルで定義したデータで置換します。文字列、画像に対応しているようです。  

## 検証

### 検証構成
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/301950/9d811ea1-41b6-62b8-29e4-93fe2d372fc8.png)
今回主に利用するリソース一覧を整理しました。非常にシンプルと思います。  
構成図にまでする必要なかったかと思うのですが、dwaw.ioの練習がてら作成してみました。  
事前構成済みのファンクションの場合はOCIRでイメージ管理されないようなので構成図にも記載していません。  
また、簡単のためにファンクションの起動にはCloud ShellからOCI CLIを利用、PDFの出力先はWordテンプレートを配置したバケットと同じ、置換用JSONはオブジェクト・ストレージに保存せずファンクション呼び出しの際の引数としました。


### 事前準備・前提条件など
事前に構成図中の以下リソースは作成済みとする。
- VCN
  - プライベート・サブネット
  - サービス・ゲートウェイ
- プライベート・バケット
  - .docx形式のWordテンプレート  
  `Hello {customer.first_name} {customer.last_name}!`とだけ記入したただのワードファイルを今回は使いました。
  ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/301950/82f962a6-0024-76b6-30d7-a43e9c6ff6d8.png)

:::note info
追加でプライベート・サブネットからのOracle Services Networkへの通信がサービス・ゲートウェイに向くようにルート・テーブルやセキュリティ・ルールを適切に設定しておきます。
- [コンソールでのサービス・ゲートウェイの設定](https://docs.oracle.com/ja-jp/iaas/Content/Network/Tasks/servicegateway.htm#setting_up_sgw)
:::

### 事前構成済みファンクションの作成
コンソールからポチポチして作成。

「Document Generator」
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/301950/6f122e3a-cbb3-d217-792c-0e4f043bc299.png)
「関数の作成」
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/301950/164ec5e3-2beb-f0d6-58f5-3f22ddb5383e.png)
「新規アプリケーションの作成」->VCNやサブネットを選択。シェイプはGENERIC_X86を選択必須。他はデフォルト。IAMポリシーも自動作成。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/301950/e31fcb7c-4268-2e43-d09e-ecbc02b9b4a4.png)
「動的グループとIAMポリシーを作成しないでください」にチェックを入れなければ自動作成されたリソースの情報がでてくる。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/301950/1a9380cc-e616-c7e3-e427-e360c407520a.png)
自動作成されたポリシー
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/301950/29ab3697-b962-5576-e23f-b9c78f98c6c4.png)
自動作成された動的グループ
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/301950/917442cc-44f8-8871-39a6-6c92dafa50f3.png)

:::note info
自動の場合、動的グループは現在ログインしているユーザーのアイデンティティ・ドメインでなくデフォルト・ドメインで作成されるので注意。自分でポリシーや動的グループを管理したい場合は手動で実施。
- [ドキュメントジェネレータ関数-構成オプション-権限](https://docs.oracle.com/ja-jp/iaas/Content/Functions/Tasks/functions_pbf_catalog_document_generator.htm#functions_pbf_catalog_document_generator_plus__permissions-document-generator)
:::

### Cloud Shellからファンクション呼び出し
Cloud ShellにはデフォルトでOCI CLIがインストールされており、さらに認証情報の設定も構成済みのためすぐにCLIコマンドを実行することができます。
#### JSONファイルの準備
Wordファイル中のタグ変数をJSONデータで置換するのですが、今回はテンプレート置換用のJSONをオブジェクトストレージに置かず、インラインで指定して実行しました。
マニュアルの例をそのまま利用しています。
  - [文書ジェネレータ・タグ](https://docs.oracle.com/ja-jp/iaas/Content/Functions/non-dita/DocGenPBF-doc/markdown/DocGen-Template-Tags.htm#document-generator-tags)
  - [リクエストとレスポンスの例](https://docs.oracle.com/ja-jp/iaas/Content/Functions/Tasks/functions_pbf_catalog_document_generator.htm#functions_pbf_catalog_document_generator_plus__pbf-document-generator-example-requests-responses)

置換用JSONファイル
```
{
  "customer": {
    "first_name": "Jack",
    "last_name": "Smith"
  }
}
```

CLIのリクエストパラメータに投げるJSON
```
{
  "requestType": "SINGLE",
  "tagSyntax": "DOCGEN_1_0",
  "data": {    
    "source": "INLINE",
    "content": {
		"customer": {
			"first_name": "Jack",
			"last_name": "Smith"  
		}
	}
  },
  "template": {
    "source": "OBJECT_STORAGE",
    "namespace": "<オブジェクト・ストレージのネームスペース>",
    "bucketName": "<バケット名>",
    "objectName": "<テンプレートWordファイル名>"
  },
  "output": {
    "target": "OBJECT_STORAGE",
    "namespace": "<オブジェクト・ストレージのネームスペース>",
    "bucketName": "<バケット名>",
    "objectName": "<PDFファイル名>",
    "contentType": "application/pdf"
   }
}
```

CLIでファンクション呼び出し(結局長くなったからJSONファイルで別だししてもよかった)  
```
oci fn function invoke --function-id <ファンクションのOCID> --body '{"requestType":"SINGLE","tagSyntax":"DOCGEN_1_0","data":{"source":"INLINE","content":{"customer":{"first_name":"Jack","last_name":"Smith"}}},"template":{"source":"OBJECT_STORAGE","namespace":"<オブジェクト・ストレージのネームスペース>","bucketName":"<バケット名>","objectName":"<テンプレートWordファイル名>"},"output":{"target":"OBJECT_STORAGE","namespace":"<オブジェクト・ストレージのネームスペース>","bucketName":"<バケット名>","objectName":"<PDFファイル名>","contentType":"application/pdf"}}' --file "-"
```

#### 実行結果
意外とレスポンスくるまでに時間がかかった印象。1-2分くらい？
```
{"responseType":"SINGLE","code":200,"status":"OK","metadata":{"version":"1.0.1","configurationParameters":{"FN_FN_NAME":"DocGenFunc","FN_APP_NAME":"DocGenApp","FN_TYPE":"sync","FN_APP_ID":"ocid1.fnapp.oc1.ap-osaka-1.xxx","OCI_REGION_METADATA":"{\"realmDomainComponent\":\"oraclecloud.com\",\"realmKey\":\"oc1\",\"regionIdentifier\":\"ap-osaka-1\",\"regionKey\":\"KIX\"}","FN_FN_ID":"ocid1.fnfunc.oc1.ap-osaka-1.xxx","FN_MEMORY":"512"}},"document":{"type":"OBJECT_STORAGE","namespace":"xxx","bucketName":"bucket_temp","objectName":"output.pdf","contentType":"application/pdf"}}
```
指定したバケットにpdfが生成されている。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/301950/b74b648e-a893-1896-1314-34ad47a99ed3.png)
PDF中身(質素)
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/301950/d2ef763b-1174-2b37-b499-e8d2b52ebc04.png)



## まとめ・気づき
- 事前構成済みファンクションの利用のためにOCIRは用意しなくてよかった。
- 後で気づいたのですが、事前構成済みファンクションはファンクションサービスに対する以下のポリシーがなくても動いてました。
  - `Allow service FaaS to use virtual-network-family in compartment <コンパートメント名>`
  - `Allow service FaaS to read repos in compartment <コンパートメント名>`
- などなど、普通のファンクションとは少しずつ違うようですが、事前の準備とかを気にせずほんとにマニュアル記載のとおり簡単に動かせるという印象でした。
  - [事前構築済ファンクションを使用したファンクションの作成](https://docs.oracle.com/ja-jp/iaas/Content/Functions/Tasks/functions_pbf_creating_prebuilt.htm)
    - > 事前構築済ファンクション(PBF)は、Oracle Cloud Infrastructure Functionsを使用して実装されたすぐに使用できるタスクまたはアクションのカタログを提供します。ファンクション・コードを記述、構築、パッケージ化および保守する必要はありません。
  - [OCI Functionsのアーキテクチャを理解する](https://qiita.com/dingtianhongjie/items/8ba223df0a6e189c14ab)
    - ファンクションの構成要素が分かりやすくまとまっている。
- [事前構築済ファンクション・カタログ][docPreFunc]
  - その他の事前構成済みファンクション一覧。
  - 実機確認だと、カタログにある「Wallet関数を使用したデータベース・シークレットのローテーション」と「Walletファンクションを使用しないデータベース・シークレット・ローテーション」はコンソールから選べなさそうだった。ADB作成したらでてくる？
- [OCI Architecture Diagram Toolkit][icon]
  - Cloud Shellのアイコンは用意されてないことに気づきました。

以上。


[OCI releaseNote]: https://docs.oracle.com/en-us/iaas/releasenotes/changes/b0e941dd-9861-410c-b2a0-425334d2c98e/
[docGen]: https://docs.oracle.com/ja-jp/iaas/Content/Functions/Tasks/functions_pbf_catalog_document_generator.htm
[docPreFunc]: https://docs.oracle.com/ja-jp/iaas/Content/Functions/Tasks/functions_pbf_catalog.htm
[icon]: https://docs.oracle.com/ja-jp/iaas/Content/General/Reference/graphicsfordiagrams.htm