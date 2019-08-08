---
title: Windows コンテナーの要件
description: Windows コンテナーの要件
keywords: メタデータ、コンテナー
author: taylorb-microsoft
ms.date: 09/26/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
ms.openlocfilehash: 5fc9b5c9135e87a0d3246952c35c9755e9ad209e
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/07/2019
ms.locfileid: "9998469"
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

## <a name="supported-base-images"></a>サポートされている基本イメージ

Windows コンテナーは、Windows Server Core、Nano Server、Windows、IoT Core という4つのコンテナーベースのイメージで提供されます。 一部の構成は、どちらの OS イメージもサポートしていません。 サポートされている構成を次の表に示します。

|ホストオペレーティングシステム|Windows コンテナー|Hyper-V による分離|
|---------------------|-----------------|-----------------|
|Windows Server 2016 または Windows Server 2019 (標準またはデータセンター)|Server Core、Nano Server、Windows|Server Core、Nano Server、Windows|
|Nano Server|Nano Server|Server Core、Nano Server、Windows|
|Windows 10 Pro または Windows 10 Enterprise|使用できません|Server Core、Nano Server、Windows|
|IoT Core|IoT Core|使用できません|

> [!WARNING]  
> Windows Server バージョン1709以降では、Nano Server はコンテナーホストとしては利用できなくなりました。

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

### <a name="base-image-differences"></a>基本イメージの相違点

どのようにして適切な基本イメージを作成するか。 必要に応じてビルドを無料で行うことができます。各画像の一般的なガイドラインを次に示します。

- [Windows Server Core](https://hub.docker.com/_/microsoft-windows-servercore): アプリケーションで .net framework が完全に必要な場合は、この画像を使用するのが最適です。
- [Nano server](https://hub.docker.com/_/microsoft-windows-nanoserver): .net Core のみを必要とするアプリケーションの場合、nano server では、フルな画像が提供されます。
- [Windows](https://hub.docker.com/_/microsoft-windowsfamily-windows): アプリケーションは、サーバー Core または Nano server のイメージ (GDI ライブラリなど) に含まれていないコンポーネントまたは .dll に依存している可能性があります。 この画像は、すべての Windows の依存関係のセットを保持します。
- [Iot Core](https://hub.docker.com/_/microsoft-windows-iotcore): この画像は、 [iot アプリケーション](https://developer.microsoft.com/windows/iot)で使用することを目的としています。 IoT Core ホストをターゲットとする場合は、このコンテナーイメージを使用する必要があります。

ほとんどのユーザーは、Windows Server Core または Nano Server を使用するのに最適なイメージとなります。 Nano Server 上での構築について考えている場合は、次の点に注意してください。

- サービス スタックが削除されている
- .NET Core が含まれていない ([.NET Core の Nano Server イメージ](https://hub.docker.com/r/microsoft/dotnet/) は使用できます)
- PowerShell が削除されている
- WMI が削除されている
- Windows Server Version 1709 からは、アプリケーションがユーザー コンテキストで実行されるため、管理者特権を必要とするコマンドは失敗となります。 ユーザーフラグ (docker run--user ContainerAdministrator など) を使用して、コンテナー管理者アカウントを指定できますが、今後は NanoServer から管理者アカウントを完全に削除することを意図しています。

ここには特に重要な相違点が記載されていますが、すべてが網羅されているわけではありません。 上に記載されている以外にも、除外されているコンポーネントがあります。 Nano Server 上には、いつでもレイヤーを追加できることに注意してください。 この例については、[.NET Core の Nano Server の Dockerfile](https://github.com/dotnet/dotnet-docker/blob/master/2.1/sdk/nanoserver-1803/amd64/Dockerfile) をご覧ください。