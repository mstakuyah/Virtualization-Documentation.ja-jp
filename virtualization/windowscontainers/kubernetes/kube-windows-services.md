---
title: Windows サービスとしての Kubernetes の実行
author: daschott
ms.author: daschott
ms.date: 02/12/2019
ms.topic: get-started-article
ms.prod: containers
description: Kubernetes コンポーネントを Windows サービスとして実行する方法について説明します。
keywords: kubernetes、1.14、windows、はじめに
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5c18
ms.openlocfilehash: cd5026a244b57b5c70d4abfe076839130315a4f5
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909802"
---
# <a name="kubernetes-components-as-windows-services"></a>Windows サービスとしての Kubernetes コンポーネント 

一部のユーザーは、flanneld、kubelet、kube-proxy などのプロセスを Windows サービスとして実行するように構成したい場合があります。 これにより、プロセスが予期しないプロセスまたはノードクラッシュに応じて自動的に再起動するなど、追加のフォールトトレランスの利点がもたらされます。


## <a name="prerequisites"></a>前提条件
1. `c:\k` ディレクトリに[nssm.exe](https://nssm.cc/download)をダウンロードしました。
2. ノードをクラスターに参加させ、前のノードで[install.](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/install.ps1) ps1 または[start. ps1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/start.ps1)スクリプトを実行します。

## <a name="registering-windows-services"></a>Windows サービスの登録
`kubelet`、`kube-proxy`、`flanneld.exe` を登録する nssm.exe を使用して、バックグラウンドで Windows サービスとして実行する[サンプルスクリプトを](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/register-svc.ps1)実行できます。

```
C:\k\register-svc.ps1 -NetworkMode <Network mode> -ManagementIP <Windows Node IP> -ClusterCIDR <Cluster subnet> -KubeDnsServiceIP <Kube-dns Service IP> -LogDir <Directory to place logs>
```

# <a name="managementiptabmanagementip"></a>[ManagementIP](#tab/ManagementIP)
Windows ノードに割り当てられた IP アドレス。 `ipconfig` を使用すると、これを見つけることができます。

|  |  | 
|---------|---------|
|パラメーター     | `-ManagementIP`        |
|既定値    | n.A.        |


# <a name="networkmodetabnetworkmode"></a>[NetworkMode](#tab/NetworkMode)
ネットワークモード `l2bridge` (flannel host gw)、または[ネットワークソリューション](./network-topologies.md)として選択された `overlay` (flannel vxlan)。

> [!Important] 
> `overlay` ネットワークモード (flannel vxlan) には、Kubernetes v 1.14 バイナリ以上が必要です。

|  |  | 
|---------|---------|
|パラメーター     | `-NetworkMode`        |
|既定値    | `l2bridge`        |


# <a name="clustercidrtabclustercidr"></a>[ClusterCIDR](#tab/ClusterCIDR)
[クラスターサブネットの範囲](./getting-started-kubernetes-windows.md#cluster-subnet-def)。

|  |  | 
|---------|---------|
|パラメーター     | `-ClusterCIDR`        |
|既定値    | `10.244.0.0/16`        |


# <a name="kubednsserviceiptabkubednsserviceip"></a>[KubeDnsServiceIP](#tab/KubeDnsServiceIP)
[KUBERNETES DNS サービス IP](./getting-started-kubernetes-windows.md#kube-dns-def)。

|  |  | 
|---------|---------|
|パラメーター     | `-KubeDnsServiceIP`        |
|既定値    | `10.96.0.10`        |


# <a name="logdirtablogdir"></a>[LogDir](#tab/LogDir)
Kubelet および kube ログがそれぞれの出力ファイルにリダイレクトされるディレクトリ。

|  |  | 
|---------|---------|
|パラメーター     | `-LogDir`        |
|既定値    | `C:\k`        |

---


> [!TIP] 
> 何か問題が発生した場合は、 [「トラブルシューティング」セクション](./common-problems.md#i-have-problems-running-kubernetes-processes-as-windows-services)を参照してください。

## <a name="manual-approach"></a>手動によるアプローチ
上記の[参照スクリプト](#registering-windows-services)が機能しないようにするために、このセクションでは、これらのサービスを手動で登録するために使用できる*サンプルコマンド*をいくつか紹介します。

> [!TIP] 
> `sc`を介してネイティブ Windows サービスとして実行する `kubelet` および `kube-proxy` を構成する方法の詳細については、「 [Kubelet and kube To Windows services](https://kubernetes.io/docs/getting-started-guides/windows/#kubelet-and-kube-proxy-can-now-run-as-windows-services) 」を参照してください。

### <a name="register-flanneldexe"></a>Flanneld の登録
```
nssm install flanneld C:\flannel\flanneld.exe
nssm set flanneld AppParameters --kubeconfig-file=c:\k\config --iface=<ManagementIP> --ip-masq=1 --kube-subnet-mgr=1
nssm set flanneld AppEnvironmentExtra NODE_NAME=<hostname>
nssm set flanneld AppDirectory C:\flannel
nssm start flanneld
```

### <a name="register-kubeletexe"></a>Kubelet の登録
```
nssm install kubelet C:\k\kubelet.exe
nssm set kubelet AppParameters --hostname-override=<hostname> --v=6 --pod-infra-container-image=kubeletwin/pause --resolv-conf="" --allow-privileged=true --enable-debugging-handlers --cluster-dns=<DNS-service-IP> --cluster-domain=cluster.local --kubeconfig=c:\k\config --hairpin-mode=promiscuous-bridge --image-pull-progress-deadline=20m --cgroups-per-qos=false  --log-dir=<log directory> --logtostderr=false --enforce-node-allocatable="" --network-plugin=cni --cni-bin-dir=c:\k\cni --cni-conf-dir=c:\k\cni\config
nssm set kubelet AppDirectory C:\k
nssm start kubelet
```

### <a name="register-kube-proxyexe-l2bridge--host-gw"></a>Kube-proxy (l2bridge/host gw) を登録します。
```
nssm install kube-proxy C:\k\kube-proxy.exe
nssm set kube-proxy AppDirectory c:\k
nssm set kube-proxy AppParameters --v=4 --proxy-mode=kernelspace --hostname-override=<hostname>--kubeconfig=c:\k\config --enable-dsr=false --log-dir=<log directory> --logtostderr=false
nssm.exe set kube-proxy AppEnvironmentExtra KUBE_NETWORK=cbr0
nssm set kube-proxy DependOnService kubelet
nssm start kube-proxy
```

### <a name="register-kube-proxyexe-overlay--vxlan"></a>Kube-proxy を登録します (オーバーレイ/vxlan)
```
PS C:\k> nssm install kube-proxy C:\k\kube-proxy.exe
PS C:\k> nssm set kube-proxy AppDirectory c:\k
PS C:\k> nssm set kube-proxy AppParameters --v=4 --proxy-mode=kernelspace --feature-gates="WinOverlay=true" --hostname-override=<hostname> --kubeconfig=c:\k\config --network-name=vxlan0 --source-vip=<source-vip> --enable-dsr=false --log-dir=<log directory> --logtostderr=false
nssm set kube-proxy DependOnService kubelet
nssm start kube-proxy
```