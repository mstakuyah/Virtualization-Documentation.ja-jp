---
title: "Kubernetes のトラブルシューティング"
author: gkudra-msft
ms.author: gekudray
ms.date: 11/16/2017
ms.topic: troubleshooting
ms.prod: containers
description: "Kubernetes の展開と Windows ノードの参加で発生する一般的な問題の解決方法。"
keywords: "kubernetes, 1.9, linux, コンパイル"
ms.openlocfilehash: 73b44ffd12fba58ac4ef38352c012061a6817945
ms.sourcegitcommit: ad5f6344230c7c4977adf3769fb7b01a5eca7bb9
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/05/2017
---
# <a name="troubleshooting-kubernetes"></a>Kubernetes のトラブルシューティング #
このページでは、Kubernetes のセットアップ、ネットワーク、および展開に関する一般的な問題について説明します。

> [!tip]
> PR を[ドキュメント リポジトリ](https://github.com/MicrosoftDocs/Virtualization-Documentation/)に投稿することで、よく寄せられる質問の項目をご提案ください。


## <a name="common-deployment-errors"></a>一般的な展開エラー ##
Kubernetes マスターのデバッグは、(発生しやすい順で) 次の 3 つのカテゴリに分類されます。

  - Kubernetes システム コンテナーに問題がある。
  - `kubelet` の実行状態に問題がある。
  - システムに問題がある。


Kubernetes によるポッドの作成を確認するには、`kubectl get pods -n kube-system` を実行します。これにより、いずれかがクラッシュしていたり正しく開始していないとわかることがあります。   次に `docker ps -a` を実行して、これらのポッドの背後にあるすべての未加工コンテナーを表示します。 最後に、問題の原因となっている疑いがあるコンテナーに対して `docker logs [ID]` を実行し、プロセスの未加工出力を確認します。


### <a name="permission-denied-errors"></a>_"アクセス拒否"_ エラー ###
スクリプトに実行アクセス許可があることを確認します。

```bash
chmod +x [script name]
```

さらに、特定のスクリプト (`kubelet` など) は管理者特権で実行し、`sudo` というプレフィックスを付ける必要があります。


### <a name="cannot-connect-to-the-api-server-at-httpsaddressport"></a>`https://[address]:[port]` で API サーバーに接続できない ###
このエラーはほとんどの場合、証明書の問題を示しています。 構成ファイルを正しく生成していること、その中の IP アドレスがホストに対応していること、API サーバーによってマウントされたディレクトリにコピー済みであることを確認します。

[手順](./creating-a-linux-master)に従って操作した場合、ディレクトリは `~/kube/kubelet/` になります。それ以外の場合は、API のサーバーのマニフェスト ファイルを参照してマウント ポイントを確認します。


## <a name="common-networking-errors"></a>一般的なネットワーク エラー ##
ネットワークまたはホストに追加の制約があり、ノード間で特定の種類の通信が妨げられている場合があります。 次のことを確認してください。

  - ポッドからと思われるトラフィックが許可されていること
  - HTTP トラフィックが許可されていること (Web サービスを展開する場合)
  - ICMP パケットが破棄されていないこと


<!-- ### My Linux node cannot ping my Windows pods ### -->

## <a name="common-windows-errors"></a>一般的な Windows エラー ##


### <a name="my-windows-pods-cannot-access-the-linux-master-or-vice-versa"></a>My Windows ポッドから Linux マスター、またはその逆のアクセスはできません。 ###
Hyper-V 仮想マシンを使用している場合は、ネットワーク アダプターで MAC スプーフィングが有効になっていることを確認します。


### <a name="my-windows-node-cannot-access-my-services-using-the-service-ip"></a>My Windows ノードは、サービス IP を使用してサービスにアクセスすることはできません。 ###
これは、Windows の現在のネットワーク スタックの既知の制限です。
