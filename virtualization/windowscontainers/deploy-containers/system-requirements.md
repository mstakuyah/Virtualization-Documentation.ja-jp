---
title: Windows コンテナーの要件
description: Windows コンテナーの要件
keywords: メタデータ、コンテナー
author: taylorb-microsoft
ms.date: 09/26/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
ms.openlocfilehash: d3df0631a8a61db16ad207f49163a7304c5db717
ms.sourcegitcommit: a7f9ab96be359afb37783bbff873713770b93758
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/28/2019
ms.locfileid: "9681052"
---
# <a name="windows-container-requirements"></a>Windows コンテナーの要件

このガイドでは、Windows コンテナーホストの要件を示します。

## <a name="os-requirements"></a>OS 要件

- Windows コンテナー機能は、Windows Server 2016 (コアおよびデスクトップエクスペリエンス)、Windows 10 Professional、Enterprise (記念日) 以降でのみ使用できます。
<<<<<<< HEAD
- Hyper-v の分離を実行する前に、Hyper-v の役割をインストールする必要があります。
- Windows Server コンテナー ホストでは、Windows を c:\ にインストールする必要があります。 この制限は、Hyper-v の分離されたコンテナーのみを展開する場合には適用されません。
=======
- Hyper-v ロールをインストールしてから、Hyper-v 分離でコンテナーを実行する必要があります。
- Windows Server コンテナー ホストでは、Windows を c:\ にインストールする必要があります。 Hyper-V コンテナーのみを展開する場合、この制限は適用されません。
>>>>>>> 原点/マスター

## <a name="virtualized-container-hosts"></a>仮想化されたコンテナーホスト

<<<<<<< HEAD Windows コンテナーホストが Hyper-v 仮想マシンから実行され、Hyper-v 分離をホストする場合は、入れ子になった仮想化を有効にする必要があります。 入れ子になった仮想化には、次の要件があります。 = = = = = = = = = = = =: Windows コンテナーホストは、hyper-v 仮想マシンから実行され、Hyper-v 分離を備えたコンテナーとしても、入れ子になった仮想化を有効にする必要があります。 入れ子になった仮想化には次の要件があります。
>>>>>>> 原点/マスター

- 仮想化された Hyper-V ホスト用に少なくとも 4 GB の RAM を利用できる。
- Windows Server 2019、Windows server バージョン1803、windows Server バージョン1709、windows Server 2016、ホストシステム上の windows 10、仮想マシンの windows Server (フル、コア)。
- Intel VT-x に対応したプロセッサ (この機能は現在 Intel プロセッサのみで使用可能です)。
<<<<<<< HEAD
- コンテナーホスト VM には、少なくとも2つの仮想プロセッサが必要です。

## <a name="supported-base-images"></a>サポートされている基本イメージ

<a name="windows-containers-are-offered-with-four-container-base-images-windows-server-core-nano-server-windows-and-iot-core-not-all-configurations-support-both-os-images-this-table-details-the-supported-configurations"></a>Windows コンテナーは、Windows Server Core、Nano Server、Windows、IoT Core という4つのコンテナーベースのイメージで提供されます。 一部の構成は、どちらの OS イメージもサポートしていません。 サポートされている構成を次の表に示します。
=======
- コンテナー ホスト VM には、少なくとも 2 つの仮想プロセッサも必要になります。

## <a name="supported-base-images"></a>サポートされる基本イメージ

Windows コンテナーは、Windows Server Core、Nano Server、Windows、IoT Core という4つのコンテナーベースのイメージで提供されます。 一部の構成は、どちらの OS イメージもサポートしていません。 サポートされている構成を次の表に示します。

<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:75%" cellpadding="5" cellspacing="5">
<thead>
<tr valign="top">
<th><center>ホスト オペレーティング システム</center></th>
<th><center>Windows Server コンテナー</center></th>
<th><center>Hyper-V による分離</center></th>
</tr>
</thead>
<tbody>
<tr valign="top">
<td><center>Windows Server 2016/2019 (標準またはデータセンター)</center></td>
<td><center>Server Core、Nano Server、Windows</center></td>
<td><center>Server Core、Nano Server、Windows</center></td>
</tr>
<tr valign="top">
<td><center>Nano Server<a href="#warn-1">*</a></center></td>
<td><center> Nano Server</center></td>
<td><center>Server Core、Nano Server、Windows</center></td>
</tr>
<tr valign="top">
<td><center>Windows 10 Pro / Enterprise</center></td>
<td><center>窓<a href="#warn-2">**</a></center></td>
<td><center>Server Core、Nano Server、Windows</center></td>
</tr>
<tr valign="top">
<td><center>IoT Core</center></td>
<td><center>IoT Core</center></td>
<td><center>利用不可</center></td>
</tr>
</tbody>
</table>

> [!Warning]  
> <span id="warn-1">* Windows Server 以降、バージョン 1709 Nano Server は、コンテナーホストとしては使用できなくなりました。</span>

> <span id="warn-2">* * Windows 10 年 2018 10 月の更新プログラムが必要です。また、を使用してコンテナーを実行するときに、プロセスの分離を直接要求することもできます。</span>
>>>>>>> 原点/マスター

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