---
title: ネットワークトポロジ
author: daschott
ms.author: daschott
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: Windows と Linux でサポートされているネットワークトポロジ。
keywords: kubernetes、1.14、windows、はじめに
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 6b0e13258b749ad3dfd5c8349200ca8a54908952
ms.sourcegitcommit: 42cb47ba4f3e22163869d094bd0c9cff415a43b0
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/02/2019
ms.locfileid: "9884983"
---
# <a name="network-solutions"></a>Network Solutions #

[Kubernetes master ノードのセットアップ](./creating-a-linux-master.md)が完了したら、ネットワークソリューションを選ぶことができます。 複数の方法で、複数のノードにわたって仮想[クラスターサブネット](./getting-started-kubernetes-windows.md#cluster-subnet-def)をルーティングすることができます。 Windows の Kubernetes で、次のいずれかのオプションを選びます。

1. [フランネル](#flannel-in-vxlan-mode)などの CNI プラグインを使用して、オーバーレイネットワークをセットアップします。
2. [フランネル](#flannel-in-host-gateway-mode)などの CNI プラグインを使ってルートをプログラムします (l2bridge のネットワークモードを使用します)。
3. スマートな[上位ラック (ToR) スイッチ](#configuring-a-tor-switch)を構成して、サブネットをルーティングします。

> [!tip]  
> Windows には4番目のネットワークソリューションがあります。このソリューションは、オープン vSwitch (OvS) と Open Virtual Network ("Open n") を使用します。 このドキュメントは、このドキュメントの範囲外ですが、これらの[手順](https://kubernetes.io/docs/getting-started-guides/windows/#for-3-open-vswitch-ovs-open-virtual-network-ovn-with-overlay)を読み、セットアップすることができます。

## <a name="flannel-in-vxlan-mode"></a>Vxlan モードでのフランネル

Vxlan モードでフランネルを使用すると、構成可能な仮想オーバーレイネットワーク (VXLAN トンネリングを使用して、ノード間でパケットをルーティングする) を設定できます。

### <a name="prepare-kubernetes-master-for-flannel"></a>フランネルの Kubernetes master を準備する
クラスターの[Kubernetes master](./creating-a-linux-master.md)では、いくつかの小さな準備を行うことをお勧めします。 フランネルを使用する場合は、ブリッジされた IPv4 トラフィックを iptables チェーンに有効にすることをお勧めします。 これは、次のコマンドを使用して行うことができます。

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```

###  <a name="download--configure-flannel"></a>フランネルをダウンロード & 構成する ###
最新のフランネルマニフェストをダウンロードします。

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

Vxlan ネットワークバックエンドを有効にするには、次の2つのセクションを変更する必要があります。

1. の`net-conf.json`セクションで`kube-flannel.yml`、次のようにします。
 * クラスターのサブネット ("10.244.0.0/16" など) は、必要に応じて設定されます。
 * VNI 4096 はバックエンドで設定されています
 * ポート4789はバックエンドで設定されています
2. の`cni-conf.json`セクションで`kube-flannel.yml`、ネットワーク名をに`"vxlan0"`変更します。

上記の手順を適用`net-conf.json`すると、次のようになります。
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
> Windows のフランネルと相互運用するには、Linux のフランネル用に VNI を4096およびポート4789に設定する必要があります。 他の VNIs のサポートは近日中にサポートされます。 これらのフィールドの説明については、「 [VXLAN](https://github.com/coreos/flannel/blob/master/Documentation/backends.md#vxlan) 」を参照してください。

次`cni-conf.json`のようになります。
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
> 上記のオプションの詳細については、Linux 向けの公式の CNI[フランネル](https://github.com/containernetworking/plugins/tree/master/plugins/meta/flannel#network-configuration-reference)、[ポートマップ](https://github.com/containernetworking/plugins/tree/master/plugins/meta/portmap#port-mapping-plugin)、および[ブリッジ](https://github.com/containernetworking/plugins/tree/master/plugins/main/bridge#network-configuration-reference)プラグインに関するドキュメントを参照してください。

### <a name="launch-flannel--validate"></a>フランネルの起動 & 検証 ###
次を使用してフランネルを起動します。

```bash
kubectl apply -f kube-flannel.yml
```

次に、フランネルポッドは Linux ベースであるため、DaemonSet に`kube-flannel-ds` Linux の[nodeselector](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml)パッチを適用して、ターゲットとなる linux のみを対象としています (これは、後で参加するときに、Windows でフランネル "flanneld" のホストエージェントのプロセスを起動します)。

```
kubectl patch ds/kube-flannel-ds-amd64 --patch "$(cat node-selector-patch.yml)" -n=kube-system
```
> [!tip]  
> どのノードも x86 ベース以外の場合は、 `-amd64`上記のプロセッサアーキテクチャに置き換えます。

数分後に、フランネルポッドネットワークが展開された場合は、すべての pod が実行中であることを確認する必要があります。

```bash
kubectl get pods --all-namespaces
```

![テキスト](media/kube-master.png)

フランネル DaemonSet には、NodeSelector `beta.kubernetes.io/os=linux`も適用されている必要があります。

```bash
kubectl get ds -n kube-system
```

![テキスト](media/kube-daemonset.png)

> [!tip]  
> 残りのフランネル-* デーモンセットでは、プロセッサアーキテクチャに一致するノードがない場合は、これらを無視するか、削除することができます。

> [!tip]  
> 困惑? 次に、フランネル v 0.11.0 の[yml の例](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/overlay/manifests/kube-flannel-example.yml)を示します。これらの手順は、既定のクラスター `10.244.0.0/16`サブネットで事前に適用されています。

成功したら、次の[手順](#next-steps)に進みます。

## <a name="flannel-in-host-gateway-mode"></a>フランネル in host-gateway モード

[フランネル vxlan](#flannel-in-vxlan-mode)では、フランネルネットワーキング用のもう1つのオプションは*host-gateway モード*(ホスト gw) です。これにより、ターゲットノードのホストアドレスを次のホップとして使用して、各ノードの静的ルートのプログラミングが、他のノードの pod サブネットにも適用されます。

### <a name="prepare-kubernetes-master-for-flannel"></a>フランネルの Kubernetes master を準備する

クラスターの[Kubernetes master](./creating-a-linux-master.md)では、いくつかの小さな準備を行うことをお勧めします。 フランネルを使用する場合は、ブリッジされた IPv4 トラフィックを iptables チェーンに有効にすることをお勧めします。 これは、次のコマンドを使用して行うことができます。

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```


###  <a name="download--configure-flannel"></a>フランネルをダウンロード & 構成する ###
最新のフランネルマニフェストをダウンロードします。

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

Windows/Linux の両方でホスト/ゲートウェイのネットワークを有効にするには、1つのファイルに変更を行う必要があります。

Kube-flannel の`net-conf.json`セクションで、次の項目をダブルチェックします。 yml
1. 使用されているネットワークバックエンドの種類`host-gw`は、 `vxlan`ではなくに設定されています。
2. クラスターのサブネット ("10.244.0.0/16" など) は、必要に応じて設定されます。

2つの手順を適用`net-conf.json`すると、次のようになります。
```json
net-conf.json: |
    {
      "Network": "10.244.0.0/16",
      "Backend": {
        "Type": "host-gw"
      }
    }
```

### <a name="launch-flannel--validate"></a>フランネルの起動 & 検証 ###
次を使用してフランネルを起動します。

```bash
kubectl apply -f kube-flannel.yml
```

次に、フランネルポッドは Linux ベースであるため、linux の[Nodeselector](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml)パッチを`kube-flannel-ds` DaemonSet に適用して、ターゲットとなる linux のみを対象としています (これは、後で参加するときに、Windows でフランネル "flanneld" のホストエージェントのプロセスを起動します)。

```
kubectl patch ds/kube-flannel-ds-amd64 --patch "$(cat node-selector-patch.yml)" -n=kube-system
```
> [!tip]  
> どのノードも x86 ベース以外の場合は、 `-amd64`上記のように目的のプロセッサアーキテクチャに置き換えます。

数分後に、フランネルポッドネットワークが展開された場合は、すべての pod が実行中であることを確認する必要があります。

```bash
kubectl get pods --all-namespaces
```

![テキスト](media/kube-master.png)

フランネル DaemonSet には、NodeSelector も適用されている必要があります。

```bash
kubectl get ds -n kube-system
```

![テキスト](media/kube-daemonset.png)

> [!tip]  
> 残りのフランネル-* デーモンセットでは、プロセッサアーキテクチャに一致するノードがない場合は、これらを無視するか、削除することができます。

> [!tip]  
> 困惑? フランネル v 0.11.0 の[yml の例](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/manifests/kube-flannel-example.yml)を次に示します。これらの2つの手順は、既定の`10.244.0.0/16`クラスターサブネットで事前に適用されています。

成功したら、次の[手順](#next-steps)に進みます。

## <a name="configuring-a-tor-switch"></a>ToR スイッチの構成 ##
> [!NOTE]
> [ネットワークソリューションとしてフランネル](#flannel-in-host-gateway-mode)を選択した場合は、このセクションをスキップできます。
ToR スイッチの構成は、実際のノードの外部で行われます。 詳細については、「[公式の Kubernetes ドキュメント](https://kubernetes.io/docs/getting-started-guides/windows/#upstream-l3-routing-topology)」を参照してください。


## <a name="next-steps"></a>次のステップ ## 
このセクションでは、ネットワークソリューションを選択して構成する方法について説明します。 これで、手順4の準備が整いました。

> [!div class="nextstepaction"]
> [Windows ワーカーへの参加](./joining-windows-workers.md)