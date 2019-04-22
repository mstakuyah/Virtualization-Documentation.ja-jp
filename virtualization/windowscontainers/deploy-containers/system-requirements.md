---
title: Windows コンテナーの要件
description: Windows コンテナーの要件
keywords: メタデータ、コンテナー
author: taylorb-microsoft
ms.date: 09/26/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
ms.openlocfilehash: 942676be30760cbe1701d75f7d2fbca9539ce03b
ms.sourcegitcommit: a5ff22c205149dac4fc05325ef3232089826f1ef
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/22/2019
ms.locfileid: "9380226"
---
# <a name="windows-container-requirements"></a>Windows コンテナーの要件

このガイドは、Windows のコンテナーをホストする場合の要件を一覧表示します。

## <a name="os-requirements"></a>OS の要件

- Windows コンテナーの機能は、Windows Server 2016 でのみ (コアとデスクトップ エクスペリエンス)、Windows 10 Professional および Enterprise (記念日のエディション) 以降。
- HYPER-V 分離を実行する前に HYPER-V 役割をインストールする必要があります。
- Windows Server コンテナー ホストでは、Windows を c:\ にインストールする必要があります。 この制限は、分離 HYPER-V コンテナーを展開するだけの場合は適用されません。

## <a name="virtualized-container-hosts"></a>コンテナーの仮想化されたホスト

Windows コンテナー ホスト HYPER-V 仮想マシンの実行が HYPER-V 分離されますもホストする場合は、入れ子になった仮想化が有効にする必要があります。 入れ子になった仮想化には次の要件があります。

- 仮想化された Hyper-V ホスト用に少なくとも 4 GB の RAM を利用できる。
- Windows Server 2019、Windows Server バージョン 1803、ホスト システム、1709、Windows Server 2016、または Windows 10 のバージョンの Windows Server と Windows Server (全、コア)、仮想マシンでします。
- Intel VT-x に対応したプロセッサ (この機能は現在 Intel プロセッサのみで使用可能です)。
- コンテナー ホスト VM に少なくとも 2 つの仮想プロセッサ必要があります。

## <a name="supported-base-images"></a>サポートされている基本の画像

コンテナーの 4 つの基本画像 Windows コンテナーが用意されています。 Windows Server Core、Nano サーバー、Windows、および IoT コアします。 一部の構成は、どちらの OS イメージもサポートしていません。 サポートされている構成を次の表に示します。

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
<td><center>Windows Server 2016/2019 (標準またはデータ センター)</center></td>
<td><center>サーバーの主要な Nano サーバー、Windows</center></td>
<td><center>サーバーの主要な Nano サーバー、Windows</center></td>
</tr>
<tr valign="top">
<td><center>Nano Server<a href="#warn-1">*</a></center></td>
<td><center> Nano Server</center></td>
<td><center>サーバーの主要な Nano サーバー、Windows</center></td>
</tr>
<tr valign="top">
<td><center>Windows 10 Pro / Enterprise</center></td>
<td><center>利用不可</center></td>
<td><center>サーバーの主要な Nano サーバー、Windows</center></td>
</tr>
<tr valign="top">
<td><center>IoT Core</center></td>
<td><center>IoT Core</center></td>
<td><center>利用不可</center></td>
</tr>
</tbody>
</table>

> [!WARNING]  
> 1709 のバージョンの Windows Server から始めて、Nano サーバーが不要になったコンテナー ホストとして使用します。

### <a name="memory-requirements"></a>メモリの要件

コンテナーに対して利用可能なメモリの制限は、[リソース コントロール](https://docs.microsoft.com/en-us/virtualization/windowscontainers/manage-containers/resource-controls)を使用するか、コンテナー ホストをオーバーロードすることによって構成できます。  コンテナーを起動し、(ipconfig や dir) の基本的なコマンドが実行に必要なメモリ量の最小値は、以下に示します。

>[!NOTE]
>これらの値は、アカウント間でリソース共有コンテナーまたはコンテナーで実行されているアプリケーションからの要件を考慮しません。  たとえば、512 MB の空きメモリを持つホストでは、Hyper-V による分離の下で複数の Server Core コンテナーを実行できます。これは、それらのコンテナーがリソースを共有するためです。

#### <a name="windows-server-2016"></a>Windows Server 2016

| 基本の画像  | Windows Server コンテナー | Hyper-V による分離    |
| ----------- | ------------------------ | -------------------- |
| Nano Server | 40 MB                     | 130 MB + 1 GB ページファイル |
| Server Core | 50 MB                     | 325 MB + 1 GB ページファイル |

#### <a name="windows-server-version-1709"></a>Windows Server Version 1709

| 基本の画像  | Windows Server コンテナー | Hyper-V による分離    |
| ----------- | ------------------------ | -------------------- |
| Nano Server | 30 MB                     | 110 MB + 1 GB ページファイル |
| Server Core | 45 MB                     | 360 MB + 1 GB ページファイル |

### <a name="base-image-differences"></a>基本イメージの相違点

基に適切な基本の画像を選ぶにはどうはいずれかのか。 無料で作成するのには、好きは、各画像の一般的なガイドライン。

- [Windows Server Core](https://hub.docker.com/_/microsoft-windows-servercore): これは、使用する最適なイメージ、アプリケーションでは、.NET framework を必要とする場合。
- [Nano サーバー](https://hub.docker.com/_/microsoft-windows-nanoserver): アプリケーションのみ .NET コア必要がある、Nano サーバーが提供するフルバージョンの量の画像。
- [Windows](https://hub.docker.com/_/microsoft-windowsfamily-windows): コンポーネントまたは Server Core でが表示されない .dll に依存するアプリケーションまたは Nano サーバー イメージ、GDI ライブラリなどがあります。 このイメージは、Windows の完全な依存関係の設定を実行します。
- [「IoT コア](https://hub.docker.com/_/microsoft-windows-iotcore): このイメージは、 [「IoT アプリケーション](https://developer.microsoft.com/en-us/windows/iot)に特化します。 「IoT コア ホストを対象とするときに、このコンテナー画像を使用していますください。

ほとんどのユーザーでは、Windows Server Core Nano サーバーなりますほうの画像を使用します。 以下は、次のことを Nano サーバー上に文書を考慮するように注意してください。

- サービス スタックが削除されている
- .NET Core が含まれていない ([.NET Core の Nano Server イメージ](https://hub.docker.com/r/microsoft/dotnet/) は使用できます)
- PowerShell が削除されている
- WMI が削除されている
- Windows Server Version 1709 からは、アプリケーションがユーザー コンテキストで実行されるため、管理者特権を必要とするコマンドは失敗となります。 コンテナーの管理者アカウント (docker 実行 - ユーザー ContainerAdministrator) などの-ユーザー フラグで NanoServer から管理者アカウントを完全に削除するただしこれを指定できます。

ここには特に重要な相違点が記載されていますが、すべてが網羅されているわけではありません。 上に記載されている以外にも、除外されているコンポーネントがあります。 Nano Server 上には、いつでもレイヤーを追加できることに注意してください。 この例については、[.NET Core の Nano Server の Dockerfile](https://github.com/dotnet/dotnet-docker/blob/master/2.1/sdk/nanoserver-1803/amd64/Dockerfile) をご覧ください。