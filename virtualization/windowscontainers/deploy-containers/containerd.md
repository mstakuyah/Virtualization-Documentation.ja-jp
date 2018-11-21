---
title: Windows コンテナー プラットフォーム
description: 新しいコンテナー内の文書パーツ利用可能な Windows の詳細を表示します。
keywords: LCOW、linux コンテナー、docker、コンテナー、containerd、cri、runhcs、runc
author: scooley
ms.date: 11/19/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: a0e62b32-0c4c-4dd4-9956-8056e9abd9e5
ms.openlocfilehash: 5811ea0761567c3a7db036358b24d1a3e7c51baf
ms.sourcegitcommit: fdaf666973fca37d8c428e0247454dd47c01f1c3
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/20/2018
ms.locfileid: "7460601"
---
# <a name="container-platform-tools-on-windows"></a>Windows にコンテナー プラットフォーム ツール

Windows コンテナー プラットフォームを展開します。  コンテナーの他のプラットフォーム ツールで作ったはこれで、docker はしますコンテナーの旅の最初の部分をでした。

1. [containerd/cri](https://github.com/containerd/cri) - 新しい Windows Server 2019/Windows 10 1809 します。
1. [runhcs](https://github.com/Microsoft/hcsshim/tree/master/cmd/runhcs) - runc に Windows のコンテナーのホストに対応します。
1. [hcs](https://docs.microsoft.com/virtualization/api/) - ホスト計算サービス + 便利な shim を使用するが簡単にします。

    * [hcsshim](https://github.com/microsoft/hcsshim)
    * [dotnet computevirtualization](https://github.com/microsoft/dotnet-computevirtualization)

この記事は Windows および Linux コンテナー プラットフォームと各コンテナー プラットフォーム ツールについて説明します。

## <a name="windows-and-linux-container-platform"></a>Windows と Linux コンテナー プラットフォーム

Linux 環境で Docker などのコンテナーの管理ツールの [コンテナー ツール - [runc](https://github.com/opencontainers/runc)と[containerd](https://containerd.io/)のより詳細な設定で構築します。

![Linux docker アーキテクチャ](media/docker-on-linux.png)

`runc` 作成して実行[OCI コンテナー ランタイムの仕様](https://github.com/opencontainers/runtime-spec)に従ってコンテナー Linux コマンド ライン ツールです。

`containerd` ダウンロードして、コンテナーの実行、監視して、コンテナーの画像を展開して、コンテナーのライフ サイクルを管理するデーモンです。

Windows では、別の方法がわかったします。  Windows のコンテナーをサポートする Docker で作業を開始したときは、HCS (計算サービスをホストする) に直接開発されています。  [ブログの投稿](https://blogs.technet.microsoft.com/virtualization/2017/01/27/introducing-the-host-compute-service-hcs/)は、HCS を作成した理由と理由はこのアプローチ コンテナーに最初にについての完全なです。

![Windows の初期 Docker エンジン アーキテクチャ](media/hcs.png)

この時点では、まだ Docker が、HCS に直接呼び出します。 今後、ただし、コンテナーの管理ツールが Windows コンテナー コンテナー ホスト containerd と runhcs に containerd と linux runc に電話をかける方法との通話の可能性のある Windows など、展開します。

## <a name="runhcs"></a>runhcs

`runhcs` 分岐が`runc`します。  ような`runc`、`runhcs`コマンド ライン クライアント アプリケーションを開いているコンテナー initiative (英語) (OCI) 書式に従ってパッケージを実行するのには、準拠実装で開いているコンテナー initiative (英語) の仕様のします。

Runc と runhcs の機能の違いは次のとおりです。

* `runhcs` windows を実行します。  作成し、管理コンテナーに[HCS](containerd.md#hcs)と通信します。
* `runhcs` さまざまな種類の別のコンテナーを実行できます。

  * Windows と Linux [HYPER-V コンテナー](../manage-containers/hyperv-container.md)
  * Windows コンテナー (コンテナーの画像は、コンテナーのホストを一致する必要があります) を処理します。

**使い方:**

``` cmd
runhcs run [ -b bundle ] <container-id>
```

`<container-id>` 開始しているコンテナーのインスタンスの名前です。 名前は、コンテナーのホスト上で一意である必要があります。

バンドル ディレクトリ (を使用して`-b bundle`) はオプションです。  
Runc と同様、バンドルを使用するコンテナーが構成されます。 コンテナーのバンドルが、コンテナーの OCI 仕様ファイル ディレクトリ"config.json"します。  「バンドル」の既定値は、現在のディレクトリです。

OCI 仕様ファイル、"config.json"、2 つのフィールドが正常に実行するには。

1. コンテナーのスクラッチ領域へのパス
1. コンテナーのレイヤー ディレクトリへのパス

Runhcs で使用できるコンテナー コマンドは、次のとおりです。

* 作成して、コンテナーを実行するためのツール
  * **実行**作成し、コンテナーの実行
  * コンテナーを**作成する**作成します。

* コンテナーの実行中のプロセスを管理するためのツール。
  * 作成したコンテナー内のユーザー定義処理を実行**開始**
  * **実行**コンテナー内の新しいプロセスを実行します。
  * 一時停止] を**ポイント**しますコンテナー内のすべてのプロセスを停止します。
  * 以前に一時停止されているすべてのプロセスを再開する**再開します。**
  * **ps** ps は、コンテナーの内部を実行しているプロセスを表示します。

* コンテナーの状態を管理するためのツール
  * **状態**をコンテナーの状態を出力します。
  * 指定されたシグナルを送信**を強制終了**(既定: SIGTERM) コンテナーの初期プロセス
  * **コンテナーのデタッチ コンテナーと使用頻度が保持しているすべてのリソースを削除します。**

複数のコンテナーを考慮する可能性のあるだけで、コマンドは、**リスト**です。  指定されたルートと runhcs が開始したコンテナーを実行している (または一時停止している) が表示されます。

### <a name="hcs"></a>HCS

GitHub に、HCS で利用可能な 2 つのラッパーがあります。 C API、HCS なので、ラッパーしやすいようにより高いレベルの言語から、HCS を発信します。  

* [hcsshim](https://github.com/microsoft/hcsshim) - HCSShim は、外出先で記述されてれ、runhcs の基本です。
AppVeyor 最新情報を取得するか、自分で作成します。
* [dotnet computevirtualization](https://github.com/microsoft/dotnet-computevirtualization) -dotnet computevirtualization、HCS の c# 包み紙には

(直接または経由の包み紙に)、HCS を使用する場合は、HCS 周囲錆び/Haskell/InsertYourLanguage の包み紙にする、コメントのままにしてください。

HCS の詳細については、 [John Stark の DockerCon プレゼンテーション](https://www.youtube.com/watch?v=85nCF5S8Qok)をご覧ください。

## <a name="containerdcri"></a>containerd/cri

> [!IMPORTANT]
> CRI サポートは、サーバー 2019年/Windows 10 1809 で使用できると、後でのみです。  Windows 版の containerd を開発している場合も引き続き積極的にします。
> 開発/テストのみです。

[CRI](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto) (コンテナー ランタイム インターフェイス) が共有サンド ボックスで workload(s) としてコンテナーを説明する OCI 仕様には、1 つのコンテナーが定義されているときに環境にポッドが呼び出されます。  ポッドには、1 つ以上のコンテナー ワークロードを含めることができます。  ポッドは Kubernetes とサービス布地へメッシュ処理のメモリ、vNETs などのいくつかの共有リソースと同じホストに置かれるべき作業負荷をグループ化されたようにコンテナー orchestrators ことができます。

CRI 仕様へのリンク。

* [RunPodSandbox](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto#L24) - ポッドの仕様
* [CreateContainer](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto#L47) - ワークロードの仕様

![ベース Containerd コンテナーが混在する環境](media/containerd-platform.png)

RunHCS と containerd は、任意の Windows システム Server 2016 以降で管理できる、ポッド (コンテナーのグループ) をサポートするために必要なコンテナー ツールに Windows で変更を解除します。  CRI サポート、Windows Server 2019/Windows 10 1809 以降使用します。