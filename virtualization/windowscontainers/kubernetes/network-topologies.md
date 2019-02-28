---
title: ネットワーク トポロジ
author: daschott
ms.author: daschott
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: Windows と Linux ではサポートされているネットワーク トポロジします。
keywords: kubernetes、1.13、windows、作業の開始
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 9f96fcc80c533b74ab46d93beecc7ca8629ce395
ms.sourcegitcommit: 41318edba7459a9f9eeb182bf8519aac0996a7f1
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/28/2019
ms.locfileid: "9120450"
---
# <a name="network-solutions"></a>Network Solutions #

[Kubernetes マスター ノードをセットアップ](./creating-a-linux-master.md)した後は、ネットワーク ソリューションを選択する準備が整います。 仮想[クラスター サブネット](./getting-started-kubernetes-windows.md#cluster-subnet-def)ルーティング可能なノード間での複数の方法があります。 Windows Kubernetes のオプションを次のいずれかを今日選びます。

1. オーバーレイ ネットワークをセットアップするのにには、 [Flannel](#flannel-in-vxlan-mode)などいる CNI プラグインを使用します。
2. プログラムのルートに[Flannel](#flannel-in-host-gateway-mode)などいる CNI プラグインを使用します。
3. スマート[ラック上部 (ToR) の切り替え](#configuring-a-tor-switch)にサブネットのルーティングを構成します。

> [!tip]  
> 4 つ目のネットワーク ソリューションを開いているとして (OvS) を活用するウィンドウと開いている仮想ネットワーク (OVN) があります。 これをドキュメント化するが、このドキュメントの範囲外を設定するのには、[次の手順](https://kubernetes.io/docs/getting-started-guides/windows/#for-3-open-vswitch-ovs-open-virtual-network-ovn-with-overlay)をお読みください。

## <a name="flannel-in-vxlan-mode"></a>Vxlan モードで flannel します。

Flannel vxlan モードでは、VXLAN のトンネル ノードの間でのルーティング パケットを使用する設定可能な仮想オーバーレイ ネットワークのセットアップに使用できます。

### <a name="prepare-kubernetes-master-for-flannel"></a>Flannel 用 Kubernetes マスターを準備します。
クラスターで[Kubernetes マスター](./creating-a-linux-master.md)では、いくつかマイナーの準備をお勧めします。 Flannel を使用する場合は、iptables チェーン ブリッジ IPv4 トラフィックを有効にすることをお勧めします。 これは、次のコマンドを使用します。

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```

###  <a name="download--configure-flannel"></a>ダウンロード & Flannel を構成します。 ###
最新の Flannel マニフェストをダウンロードしてください。

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

Vxlan ネットワーク バックエンドを有効に変更することは 2 つのセクションがあります。

1. `net-conf.json`のセクションで、 `kube-flannel.yml`、確認してください。
 * クラスター サブネット (「10.244.0.0/16」など) の設定に応じてします。
 * バックエンドで VNI 4096 が設定します。
 * ポート 4789 がバックエンドを設定します。
2. `cni-conf.json`のセクションで、 `kube-flannel.yml`、ネットワーク名に変更`"vxlan0"`します。

上記の手順を適用した後、`net-conf.json`は次のようになります。
```json
  net-conf.json: |
    {
      "Network": "10.244.0.0/16",
      "Backend": {
        "Type": "vxlan",
        "VNI" : 4096,
        "Port": 4789
      }
    }
```

> [!NOTE]  
> Windows Flannel と相互運用する 4096 と linux Flannel のポート 4789、VNI を設定する必要があります。 その他の VNIs のサポートは近日公開します。 これらのフィールドの詳細については、 [VXLAN](https://github.com/coreos/flannel/blob/master/Documentation/backends.md#vxlan)を参照してください。

`cni-conf.json`は次のようになります。
```json
cni-conf.json: |
    {
      "name": "vxlan0",
      "plugins": [
        {
          "type": "flannel",
          "delegate": {
            "hairpinMode": true,
            "isDefaultGateway": true
          }
        },
        {
          "type": "portmap",
          "capabilities": {
            "portMappings": true
          }
        }
      ]
    }
```
> [!tip]  
> 上記のオプションの詳細については、正式ないる CNI [flannel](https://github.com/containernetworking/plugins/tree/master/plugins/meta/flannel#network-configuration-reference)、 [portmap](https://github.com/containernetworking/plugins/tree/master/plugins/meta/portmap#port-mapping-plugin)、[ブリッジ](https://github.com/containernetworking/plugins/tree/master/plugins/main/bridge#network-configuration-reference)プラグイン ドキュメント linux を参照してください。

### <a name="launch-flannel--validate"></a>サイド バー Flannel & を検証します。 ###
Flannel の使用を開始するには。

```bash
kubectl apply -f kube-flannel.yml
```

次に、Linux ベース Flannel ポッドので、Linux [NodeSelector](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml)の更新プログラムを適用`kube-flannel-ds`DaemonSet のみ Linux (おはプロセスが起動 Flannel"flanneld"ホスト エージェント windows 後に参加するとき) を対象とします。

```
kubectl patch ds/kube-flannel-ds-amd64 --patch "$(cat node-selector-patch.yml)" -n=kube-system
```
> [!tip]  
> X86 64 ベース ノードがある場合は、置き換える`-amd64`上にある、プロセッサ アーキテクチャとします。

しばらくすると、すべてのポッド Flannel ポッド ネットワークが配置されている場合に実行するとが表示されます。

```bash
kubectl get pods --all-namespaces
```

![テキスト](media/kube-master.png)

Flannel DaemonSet には、NodeSelector する必要があります。`beta.kubernetes.io/os=linux`を適用します。

```bash
kubectl get ds -n kube-system
```

![テキスト](media/kube-daemonset.png)

> [!tip]  
> 残りの flannel ds - の * DaemonSets、これらは無視されますか、プロセッサ アーキテクチャに一致するノードがない場合、スケジュールされず削除します。

> [!tip]  
> わかりにくいですか。 ここでは、既定クラスター サブネットの適用済みの次の手順で Flannel v0.11.0 用の完全な[例 kube-flannel.yml](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/overlay/manifests/kube-flannel-example.yml) `10.244.0.0/16`します。

成功した後は、[次の手順](#next-steps)に進みます。

## <a name="flannel-in-host-gateway-mode"></a>ホスト ゲートウェイ モードで flannel します。

[Flannel vxlan](#flannel-in-vxlan-mode)、と共に Flannel ネットワークの他のオプションは*ホスト ゲートウェイ モード*(ホスト-ゲートウェイ) は、各ターゲット ノードのホストのアドレスを使用して、次のホップとして、その他のノードのポッド サブネット ノードの静的ルートのプログラミングします。

### <a name="prepare-kubernetes-master-for-flannel"></a>Flannel 用 Kubernetes マスターを準備します。

クラスターで[Kubernetes マスター](./creating-a-linux-master.md)では、いくつかマイナーの準備をお勧めします。 Flannel を使用する場合は、iptables チェーン ブリッジ IPv4 トラフィックを有効にすることをお勧めします。 これは、次のコマンドを使用します。

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```


###  <a name="download--configure-flannel"></a>ダウンロード & Flannel を構成します。 ###
最新の Flannel マニフェストをダウンロードしてください。

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

1 つのファイルをホスト ゲートウェイの両方の Windows/Linux でネットワークを有効にするために変更する必要があります。

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

### <a name="launch-flannel--validate"></a>サイド バー Flannel & を検証します。 ###
Flannel の使用を開始するには。

```bash
kubectl apply -f kube-flannel.yml
```

次に、Linux ベース Flannel ポッドのため、適用 Linux [NodeSelector](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml)当社パッチ`kube-flannel-ds`DaemonSet のみ Linux (おはプロセスが起動 Flannel"flanneld"ホスト エージェント windows 後に参加するとき) を対象とします。

```
kubectl patch ds/kube-flannel-ds-amd64 --patch "$(cat node-selector-patch.yml)" -n=kube-system
```
> [!tip]  
> X86 64 ベース ノードがある場合は、置き換える`-amd64`上にある目的のプロセッサ アーキテクチャでします。

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
> 残りの flannel ds - の * DaemonSets、これらは無視されますか、プロセッサ アーキテクチャに一致するノードがない場合、スケジュールされず削除します。

> [!tip]  
> わかりにくいですか。 ここでは、既定クラスター サブネットの適用済み次の 2 つの手順で完全な[例 kube-flannel.yml](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/manifests/kube-flannel-example.yml)の Flannel v0.11.0`10.244.0.0/16`します。

成功した後は、[次の手順](#next-steps)に進みます。

## <a name="configuring-a-tor-switch"></a>ToR スイッチを設定します。 ##
> [!NOTE]
> [ネットワーク ソリューションとして Flannel](#flannel-in-host-gateway-mode)を選択した場合は、このセクションを省略できます。
ToR スイッチの構成は、実際のノードの外部で発生します。 詳細については、[正式な Kubernetes ドキュメント](https://kubernetes.io/docs/getting-started-guides/windows/#upstream-l3-routing-topology)を参照してください。


## <a name="next-steps"></a>次のステップ ## 
このセクションは、選択して、ネットワーク ソリューションを構成する方法を説明します。 手順 4 の準備が整いました。

> [!div class="nextstepaction"]
> [Windows の作業者への参加](./joining-windows-workers.md)