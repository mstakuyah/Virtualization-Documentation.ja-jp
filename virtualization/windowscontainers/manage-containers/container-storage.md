---
title: コンテナーストレージの概要
description: Windows Server コンテナーがホストとその他の種類の記憶域を使用する方法
keywords: コンテナー, ボリューム, 記憶域, マウント, bindmount
author: cwilhit
ms.openlocfilehash: fba08de884d59cc1b656895ec2b7078ba3975269
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910272"
---
# <a name="container-storage-overview"></a>コンテナーストレージの概要

<!-- Great diagram would be great! -->

このトピックでは、コンテナーが Windows 上でストレージを使用するさまざまな方法の概要について説明します。 ストレージに関しては、コンテナーの動作は仮想マシンとは異なります。 本質的には、コンテナーは、その中で実行されるアプリがホストのファイルシステム全体に状態を書き込むことがないように構築されています。 コンテナーは既定では "スクラッチ" 領域を使用しますが、Windows ではストレージを永続化する手段も提供されます。

## <a name="scratch-space"></a>スクラッチ領域

既定では、Windows コンテナーは短期ストレージを使用します。 すべてのコンテナー i/o は "スクラッチ領域" で発生し、各コンテナーは独自のスクラッチを取得します。 ファイルの作成とファイルの書き込みは、スクラッチ領域でキャプチャされ、ホストにエスケープされません。 コンテナーインスタンスが停止すると、スクラッチ領域で発生したすべての変更が破棄されます。 新しいコンテナーインスタンスが開始されると、インスタンスに新しいスクラッチ領域が提供されます。

## <a name="layer-storage"></a>レイヤー記憶域

「コンテナーの[概要](../about/index.md)」で説明されているように、コンテナーイメージは一連のレイヤーとして表現されるファイルのバンドルです。 レイヤーのストレージは、コンテナーに組み込まれているすべてのファイルです。 コンテナーに対する `docker pull` および `docker run` を実行すると、結果は毎回同じになります。

### <a name="where-layers-are-stored-and-how-to-change-it"></a>レイヤーの保存場所と変更方法

既定のインストールでは、レイヤーが `C:\ProgramData\docker` に格納され、"image" ディレクトリおよび "windowsfilter" ディレクトリに分割されます。 レイヤーの保存場所は、「[Windows 上の Docker エンジン](../manage-docker/configure-docker-daemon.md)」の記載に従い、`docker-root` 構成を使用して変更できます。

> [!NOTE]
> レイヤー記憶域は、NTFS のみでサポートされます。 ReFS はサポートされません。

レイヤー ディレクトリ内のファイルは変更しないでください。これらは以下のようなコマンドを使用して慎重に管理します。

- [docker images](https://docs.docker.com/engine/reference/commandline/images/)
- [docker rmi](https://docs.docker.com/engine/reference/commandline/rmi/)
- [docker プル](https://docs.docker.com/engine/reference/commandline/pull/)
- [docker 負荷](https://docs.docker.com/engine/reference/commandline/load/)
- [docker save](https://docs.docker.com/engine/reference/commandline/save/)

### <a name="supported-operations-in-layer-storage"></a>レイヤー記憶域でサポートされる操作

コンテナーの実行では、ほとんどの NTFS 操作を使用できます (トランザクションを除く)。 これには ACL の設定が含まれており、すべての ACL はコンテナー内でチェックされます。 1 つのコンテナー内で複数のユーザーとしてプロセスを実行するには、`RUN net user /create ...` で `Dockerfile` 内にユーザーを作成し、ファイルの ACL を設定します。次に、[Dockerfile USER ディレクティブ](https://docs.docker.com/engine/reference/builder/#user)を使用して、そのユーザーで実行するプロセスを構成します。

## <a name="persistent-storage"></a>Persistent Storage

Windows コンテナーは、バインドマウントとボリュームを介して永続ストレージを提供するメカニズムをサポートしています。 詳細については、「[コンテナーの永続ストレージ](./persistent-storage.md)」を参照してください。

## <a name="storage-limits"></a>ストレージの制限

Windows アプリケーションでは、新しいファイルをインストールまたは作成する前に、または一時ファイルのクリーンアップのトリガーとして、空きディスク領域の量を照会することが一般的なパターンとなっています。  アプリケーションの互換性を最大化することを目標として、Windows コンテナーの C: ドライブは、20 GB の仮想ハードディスクサイズを表します。

ユーザーによっては、この既定値を上書きして、空き領域を小さい値またはより大きい値に構成することが必要になる場合があります。 これを行うには、"ストレージの選択" 構成内の "サイズ" オプションを使用します。

### <a name="examples"></a>例

コマンドライン: `docker run --storage-opt "size=50GB" mcr.microsoft.com/windows/servercore:ltsc2019 cmd`

または、docker 構成ファイルを直接変更することもできます。

```Docker Configuration File
"storage-opts": [
    "size=50GB"
  ]
```

> [!TIP]
> この方法は、docker ビルドにも使用できます。 Docker 構成ファイルの変更について詳しくは、[Docker の構成](https://docs.microsoft.com/virtualization/windowscontainers/manage-docker/configure-docker-daemon#configure-docker-with-configuration-file)ドキュメントをご覧ください。
