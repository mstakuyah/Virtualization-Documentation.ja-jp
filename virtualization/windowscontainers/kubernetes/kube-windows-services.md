---
title: Windows サービスとして Kubernetes を実行します。
author: daschott
ms.author: daschott
ms.date: 02/12/2019
ms.topic: get-started-article
ms.prod: containers
description: Windows サービスとして Kubernetes コンポーネントを実行する方法。
keywords: kubernetes、1.13、windows、作業の開始
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5c18
ms.openlocfilehash: 6c68edda6e2017640b0a490c3c30f063c81698b3
ms.sourcegitcommit: 41318edba7459a9f9eeb182bf8519aac0996a7f1
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/28/2019
ms.locfileid: "9120593"
---
# <a name="kubernetes-components-as-windows-services"></a>Windows サービスとして Kubernetes コンポーネント 

一部のユーザーは、flanneld.exe、kubelet.exe、kube proxy.exe またはその他のユーザーとして Windows サービスを実行するなどのプロセスを構成する必要があります。 追加のフォールト トレランスが、予期しないプロセスまたはノードのクラッシュ時に自動的に再起動プロセスなどの主な利点が表示されます。


## <a name="prerequisites"></a>前提条件
1. ダウンロードしたに[nssm.exe](https://nssm.cc/download) `c:\k`ディレクトリ
2. 自分のクラスターにノードを結合してスクリプトを実行する[install.ps1](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/install.ps1)または[start.ps1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/start.ps1)ノード以前

## <a name="registering-windows-services"></a>Windows サービスを登録します。
[サンプルのスクリプト](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/register-svc.ps1)を実行するとする関数を使用して nssm.exe に登録する`kubelet`、 `kube-proxy`、および`flanneld.exe`として Windows サービスをバック グラウンドで実行します。

```
C:\k\register-svc.ps1 -NetworkMode <Network mode> -ManagementIP <Windows Node IP> -ClusterCIDR <Cluster subnet> -KubeDnsServiceIP <Kube-dns Service IP> -LogDir <Directory to place logs>
```

# [<a name="managementip"></a>ManagementIP](#tab/ManagementIP)
Windows ノードに割り当てられている IP アドレス。 使える`ipconfig`を確認します。

|  |  | 
|---------|---------|
|パラメーター     | `-ManagementIP`        |
|既定値    | n.A.        |


# [<a name="networkmode"></a>NetworkMode](#tab/NetworkMode)
ネットワーク モード`l2bridge`(flannel ホスト校) または`overlay`(flannel vxlan) として[ネットワーク ソリューション](./network-topologies.md)を選択します。

> [!Important] 
> `overlay` ネットワークのモード (flannel vxlan) 必要 Kubernetes v1.14 バイナリまたはそれ以降。

|  |  | 
|---------|---------|
|パラメーター     | `-NetworkMode`        |
|既定値    | `l2bridge`        |


# [<a name="clustercidr"></a>ClusterCIDR](#tab/ClusterCIDR)
[クラスター サブネット範囲](./getting-started-kubernetes-windows.md#cluster-subnet-def)です。

|  |  | 
|---------|---------|
|パラメーター     | `-ClusterCIDR`        |
|既定値    | `10.244.0.0/16`        |


# [<a name="kubednsserviceip"></a>KubeDnsServiceIP](#tab/KubeDnsServiceIP)
[Kubernetes DNS サービス IP](./getting-started-kubernetes-windows.md#kube-dns-def)します。

|  |  | 
|---------|---------|
|パラメーター     | `-KubeDnsServiceIP`        |
|既定値    | `10.96.0.10`        |


# [<a name="logdir"></a>LogDir](#tab/LogDir)
それぞれのそれぞれの出力ファイルに kubelet と kube プロキシのログのリダイレクト先のディレクトリです。

|  |  | 
|---------|---------|
|パラメーター     | `-LogDir`        |
|既定値    | `C:\k`        |

---


> [!TIP] 
> 移動か。 何かが正しくない、[セクションのトラブルシューティング](./common-problems.md#i-have-problems-running-kubernetes-processes-as-windows-services)を参照してください。

## <a name="manual-approach"></a>手動による方法
必要があります、[上記の参照先のスクリプト](#registering-windows-services)されませんが、このセクションは、いくつかの*コマンドのサンプル*これらのサービスを手動でステップ バイ ステップを登録するために使用します。

> [!TIP] 
> 構成する方法の詳細については[Kubelet と kube プロキシが、Windows サービスとして実行できるようになりました](https://kubernetes.io/docs/getting-started-guides/windows/#kubelet-and-kube-proxy-can-now-run-as-windows-services)を参照してください`kubelet`と`kube-proxy`で Windows サービスをネイティブとして実行する`sc`します。

### <a name="register-flanneldexe"></a>Flanneld.exe を登録します。
```
nssm install flanneld C:\flannel\flanneld.exe
nssm set flanneld AppParameters --kubeconfig-file=c:\k\config --iface=<ManagementIP> --ip-masq=1 --kube-subnet-mgr=1
nssm set flanneld AppEnvironmentExtra NODE_NAME=<hostname>
nssm set flanneld AppDirectory C:\flannel
nssm start flanneld
```

### <a name="register-kubeletexe"></a>Kubelet.exe を登録します。
```
nssm install kubelet C:\k\kubelet.exe
nssm set kubelet AppParameters --hostname-override=<hostname> --v=6 --pod-infra-container-image=kubeletwin/pause --resolv-conf="" --allow-privileged=true --enable-debugging-handlers --cluster-dns=<DNS-service-IP> --cluster-domain=cluster.local --kubeconfig=c:\k\config --hairpin-mode=promiscuous-bridge --image-pull-progress-deadline=20m --cgroups-per-qos=false  --log-dir=<log directory> --logtostderr=false --enforce-node-allocatable="" --network-plugin=cni --cni-bin-dir=c:\k\cni --cni-conf-dir=c:\k\cni\config
nssm set kubelet AppDirectory C:\k
nssm start kubelet
```

### <a name="register-kube-proxyexe-l2bridge--host-gw"></a>登録 kube proxy.exe (l2bridge/ホスト校)
```
nssm install kube-proxy C:\k\kube-proxy.exe
nssm set kube-proxy AppDirectory c:\k
nssm set kube-proxy AppParameters --v=4 --proxy-mode=kernelspace --hostname-override=<hostname>--kubeconfig=c:\k\config --enable-dsr=false --log-dir=<log directory> --logtostderr=false
nssm.exe set kube-proxy AppEnvironmentExtra KUBE_NETWORK=cbr0
nssm set kube-proxy DependOnService kubelet
nssm start kube-proxy
```

### <a name="register-kube-proxyexe-overlay--vxlan"></a>登録 kube proxy.exe (オーバーレイ/vxlan)
```
PS C:\k> nssm install kube-proxy C:\k\kube-proxy.exe
PS C:\k> nssm set kube-proxy AppDirectory c:\k
PS C:\k> nssm set kube-proxy AppParameters --v=4 --proxy-mode=kernelspace --feature-gates="WinOverlay=true" --hostname-override=<hostname> --kubeconfig=c:\k\config --network-name=vxlan0 --source-vip=<source-vip> --enable-dsr=false --log-dir=<log directory> --logtostderr=false
nssm set kube-proxy DependOnService kubelet
nssm start kube-proxy
```