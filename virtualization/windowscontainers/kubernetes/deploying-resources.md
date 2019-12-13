---
title: Linux ノードの結合
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: 混合 OS Kubernetes クラスターに Kubernetes resoureces をデプロイします。
keywords: kubernetes、1.14、windows、はじめに
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: e6c569ae8d5bf50e24ea0fc7a6dd04734b60a863
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909962"
---
# <a name="deploying-kubernetes-resources"></a>Kubernetes リソースのデプロイ #
Kubernetes クラスターが少なくとも1つのマスターと1ワーカーで構成されている場合は、Kubernetes リソースをデプロイすることができます。
> [!TIP] 
> Windows で現在どのような Kubernetes リソースがサポートされていますか。 詳細については、「 [Windows ロードマップで公式に](https://github.com/orgs/kubernetes/projects/8)[サポートされている機能](https://kubernetes.io/docs/setup/production-environment/windows/intro-windows-in-kubernetes/#supported-functionality-and-limitations)」および「Kubernetes」を参照してください。


## <a name="running-a-sample-service"></a>サンプルサービスの実行 ##
ここでは、確実にクラスターへの参加に成功し、ネットワークを正しく構成できるように、非常に単純な [PowerShell ベースの Web サービス](https://github.com/Microsoft/SDN/blob/master/Kubernetes/WebServer.yaml)を展開します。

これを行う前に、すべてのノードが正常であることを確認することをお勧めします。
```bash
kubectl get nodes
```

すべて問題がなければ、次のサービスをダウンロードして実行できます。
> [!Important] 
> `kubectl apply`する前に、サンプルファイルの `microsoft/windowsservercore` イメージを、[ノードによって](https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/version-compatibility#choosing-container-os-versions)実行可能なコンテナーイメージに再確認または変更してください。

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/l2bridge/manifests/simpleweb.yml -O win-webserver.yaml
kubectl apply -f win-webserver.yaml
watch kubectl get pods -o wide
```

これにより、デプロイとサービスが作成されます。 最後の watch コマンドは、ポッドを無制限に照会してその状態を追跡します。`Ctrl+C` を押して、観察の完了時に `watch` コマンドを終了するだけです。

すべての作業を正しく行った場合は、次のことを確認できます。

  - Windows ノードの `docker ps` コマンドの下にあるポッドあたり2つのコンテナーを参照してください。
  - Linux マスターで `kubectl get pods` コマンドにより 2 つのポッドが表示されること。
  - Linux マスターからポート80の*ポッド*ip を `curl` すると、web サーバーの応答が取得されます。これは、ネットワーク経由で通信するための適切なノードを示しています。
  - `docker exec` で*ポッド間* (複数の Windows ノードがある場合はホスト間も含めて) の ping を実行できること。これは、ポッド間の通信が適切であることを示します。
  - Linux マスターと個々のポッドから仮想*サービス IP* (`kubectl get services`の下に表示されます) を `curl` します。これは、適切なサービスとの通信を行うことを示しています。
  - Kubernetes の[既定の DNS サフィックス](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#services)を使用して*サービス名*を `curl` し、適切なサービス検出をデモンストレーションします。
  - Linux マスターまたはクラスター外のコンピューターから*Nodeport*を `curl` します。これは、受信接続を示しています。
  - ポッド内から外部 Ip を `curl` します。これは、送信接続を示しています。

> [!Note]  
> Windows*コンテナーホスト*は、スケジュールされているサービスからサービス IP にアクセスすることはでき**ません**。 これは、今後のバージョンの Windows Server で改善されることがわかっている[プラットフォームの制限](./common-problems.md#my-windows-node-cannot-access-my-services-using-the-service-ip)です。 ただし、Windows*ポッド***は**サービス IP にアクセスできます。

## <a name="next-steps"></a>次のステップ ##
このセクションでは、Windows ノードで Kubernetes リソースをスケジュールする方法について説明します。 これでガイドは終わりです。 問題が発生した場合は、「トラブルシューティング」セクションを参照してください。

> [!div class="nextstepaction"]
> [トラブルシューティング](./common-problems.md)

それ以外の場合は、Kubernetes コンポーネントを Windows サービスとして実行することにも興味があります。
> [!div class="nextstepaction"]
> [Windows サービス](./kube-windows-services.md)
