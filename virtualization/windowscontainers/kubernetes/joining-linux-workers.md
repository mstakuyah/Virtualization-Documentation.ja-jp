---
title: Linux ノードへの参加
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: Kubernetes v1.12 を使用するには、Linux ノードを結合できます。
keywords: kubernetes、1.12、windows、作業の開始
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 97c12d70db9679dbb85877f0985c6053f95fa500
ms.sourcegitcommit: 4412583b77f3bb4b2ff834c7d3f1bdabac7aafee
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/15/2018
ms.locfileid: "6947981"
---
# <a name="joining-linux-nodes-to-a-cluster"></a>Linux ノード クラスターへの参加

[Kubernetes マスター ノードをセットアップ](creating-a-linux-master.md)して[、目的のネットワーク ソリューションを選択](network-topologies.md)したらは、自分のクラスターにノードを結合する準備が整います。 参加する前に、いくつかの[Linux ノードの準備](joining-linux-workers.md#preparing-a-linux-node)が必要です。
> [!tip]
> Linux 手順は**Ubuntu 16.04**にカスタマイズされています。 Kubernetes を実行するための他の Linux 配布は、代わりに使用できると同じのコマンドを提供もする必要があります。 これらはもと相互運用正常に Windows。

## <a name="preparing-a-linux-node"></a>Linux ノードを準備します。

> [!NOTE]
> 明示的に指定しない限り、しない限り、**管理者特権、ルート ユーザー シェル**でコマンドを実行します。

最初に、ルート シェルに理解します。

```bash
sudo –s
```

コンピューターが最新の状態を確認します。

```bash
apt-get update && apt-get upgrade
```

## <a name="install-docker"></a>Docker のインストール

コンテナーを使用できるようにするには、Docker など、コンテナーのエンジンが必要です。 最新バージョンを移動するには、インストール済みの Docker の[次の手順](https://docs.docker.com/install/linux/docker-ce/ubuntu/)を使用できます。 実行して、その docker が正しくインストールされていることを確認できる`hello-world`の画像。

```bash
docker run hello-world
```

## <a name="install-kubeadm"></a>Kubeadm をインストールします。

ダウンロード`kubeadm`、Linux 分布バイナリし、自分のクラスターを初期化します。

> [!Important]  
> Linux 製品によってを置き換える必要があります`kubernetes-xenial`の下にある正しい[コードネーム](https://wiki.ubuntu.com/Releases)とします。

``` bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update && apt-get install -y kubelet kubeadm kubectl 
```

## <a name="disable-swap"></a>共にを無効にします。

Kubernetes Linux では、共にスペースをオフにする必要があります。

``` bash
nano /etc/fstab  # (remove a line referencing 'swap.img' , if it exists)
swapoff -a
```

## <a name="flannel-only-enable-bridged-ipv4-traffic-to-iptables"></a>(Flannel のみ)Iptables IPv4 トラフィック ブリッジを有効にします。

ネットワーク ソリューションを有効にすることをお勧めに Flannel を選択した場合は、iptables チェーンに IPv4 トラフィックをブリッジします。 必要があります[マスターの場合は、この処理を実行](network-topologies.md#flannel-in-host-gateway-mode)してへの参加 Linux ノードの繰り返す必要があります。 それを実行できる、次のコマンドを使用します。

``` bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```

## <a name="copy-kubernetes-certificate"></a>Kubernetes 証明書をコピーします。

**通常、(ルートではない) ユーザーとして**、次の 3 つの手順を実行します。

1. Linux ディレクトリの Kubernetes を作成します。

```bash
mkdir -p $HOME/.kube
```

1. Kubernetes 証明書ファイルをコピー (`$HOME/.kube/config`)[マスターの](./creating-a-linux-master.md#collect-cluster-information)名前を付けて保存`$HOME/.kube/config`作業者にします。

1. コピーした構成ファイルのファイルの所有権を次のように設定します。

``` bash
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## <a name="joining-node"></a>ノードへの参加

最後に、クラスターへの参加] を実行、 `kubeadm join` **root**[述べたダウン](./creating-a-linux-master.md#initialize-master)コマンドします。

```bash
kubeadm join <Master_IP>:6443 --token <some_token> --discovery-token-ca-cert-hash <some_hash>
```

成功した場合、次のような出力が表示されます。

![テキスト](./media/node-join.png)

## <a name="next-steps"></a>次のステップ

このセクションは、Linux 作業者を Kubernetes クラスターへの参加方法を説明します。 手順 6 の準備が整いました。
> [!div class="nextstepaction"]
> [Kubernetes リソースを展開します。](./deploying-resources.md)