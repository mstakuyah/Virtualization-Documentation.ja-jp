---
title: Linux ノードへの参加
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: OS 混在の Kubernetes クラスターに Kubernetes resoureces を展開する。
keywords: kubernetes、1.14、windows、はじめに
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: e6c569ae8d5bf50e24ea0fc7a6dd04734b60a863
ms.sourcegitcommit: d252f356a3de98f224e1550536810dfc75345303
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/04/2019
ms.locfileid: "10069946"
---
# <a name="deploying-kubernetes-resources"></a>Kubernetes リソースの展開 #
少なくとも1つの master と1つのワーカーで構成されている Kubernetes クラスターがある場合は、Kubernetes リソースを展開することができます。
> [!TIP] 
> Windows で現在サポートされている Kubernetes のリソースについて 詳しくは、「[正式にサポートされている機能](https://kubernetes.io/docs/setup/production-environment/windows/intro-windows-in-kubernetes/#supported-functionality-and-limitations)と[Kubernetes Windows ロードマップ](https://github.com/orgs/kubernetes/projects/8)」をご覧ください。


## <a name="running-a-sample-service"></a>サンプルサービスの実行 ##
ここでは、確実にクラスターへの参加に成功し、ネットワークを正しく構成できるように、非常に単純な [PowerShell ベースの Web サービス](https://github.com/Microsoft/SDN/blob/master/Kubernetes/WebServer.yaml)を展開します。

これを行う前に、すべてのノードが正常に動作していることを確認することをお勧めします。
```bash
kubectl get nodes
```

すべてが適切に表示される場合は、次のサービスをダウンロードして実行できます。
> [!Important] 
> 以前`kubectl apply`[は、ノードによって](https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/version-compatibility#choosing-container-os-versions)実行可能なコンテナー `microsoft/windowsservercore`イメージに、サンプルファイル内の画像をダブルチェックまたは修正してください。

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/l2bridge/manifests/simpleweb.yml -O win-webserver.yaml
kubectl apply -f win-webserver.yaml
watch kubectl get pods -o wide
```

これにより、展開とサービスが作成されます。 最後の watch コマンドは、pod を無限に照会して、その状態を追跡します。クリック`Ctrl+C`するだけで、 `watch`監視が完了したらコマンドが終了します。

すべての作業を正しく行った場合は、次のことを確認できます。

  - Windows ノードのコマンドの下`docker ps`の [pod あたり2つのコンテナー] を参照してください。
  - Linux マスターで `kubectl get pods` コマンドにより 2 つのポッドが表示されること。
  - `curl` : Linux マスターからポート 80 の*ポッド* IP で実行して Web サーバー応答を受信できること。これは、ネットワーク経由でのノードからポッドへの通信が適切であることを示します。
  - `docker exec` で*ポッド間* (複数の Windows ノードがある場合はホスト間も含めて) の ping を実行できること。これは、ポッド間の通信が適切であることを示します。
  - `curl` 仮想*サービス IP* (以下`kubectl get services`に表示) は、Linux マスターと個々のポッドからのものです。これは、通信を pod する適切なサービスを示しています。
  - `curl` 適切なサービス検出を示す、Kubernetes の[既定の DNS サフィックス](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#services)を持つ*サービス名*。
  - `curl` クラスター外部の Linux マスターまたはコンピューターからの*Nodeport* 。これは、受信接続を示しています。
  - `curl` ポッド内の外部 Ipこれは、送信接続の例です。

> [!Note]  
> Windows*コンテナホスト*は、スケジュール設定されているサービスからはサービス IP にアクセスでき**ません**。 これは、Windows Server 向けの将来のバージョンで強化される[既知のプラットフォームの制限](./common-problems.md#my-windows-node-cannot-access-my-services-using-the-service-ip)です。 ただし、Windows*ポッド***は**サービス IP にアクセスできます。

## <a name="next-steps"></a>次のステップ ##
このセクションでは、Windows ノードでの Kubernetes リソースのスケジュール方法について説明します。 これでガイドが終わります。 問題がある場合は、「トラブルシューティング」を参照してください。

> [!div class="nextstepaction"]
> [トラブルシューティング](./common-problems.md)

それ以外の場合は、Kubernetes のコンポーネントを Windows サービスとして実行することをお勧めします。
> [!div class="nextstepaction"]
> [Windows サービス](./kube-windows-services.md)
