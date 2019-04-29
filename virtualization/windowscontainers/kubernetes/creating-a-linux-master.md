---
title: Kubernetes マスターの新規作成
author: daschott
ms.author: daschott
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: Kubernetes クラスター マスターを作成します。
keywords: kubernetes、1.13、マスター、linux
ms.openlocfilehash: 8a3fb073616d115ab84e6cc36f0fb6cedbcf1f7d
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/26/2019
ms.locfileid: "9578253"
---
# <a name="creating-a-kubernetes-master"></a>Kubernetes マスターを作成します。 #
> [!NOTE]
> このガイドは、[Kubernetes v1.13 検証されました。 揮発性 Kubernetes のバージョンからのバージョン] の理由により、このセクションと、前提条件を満たす将来のすべてのバージョンの保持しません。 Kubeadm を使用して Kubernetes マスター シェイプを初期化するための公式のマニュアルを参照して[次のとおり](https://kubernetes.io/docs/setup/independent/install-kubeadm/)です。 単に[混在 OS のスケジュール] セクション](#enable-mixed-os-scheduling)を有効にします。

> [!NOTE]  
> 更新したばかりの Linux コンピューターが必要に追従します。Kubernetes はマスター [kube dns](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)、 [kube スケジューラ](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/)、および[kube apiserver](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/)移植されていないウィンドウをまだようなリソースです。 

> [!tip]
> Linux 手順は**Ubuntu 16.04**にカスタマイズされています。 Kubernetes を実行するための他の Linux 配布は、代わりに使用できると同じのコマンドを提供もする必要があります。 これらはもと相互運用正常に Windows。


## <a name="initialization-using-kubeadm"></a>初期化 kubeadm を使用します。 ##
明示的に指定がない限り、 **root**の下にあるコマンドを実行します。

最初に、管理者特権のルートをシェルに理解します。

```bash
sudo –s
```

コンピューターが最新の状態を確認します。

```bash
apt-get update -y && apt-get upgrade -y
```

### <a name="install-docker"></a>Docker のインストール ###
コンテナーを使用できるようにするには、Docker など、コンテナーのエンジンが必要です。 最新バージョンを移動するには、インストール済みの Docker の[次の手順](https://docs.docker.com/install/linux/docker-ce/ubuntu/)を使用できます。 実行して、その docker が正しくインストールされていることを確認することができますが、`hello-world`コンテナー。

```bash
docker run hello-world
```

### <a name="install-kubeadm"></a>Kubeadm をインストールします。 ###
ダウンロード`kubeadm`、Linux 分布バイナリし、自分のクラスターを初期化します。

> [!Important]  
> Linux 製品によってを置き換える必要があります`kubernetes-xenial`の下にある正しい[コードネーム](https://wiki.ubuntu.com/Releases)とします。

```bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update && apt-get install -y kubelet kubeadm kubectl 
```

### <a name="prepare-the-master-node"></a>マスター ノードを準備します。 ###
Kubernetes Linux では、共にスペースをオフにする必要があります。

```bash
nano /etc/fstab  # (remove a line referencing 'swap.img' , if it exists)
swapoff -a 
```

### <a name="initialize-master"></a>マスター シェイプを初期化します。 ###
サービス サブネット (10.96.0.0/12 など)、クラスター サブネット (10.244.0.0/16 など) を書き留め、kubeadm を使用して、マスター シェイプを初期化します。

```bash
kubeadm init --pod-network-cidr=10.244.0.0/16 --service-cidr=10.96.0.0/12
```

この手順が完了するまで数分かかる場合があります。 完了すると、初期化されて、マスター シェイプの確認はこのような画面が表示されます。

![テキスト](media/kubeadm-init.png)

> [!tip]
> この kubeadm への参加] コマンドのメモを行う必要があります。 使用する kubeadm トークンの有効期限が切れるとして、`kubeadm token create --print-join-command`新しいトークンを作成します。

> [!tip]
> 必要な Kubernetes バージョンがある場合を使用するには、渡すことができます、 `--kubernetes-version` kubeadm にフラグを設定します。

おがまだ終了しましたされません。 使用する`kubectl`正規ユーザーは、[次__**を unelevated、ルート以外のユーザー シェルで**__ 実行

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
今すぐ編集、または自分のクラスターに関する情報を表示する kubectl を使用することができます。

### <a name="enable-mixed-os-scheduling"></a>混合 OS のスケジュールを有効にします。 ###
既定では、特定の Kubernetes リソースはすべてのノードでスケジュールされているように書き込まれます。 ただし、環境で書かたくない Linux リソース妨害するし、Windows ノード、その逆に二重スケジュールされていることになります。 このため、 [NodeSelector](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#nodeselector)ラベルを適用する必要があります。 

ここでは、しようとして更新プログラムの linux kube プロキシ[DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)ターゲット Linux だけにします。

最初に、.yaml マニフェスト ファイルを保存するディレクトリを作成します。
```bash
mkdir -p kube/yaml && cd kube/yaml
```

確認の更新方法`kube-proxy`DaemonSet [RollingUpdate](https://kubernetes.io/docs/tasks/manage-daemon/update-daemon-set/)に設定します。

```bash
kubectl get ds/kube-proxy -o go-template='{{.spec.updateStrategy.type}}{{"\n"}}' --namespace=kube-system
```

次に、[この nodeSelector](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml)をダウンロードして、DaemonSet を修正し、Linux をターゲットのみに適用します。

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml
kubectl patch ds/kube-proxy --patch "$(cat node-selector-patch.yml)" -n=kube-system
```

成功すると後の「ノード セレクター」を表示する必要があります`kube-proxy`あり、その他の DaemonSets の設定 `beta.kubernetes.io/os=linux`

```bash
kubectl get ds -n kube-system
```

![テキスト](media/kube-proxy-ds.png)

### <a name="collect-cluster-information"></a>クラスター情報を収集します。 ###
正常に参加するには将来のノードをマスターする必要があるの管理、次の情報。
  1. `kubeadm join` 出力 ([次のとおり](#initialize-master)) から] コマンド
    * 例: `kubeadm join <Master_IP>:6443 --token <some_token> --discovery-token-ca-cert-hash <some_hash>`
  2. クラスターの中に定義されたサブネット`kubeadm init`([次のとおり](#initialize-master))
    * 例: `10.244.0.0/16`
  3. 中に定義されているサービス サブネット`kubeadm init`([次のとおり](#initialize-master))
    * 例: `10.96.0.0/12`
    * 使用してあることができます。 `kubectl cluster-info dump | grep -i service-cluster-ip-range`
  4. Kube dns サービス IP 
    * 例: `10.96.0.10`
    * "クラスター IP"フィールドを使用してで見つかる `kubectl get svc/kube-dns -n kube-system`
  5. Kubernetes`config`後に生成されたファイル`kubeadm init`([次のとおり](#initialize-master))。 手順を実行している場合、これは、次のパスに記載されています。
    * `/etc/kubernetes/admin.conf`
    * `$HOME/.kube/config`

## <a name="verifying-the-master"></a>マスター シェイプを確認します。 ##
数分後、システムは次の状態になります。

  - `kubectl get pods -n kube-system`が必要となるため、 [Kubernetes マスター コンポーネント](https://kubernetes.io/docs/concepts/overview/components/#master-components)のポッド`Running`状態です。
  - 呼び出し`kubectl cluster-info`DNS アドオンに加えて Kubernetes マスター API サーバーについての情報が表示されます。
  
> [!tip]
> DNS ポッド内に残って kubeadm がネットワークをセットアップしていないため`ContainerCreating`または`Pending`状態です。 移行されます`Running`[ネットワーク ソリューションを選択](./network-topologies.md)した後の状態。

## <a name="next-steps"></a>次のステップ ## 
ここでは、kubeadm を使用して Kubernetes マスター シェイプをセットアップする方法を説明します。 手順 3 の準備が整いました。

> [!div class="nextstepaction"]
> [ネットワーク ソリューションを選択します。](./network-topologies.md)