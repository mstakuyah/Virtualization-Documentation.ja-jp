---
title: "Windows コンテナーの要件"
description: "Windows コンテナーの要件"
keywords: "メタデータ、コンテナー"
author: enderb-ms
ms.date: 09/26/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
ms.openlocfilehash: ff9b24ccc802b228b649c2330333cbe008350aec
ms.sourcegitcommit: ea6edc5bac5705a19d48ffdf1ba676c940c2eb67
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/09/2018
---
# <a name="windows-container-requirements"></a>Windows コンテナーの要件

このガイドでは、Windows コンテナー ホストの要件を一覧で示します。

## <a name="os-requirements"></a>OS 要件

- Windows コンテナーの機能は、Windows Server ビルド 1709、Windows Server 2016 (Core、デスクトップ エクスペリエンス搭載)、Windows 10 Professional および Enterprise (Anniversary Edition) でのみ使用できます。
- Hyper-V コンテナーを実行する前に、Hyper-V ロールをインストールする必要があります。
- Windows Server コンテナー ホストでは、Windows を c:\ にインストールする必要があります。 Hyper-V コンテナーのみを展開する場合、この制限は適用されません。

## <a name="virtualized-container-hosts"></a>仮想化されたコンテナー ホスト

Windows コンテナー ホストが Hyper-V 仮想マシンから実行され、Hyper-V コンテナーもホストする場合、入れ子になった仮想化を有効にする必要があります。 入れ子になった仮想化には次の要件があります。

- 仮想化された Hyper-V ホスト用に少なくとも 4 GB の RAM を利用できる。
- Windows Server ビルド 1709、Windows Server 2016、または ホスト システム上の Windows 10、および仮想マシン内の Windows Server (Full、Core)
- Intel VT-x に対応したプロセッサ (この機能は現在 Intel プロセッサのみで使用可能です)。
- コンテナー ホスト VM には、少なくとも 2 つの仮想プロセッサも必要になります。

## <a name="supported-base-images"></a>サポートされる基本イメージ

Windows コンテナーには、Windows Server Core と Nano Server という 2 つのコンテナー イメージが付属しています。 一部の構成は、どちらの OS イメージもサポートしていません。 サポートされている構成を次の表に示します。

<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:75%" cellpadding="5" cellspacing="5">
<thead>
<tr valign="top">
<th><center>ホスト オペレーティング システム</center></th>
<th><center>Windows Server コンテナー</center></th>
<th><center>Hyper-V コンテナー</center></th>
</tr>
</thead>
<tbody>
<tr valign="top">
<td><center>Windows Server 2016 (Standard または Datacenter)</center></td>
<td><center>Server Core / Nano Server</center></td>
<td><center>Server Core / Nano Server</center></td>
</tr>
<tr valign="top">
<td><center>Nano Server<a href="#warn-1">*</a></center></td>
<td><center> Nano Server</center></td>
<td><center>Server Core / Nano Server</center></td>
</tr>
<tr valign="top">
<td><center>Windows 10 Pro / Enterprise</center></td>
<td><center>利用不可</center></td>
<td><center>Server Core / Nano Server</center></td>
</tr>
</tbody>
</table>

> [!Warning]  
> <span id="warn-1">Windows Server Version 1709 から、Nano Server はコンテナー ホストとして使用できなくなりました。</span>


### <a name="memory-requirements"></a>メモリの要件
コンテナーに対して利用可能なメモリの制限は、[リソース コントロール](https://docs.microsoft.com/en-us/virtualization/windowscontainers/manage-containers/resource-controls)を使用するか、コンテナー ホストをオーバーロードすることによって構成できます。  コンテナーの起動と基本的なコマンド (ipconfig、dir など) の実行に必要なメモリの最小量を以下に示します。  __これらの値では、コンテナー間で共有しているリソースや、そのコンテナーで実行されるアプリケーションの要件が考慮されていない点に注意してください。  たとえば、512 MB の空きメモリを持つホストでは、Hyper-V による分離の下で複数の Server Core コンテナーを実行できます。これは、それらのコンテナーがリソースを共有するためです。__

#### <a name="windows-server-2016"></a>Windows Server 2016
| 基本イメージ  | Windows Server コンテナー | Hyper-V による分離    |
| ----------- | ------------------------ | -------------------- |
| Nano Server | 40 MB                     | 130 MB + 1 GB のページファイル |
| Server Core | 50 MB                     | 325 MB + 1 GB のページファイル |

#### <a name="windows-server-version-1709"></a>Windows Server Version 1709
| 基本イメージ  | Windows Server コンテナー | Hyper-V による分離    |
| ----------- | ------------------------ | -------------------- |
| Nano Server | 30 MB                     | 110 MB + 1 GB のページファイル |
| Server Core | 45 MB                     | 360 MB + 1 GB のページファイル |


### <a name="nano-server-vs-windows-server-core"></a>Nano Server と Windows Server Core の比較

Windows Server Core と Nano Server のいずれを選択するかは、どのような点を検討すればよいでしょうか。 いずれもユーザーが自由に構築できますが、アプリケーションが .NET Framework との完全な互換性を必要とする場合は、[Windows Server Core](https://hub.docker.com/r/microsoft/windowsservercore/) を使用する必要があります。 それとは逆に、アプリケーションがクラウド用に構築され、.NET Core を使用する場合は、[Nano Server](https://hub.docker.com/r/microsoft/nanoserver/)を使用する必要があります。 これは、Nano Server が可能な限りフットプリントを小さく抑える目的で構築されており、必須でない複数のライブラリが削除されているためです。 Nano Server での構築を検討する場合、以下の点に注意してください。

- サービス スタックが削除されている
- .NET Core が含まれていない ([.NET Core の Nano Server イメージ](https://hub.docker.com/r/microsoft/dotnet/) は使用できます)
- PowerShell が削除されている
- WMI が削除されている

この他にも相違点があり、これら以上に重要な違いもあります。 上に記載されている以外にも、除外されているコンポーネントがあります。 Nano Server 上には、いつでもレイヤーを追加できることに注意してください。 この例については、[.NET Core の Nano Server の Dockerfile](https://github.com/dotnet/dotnet-docker/blob/master/2.0/sdk/nanoserver/amd64/Dockerfile) をご覧ください。

