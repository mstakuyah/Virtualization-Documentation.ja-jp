---
title: Windows コンテナーの要件
description: Windows コンテナーの要件
keywords: メタデータ、コンテナー
author: taylorb-microsoft
ms.author: taylorb
ms.date: 10/22/2019
ms.topic: deployment-article
ms.prod: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
ms.openlocfilehash: 74f501e5efab3a93e60c9d4797464cea283cdc0b
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910502"
---
# <a name="windows-container-requirements"></a>Windows コンテナーの要件

このガイドでは、Windows コンテナーホストの要件を示します。

## <a name="operating-system-requirements"></a>オペレーティング システムの要件

- Windows コンテナー機能は、windows Server (半期チャネル)、Windows Server 2019、Windows Server 2016、Windows 10 Professional および Enterprise の各エディション (バージョン1607以降) で利用できます。
- Hyper-v の分離を実行するには、hyper-v の役割がインストールされている必要があります。
- Windows Server コンテナー ホストでは、Windows を c:\ にインストールする必要があります。 Hyper-v の分離されたコンテナーのみを展開する場合、この制限は適用されません。

## <a name="virtualized-container-hosts"></a>仮想化されたコンテナーホスト

Windows コンテナーホストが Hyper-v 仮想マシンから実行され、Hyper-v の分離もホストする場合は、入れ子になった仮想化を有効にする必要があります。 入れ子になった仮想化には次の要件があります。

- 仮想化された Hyper-V ホスト用に少なくとも 4 GB の RAM を利用できる。
- ホストシステム上の windows Server (半期チャネル)、Windows Server 2019、Windows Server 2016、または Windows 10と Windows Server (フルまたは Server Core) が仮想マシンに存在します。
- Intel VT-x に対応したプロセッサ (この機能は現在 Intel プロセッサのみで使用可能です)。
- コンテナーホスト VM には、少なくとも2つの仮想プロセッサが必要です。

### <a name="memory-requirements"></a>メモリの要件

コンテナーに対して利用可能なメモリの制限は、[リソース コントロール](https://docs.microsoft.com/virtualization/windowscontainers/manage-containers/resource-controls)を使用するか、コンテナー ホストをオーバーロードすることによって構成できます。  コンテナーを起動し、基本的なコマンド (ipconfig、dir など) を実行するために必要な最小限のメモリ量を以下に示します。

>[!NOTE]
>これらの値は、コンテナーまたはコンテナーで実行されているアプリケーションからの要件との間のリソース共有を考慮しません。  たとえば、512 MB の空きメモリを持つホストでは、Hyper-V による分離の下で複数の Server Core コンテナーを実行できます。これは、それらのコンテナーがリソースを共有するためです。

#### <a name="windows-server-2016"></a>Windows Server 2016

| TestVM  | Windows Server コンテナー | Hyper-V による分離    |
| ----------- | ------------------------ | -------------------- |
| Nano Server | 40 MB                     | 130 MB + 1 GB ページファイル |
| Server Core | 50 MB                     | 325 MB + 1 GB ページファイル |

#### <a name="windows-server-semi-annual-channel"></a>Windows Server (半期チャネル)

| TestVM  | Windows Server コンテナー | Hyper-V による分離    |
| ----------- | ------------------------ | -------------------- |
| Nano Server | 30 MB                     | 110 MB + 1 GB ページファイル |
| Server Core | 45 MB                     | 360 MB + 1 GB ページファイル |

## <a name="see-also"></a>関連項目

[オンプレミスのシナリオでの Windows コンテナーと Docker のサポートポリシー](https://support.microsoft.com/help/4489234/support-policy-for-windows-containers-and-docker-on-premises)