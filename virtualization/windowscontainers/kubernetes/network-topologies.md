---
title: ネットワーク トポロジ
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: Windows と Linux ではサポートされているネットワーク トポロジします。
keywords: kubernetes、1.12、windows、作業の開始
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: bcbd7b530b58b663305ea5d8b84a75eaf971f997
ms.sourcegitcommit: 4412583b77f3bb4b2ff834c7d3f1bdabac7aafee
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/15/2018
ms.locfileid: "6948061"
---
# <a name="network-solutions"></a>Network Solutions #

[Kubernetes マスター ノードをセットアップ](./creating-a-linux-master.md)した後は、ネットワーク ソリューションを選択する準備が整います。 仮想[クラスター サブネット](./getting-started-kubernetes-windows.md#cluster-subnet-def)ルーティング可能なノード間での複数の方法があります。 Windows Kubernetes のオプションを次のいずれかを今日選びます。

1. セットアップのルートに[Flannel](network-topologies.md#flannel-in-host-gateway-mode)など、サード パーティいる CNI プラグインを使用します。
1. スマート[ラック上部 (ToR) の切り替え](network-topologies.md#configuring-a-tor-switch)にサブネットのルーティングを構成します。

> [!tip]  
> 3 つ目のネットワーク ソリューションを開いているとして (OvS) を活用するウィンドウと開いている仮想ネットワーク (OVN) があります。 これをドキュメント化するが、このドキュメントの範囲外を設定するのには、[次の手順](https://kubernetes.io/docs/getting-started-guides/windows/#for-3-open-vswitch-ovs-open-virtual-network-ovn-with-overlay)をお読みください。

## <a name="flannel-in-host-gateway-mode"></a>ホスト ゲートウェイ モードで flannel します。

Flannel ネットワークに使用できるオプションのいずれかの*ホスト ゲートウェイ モード*(ホスト-ゲートウェイ) は、すべてのノードのポッド サブネット間の静的なルートを構成します。
> [!NOTE]  
> これは*VXLAN カプセル化を使用すると、開発現時点では、Flannel に重ねてネットワーク*に異なります。 この領域で表示する.

### <a name="prepare-kubernetes-master-for-flannel"></a>Flannel 用 Kubernetes マスターを準備します。

クラスターで[Kubernetes マスター](./creating-a-linux-master.md)では、いくつかマイナーの準備をお勧めします。 Flannel を使用する場合は、iptables チェーン ブリッジ IPv4 トラフィックを有効にすることをお勧めします。 これは、次のコマンドを使用します。

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```

###  <a name="download--configure-flannel"></a>ダウンロードし、Flannel を構成します。 ###
最新の Flannel マニフェストをダウンロードしてください。

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

次の 2 つのことをホスト ゲートウェイの両方の Windows/Linux でネットワークを有効にする必要があります。

`net-conf.json` ] セクションの kube flannel.yml を確認してください。
1. 使用されているネットワーク バックエンドの種類に設定されて`host-gw`の代わりに`vxlan`します。
2. クラスター サブネット (「10.244.0.0/16」など) の設定に応じてします。

2 つの手順を適用した後、`net-conf.json`は次のようになります。
```json
net-conf.json: |
    {
      "Network": "10.244.0.0/16",
      "Backend": {
        "Type": "host-gw"
      }
    }
```

### <a name="launch-flannel--validate"></a>Flannel を起動すると、検証します。 ###
Flannel の使用を開始するには。

```bash
kubectl apply -f kube-flannel.yml
```

次に、Linux ベース Flannel ポッドのため、適用 Linux [NodeSelector](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml)当社パッチ`kube-flannel-ds`DaemonSet のみ Linux (おはプロセスが起動 Flannel"flanneld"ホスト エージェント windows 後に参加するとき) を対象とします。

```
kubectl patch ds/kube-flannel-ds --patch "$(cat node-selector-patch.yml)" -n=kube-system
```

しばらくすると、すべてのポッド Flannel ポッド ネットワークが配置されている場合に実行するとが表示されます。

```bash
kubectl get pods --all-namespaces
```

![テキスト](media/kube-master.png)

Flannel DaemonSet も適用される NodeSelector が必要です。

```bash
kubectl get ds -n kube-system
```

![テキスト](media/kube-daemonset.png)
> [!tip]  
> わかりにくいですか。 ここでは、既定クラスター サブネットの適用済み次の 2 つの手順で完全な[例 kube-flannel.yml](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/manifests/kube-flannel-example.yml)の Flannel v0.9.1`10.244.0.0/16`します。

## <a name="configuring-a-tor-switch"></a>ToR スイッチを設定します。 ##
> [!NOTE]
> [ネットワーク ソリューションとして Flannel](#flannel-in-host-gateway-mode)を選択した場合は、このセクションを省略できます。
ToR スイッチの構成は、実際のノードの外部で発生します。 詳細については、[正式な Kubernetes ドキュメント](https://kubernetes.io/docs/getting-started-guides/windows/#upstream-l3-routing-topology)を参照してください。


## <a name="next-steps"></a>次のステップ ## 
このセクションで、ネットワーク ソリューションを選択する方法を説明します。 手順 4 の準備が整いました。

> [!div class="nextstepaction"]
> [Windows の作業者への参加](./joining-windows-workers.md)