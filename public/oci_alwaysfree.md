---
title: OCI Always Free コンピュート 5つのポイント
tags:
  - oci
  - always_free
private: false
updated_at: '2024-03-18T12:37:19+09:00'
id: b6bdc928fa79b1eaaff8
organization_url_name: null
slide: false
ignorePublish: false
---
## はじめに
今更ですがOCI Always Freeのコンピュートをちょっとした遊びや学習で使う分にはこのポイントを押さえれば結構便利に使えるのではないか、という観点で改めて整理しました。
内容としては既出情報の再整理となっていそうですがご了承ください。

## Always Free のコンピュートリソースの種類
OCIではアカウントを作るだけでクラウド上に無料でVMを作成でき、自由に使うことができます。以下の２種類のシェイプが作成できます。
- マイクロ・インスタンス(AMDプロセッサ): 
すべてのテナンシは、AMDプロセッサを含むVM.Standard.E2.1.Microシェイプを使用して最大2つのAlways Free VMインスタンスを取得します。
- Ampere A1 Computeインスタンス(Armプロセッサ): 
すべてのテナンシは、Armプロセッサを含むVM.Standard.A1.Flexシェイプを使用するVMインスタンスに対して、月ごとに最初の3,000 OCPU時間および18,000 GB時間を無料で取得します。Always Free テナンシの場合、これは4 OCPUおよび24 GBのメモリーと同等です。

https://docs.oracle.com/ja-jp/iaas/Content/FreeTier/freetier_topic-Always_Free_Resources.htm#freetier_topic_Always_Free_Resources_Infrastructure

## Always Freeでコンピュートを使うときのポイント
### Always Free環境で作成できる最大インスタンス数は4
- A1インスタンスでは4oCPU、24GBメモリー分のリソースが使えます。この範囲のなかで、例えば4oCPU/24GBメモリのインスタンスを1つ作ることもできますし、1oCPUずつ切り出して使うことで最大で4つまでインスタンスを作成することもできます。
- マイクロ・インスタンスは最大で２つ使えるため、Always Free環境で作成できる最大インスタンス数はA1インスタンスと足して6になりそうですが、ブロックボリュームのリソース制限(Always Freeでは200GBまで)とブートボリュームの最小サイズ(50GB)の制限があるため、どんなに頑張っても4つが最大になります。
    
### A1インスタンスは最大性能高く、マイクロ・インスタンスは汎用性が高い
- 例えばDockerを使ってコンテナたてたいみたいなときに、arm対応のDockerイメージが存在しなかったり、あとはインストールしたいソフトウェアのarm向けビルドが存在しないこともあります。
- とはいえA1インスタンスでもNextCloudとかは動かせる！ブロックボリューム200GBとオブジェクトストレージ20GBをマウントして活用することで最大220GBのオンラインストレージとして無料で利用可能。
    参考：https://docs.oracle.com/ja/learn/oci_nextcloud_ampere_a1/index.html
    - minecraftのサーバもA1で建てれる！
      実際にJava版を4oCPU/24GBのインスタンスで遊んでみましたが、4人くらいで遊ぶには十分でした。
- マイクロ・インスタンスはCentOSをイメージとして選択可能。A1は選択不可。またWindowsはライセンス料かかるためどちらも選択不可
- ADの選択でもA1とE2マイクロで差がありそう。A1は任意のAD1-3に、E2マイクロはAD1のみに作成できる？それともE2インスタンスを複数ADに分散させて配置できないってこと？？どちらにしてもADの選択という点ではA1インスタンスの方が幅が広い。  
私のAlways Free環境はホーム・リージョンのADがAD1のみのため確認できず。  
    参考：https://docs.oracle.com/ja-jp/iaas/Content/FreeTier/freetier_topic-Always_Free_Resources.htm#freetier_topic_Always_Free_Resources_Infrastructure
    > 複数の可用性ドメインがあるリージョン:
    > - 任意の可用性ドメインにAmpere A1 Computeインスタンスを作成できます。
    > - M.Standard.E2.1.Microシェイプを使用するインスタンスは、1つの可用性ドメインにのみ作成できます。

### マイクロ・インスタンスでのメモリ有効活用
マイクロ・インスタンスはメモリが1GBでけっこうきつく、メモリをどうにかしてうまく活用したり節約したくなります。せこい...
- dnfのautoupgradeを無効にする
    公開していたwebページがたまにつながらなくなることがあったのでモニタリングから確認してみるとメモリ使用率が一定間隔で100%にスパイクしていることがわかりました。syslogから「Failed to start dnf makecache.」が確認できたので、一旦無効にしました。(Oracle Linuxをイメージとして選択したときのみ？)
    `sudo systemctl disable dnf-makecache.timer`
参考：https://note.com/t_ak66/n/n023eeb753a99
- swapを2GBで設定
実メモリの2倍が適正値とのことでいつも2GBで設定していますが、どうなんでしょうか。
参考：https://docs.oracle.com/ja/learn/file_system_linux_8/#task-9-increase-swap-space

###  A1インスタンス作成時の Out of host capacity. 対策
A1インスタンスは人気のため新規で作ろうとしても上記の、クラウドリソースが足らない旨のエラーがでることがあります。これはもう空いた時に作成するしかないので、Terraformのマネージド環境であるリソースマネージャーを使って定期的にインスタンス作成を施行する方法を紹介します。具体的にはインスタンス作成画面からA1シェイプを選んでスタックだけを作成し、その後マイクロ・インスタンスのcronジョブでリソースマネージャーをCLIから定期実行します。
- 準備
    - インスタンス作成の画面からA1インスタンスを選択したコンピュートのスタックを作成し、スタックのOCIDを控える。
    - マイクロ・インスタンスでCLIを実行できる環境を整える。Oracle Linux Developperイメージでインスタンスを作成した場合、CLI自体はインストール済み。
- cronに以下のCLIを登録し、定期実行する
`oci resource-manager job create-apply-job --stack-id <スタックのOCID> --execution-plan-strategy AUTO_APPROVED`
- リソースマネージャーのジョブが成功したらイベント・サービスを使ってメール通知したいところですが、Always Freeだとイベントが使えないため、通知したい場合はPayG環境へアップグレードしましょう。
- 参考：Osaka regionで5分ごとに実行してみて、約5時間ほどでインスタンス作成に成功しました。
### アイドル状態のコンピュートリソースについて  
https://docs.oracle.com/en-us/iaas/Content/FreeTier/freetier_topic-Always_Free_Resources.htm#compute__idleinstances
- 7日間のうち以下の状態であった場合、アイドル状態とみなされコンピュートが停止されるようです。少し前は10%とかだったようなので徐々にハードルが上がってきている模様。
    - 95%の期間でCPU使用率が20%未満？
    - ネットワーク使用率が20%未満
    - メモリー使用率が20%未満(A1シェイプにのみ適用)
- アイドル状態とみなされたインスタンスが停止された**後**、メールで通知がきます。
    - 以下はメール抜粋ですが、PayGでAlways Freeリソースを運用している場合はこのアイドル状態の対象であっても停止されることはなさそうです。
        > In the future, you can keep idle compute instances from being stopped by converting your account to Pay As You Go (PAYG). With PAYG, you will not be charged as long as your usage for all OCI resources remains within the Always Free limits.


## さいごに
PayGにするだけで無料で使えるサービスもたくさんあるし、高額の課金がされないように運用するスキルも身につきそうなため、できるならPayGにしたいと思いました。


## 追記
- 記事構成変更。選択できるADについて追記。2024/3
