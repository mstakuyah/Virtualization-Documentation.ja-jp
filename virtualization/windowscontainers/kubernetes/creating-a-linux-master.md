---
title: "Kubernetes マスターの新規作成"
author: gkudra-msft
ms.author: gekudray
ms.date: 11/16/2017
ms.topic: get-started-article
ms.prod: containers
description: "Kubernetes クラスター マスターを 1 から作成します。"
keywords: "kubernetes, 1.9, マスター, linux"
ms.openlocfilehash: 3ea338f7af3dd921731fce0ec5a8b2cf8c4fef0c
ms.sourcegitcommit: f542e8c95b5bb31b05b7c88f598f00f76779b519
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/01/2018
---
# <a name="kubernetes-master--from-scratch"></a>Kubernetes マスターの新規作成 #
このページでは、Kubernetes マスターの手動展開手順を最初から最後まで説明します。

最近更新された Ubuntu のような Linux コンピューターについて理解する必要があります。 この場合、Windows はまったく関与しません。バイナリは、Linux でクロス コンパイルします。


> [!Warning]  
> Kubernetes のバージョン間の揮発性のため、このガイドの内容は将来的に正しくなくなる可能性があります。


## <a name="preparing-the-master"></a>マスターの準備 ##
まず、すべての前提条件をインストールします。

```bash
sudo apt-get install curl git build-essential docker.io conntrack python2.7
```

プロキシの内側にいる場合は、現在のセッションの環境変数を定義します。
```bash
HTTP_PROXY=http://proxy.example.com:80/
HTTPS_PROXY=http://proxy.example.com:443/
http_proxy=http://proxy.example.com:80/
https_proxy=http://proxy.example.com:443/
```
この設定を永続化する場合は、/etc/environment に変数を追加します (変更を適用するには、ログアウトして再びログインする必要があります)。

[このリポジトリ](https://github.com/Microsoft/SDN/tree/master/Kubernetes/linux)には、セットアップ プロセスを支援するスクリプトのコレクションがあります。 これらを `~/kube/` にチェックアウトします。このディレクトリ全体が、この後の手順で多くの Docker コンテナー用にマウントされるため、このガイドに記載されているものと同じ構造を維持する必要があります。

```bash
mkdir ~/kube
mkdir ~/kube/bin
git clone https://github.com/Microsoft/SDN /tmp/k8s 
cd /tmp/k8s/Kubernetes/linux
chmod -R +x *.sh
chmod +x manifest/generate.py
mv * ~/kube/
```


### <a name="installing-the-linux-binaries"></a>Linux バイナリのインストール ###

> [!Note]  
> 事前ビルドされたバイナリをダウンロードする代わりにパッチを含める場合や最新の Kubernetes コードを使用する場合は、[このページ](./compiling-kubernetes-binaries.md)をご覧ください。

正式な Linux バイナリを [Kubernetes メインライン](https://github.com/kubernetes/kubernetes/releases/tag/v1.9.1)からダウンロードして、次のようにインストールします。

```bash
wget -O kubernetes.tar.gz https://github.com/kubernetes/kubernetes/releases/download/v1.9.1/kubernetes.tar.gz
tar -vxzf kubernetes.tar.gz 
cd kubernetes/cluster 
# follow the prompts from this command, the defaults are generally fine:
./get-kube-binaries.sh
cd ../server
tar -vxzf kubernetes-server-linux-amd64.tar.gz 
cd kubernetes/server/bin
cp hyperkube kubectl ~/kube/bin/
```

どこからでも実行できるように、バイナリを `$PATH` に追加します。 この時点ではセッション用にパスが設定されただけです。永続的な設定を行うには、この行を `~/.profile` に追加します。

```bash
$ PATH="$HOME/kube/bin:$PATH"
```

### <a name="install-cni-plugins"></a>CNI プラグインのインストール ###
Kubernetes ネットワークには基本的な CNI プラグインが必要です。 これらのプラグインは[こちら](https://github.com/containernetworking/plugins/releases)からダウンロードできます。`/opt/cni/bin/` に展開してください。

```bash
DOWNLOAD_DIR="${HOME}/kube/cni-plugins"
CNI_BIN="/opt/cni/bin/"
mkdir ${DOWNLOAD_DIR}
cd $DOWNLOAD_DIR
curl -L $(curl -s https://api.github.com/repos/containernetworking/plugins/releases/latest | grep browser_download_url | grep 'amd64.*tgz' | head -n 1 | cut -d '"' -f 4) -o cni-plugins-amd64.tgz
tar -xvzf cni-plugins-amd64.tgz
sudo mkdir -p ${CNI_BIN}
sudo cp -r !(*.tgz) ${CNI_BIN}
ls ${CNI_BIN}
```


### <a name="certificates"></a>証明書 ###
まず、`ifconfig` または次のコマンドを使用して、ローカル IP アドレスを取得します。

```bash
$ ip addr show dev eth0
```

上のコマンドは、インターフェイス名がわかっている場合に使用します。 インターフェイス名は、このプロセス全体で何度も必要になります。環境変数に設定しておくと、作業が簡単になります。 次のスニペットでは、一時的な設定が行われます。セッションまたはシェルが終了した場合は、もう一度設定する必要があります。

```bash
$ MASTER_IP=10.123.45.67   # example! replace
```

クラスター内でノード間の通信に使用される証明書を準備します。

```bash
cd ~/kube/certs
chmod u+x generate-certs.sh
./generate-certs.sh $MASTER_IP
```

### <a name="prepare-manifests--addons"></a>マニフェストとアドオンの準備 ###
Kubernetes システム ポッドを指定した一連の YAML ファイルを生成します。これには、`manifest` フォルダー内の Python スクリプトにマスターの IP アドレスと*完全*クラスター CIDR を渡します。

```bash
cd ~/kube/manifest
./generate.py $MASTER_IP --cluster-cidr 192.168.0.0/16
```

マニフェストで Kubernetes による混乱が生じないように、Python スクリプトを削除または移動します。この操作を行わないと、後で問題につながります。

> [!Important]  
> Kubernetes バージョンがこのガイドの記述と異なる場合は、スクリプトでさまざまなバージョン管理フラグ (`--api-version` など) を使用して、ポッドによって展開される[イメージをカスタマイズ](https://console.cloud.google.com/gcr/images/google-containers/GLOBAL/hyperkube-amd64)します。 すべてのマニフェストで同じイメージが使用されるわけではなく、バージョン管理スキーマも異なる場合があります (特に `etcd` およびアドオン マネージャー)。


#### <a name="manifest-customization"></a>マニフェストのカスタマイズ ####
この時点では、セットアップ固有の変更が望ましいことがあります。 たとえば、Kubernetes による自動管理を許可するのではなく、手動でサブネットをノードに割り当てる必要性が生じることもあります。 この構成では、スクリプトにオプションを使用できます (`--im-sure` パラメーターについては、`--help` をご覧ください)。

```bash
./generate.py $MASTER_IP --im-sure
```

その他のカスタム構成オプションでは、生成されたマニフェストに対する手動の変更が必要です。


### <a name="configure--run-kubernetes"></a>Kubernetes の構成と実行 ###
生成された証明書を使用できるように Kubernetes を構成します。 これにより、`~/.kube/config` に構成が作成されます。

```bash
cd ~/kube
./configure-kubectl.sh $MASTER_IP
```

ここで、後でポッドが必要になる場所にファイルをコピーします。

```bash
mkdir ~/kube/kubelet
sudo cp ~/.kube/config ~/kube/kubelet/
```

Kubernetes の "クライアント" である `kubelet` を開始する準備ができました。 以下のスクリプトはどちらも無期限に実行されます。作業を続けるには、各スクリプトの後に別のターミナル セッションを開きます。

```bash
cd ~/kube
sudo ./start-kubelet.sh
```

Kubeproxy スクリプトを実行し、部分クラスター CIDR を渡します。

```bash
cd ~/kube
sudo ./start-kubeproxy.sh 192.168
```


> [!Important]  
> これは、予測された*完全な* /16 CIDR ですが、この場合は *CIDR にあるトラフィックが Kubernetes 以外であっても*ノードの実行が失敗となります。 Kubeproxy は、他のホストのトラフィックを妨害しないように、*サービス* サブネットへの Kubernetes トラフィックに*のみ*適用されます。

> [!Note]  
> これらのスクリプトはデーモン化することができます。 このガイドでは、セットアップ時にエラーを検出するために最も有効な、手動の実行についてのみ説明します。


## <a name="verifying-the-master"></a>マスターの検証 ##
数分後、システムは次の状態になります。

  - `docker ps` に、最大 23 のワーカー コンテナーおよびポッド コンテナーが作成されています。
  - `kubectl cluster-info` を呼び出すと、DNS および Heapster アドオンに加えて Kubernetes マスター API サーバーに関する情報が表示されます。
  - `ifconfig` により、選択したクラスター CIDR で新しいインターフェイス `cbr0` が表示されます。

