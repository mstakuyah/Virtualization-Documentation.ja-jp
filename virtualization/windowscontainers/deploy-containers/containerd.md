---
title: Windows コンテナプラットフォーム
description: 詳細については、「Windows で利用できる新しいコンテナー文書パーツ」を参照してください。
keywords: LCOW、linux コンテナー、docker、コンテナー、cri の入っている erd、、runhcs、runc
author: scooley
ms.date: 11/19/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: a0e62b32-0c4c-4dd4-9956-8056e9abd9e5
ms.openlocfilehash: 3107eb48dc9c75224b0c9dd9b436af6f0f451871
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/07/2019
ms.locfileid: "9998419"
---
# <a name="container-platform-tools-on-windows"></a>Windows のコンテナプラットフォームツール

Windows コンテナプラットフォームが拡張されています。 Docker は、現在のコンテナーの最初の部分であるため、他のコンテナープラットフォームツールを構築しています。

* 含まれる[erd/cri](https://github.com/containerd/cri) -windows Server 2019/windows 10 1809 に新しく追加されています。
* [runhcs](https://github.com/Microsoft/hcsshim/tree/master/cmd/runhcs) -runc に対応する Windows コンテナーホスト。
* [hcs](https://docs.microsoft.com/virtualization/api/) -ホストコンピューティングサービス + 便利な shim を使って、使いやすくすることができます。
  * [hcsshim](https://github.com/microsoft/hcsshim)
  * [.net-computevirtualization](https://github.com/microsoft/dotnet-computevirtualization)

この記事では、Windows と Linux のコンテナプラットフォーム、および各コンテナプラットフォームツールについて説明します。

## <a name="windows-and-linux-container-platform"></a>Windows と Linux のコンテナプラットフォーム

Linux 環境では、Docker などのコンテナー管理ツールは、 [runc](https://github.com/opencontainers/runc)と "いい[erd](https://containerd.io/)" のように、より細かいコンテナーツールのセットに組み込まれています。

![Linux の Docker アーキテクチャ](media/docker-on-linux.png)

`runc` は、 [OCI コンテナーランタイムの仕様](https://github.com/opencontainers/runtime-spec)に従ってコンテナーを作成して実行するための Linux コマンドラインツールです。

`containerd` コンテナのライフサイクルを管理するデーモンであり、コンテナの実行と監督へのコンテナーイメージのダウンロードとアンパックを行います。

Windows では、別のアプローチが採用されました。  Windows コンテナーをサポートするために、Docker を使って作業を開始した場合は、HCS (ホストコンピューティングサービス) に直接組み込まれています。  [このブログの投稿](https://techcommunity.microsoft.com/t5/Containers/Introducing-the-Host-Compute-Service-HCS/ba-p/382332)には、なぜ hcs を構築したのかに関する情報があります。この方法では、最初にこの方法でコンテナーを作成しました。

![Windows 上の最初の Docker エンジンアーキテクチャ](media/hcs.png)

この時点では、Docker はまだ HCS に直接通話を発信しています。 ただし、Windows コンテナーを含めるためにコンテナー管理ツールが拡張されています。 Windows コンテナーホストでは、windows のコンテナーが含まれています。この場合、Windows コンテナーホストは、Linux 上での erd と runc での呼び出し方法として、コンテナーと runhcs に呼び出します。

## <a name="runhcs"></a>runhcs

`runhcs` はの`runc`フォークです。  Like `runc`は`runhcs` 、open container イニシアチブ (OCI) 形式に従ってパッケージ化されたアプリケーションを実行するためのコマンドラインクライアントであり、open container イニシアチブ仕様の準拠実装です。

Runc と runhcs の機能の違いは次のとおりです。

* `runhcs` Windows で実行されます。  このアプリは、 [Hcs](containerd.md#hcs)と通信して、コンテナーの作成と管理を行います。
* `runhcs` さまざまな種類のコンテナーを実行できます。

  * Windows と Linux[の hyper-v 分離](../manage-containers/hyperv-container.md)
  * Windows プロセスコンテナー (コンテナーの画像はコンテナーのホストと一致する必要があります)

**使い方:**

``` cmd
runhcs run [ -b bundle ] <container-id>
```

`<container-id>` は、開始するコンテナーインスタンスの名前です。 この名前は、コンテナーホストで一意である必要があります。

バンドルディレクトリ (使用`-b bundle`) はオプションです。  
Runc の場合と同様に、コンテナーはバンドルを使って構成されます。 コンテナーのバンドルは、コンテナーの OCI 仕様ファイル "config. json" を含むディレクトリです。  "バンドル" の既定値は、現在のディレクトリです。

OCI spec ファイル "config.xml" は、正しく実行するために2つのフィールドを持つ必要があります。

* コンテナーのスクラッチ領域へのパス
* コンテナーのレイヤーディレクトリへのパス

Runhcs で使用できるコンテナーコマンドには次のものがあります。

* コンテナーを作成して実行するためのツール
  * **run**コンテナーを作成して実行します。
  * コンテナーの作成を**作成**する

* コンテナーで実行されているプロセスを管理するためのツール:
  * **start**作成したコンテナーでユーザー定義のプロセスを実行します。
  * **exec**は、コンテナー内で新しいプロセスを実行します。
  * **** 一時停止 pause は、コンテナー内のすべてのプロセスを停止します
  * **resume**は、以前に一時停止されているすべてのプロセスを再開します
  * **ps** ps は、コンテナー内で実行されているプロセスを表示する

* コンテナーの状態を管理するためのツール
  * **状態**はコンテナーの状態を出力します。
  * **kill**は、指定されたシグナル (既定: SIGTERM) をコンテナーの init プロセスに送信します。
  * **delete**は、デタッチされたコンテナーで頻繁に使われるコンテナーが保持しているリソースを削除します

複数のコンテナーと見なされる唯一のコマンドは**list**です。  これは、指定されたルートを使って、runhcs で開始される実行中または一時停止のコンテナーを一覧表示します。

### <a name="hcs"></a>HCS

HCS とのインターフェイスには、GitHub で利用可能な2つのラッパがあります。 HCS は C API であるため、ラッパーでは、より高いレベルの言語から HCS を簡単に呼び出すことができます。  

* [hcsshim](https://github.com/microsoft/hcsshim) -Hcsshim は Go で書かれており、runhcs の基礎となっています。
AppVeyor から最新情報を入手するか、自分で作成します。
* [.net-computevirtualization](https://github.com/microsoft/dotnet-computevirtualization) -.net-computevirtualization は、Hcs の C# ラッパーです。

HCS (直接またはラッパー経由) を使用する場合、または、Rust/Haskell/Inserta the the the the the the the the the the the the comment (英語)

HCS の詳細については、 [John はっきりの DockerCon プレゼンテーション](https://www.youtube.com/watch?v=85nCF5S8Qok)をご覧ください。

## <a name="containerdcri"></a>cri

> [!IMPORTANT]
> CRI のサポートは、サーバー 2019/Windows 10 1809 以降でのみ使用できます。  また、Windows 用のお客様のために、現在も活発な erd を開発しています。
> 開発/テストのみ。

OCI の仕様では1つのコンテナーが定義されていますが、 [CRI](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto) (container runtime interface) は pod と呼ばれる共有サンドボックス環境のワークロードとしてコンテナーを記述します。  ポッドには、1つ以上のコンテナーのワークロードを含めることができます。  Pod は、Kubernetes やサービスファブリックメッシュなどのグループ化されたワークロードを、メモリや vNETs などの共有リソースを使って、同じホスト上にある必要があります。

cri は、pod の次の互換性マトリックスを有効にします。

| ホスト OS | コンテナー OS | 隔離 | ポッドのサポート |
|:-------------------------------------------------------------------------|:-----------------------------------------------------------------------------|:---------------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------|
| <ul><li>Windows Server 2019/1809</ul></li><ul><li>Windows 10 1809</ul></li> | Linux | `hyperv` | Yes —真の複数コンテナーポッドをサポートします。 |
|  | Windows Server 2019/1809 | `process`* または `hyperv` | Yes —各ワークロードコンテナー OS がユーティリティ VM OS と一致する場合に、真の複数コンテナーポッドをサポートします。 |
|  | Windows Server 2016</br>Windows Server 1709</br>Windows Server 1803 | `hyperv` | Partial —コンテナー OS がユーティリティ VM OS と一致した場合に、1つのプロセスで分離されたコンテナーをユーティリティ VM ごとにサポートできる pod サンドボックスをサポートします。 |

\ * Windows 10 ホストは、Hyper-v 分離のみをサポートします

CRI spec へのリンク:

* [Runpodsandbox](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto#L24) -ポッドスペック
* [Createcontainer](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto#L47) -ワークロードの仕様

![格納用 erd ベースのコンテナー環境](media/containerd-platform.png)

RunHCS とインプレース erd はどちらも Windows システムサーバー2016以降で管理できますが、Pod (コンテナーのグループ) をサポートするには、Windows のコンテナーツールへの変更を壊す必要があります。  CRI のサポートは、Windows Server 2019/Windows 10 1809 以降で利用できます。
