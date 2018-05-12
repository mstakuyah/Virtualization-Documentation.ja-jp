---
title: Windows で使用する Kubernetes
author: gkudra-msft
ms.author: gekudray
ms.date: 11/16/2017
ms.topic: get-started-article
ms.prod: containers
description: Windows ノードを v1.9 ベータ版の Kubernetes クラスターに参加させます。
keywords: kubernetes, 1.9, windows, 作業の開始
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: c6127fe8ab9de6a56816fb8187d4dec525425510
ms.sourcegitcommit: 7c3af076eb8bad98e1c3de0af63dacd842efcfa3
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/07/2018
---
# <a name="kubernetes-on-windows"></a>Windows で使用する Kubernetes #

Kubernetes 1.9 および Windows Server [Version 1709](https://docs.microsoft.com/en-us/windows-server/get-started/whats-new-in-windows-server-1709#networking) の最新リリースを使用すると、ユーザーは Windows ネットワークの最新機能を利用できます。

  - **共有ポッド コンパートメント**: インフラストラクチャ ポッドとワーカー ポッドで、ネットワーク コンパートメントを共有できるようになりました (Linux 名前空間と同様)。
  - **エンドポイントの最適化**: コンパートメントの共有により、コンテナー サービスは 以前の半数以上のエンドポイントを追跡する必要があります。
  - **データ パスの最適化**: 仮想フィルタリング プラットフォームおよびホスト ネットワーク サービスへの機能強化により、カーネルに基づく負荷分散が可能になります。


このページは、新しい Windows ノードを Linux ベースの既存クラスターに参加させる作業のガイドとしてお使いください。 1 から開始するには、[このページ](./creating-a-linux-master.md) &mdash; Kubernetes クラスターの展開に使用できる豊富なリソースの 1 つ &mdash; を参照して、同じようにマスターを 1 からセットアップします。

> [!TIP] 
> Azure でクラスターを展開する場合は、オープン ソースの ACS Engine ツールを使用すると簡単です。 手順を説明した[チュートリアル](https://github.com/Azure/acs-engine/blob/master/docs/kubernetes/windows.md)をご利用ください。

<a name="definitions"></a> このガイドで使用されている用語の定義を以下に示します。

  - **外部ネットワーク**は、ノード間で通信する際のネットワークです。
  - <a name="cluster-subnet-def"></a>**クラスター サブネット**は、ルーティング可能な仮想ネットワークです。ノードには、ポッドが使用できるように、これより小さなサブネットが割り当てられます。
  - **サービス サブネット**は、ルーティング不可能で純然たる仮想の 11.0/16 サブネットです。これは、ネットワーク トポロジに関係なく同じようにポッドがサービスにアクセスできるように使用されます。 ノードで実行されている `kube-proxy` によって、ルーティング可能なアドレス空間との間で変換が行われます。

## <a name="what-you-will-accomplish"></a>作業内容 ##

このガイドでは、次のことを行います。

> [!div class="checklist"]  
> * [Linux マスター](#preparing-the-linux-master) ノードを構成する。  
> * [Windows ワーカー ノード](#preparing-a-windows-node)を参加させる。  
> * [ネットワーク トポロジ](#network-topology)を準備する。  
> * [サンプル Windows サービス](#running-a-sample-service)を展開する。  
> * [一般的な問題とよくある誤り](./common-problems.md)を理解する。  

## <a name="preparing-the-linux-master"></a>Linux マスターの準備 ##

[手順](./creating-a-linux-master.md)に従った場合も既存クラスターが存在する場合も、Linux マスターから唯一必要なものは、Kubernetes の証明書の構成です。 この情報は、セットアップによって `/etc/kubernetes/admin.conf`、`~/.kube/config`、またはこれ以外の場所にあります。

## <a name="preparing-a-windows-node"></a>Windows のノードの準備 ##

> [!NOTE]  
> Windows セクションでのすべてのコード スニペットは、_管理者特権の_ PowerShell で実行します。

Kubernetes ではコンテナー オーケストレータとして [Docker](https://www.docker.com/) が使用されるため、これをインストールする必要があります。 [Docs の公式手順](../manage-docker/configure-docker-daemon.md#install-docker)または [Docker の手順](https://store.docker.com/editions/enterprise/docker-ee-server-windows)に従うことも、以下の手順を試すこともできます。

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

[この Microsoft リポジトリ](https://github.com/Microsoft/SDN)には、このノードをクラスターに参加させるために役立つスクリプトのコレクションがあります。 ZIP ファイルは、[こちら](https://github.com/Microsoft/SDN/archive/master.zip)から直接ダウンロードできます。 必要になるのは `Kubernetes/windows` フォルダーだけです。このフォルダーの内容を `C:\k\` に移動する必要があります。

```powershell
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
wget https://github.com/Microsoft/SDN/archive/master.zip -o master.zip
Expand-Archive master.zip -DestinationPath master
mkdir C:/k/
mv master/SDN-master/Kubernetes/windows/* C:/k/
rm -recurse -force master,master.zip
```

[前述](#preparing-the-linux-master)の証明書ファイルを、この新しい `C:\k` ディレクトリにコピーします。

## <a name="network-topology"></a>ネットワーク トポロジ ##

仮想[クラスター サブネット](#cluster-subnet-def)をルーティング可能にするには、いくつかの方法があります。 以下が可能です。

  - [ホスト ゲートウェイ モード](./configuring-host-gateway-mode.md)を構成し、ノード間の静的なネクストホップ ルートを設定して、ポッド間通信を可能にします。
  - サブネットでルーティングを行うためのスマート Top-of-Rack (ToR) スイッチを構成します。
  - [Flannel](https://coreos.com/flannel/docs/latest/kubernetes.html) など、サード パーティのオーバーレイ プラグインを使用します (Windows の Flannel サポートはベータ版です)。

### <a name="creating-the-pause-image"></a>"pause" イメージの作成 ###

`docker` のインストールが完了したため、Kubernetes インフラストラクチャ ポッドを準備するために Kubernetes で使用される "pause" イメージを準備する必要があります。

```powershell
docker pull microsoft/windowsservercore:1709
docker tag microsoft/windowsservercore:1709 microsoft/windowsservercore:latest
cd C:/k/
docker build -t kubeletwin/pause .
```

> [!NOTE]
> 後で展開するサンプル サービスは最新のイメージに依存するため、このイメージに `:latest` というタグを付けます。ただし、これが実際に利用可能な最新の Windows Server Core イメージ_である_とは限りません。 競合しているコンテナー イメージに注意する必要があります。必要なタグがないために互換性のないコンテナー イメージの `docker pull` が発生し、[展開に関する問題](./common-problems.md#when-deploying-docker-containers-keep-restarting)の原因となる可能性があります。 


### <a name="downloading-binaries"></a>バイナリのダウンロード ###
`pull` が発生するまでの間に、Kubernetes から次のクライアント側バイナリをダウンロードします。

  - `kubectl.exe`
  - `kubelet.exe`
  - `kube-proxy.exe`

これらは、最新の 1.9 リリースの `CHANGELOG.md` ファイル内のリンクからダウンロードできます。 このドキュメントの作成時点の最新リリースは [1.9.1](https://github.com/kubernetes/kubernetes/releases/tag/v1.9.1) であり、Windows バイナリは[こちら](https://storage.googleapis.com/kubernetes-release/release/v1.9.1/kubernetes-node-windows-amd64.tar.gz)です。 [7-Zip](http://www.7-zip.org/) などのツールを利用してアーカイブを展開し、バイナリを `C:\k\` に配置します。

`kubectl` コマンドを `C:\k\` ディレクトリの外部で使用できるようにするために、`PATH` 環境変数を変更します。

```powershell
$env:Path += ";C:\k"
```

この変更を永続化する場合は、コンピューター ターゲット内の変数を変更します。

```powershell
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\k", [EnvironmentVariableTarget]::Machine)
```

### <a name="joining-the-cluster"></a>クラスターへの参加 ###
以下を使用して、クラスター構成が有効であることを確認します。

```powershell
kubectl version
```

接続エラーが発生する場合は、

```
Unable to connect to the server: dial tcp [::1]:8080: connectex: No connection could be made because the target machine actively refused it.
```

構成が正しく検出されているかどうかを確認します。

```powershell
kubectl config view
```

`kubectl` が構成ファイルを検索する場所を変更するには、`--kubeconfig` パラメーターを渡すか、`KUBECONFIG` 環境変数を変更します。 たとえば、`C:\k\config` に構成がある場合は、次のように設定します。

```powershell
$env:KUBECONFIG="C:\k\config"
```

現在のユーザーのスコープでこの設定を永続化するには、次のように設定します。

```powershell
[Environment]::SetEnvironmentVariable("KUBECONFIG", "C:\k\config", [EnvironmentVariableTarget]::User)
```

ノードをクラスターに参加させる準備ができました。 *管理者特権*で開いた 2 つの別々の PowerShell ウィンドウで、以下のスクリプトを (この順に) 実行します。 最初のスクリプトの `-ClusterCidr` パラメーターは、構成された[クラスター サブネット](#cluster-subnet-def)で、ここでは `192.168.0.0/16` です。

```powershell
./start-kubelet.ps1 -ClusterCidr 192.168.0.0/16
./start-kubeproxy.ps1
```

1 分以内に、Linux マスターの `kubectl get nodes` に Windows ノードが表示されるようになります。


### <a name="validating-your-network-topology"></a>ネットワーク トポロジの検証 ###

適切なネットワーク構成の検証には、いくつかの基本的なテストを使用します。

  - **ノード間の接続**: マスター ノードと Windows ワーカー ノードの間で双方向の ping が成功する必要があります。

  - **ポッド サブネットからノードへの接続**: 仮想ポッド インターフェイスとノードの間で ping を実行します。 Linux と Windows で、それぞれ `route -n` と `ipconfig` を使用して `cbr0` インターフェイスを探し、ゲートウェイ アドレスを見つけます。

これらの基本テスト (一方または両方) に失敗した場合は、[トラブルシューティング ページ](./common-problems.md#common-networking-errors)を参照して一般的な問題を解決します。


## <a name="running-a-sample-service"></a>サンプル サービスの実行 ##

ここでは、確実にクラスターへの参加に成功し、ネットワークを正しく構成できるように、非常に単純な [PowerShell ベースの Web サービス](https://github.com/Microsoft/SDN/blob/master/Kubernetes/WebServer.yaml)を展開します。

Linux マスターで、サービスをダウンロードして実行します。

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/WebServer.yaml -O win-webserver.yaml
kubectl apply -f win-webserver.yaml
watch kubectl get pods -o wide
```

これにより、展開とサービスが作成されます。ポッドが無期限に監視され、状態が追跡されます。監視を終了するときは、`Ctrl+C` を押して `watch` コマンドを終了します。

すべての作業を正しく行った場合は、次のことを確認できます。

  - Windows ノードで `docker ps` コマンドにより 4 つのコンテナーが表示されること。
  - Linux マスターで `kubectl get pods` コマンドにより 2 つのポッドが表示されること。
  - `curl` : Linux マスターからポート 80 の*ポッド* IP で実行して Web サーバー応答を受信できること。これは、ネットワーク経由でのノードからポッドへの通信が適切であることを示します。
  - `docker exec` で*ポッド間* (複数の Windows ノードがある場合はホスト間も含めて) の ping を実行できること。これは、ポッド間の通信が適切であることを示します。
  - `curl` : Linux マスターおよび個々のポッドから仮想*サービス IP* (`kubectl get services` で確認) で実行できること。
  - `curl` : Kubernetes の[既定の DNS サフィックス](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#services)を指定した*サービス名*に対して実行できること。これにより、DNS が正しく機能していることを確認します。

> [!Warning]  
> Windows ノードでは、サービス IP にアクセスできません。 これは、Windows Server への次の更新プログラムで改善が予定されている[既知のプラットフォーム制限](./common-problems.md#my-windows-node-cannot-access-my-services-using-the-service-ip)です。


### <a name="port-mapping"></a>ポートのマッピング ### 
ノードのポートをマップすることによって、ポッドでホストされているサービスにそれぞれのノードからアクセスすることもできます。 この機能を示すための[サンプル YAML](https://github.com/Microsoft/SDN/blob/master/Kubernetes/PortMapping.yaml) では、ノードのポート 4444 をポッドのポート 80 にマップしています。 これを展開するには、さきほどと同じ手順に従います。

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/PortMapping.yaml -O win-webserver-port-mapped.yaml
kubectl apply -f win-webserver-port-mapped.yaml
watch kubectl get pods -o wide
```

これで、ポート 4444 の*ノード* IP で `curl` を実行し、Web サーバーの応答を受信することができます。 このとき 1 対 1 のマッピングを適用する必要があるため、スケーリングはノードあたり単一ポッドに制限される点に注意してください。
