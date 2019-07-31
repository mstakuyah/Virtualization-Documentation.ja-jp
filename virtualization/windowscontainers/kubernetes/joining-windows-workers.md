---
title: Windows ノードへの参加
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: Windows ノードを Kubernetes クラスターに v 1.14 で参加する。
keywords: kubernetes、1.14、windows、はじめに
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: c9dbfec968d52d9fbc528892f0e3749270e3ff70
ms.sourcegitcommit: c4a3f88d1663dd19336bfd4ede0368cb18550ac7
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/31/2019
ms.locfileid: "9882985"
---
# <a name="joining-windows-server-nodes-to-a-cluster"></a>Windows Server ノードをクラスターに参加する #
[Kubernetes master ノードをセットアップ](./creating-a-linux-master.md)し、目的の[ネットワークソリューションを選択](./network-topologies.md)したら、Windows Server ノードに参加してクラスターを形成することができます。 そのために[は、](#preparing-a-windows-node)参加する前に Windows ノードの準備が必要です。

## <a name="preparing-a-windows-node"></a>Windows のノードの準備 ##
> [!NOTE]  
> Windows セクションでのすべてのコード スニペットは、_管理者特権の_ PowerShell で実行します。

### <a name="install-docker-requires-reboot"></a>Docker をインストールする (再起動が必要) ###
Kubernetes は、コンテナーエンジンとして[Docker](https://www.docker.com/)を使います。そのため、インストールする必要があります。 [Docs の公式手順](../manage-docker/configure-docker-daemon.md#install-docker)または [Docker の手順](https://store.docker.com/editions/enterprise/docker-ee-server-windows)に従うことも、以下の手順を試すこともできます。

```powershell
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
Install-Package -Name Docker -ProviderName DockerMsftProvider
Restart-Computer -Force
```

プロキシの内側にいる場合は、次の PowerShell 環境変数を定義する必要があります。
```powershell
[Environment]::SetEnvironmentVariable("HTTP_PROXY", "http://proxy.example.com:80/", [EnvironmentVariableTarget]::Machine)
[Environment]::SetEnvironmentVariable("HTTPS_PROXY", "http://proxy.example.com:443/", [EnvironmentVariableTarget]::Machine)
```

再起動後に、次のエラーが表示されます。

![テキスト](media/docker-svc-error.png)

次に、docker サービスを手動で開始します。

```powershell
Start-Service docker
```

### <a name="create-the-pause-infrastructure-image"></a>"一時停止" (インフラストラクチャ) 画像を作成する ###
> [!Important]
> コンテナーイメージの競合には注意が必要です。予期しないタグが含まれて`docker pull`いると、互換性のないコンテナーのイメージが発生し`ContainerCreating` 、無限状態などの[展開の問題](./common-problems.md#when-deploying-docker-containers-keep-restarting)が発生する可能性があります。

`docker` のインストールが完了したため、Kubernetes インフラストラクチャ ポッドを準備するために Kubernetes で使用される "pause" イメージを準備する必要があります。 これには、次の3つの手順があります。 
  1. [画像を引き出す](#pull-the-image)
  2. microsoft/nanoserver として[タグ付け](#tag-the-image)する: 最新
  3. と[実行](#run-the-container)


#### <a name="pull-the-image"></a>画像を引き出す ####     
 特定の Windows リリースのイメージを取得します。 たとえば、Windows Server 2019 を実行している場合は、次のようになります。

 ```powershell
docker pull mcr.microsoft.com/windows/nanoserver:1809
 ```

#### <a name="tag-the-image"></a>画像にタグを付ける ####
このガイドの後半で使用する Dockerfiles は、 `:latest`イメージタグを探しています。 次に示すように、nanoserver の画像にタグを付けます。

```powershell
docker tag mcr.microsoft.com/windows/nanoserver:1809 microsoft/nanoserver:latest
```

#### <a name="run-the-container"></a>コンテナーを実行する ####
自分のコンピューターでコンテナーが実際に実行されていることをもう一度確認します。

```powershell
docker run microsoft/nanoserver:latest
```

次のようなものが表示されるはずです。

![テキスト](./media/docker-run-sample.png)

> [!tip]
> コンテナーを実行できない場合は、「コンテナーの[バージョンと一致するコンテナーホストのバージョン」を](https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/version-compatibility#matching-container-host-version-with-container-image-versions)参照してください。


#### <a name="prepare-kubernetes-for-windows-directory"></a>Kubernetes for Windows ディレクトリを準備する ####
"Windows 用 Kubernetes" ディレクトリを作成して、Kubernetes バイナリと展開スクリプトと構成ファイルを保存します。

```powershell
mkdir c:\k
```

#### <a name="copy-kubernetes-certificate"></a>Kubernetes 証明書をコピーする #### 
Kubernetes certificate file (`$HOME/.kube/config`) を[master から](./creating-a-linux-master.md#collect-cluster-information)この新しい`C:\k`ディレクトリにコピーします。

> [!tip]
> [Xcopy](https://docs.microsoft.com/windows-server/administration/windows-commands/xcopy)や[winscp](https://winscp.net/eng/download.php)などのツールを使って、ノード間で構成ファイルを転送することができます。

#### <a name="download-kubernetes-binaries"></a>Kubernetes バイナリをダウンロードする ####
Kubernetes を実行できるようにするには、まず、、 `kubectl` `kube-proxy`バイナリ`kubelet`をダウンロードする必要があります。 これらは、 `CHANGELOG.md` [最新リリース](https://github.com/kubernetes/kubernetes/releases/)のファイル内のリンクからダウンロードできます。
 - たとえば、次に示すように、 [v 1.14 ノードのバイナリ](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.14.md#node-binaries)を示します。
 - [展開/アーカイブ](https://docs.microsoft.com/powershell/module/microsoft.powershell.archive/expand-archive?view=powershell-6)などのツールを使用してアーカイブを抽出し、バイナリ`C:\k\`をに配置します。

#### <a name="optional-setup-kubectl-on-windows"></a>省略Windows でのセットアップ kubectl ####
Windows からクラスターを制御したい場合は、 `kubectl`コマンドを使用します。 まず、 `kubectl` `C:\k\`ディレクトリ外で利用できるようにするには`PATH` 、環境変数を変更します。

```powershell
$env:Path += ";C:\k"
```

この変更を永続化する場合は、コンピューター ターゲット内の変数を変更します。

```powershell
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\k", [EnvironmentVariableTarget]::Machine)
```

次に、[クラスター証明書](#copy-kubernetes-certificate)が有効であることを確認します。 構成ファイルを`kubectl`検索する場所を設定するには、 `--kubeconfig`パラメーターを渡すか、 `KUBECONFIG`環境変数を変更します。 たとえば、`C:\k\config` に構成がある場合は、次のように設定します。

```powershell
$env:KUBECONFIG="C:\k\config"
```

現在のユーザーのスコープでこの設定を永続化するには、次のように設定します。

```powershell
[Environment]::SetEnvironmentVariable("KUBECONFIG", "C:\k\config", [EnvironmentVariableTarget]::User)
```

最後に、構成が正しく検出されたかどうかを確認するには、次のように使用できます。

```powershell
kubectl config view
```

接続エラーが発生する場合は、

```
Unable to connect to the server: dial tcp [::1]:8080: connectex: No connection could be made because the target machine actively refused it.
```

Kubeconfig の場所をもう一度確認するか、もう一度コピーしてみてください。

エラーが表示されない場合は、ノードがクラスターに参加する準備ができました。

## <a name="joining-the-windows-node"></a>Windows ノードへの参加 ##
[選択したネットワークソリューション](./network-topologies.md)に応じて、次のことができます。
1. [Windows Server ノードをフランネル (vxlan または host-gw) クラスターに参加する](#joining-a-flannel-cluster)
2. [Windows Server ノードを ToR スイッチを使用してクラスターに参加する](#joining-a-tor-cluster)

### <a name="joining-a-flannel-cluster"></a>フランネルクラスターへの参加 ###
この[Microsoft リポジトリ](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/overlay)には、このノードをクラスターに参加するのに役立つフランネル展開スクリプトのコレクションが含まれています。

フランネルの[最初](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/start.ps1)のスクリプトをダウンロードします。次の内容を`C:\k`抽出します。

```powershell
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/start.ps1 -o c:\k\start.ps1
```

[Windows ノードを準備](#preparing-a-windows-node)したことを前提`c:\k`としている場合、ディレクトリは以下のようになっているため、ノードに参加することができます。

![テキスト](./media/flannel-directory.png)

#### <a name="join-node"></a>参加ノード #### 
Windows ノードへの参加プロセスを簡単にするために、1つの windows スクリプト`kubelet` `kube-proxy` `flanneld`を実行して、、、の各ノードに参加する必要があります。

> [!Note]
> [start](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/start.ps1)を参照する[](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/install.ps1)と、 `flanneld`実行可能ファイルや、[インフラストラクチャポッド用の dockerfile](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/Dockerfile)などの追加ファイルをダウンロードし、それを*インストールすること*になります。 オーバーレイネットワークモードの場合、[ファイアウォール](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/helper.psm1#L111)はローカル UDP ポート4789用に開かれます。 Pod ネットワークの新しい外部 vSwitch が初めて作成されている間、複数の powershell ウィンドウを開いたり閉じたりすることができます。数秒のネットワークが停止する場合もあります。

```powershell
cd c:\k
.\start.ps1 -ManagementIP <Windows Node IP> -NetworkMode <network mode>  -ClusterCIDR <Cluster CIDR> -ServiceCIDR <Service CIDR> -KubeDnsServiceIP <Kube-dns Service IP> -LogDir <Log directory>
```
# [<a name="managementip"></a>ManagementIP](#tab/ManagementIP)
Windows ノードに割り当てられている IP アドレス。 これは、 `ipconfig`を使用して見つけることができます。

|  |  | 
|---------|---------|
|パラメーター     | `-ManagementIP`        |
|既定値    | n.A. **required**        |

# [<a name="networkmode"></a>NetworkMode](#tab/NetworkMode)
ネットワークソリューションと`l2bridge`して選ばれたネットワークモード`overlay` (フランネル host gw) または[](./network-topologies.md)(フランネル vxlan)。

> [!Important] 
> `overlay` ネットワークモード (フランネル vxlan) には、Kubernetes v 1.14 バイナリ (または上記) と[KB4489899](https://support.microsoft.com/help/4489899)が必要です。

|  |  | 
|---------|---------|
|パラメーター     | `-NetworkMode`        |
|既定値    | `l2bridge`        |


# [<a name="clustercidr"></a>ClusterCIDR](#tab/ClusterCIDR)
[クラスターサブネットの範囲](./getting-started-kubernetes-windows.md#cluster-subnet-def)。

|  |  | 
|---------|---------|
|パラメーター     | `-ClusterCIDR`        |
|既定値    | `10.244.0.0/16`        |


# [<a name="servicecidr"></a>このワトソン博士](#tab/ServiceCIDR)
[サービスサブネットの範囲](./getting-started-kubernetes-windows.md#service-subnet-def)。

|  |  | 
|---------|---------|
|パラメーター     | `-ServiceCIDR`        |
|既定値    | `10.96.0.0/12`        |


# [<a name="kubednsserviceip"></a>KubeDnsServiceIP](#tab/KubeDnsServiceIP)
[KUBERNETES DNS サービス IP](./getting-started-kubernetes-windows.md#plan-ip-addressing-for-your-cluster)。

|  |  | 
|---------|---------|
|パラメーター     | `-KubeDnsServiceIP`        |
|既定値    | `10.96.0.10`        |


# [<a name="interfacename"></a>InterfaceName](#tab/InterfaceName)
Windows ホストのネットワークインターフェイスの名前です。 これは、 `ipconfig`を使用して見つけることができます。

|  |  | 
|---------|---------|
|パラメーター     | `-InterfaceName`        |
|既定値    | `Ethernet`        |


# [<a name="logdir"></a>LogDir](#tab/LogDir)
Kubelet と kube のログがそれぞれの出力ファイルにリダイレクトされるディレクトリ。

|  |  | 
|---------|---------|
|パラメーター     | `-LogDir`        |
|既定値    | `C:\k`        |


---

> [!tip]
> すでに説明したように、Linux master のクラスターサブネット、サービスサブネット、および kube DNS IP は、[前](./creating-a-linux-master.md#collect-cluster-information)に説明した手順に従ってください。

この操作を実行すると、次のことが可能になります。
  * を使用して参加した Windows ノードを表示する `kubectl get nodes`
  * Powershell の3つのウィンドウが`kubelet` `flanneld`開いていることを確認してください。 `kube-proxy`
  * ノードでのホストエージェントの`flanneld`プロセス`kubelet`と実行`kube-proxy`の詳細については、「」を参照してください。

成功した場合は、[次の手順](#next-steps)に進みます。

## <a name="joining-a-tor-cluster"></a>ToR クラスターへの参加 ##
> [!NOTE]
> [以前](./network-topologies.md#flannel-in-host-gateway-mode)にネットワークソリューションとしてフランネルを選択した場合は、このセクションをスキップできます。

これを行うには、[上流の Kubernetes で Windows Server コンテナーを設定](https://kubernetes.io/docs/getting-started-guides/windows/#for-1-upstream-l3-routing-topology-and-2-host-gateway-topology)するための手順に従います。 これには、ノードに割り当てられている pod CIDR プレフィックスがそれぞれのノード IP にマップされるように、上流のルーターを構成していることを確認することが含まれます。

新しいノードが "Ready"、kubelet + kube の`kubectl get nodes`順に表示されていて、上流の ToR ルーターを構成している場合は、次の手順を実行することができます。

## <a name="next-steps"></a>次のステップ ##
このセクションでは、Windows ワーカーを Kubernetes クラスターに参加させる方法について説明します。 これで、手順5の準備ができました。

> [!div class="nextstepaction"]
> [Linux ワーカに参加する](./joining-linux-workers.md)

また、Linux ワーカをお持ちでない場合は、手順6に進んでください。

> [!div class="nextstepaction"]
> [Kubernetes リソースの展開](./deploying-resources.md)
