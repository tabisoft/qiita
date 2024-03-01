---
title: 【OCI Functions】 pre-built の Document Generator 試してみた
tags:
  - oci
  - Functions
private: true
updated_at: '2024-03-01T21:56:02+09:00'
id: ea880796bce04ba824dc
organization_url_name: null
slide: false
ignorePublish: false
---
2024/2/20にリリースされた新しい事前構成済みFunctionsの「Document Generator」の動作確認をしてみました。  
以下マニュアルをなぞりながら整理した内容になります。

- [Release Notes-Document Generator pre-built function now available][OCI releaseNote]
- [ドキュメントジェネレータ関数][docGen]

## 概要
OfficeテンプレートおよびJSONデータに基づいてPDFドキュメントを生成するFunctionsです。  
オブジェクト・ストレージに保存した.docx形式のテンプレートファイル中の変数(タグで指定します。後述)をJSONファイルで定義したデータで置換します。文字列、画像に対応しているようです。  





[OCI releaseNote]: https://docs.oracle.com/en-us/iaas/releasenotes/changes/b0e941dd-9861-410c-b2a0-425334d2c98e/
[docGen]: https://docs.oracle.com/ja-jp/iaas/Content/Functions/Tasks/functions_pbf_catalog_document_generator.htm
