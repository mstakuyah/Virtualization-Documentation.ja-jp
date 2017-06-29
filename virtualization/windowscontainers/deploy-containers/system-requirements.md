---
title: "Windows コンテナーの要件"
description: "Windows コンテナーの要件"
keywords: "メタデータ、コンテナー"
author: enderb-ms
ms.date: 09/26/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
ms.openlocfilehash: c6f8dd68c6c9f346e26d29b0072a0e3d8c18759f
ms.sourcegitcommit: 8f096e9d557c60985c8512b43e4e4058cb307ed2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/09/2017
---
# <a name="windows-container-requirements"></a>Windows コンテナーの要件

このガイドでは、Windows コンテナー ホストの要件を一覧で示します。

## <a name="os-requirements"></a>OS 要件

- Windows コンテナーの機能は、Windows Server 2016 (Core、デスクトップ エクスペリエンス搭載)、Nano Server、Windows 10 Professional および Enterprise (Anniversary Edition) でのみ使用できます。
- Hyper-V コンテナーを実行する前に、Hyper-V ロールをインストールする必要があります。
- Windows Server コンテナー ホストでは、Windows を c:\ にインストールする必要があります。 Hyper-V コンテナーのみを展開する場合、この制限は適用されません。

## <a name="virtualized-container-hosts"></a>仮想化されたコンテナー ホスト

Windows コンテナー ホストが Hyper-V 仮想マシンから実行され、Hyper-V コンテナーもホストする場合、入れ子になった仮想化を有効にする必要があります。 入れ子になった仮想化には次の要件があります。

- 仮想化された Hyper-V ホスト用に少なくとも 4 GB の RAM を利用できる。
- Windows Server 2016、またはホスト システム上の Windows 10、および Windows Server (Full、Core) または仮想マシンの Nano Server
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
<td><center>Nano Server</center></td>
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

## <a name="matching-container-host-version-with-container-image-versions"></a>コンテナー ホストのバージョンとコンテナー イメージのバージョンを一致させる
### <a name="windows-server-containers"></a>Windows Server のコンテナー
Windows Server コンテナーとその基になっているホストは単一のカーネルを共有するため、コンテナーの基本イメージはホストの基本イメージと一致している必要があります。  バージョンが異なる場合、コンテナーは起動するかもしれませんが、完全に機能することは保証できません。 したがって、一致しないバージョンはサポートされません。  Windows オペレーティング システムは、メジャー、マイナー、ビルド、リビジョンの 4 つのレベルでバージョン管理されています (例: 10.0.14393.0)。 ビルド番号は、新しいバージョンの OS が公開された場合にのみ変更されます。 リビジョン番号は、Windows 更新プログラムが適用されると更新されます。 Windows Server コンテナーは、ビルド番号が異なると起動をブロックされます (例: 10.0.14300.1030 (Technical Preview 5) と 10.0.14393 (Windows Server 2016 RTM))。 ビルド番号が一致し、リビジョン番号が異なる場合は、起動をブロックされません (例: 10.0.14393 (Windows Server 2016 RTM) と 10.0.14393.206 (Windows Server 2016 GA))。 ただし、技術的にはブロックされませんが、これは状況によっては正しく機能しない可能性がある構成であるため、運用環境ではサポートできません。 

インストールされている Windows ホストのバージョンは、HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion で確認できます。  基本イメージが使用しているバージョンは、Docker ハブのタグまたはイメージの説明で提供されるイメージのハッシュ テーブルで確認できます。  「[Windows 10 の更新履歴](https://support.microsoft.com/en-us/help/12387/windows-10-update-history)」のページでは、各ビルドとリビジョンがリリースされた日付がわかります。

この例では、14393 がメジャー ビルド番号で、321 がリビジョンです。
```none
Windows PowerShell
Copyright (C) 2016 Microsoft Corporation. All rights reserved.

PS C:\Users\Administrator> (Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\').BuildLabEx
14393.321.amd64fre.rs1_release_inmarket.161004-2338
```

### <a name="hyper-v-containers"></a>Hyper-V コンテナー
コンテナーとホストの間でカーネルを共有する Windows Server コンテナーとは異なり、各 Hyper-V コンテナーは専用の Windows カーネル インスタンスを利用します。  このため、コンテナー ホストのバージョンとコンテナー イメージのバージョンが一致しないことがあります。  現時点では、ビルド番号が Windows Server 2016 GA (10.0.14393.206) と等しいかそれより大きいビルドは、リビジョン番号に関わらず、サポートされる構成において、Windows Server Core または Nano Server のどちらの Windows Server 2016 GA イメージでも実行できます。  将来的には、ユーザーのフィードバックに基づいて、どのくらい離れたビルド番号までが許容されるのかについて、詳しいガイダンスを提供します。  Windows Update によって完全な機能、信頼性、セキュリティが確実に提供されるようにするには、すべてのシステムを最新バージョンに維持する必要があることを理解しておくことが重要です。  
