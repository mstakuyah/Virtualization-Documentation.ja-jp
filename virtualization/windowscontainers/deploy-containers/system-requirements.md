---
title: Windows コンテナーの要件
description: Windows コンテナーの要件
keywords: メタデータ、コンテナー
author: taylorb-microsoft
ms.date: 09/26/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
ms.openlocfilehash: df5d8e17d0d512f7f53fcacf6c2c2a2652f3e7c0
ms.sourcegitcommit: 73134bf279f3ed18235d24ae63cdc2e34a20e7b7
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/12/2019
ms.locfileid: "10107866"
---
# <a name="windows-container-requirements"></a>Windows コンテナーの要件

このガイドでは、Windows コンテナーホストの要件を示します。

## <a name="os-requirements"></a>OS 要件

- Windows コンテナー機能は、Windows Server 2016 (コアおよびデスクトップエクスペリエンス)、Windows 10 Professional、Enterprise (記念日) 以降でのみ使用できます。
- Hyper-v の分離を実行する前に、Hyper-v の役割をインストールする必要があります。
- Windows Server コンテナー ホストでは、Windows を c:\ にインストールする必要があります。 この制限は、Hyper-v の分離されたコンテナーのみを展開する場合には適用されません。

## <a name="virtualized-container-hosts"></a>仮想化されたコンテナーホスト

Windows コンテナーホストを Hyper-v 仮想マシンから実行し、Hyper-v 分離をホストする場合は、入れ子になった仮想化を有効にする必要があります。 入れ子になった仮想化には次の要件があります。

- 仮想化された Hyper-V ホスト用に少なくとも 4 GB の RAM を利用できる。
- Windows Server 2019、Windows server バージョン1803、windows Server バージョン1709、windows Server 2016、ホストシステム上の windows 10、仮想マシンの windows Server (フル、コア)。
- Intel VT-x に対応したプロセッサ (この機能は現在 Intel プロセッサのみで使用可能です)。
- コンテナーホスト VM には、少なくとも2つの仮想プロセッサが必要です。

### <a name="memory-requirements"></a>メモリの要件

コンテナーに対して利用可能なメモリの制限は、[リソース コントロール](https://docs.microsoft.com/virtualization/windowscontainers/manage-containers/resource-controls)を使用するか、コンテナー ホストをオーバーロードすることによって構成できます。  コンテナーの起動と基本的なコマンドの実行に必要な最小メモリ量 (ipconfig、dir など) は、以下のとおりです。

>[!NOTE]
>これらの値は、コンテナー内で実行されているアプリケーションのコンテナーまたは要件間でのリソース共有を考慮していません。  たとえば、512 MB の空きメモリを持つホストでは、Hyper-V による分離の下で複数の Server Core コンテナーを実行できます。これは、それらのコンテナーがリソースを共有するためです。

#### <a name="windows-server-2016"></a>Windows Server 2016

| 基本イメージ  | Windows Server コンテナー | Hyper-V による分離    |
| ----------- | ------------------------ | -------------------- |
| Nano Server | 40 MB                     | 130 MB + 1 GB ページファイル |
| Server Core | 50 MB                     | 325 MB + 1 GB ページファイル |

#### <a name="windows-server-version-1709"></a>Windows Server Version 1709

| 基本イメージ  | Windows Server コンテナー | Hyper-V による分離    |
| ----------- | ------------------------ | -------------------- |
| Nano Server | 30 MB                     | 110 MB + 1 GB ページファイル |
| Server Core | 45 MB                     | 360 MB + 1 GB ページファイル |
