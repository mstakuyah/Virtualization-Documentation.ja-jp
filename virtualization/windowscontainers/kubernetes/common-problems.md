---
title: "Kubernetes のトラブルシューティング"
author: gkudra-msft
ms.author: gekudray
ms.date: 11/16/2017
ms.topic: troubleshooting
ms.prod: containers
description: "Kubernetes の展開と Windows ノードの参加で発生する一般的な問題の解決方法。"
keywords: "kubernetes, 1.9, linux, コンパイル"
ms.openlocfilehash: b6be43f1afabdf8ef9c2ddc6f46ed5ac43a9e7a5
ms.sourcegitcommit: 2e8f1fd06d46562e56c9e6d70e50745b8b234372
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/14/2018
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

さらに、特定のスクリプト (`kubelet` など) はスーパー ユーザー特権で実行し、`sudo` というプレフィックスを付ける必要があります。


### <a name="cannot-connect-to-the-api-server-at-httpsaddressport"></a>`https://[address]:[port]` で API サーバーに接続できない ###
このエラーはほとんどの場合、証明書の問題を示しています。 構成ファイルを正しく生成していること、その中の IP アドレスがホストに対応していること、API サーバーによってマウントされたディレクトリにコピー済みであることを確認します。

[手順](./creating-a-linux-master)に従って操作した場合、ディレクトリは `~/kube/kubelet/` になります。それ以外の場合は、API のサーバーのマニフェスト ファイルを参照してマウント ポイントを確認します。


## <a name="common-networking-errors"></a>一般的なネットワーク エラー ##
ネットワークまたはホストに追加の制約があり、ノード間で特定の種類の通信が妨げられている場合があります。 次のことを確認してください。

  - ネットワーク トポロジを正しく構成していること
  - ポッドからと思われるトラフィックが許可されていること
  - HTTP トラフィックが許可されていること (Web サービスを展開する場合)
  - ICMP パケットが破棄されていないこと


<!-- ### My Linux node cannot ping my Windows pods ### -->

## <a name="common-windows-errors"></a>一般的な Windows エラー ##

### <a name="pods-stop-resolving-dns-queries-successfully-after-some-time-alive"></a>しばらく稼働した後、ポッドが正常な DNS クエリの解決を停止する ###
これは、いくつかの設定に影響を与えるネットワーク スタックの既知の問題です。Windows のサービスを通じて優先的に対応しています。


### <a name="my-kubernetes-pods-are-stuck-at-containercreating"></a>使用している Kubernetes ポッドが "ContainerCreating" で停止する ###
この問題には多くの原因が考えられますが、最も一般的な原因の 1 つは、pause イメージが正しく構成されていないことです。 これは、次の問題の大きな兆候です。


### <a name="when-deploying-docker-containers-keep-restarting"></a>展開するときに、Docker コンテナーが再起動を繰り返す ###
pause イメージが、OS のバージョンと互換性があることを確認します。 [作業手順](./getting-started-kubernetes-windows.md)では、OS とコンテナーの両方のバージョンが 1709 であることを前提にしています。 Insider ビルドなど、新しいバージョンの Windows を使用する場合は、イメージを調整する必要があります。 イメージについては、マイクロソフトの [Docker リポジトリ](https://hub.docker.com/u/microsoft/)を参照してください。 いずれの場合も、pause イメージ Dockerfile とサンプル サービスの両方で、イメージに `microsoft/windowsservercore:latest` にタグ付けされている必要があります。


### <a name="my-windows-pods-cannot-access-the-linux-master-or-vice-versa"></a>Windows ポッドから Linux マスターにアクセスできない (その逆も同じである) ###
Hyper-V 仮想マシンを使用している場合は、ネットワーク アダプターで MAC スプーフィングが有効になっていることを確認します。


### <a name="my-windows-node-cannot-access-my-services-using-the-service-ip"></a>Windows ノードがサービス IP を使用してサービスにアクセスできない ###
これは、Windows の現在のネットワーク スタックの既知の制限です。 ポッドのみがサービス IP を参照できます。


### <a name="no-network-adapter-is-found-when-starting-kubelet"></a>Kubelet を起動するときにネットワーク アダプターが見つからない ###
Kubernetes ネットワークが機能するには、Windows ネットワーク スタックで仮想アダプターが必要です。 次のコマンドが (管理シェルに) 結果を返さない場合、仮想ネットワークの作成 &mdash; Kubelet が機能するのに必要な前提条件 &mdash; に失敗しています。

```powershell
Get-HnsNetwork | ? Name -ieq "l2bridge"
Get-NetAdapter | ? Name -Like "vEthernet (Ethernet*"
```

`start-kubelet.ps1`スクリプトの出力を調べて、仮想ネットワークの作成中にエラーが発生していないか確認してください。

