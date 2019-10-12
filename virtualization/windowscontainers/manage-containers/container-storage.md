---
title: コンテナーストレージの概要
description: Windows Server コンテナーがホストとその他の種類の記憶域を使用する方法
keywords: コンテナー, ボリューム, 記憶域, マウント, bindmount
author: cwilhit
ms.openlocfilehash: fba08de884d59cc1b656895ec2b7078ba3975269
ms.sourcegitcommit: 22dcc1400dff44fb85591adf0fc443360ea92856
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/12/2019
ms.locfileid: "10209752"
---
# <a name="container-storage-overview"></a>コンテナーストレージの概要

<!-- Great diagram would be great! -->

このトピックでは、コンテナーが Windows でストレージを使用する方法の概要について説明します。 コンテナーは、記憶域に含まれている仮想マシンとは動作が異なります。 本質的に、コンテナーは、その内部で実行されているアプリが、ホストのファイルシステム全体で状態を書き込むことを防ぐように設計されています。 コンテナーは既定で "スクラッチ" スペースを使用していますが、Windows にもストレージを保持する手段が用意されています。

## <a name="scratch-space"></a>スクラッチ領域

Windows コンテナーには、既定では短期ストレージが使用されます。 すべてのコンテナー i/o は "スクラッチスペース" で発生し、各コンテナーはそれぞれ独自のスクラッチを取得します。 ファイルの作成とファイルの書き込みはスクラッチ領域でキャプチャされ、ホストにはエスケープされません。 コンテナーインスタンスが停止すると、スクラッチ領域で発生したすべての変更が破棄されます。 新しいコンテナーインスタンスが開始されると、インスタンスに対して新しいスクラッチ領域が提供されます。

## <a name="layer-storage"></a>レイヤー記憶域

コンテナーの[概要](../about/index.md)で説明されているように、コンテナの画像は、一連のレイヤーとして表されるファイルのバンドルです。 レイヤーストレージは、コンテナーに組み込まれているすべてのファイルです。 コンテナーに対する `docker pull` および `docker run` を実行すると、結果は毎回同じになります。

### <a name="where-layers-are-stored-and-how-to-change-it"></a>レイヤーの保存場所と変更方法

既定のインストールでは、レイヤーが `C:\ProgramData\docker` に格納され、"image" ディレクトリおよび "windowsfilter" ディレクトリに分割されます。 レイヤーの保存場所は、「[Windows 上の Docker エンジン](../manage-docker/configure-docker-daemon.md)」の記載に従い、`docker-root` 構成を使用して変更できます。

> [!NOTE]
> レイヤー記憶域は、NTFS のみでサポートされます。 ReFS はサポートされません。

レイヤー ディレクトリ内のファイルは変更しないでください。これらは以下のようなコマンドを使用して慎重に管理します。

- [docker images](https://docs.docker.com/engine/reference/commandline/images/)
- [docker rmi](https://docs.docker.com/engine/reference/commandline/rmi/)
- [docker pull](https://docs.docker.com/engine/reference/commandline/pull/)
- [docker load](https://docs.docker.com/engine/reference/commandline/load/)
- [docker save](https://docs.docker.com/engine/reference/commandline/save/)

### <a name="supported-operations-in-layer-storage"></a>レイヤー記憶域でサポートされる操作

コンテナーの実行では、ほとんどの NTFS 操作を使用できます (トランザクションを除く)。 これには ACL の設定が含まれており、すべての ACL はコンテナー内でチェックされます。 1 つのコンテナー内で複数のユーザーとしてプロセスを実行するには、`RUN net user /create ...` で `Dockerfile` 内にユーザーを作成し、ファイルの ACL を設定します。次に、[Dockerfile USER ディレクティブ](https://docs.docker.com/engine/reference/builder/#user)を使用して、そのユーザーで実行するプロセスを構成します。

## <a name="persistent-storage"></a>常設ストレージ

Windows コンテナーは、バインドマウントとボリュームによって固定記憶域を提供するためのメカニズムをサポートしています。 詳細については、「[コンテナー内の固定記憶域](./persistent-storage.md)」を参照してください。

## <a name="storage-limits"></a>記憶域の制限

Windows アプリケーションでは、新しいファイルをインストールまたは作成する前に、または一時ファイルのクリーンアップのトリガーとして、空きディスク領域の量を照会することが一般的なパターンとなっています。  アプリケーションの互換性を最大化することを目標として、Windows コンテナーの C: ドライブは、仮想の空きサイズを 20 GB として表しています。

一部のユーザーは、この既定値を上書きして、空き領域をより小さい値または大きい値に構成することができます。 これは、"ストレージ選択" 構成内の "サイズ" オプションで実行できます。

### <a name="examples"></a>例

コマンド ライン: `docker run --storage-opt "size=50GB" mcr.microsoft.com/windows/servercore:ltsc2019 cmd`

または、docker 構成ファイルを直接変更することもできます。

```Docker Configuration File
"storage-opts": [
    "size=50GB"
  ]
```

> [!TIP]
> このメソッドは、docker ビルドでも動作します。 Docker 構成ファイルの変更について詳しくは、[Docker の構成](https://docs.microsoft.com/virtualization/windowscontainers/manage-docker/configure-docker-daemon#configure-docker-with-configuration-file)ドキュメントをご覧ください。
