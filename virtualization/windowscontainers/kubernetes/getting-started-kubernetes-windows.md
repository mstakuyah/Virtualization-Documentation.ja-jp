---
title: "Windows で使用する Kubernetes"
author: gkudra-msft
ms.author: gekudray
ms.date: 11/16/2017
ms.topic: get-started-article
ms.prod: containers
description: "Windows ノードを v1.9 ベータ版の Kubernetes クラスターに参加させます。"
keywords: "kubernetes, 1.9, windows, 作業の開始"
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: d88ab46dc0046256ebed9c6696a99104a7197fad
ms.sourcegitcommit: ad5f6344230c7c4977adf3769fb7b01a5eca7bb9
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/05/2017
---
# <a name="kubernetes-on-windows"></a>Windows で使用する Kubernetes #
Kubernetes 1.9 および Windows Server [Version 1709](https://docs.microsoft.com/en-us/windows-server/get-started/whats-new-in-windows-server-1709#networking) の最新リリースを使用すると、ユーザーは Windows ネットワークの最新機能を利用できます。

  - **共有ポッド コンパートメント**: インフラストラクチャ ポッドとワーカー ポッドで、ネットワーク コンパートメントを共有できるようになりました (Linux 名前空間と同様)。
  - **エンドポイントの最適化**: コンパートメントの共有により、コンテナー サービスは 以前の半数以上のエンドポイントを追跡する必要があります。
  - **データ パスの最適化**: 仮想フィルタリング プラットフォームおよびホスト ネットワーク サービスへの機能強化により、カーネルに基づく負荷分散が可能になります。


このページは、新しい Windows ノードを Linux ベースの既存クラスターに参加させる作業のガイドとしてお使いください。 1 から開始するには、[このページ](./creating-a-linux-master.md) (Kubernetes クラスターの展開に使用できる豊富なリソースの 1 つ) を参照して、同じようにマスターを 1 からセットアップします。


<a name="definitions"></a> このガイドで使用されている用語の定義を以下に示します。

  - **外部ネットワーク**は、ノード間で通信する際のネットワークです。
  - <a name="cluster-subnet-def"></a>**クラスター サブネット**は、ルーティング可能な仮想ネットワークです。ノードには、ポッドが使用できるように、これより小さなサブネットが割り当てられます。
  - **サービス サブネット**は、ルーティング不可能で純然たる仮想の 11.0/16 サブネットです。これは、ネットワーク トポロジに関係なく同じようにポッドがサービスにアクセスできるように使用されます。 ノードで実行されている `kube-proxy` によって、ルーティング可能なアドレス空間との間で変換が行われます。


## <a name="what-we-will-accomplish"></a>作業内容 ##
このガイドでは、次のことを行います。

> [!div class="checklist"]  
> * [ネットワーク トポロジ](#network-topology)を準備する。  
> * [Linux マスター](#preparing-the-linux-master) ノードを構成する。  
> * [Windows ワーカー ノード](#preparing-a-windows-node)を参加させる。  
> * [サンプル Windows サービス](#running-a-sample-service)を展開する。  
> * [一般的な問題とよくある誤り](./common-problems.md)を理解する。  


## <a name="network-topology"></a>ネットワーク トポロジ ##
仮想[クラスター サブネット](#cluster-subnet-def)をルーティング可能にするには、いくつかの方法があります。 以下が可能です。

  - [ホスト ゲートウェイ モード](./configuring-host-gateway-mode.md)を構成し、ノード間の静的なネクストホップ ルートを設定して、ポッド間通信を可能にします。
  - サブネットでルーティングを行うためのスマート Top-of-Rack (ToR) スイッチを構成します。
  - [Flannel](https://coreos.com/flannel/docs/latest/kubernetes.html) など、サード パーティーのオーバーレイ プラグインを使用します (Windows の Flannel サポートはベータ版です)。


## <a name="preparing-the-linux-master"></a>Linux マスターの準備 ##
[手順](./creating-a-linux-master.md)に従った場合も既存クラスターが存在する場合も、Linux マスターから唯一必要なものは、Kubernetes の証明書の構成です。 この情報は、セットアップによって `/etc/kubernetes/admin.conf`、`~/.kube/config`、またはこれ以外の場所にあります。


## <a name="preparing-a-windows-node"></a>Windows のノードの準備 ##
> [!Note]  
> Windows セクションでのすべてのコード スニペットは、管理者特権の PowerShell で実行します。

Kubernetes ではコンテナー オーケストレータとして [Docker](https://www.docker.com/) が使用されるため、これをインストールする必要があります。 [MSDN の公式手順](virtualization/windowscontainers/manage-docker/configure-docker-daemon.md#install-docker)または [Docker の手順](https://store.docker.com/editions/enterprise/docker-ee-server-windows)に従うことも、以下の手順を試すこともできます。

```powershell
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
Install-Package -Name Docker -ProviderName DockerMsftProvider
Restart-Computer -Force
```

[この Microsoft リポジトリ](https://github.com/Microsoft/SDN)には、このノードをクラスターに参加させるために役立つスクリプトのコレクションがあります。 ZIP ファイルは、[こちら](https://github.com/Microsoft/SDN/archive/master.zip)から直接ダウンロードできます。 必要になるのは `Kubernetes/windows` フォルダーだけです。このフォルダーの内容を `C:\k\` に移動する必要があります。

```powershell
wget https://github.com/Microsoft/SDN/archive/master.zip -o master.zip
Expand-Archive master.zip -DestinationPath master
mkdir C:/k/
mv master/SDN-master/Kubernetes/windows/* C:/k/
rm -recurse -force master,master.zip
```

[前述](#preparing-the-linux-master)の証明書ファイルを、この新しい `C:\k` ディレクトリにコピーします。


### <a name="creating-the-pause-image"></a>"Pause" イメージの作成 ###
`docker` のインストールが完了したため、Kubernetes インフラストラクチャ ポッドを準備するために Kubernetes で使用される "pause" イメージを準備する必要があります。

```powershell
docker pull microsoft/windowsservercore:1709
docker tag $(docker images -q) microsoft/windowsservercore:latest
cd C:/k/
docker build -t kubeletwin/pause .
```

> [!Note]  
> 後で展開するサンプル サービスで必要になるため、`:latest` というタグを使用しています。


### <a name="downloading-binaries"></a>バイナリのダウンロード ###
`pull` が発生するまでの間に、Kubernetes から次のクライアント側バイナリをダウンロードします。

  - `kubectl.exe`
  - `kubelet.exe`
  - `kube-proxy.exe`

これらは、最新の 1.9 リリースの `CHANGELOG.md` ファイル内のリンクからダウンロードできます。 このドキュメントの作成時点の最新リリースは [1.9.0-beta.1](https://github.com/kubernetes/kubernetes/releases/tag/v1.9.0-beta.1) であり、Windows バイナリは[こちら](https://dl.k8s.io/v1.9.0-beta.1/kubernetes-node-windows-amd64.tar.gz)です。 [7-Zip](http://www.7-zip.org/) などのツールを利用してアーカイブを展開し、バイナリを `C:\k\` に配置します。

> [!Warning]  
> このドキュメントの作成時点では、`kube-proxy.exe` を正しく使用するには保留中の Kubernetes の[プル要求](https://github.com/kubernetes/kubernetes/pull/56529)が必要になります。 問題を回避するには、[バイナリの手動作成](./compiling-kubernetes-binaries.md)が必要になる可能性があります。


### <a name="joining-the-cluster"></a>クラスターへの参加 ###
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

これらの基本テスト (一方または両方) に失敗した場合は、[トラブルシューティング ページ](./common-problems.md#network-connectivity)を参照して一般的な問題を解決します。


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

  - Windows 側の `docker ps` コマンドで 4 つのコンテナーが表示されること。
  - `curl` : Linux マスターからポート 80 の*ポッド* IP で実行して Web サーバー応答を受信できること。これは、ネットワーク経由でのノードからポッドへの通信が適切であることを示します。
  - `curl` : ポート 4444 の*ノード* IP で実行して Web サーバー応答を受信できること。これは、ホストからコンテナーへのポート マッピングが適切であることを示します。
  - `docker exec` で*ポッド間* (複数の Windows ノードがある場合はホスト間も含めて) の ping を実行できること。これは、ポッド間の通信が適切であることを示します。
  - `curl` : Linux マスターおよび個々のポッドから仮想*サービス IP* (`kubectl get services` で確認) で実行できること。
  - `curl` : Kubernetes の[既定の DNS サフィックス](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#services)を指定した*サービス名*に対して実行できること。これにより、DNS が正しく機能していることを確認します。

> [!Warning]  
> Windows ノードでは、サービス IP にアクセスできません。 これは[既知の制限](./common-problems.md#common-windows-errors)です。
