---
title: Linux ノードへの参加
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: 混合 OS Kubernetes クラスターで Kubernetes resoureces を展開します。
keywords: kubernetes、1.13、windows、作業の開始
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 7d2f1dd789a96a3ee4898ef196f872e574d6321f
ms.sourcegitcommit: 41318edba7459a9f9eeb182bf8519aac0996a7f1
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/28/2019
ms.locfileid: "9120480"
---
# <a name="deploying-kubernetes-resources"></a>Kubernetes リソースを展開します。 #
少なくとも 1 つのマスターと 1 の作業者から成る Kubernetes クラスターがあると仮定して、準備ができたら Kubernetes リソースを配置します。
> [!TIP] 
> Windows にどのような Kubernetes リソースは現在サポートされている好奇心ですか。 詳細については、[正式にサポートされている機能](https://kubernetes.io/docs/getting-started-guides/windows/#supported-features)と[Windows のロードマップで Kubernetes](https://trello.com/b/rjTqrwjl/windows-k8s-roadmap)を参照してください。


## <a name="running-a-sample-service"></a>サンプルのサービスを実行します。 ##
ここでは、確実にクラスターへの参加に成功し、ネットワークを正しく構成できるように、非常に単純な [PowerShell ベースの Web サービス](https://github.com/Microsoft/SDN/blob/master/Kubernetes/WebServer.yaml)を展開します。

これを実行する前に常に、すべてのノードが正常なことを確認することをお勧めします。
```bash
kubectl get nodes
```

すべてのアイテムが意図した場合は、ダウンロードして、次のサービスを実行します。
> [!Important] 
> 前に`kubectl apply`、変更を確認するダブルクリック-check/、 `microsoft/windowsservercore` [、ノードによって実行されるコンテナーの画像](https://docs.microsoft.com/en-us/virtualization/windowscontainers/deploy-containers/version-compatibility#choosing-container-os-versions)にサンプル ファイル内のイメージです。

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/l2bridge/manifests/simpleweb.yml -O win-webserver.yaml
kubectl apply -f win-webserver.yaml
watch kubectl get pods -o wide
```

これは、展開とサービスを作成します。 ウォッチ式の最後のコマンドは、それらの状態を追跡する無期限ポッドをクエリします。キーを押すだけで`Ctrl+C`を終了する、`watch`コマンドの確認を完了するとします。

すべての作業を正しく行った場合は、次のことを確認できます。

  - [ポッドごとの 2 つのコンテナーを参照してください`docker ps`Windows ノードのコマンド
  - Linux マスターで `kubectl get pods` コマンドにより 2 つのポッドが表示されること。
  - `curl` : Linux マスターからポート 80 の*ポッド* IP で実行して Web サーバー応答を受信できること。これは、ネットワーク経由でのノードからポッドへの通信が適切であることを示します。
  - `docker exec` で*ポッド間* (複数の Windows ノードがある場合はホスト間も含めて) の ping を実行できること。これは、ポッド間の通信が適切であることを示します。
  - `curl` 仮想*サービス IP* (表示`kubectl get services`) Linux マスターからおよび個々 のポッドです。これは、ポッド コミュニケーションに適切なサービスを示しています。
  - `curl` Kubernetes で[既定の DNS サフィックス](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#services)サービスの適切な検索を示す*サービス名*です。
  - `curl` クラスター以外のコンピューターまたは Linux マスターから*NodePort*これは、着信接続を示しています。
  - `curl` ポッド; 内の外部の ip アドレスこれは、送信接続を示しています。

> [!Note]  
> Windows*コンテナー ホスト*が**ない**サービス IP に予定されているサービスからアクセスできるようにします。 これでは、[既知のプラットフォームの制限](./common-problems.md#my-windows-node-cannot-access-my-services-using-the-service-ip)は、Windows Server を将来のバージョンの改善です。 Windows*ポッド***は**ただしサービス IP をアクセスできるようにします。

### <a name="port-mapping"></a>ポートのマッピング ### 
ノードのポートをマップすることによって、ポッドでホストされているサービスにそれぞれのノードからアクセスすることもできます。 この機能を示すための[サンプル YAML](https://github.com/Microsoft/SDN/blob/master/Kubernetes/PortMapping.yaml) では、ノードのポート 4444 をポッドのポート 80 にマップしています。 これを展開するには、さきほどと同じ手順に従います。

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/PortMapping.yaml -O win-webserver-port-mapped.yaml
kubectl apply -f win-webserver-port-mapped.yaml
watch kubectl get pods -o wide
```

これで、ポート 4444 の*ノード* IP で `curl` を実行し、Web サーバーの応答を受信することができます。 このとき 1 対 1 のマッピングを適用する必要があるため、スケーリングはノードあたり単一ポッドに制限される点に注意してください。


## <a name="next-steps"></a>次のステップ ##
このセクションで、Windows ノードの Kubernetes リソースのスケジュールを設定する方法を説明します。 これは、ガイドを終了します。 問題が発生した場合は、[トラブルシューティング] セクションを確認してください。

> [!div class="nextstepaction"]
> [トラブルシューティング](./common-problems.md)

それ以外の場合もについては、以下の Windows サービスとして Kubernetes コンポーネントを実行します。
> [!div class="nextstepaction"]
> [Windows サービス](./kube-windows-services.md)