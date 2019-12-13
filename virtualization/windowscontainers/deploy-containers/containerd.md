---
title: Windows コンテナープラットフォーム
description: Windows で使用可能な新しいコンテナーの構成要素の詳細については、こちらを参照してください。
keywords: LCOW、linux コンテナー、docker、コンテナー、cri、runhcs、runc
author: scooley
ms.date: 11/19/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: a0e62b32-0c4c-4dd4-9956-8056e9abd9e5
ms.openlocfilehash: 74e22702aa4be30055b3f4f48c7fac926d793095
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909922"
---
# <a name="container-platform-tools-on-windows"></a>Windows 上のコンテナープラットフォームツール

Windows コンテナープラットフォームが展開されています。 Docker はコンテナーの最初の段階であり、他のコンテナープラットフォームツールを構築しています。

* [cri](https://github.com/containerd/cri) -windows Server 2019/windows 10 1809 の新。
* [runhcs](https://github.com/Microsoft/hcsshim/tree/master/cmd/runhcs) -runc に対応する Windows コンテナーホスト。
* [hcs](https://docs.microsoft.com/virtualization/api/) -ホストコンピューティングサービスと便利な shim を使用して使いやすくします。
  * [hcsshim](https://github.com/microsoft/hcsshim)
  * [dotnet-computevirtualization](https://github.com/microsoft/dotnet-computevirtualization)

この記事では、Windows と Linux のコンテナープラットフォーム、および各コンテナープラットフォームツールについて説明します。

## <a name="windows-and-linux-container-platform"></a>Windows と Linux のコンテナープラットフォーム

Linux 環境では、Docker のようなコンテナー管理ツールは、 [runc](https://github.com/opencontainers/runc) [と格納されて](https://containerd.io/)いるコンテナーツールのより詳細なセットに基づいて構築されています。

![Linux 上の Docker アーキテクチャ](media/docker-on-linux.png)

`runc` は、 [OCI コンテナーのランタイム仕様](https://github.com/opencontainers/runtime-spec)に従ってコンテナーを作成および実行するための Linux コマンドラインツールです。

`containerd` は、コンテナーのライフサイクルを管理するデーモンで、コンテナーの実行と監視にコンテナーイメージをダウンロードして展開します。

Windows では、別のアプローチを採用していました。  Docker を使用して Windows コンテナーをサポートするために、HCS (Host Compute Service) に直接ビルドしました。  [このブログ投稿](https://techcommunity.microsoft.com/t5/Containers/Introducing-the-Host-Compute-Service-HCS/ba-p/382332)では、hcs を構築した理由と、このアプローチを最初にコンテナーに対して行った理由に関する情報を十分に取り上げています。

![Windows 上の Docker エンジンの初期アーキテクチャ](media/hcs.png)

この時点で、Docker は引き続き HCS に直接を呼び出します。 ただし、将来的には、コンテナー管理ツールは Windows コンテナーを含むように拡張されており、Windows コンテナーホストは、Linux 上での組み込みのために呼び出しを行う方法として、コンテナーと runhcs を呼び出すことができました。

## <a name="runhcs"></a>runhcs

`runhcs` は `runc`のフォークです。  `runc`と同様に、`runhcs` は Open Container イニシアチブ (OCI) 形式に従ってパッケージ化されたアプリケーションを実行するためのコマンドラインクライアントであり、オープンコンテナーイニシアチブ仕様の準拠した実装です。

Runc と runhcs の機能上の違いは次のとおりです。

* `runhcs` は Windows で実行されます。  [Hcs](containerd.md#hcs)と通信して、コンテナーを作成および管理します。
* `runhcs` は、さまざまな種類のコンテナーを実行できます。

  * Windows と Linux[の hyper-v の分離](../manage-containers/hyperv-container.md)
  * Windows プロセスコンテナー (コンテナーイメージはコンテナーホストと一致している必要があります)

**使用法:**

``` cmd
runhcs run [ -b bundle ] <container-id>
```

`<container-id>` には、開始するコンテナーインスタンスの名前を指定します。 名前は、コンテナーホスト上で一意である必要があります。

バンドルディレクトリ (`-b bundle`を使用) は省略可能です。  
Runc と同様に、コンテナーはバンドルを使用して構成されます。 コンテナーのバンドルとは、コンテナーの OCI 仕様ファイル "config. json" が格納されているディレクトリです。  "バンドル" の既定値は、現在のディレクトリです。

OCI spec ファイル "config. json" には、正常に実行するために2つのフィールドが必要です。

* コンテナーのスクラッチ領域へのパス
* コンテナーのレイヤーディレクトリへのパス

Runhcs で使用できるコンテナーコマンドは次のとおりです。

* コンテナーを作成して実行するためのツール
  * **実行**すると、コンテナーが作成され、実行されます。
  * コンテナー**を作成する**

* コンテナーで実行されているプロセスを管理するためのツール:
  * 作成されたコンテナー内のユーザー定義プロセスを**開始**します
  * **exec**は、コンテナー内の新しいプロセスを実行します。
  * 一時**停止**pause は、コンテナー内のすべてのプロセスを中断します
  * 前に一時停止されたすべてのプロセスを**再開**します
  * **ps** ps は、コンテナー内で実行されているプロセスを表示します。

* コンテナーの状態を管理するためのツール
  * **状態**はコンテナーの状態を出力します
  * **kill**は、指定されたシグナル (既定: SIGTERM) をコンテナーの init プロセスに送信します。
  * **削除**すると、コンテナーに保持されているすべてのリソースがデタッチされたコンテナーで使用されます。

複数コンテナーと見なされる唯一のコマンドは**list**です。  Runhcs によって開始された、実行中または一時停止中のコンテナーを、指定されたルートで一覧表示します。

### <a name="hcs"></a>HCS

GitHub では、HCS とのインターフェイスとして使用できるラッパーが2つあります。 HCS は C API であるため、ラッパーを使用すると、上位レベルの言語から HCS を簡単に呼び出すことができます。  

* hcsshim は[hcsshim](https://github.com/microsoft/hcsshim)に記述されており、runhcs の基礎となります。
AppVeyor から最新のファイルを取得するか、自分でビルドします。
* [dotnet-computevirtualization](https://github.com/microsoft/dotnet-computevirtualization) -dotnet-computevirtualization は hcs のC#ラッパーです。

HCS (直接またはラッパーを介して) を使用する場合、または HCS の Rust/Haskell/Insertlanguage ラッパーを作成する場合は、コメントを残してください。

HCS の詳細については、 [John Stark の DockerCon プレゼンテーション](https://www.youtube.com/watch?v=85nCF5S8Qok)をご覧ください。

## <a name="containerdcri"></a>cri

> [!IMPORTANT]
> CRI のサポートは、サーバー 2019/Windows 10 1809 以降でのみ使用できます。  また、Windows 用の "存在する" ためにも積極的に開発しています。
> 開発/テストのみ。

OCI specs は1つのコンテナーを定義しますが、 [CRI](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto) (container runtime インターフェイス) は、ポッドと呼ばれる共有サンドボックス環境のワークロードとしてコンテナーを記述します。  ポッドには、1つまたは複数のコンテナーワークロードを含めることができます。  ポッドを使用すると、Kubernetes と Service Fabric メッシュのようなグループ化されたワークロードを、メモリや Vnet などの共有リソースと同じホスト上に配置することができます。

cri は、ポッドに対して次の互換性マトリックスを有効にします。

| ホスト OS | Container OS | 分離性 | ポッドのサポート |
|:-------------------------------------------------------------------------|:-----------------------------------------------------------------------------|:---------------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------|
| <ul><li>Windows Server 2019/1809</ul></li><ul><li>Windows 10 1809</ul></li> | Linux | `hyperv` | はい—は、真の複数コンテナーポッドをサポートします。 |
|  | Windows Server 2019/1809 | `process`* または `hyperv` | はい—各ワークロードコンテナー OS がユーティリティ VM OS と一致する場合は、真の複数コンテナーポッドをサポートします。 |
|  | Windows Server 2016、</br>Windows Server 1709、</br>Windows Server 1803 | `hyperv` | Partial-コンテナー OS がユーティリティ VM OS と一致する場合に、ユーティリティ VM ごとに1つのプロセス分離コンテナーをサポートできる pod サンドボックスをサポートします。 |

\*Windows 10 ホストは、Hyper-v の分離のみをサポートします。

CRI 仕様へのリンク:

* [Runpodsandbox](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto#L24) -ポッド仕様
* [Createcontainer](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto#L47) -ワークロードの仕様

![コンテナー環境の格納](media/containerd-platform.png)

RunHCS と入力 erd はどちらも Windows システムサーバー2016以降で管理できますが、ポッド (コンテナーのグループ) をサポートするには、Windows のコンテナーツールに重大な変更を加える必要がありました。  CRI サポートは、Windows Server 2019/Windows 10 1809 以降で使用できます。
