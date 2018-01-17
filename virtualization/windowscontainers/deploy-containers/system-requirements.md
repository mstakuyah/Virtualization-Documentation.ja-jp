---
title: "Windows コンテナーの要件"
description: "Windows コンテナーの要件"
keywords: "メタデータ、コンテナー"
author: enderb-ms
ms.date: 09/26/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
ms.openlocfilehash: ecc11468bbd5aad2638da3c4f733e4d5068f0056
ms.sourcegitcommit: 77a6195318732fa16e7d5be727bdb88f52f6db46
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/20/2017
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
<td><center>Nano Server*</center></td>
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
* Windows Server Version 1709 から、Nano Server はコンテナー ホストとしてのみ使用できなくなりました。

### <a name="memory-requirments"></a>メモリの要件
コンテナーに対して利用可能なメモリの制限は、[リソース コントロール](https://docs.microsoft.com/en-us/virtualization/windowscontainers/manage-containers/resource-controls)を使用するか、コンテナー ホストをオーバーロードすることによって構成できます。  コンテナーの起動と基本的なコマンド (ipconfig、dir など) の実行に必要なメモリの最小量を以下に示します。  これらの値では、コンテナー間で共有しているリソースや、そのコンテナーで実行されるアプリケーションの用件が考慮されていない点に注意してください。

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

この他にも相違点があり、これら以上に重要な違いもあります。 上に記載されている以外にも、除外されているコンポーネントがあります。 Nano Server 上には、いつでもレイヤーを追加できることに注意してください。 この例については、[.NET Core の Nano Server の Dockerfile](https://github.com/dotnet/dotnet-docker/blob/master/2.0/sdk/nanoserver/amd64/Dockerfile)をご覧ください。

## <a name="matching-container-host-version-with-container-image-versions"></a>コンテナー ホストのバージョンとコンテナー イメージのバージョンを一致させる
### <a name="windows-server-containers"></a>Windows Server のコンテナー
Windows Server コンテナーとその基になっているホストは単一のカーネルを共有するため、コンテナーの基本イメージはホストの基本イメージと一致している必要があります。  バージョンが異なる場合、コンテナーは起動するかもしれませんが、完全に機能することは保証できません。 Windows オペレーティング システムは、メジャー、マイナー、ビルド、リビジョンの 4 つのレベルでバージョン管理されています (例: 10.0.14393.103)。 ビルド番号(14393 など) は、新しいバージョンの OS (Version 1709、1803、Fall Creators Update など) が公開された場合にのみ変更されます。リビジョン番号 (103 など) は、Windows 更新プログラムが適用されると更新されます。
#### <a name="build-number-new-release-of-windows"></a>ビルド番号 (Windows の新しいリリース)
Windows Server コンテナーは、コンテナー ホストとコンテナー イメージの間でビルド番号が異なると起動をブロックされます (例: 10.0.14393.* (Windows Server 2016) と 10.0.16299.* (Windows Server Version 1709))。  
#### <a name="revision-number-patching"></a>リビジョン番号 (修正プログラムの適用)
Windows Server コンテナーは、コンテナー ホストとコンテナー イメージの間でリビジョン番号が異なっても起動をブロックされません__ (例: 10.0.14393.1914 (KB4051033 が適用された Windows Server 2016) と 10.0.14393.1944 (KB4053579 が適用された Windows Server 2016))。  
Windows Server 2016 ベースのホスト/イメージの場合 – コンテナー イメージのリビジョンは、サポートされる構成でホストと一致する必要があります。  Windows Server Version 1709 以降、これは当てはまりません。ホストとコンテナー イメージのリビジョンが一致する必要はありません。  通常どおり、システムに最新のパッチおよび更新プログラムを適用しておくことをお勧めします。
#### <a name="practical-application"></a>実際の適用例
例 1: KB4041691 が適用された Windows Server 2016 をコンテナー ホストが実行している場合。  このホストに展開されている Windows Server コンテナーはすべて、10.0.14393.1770 コンテナー基本イメージに基づいている必要があります。  ホストに KB4053579 を適用した場合、サポートされた状態を維持するには、同時にコンテナー イメージも更新する必要があります。
例 2: KB4043961 が適用された Windows Server Version 1709 をコンテナー ホストが実行している場合。  このホストに展開されている Windows Server コンテナーはすべて、Windows Server Version 1709 (10.0.16299) コンテナー基本イメージに基づいている必要がありますが、ホストの KB に一致する必要はありません。  ホストに KB4054517 が適用されていれば、コンテナー イメージを更新する必要はありませんが、セキュリティの問題に十分に対応するには更新を行ってください。
#### <a name="querying-version"></a>バージョンの照会
方法 1: Version 1709 以降、cmd プロンプトと ver コマンドでリビジョン詳細が返されるようになりました。
```
Microsoft Windows [Version 10.0.16299.125]
(c) 2017 Microsoft Corporation. All rights reserved.

C:\>ver

Microsoft Windows [Version 10.0.16299.125] 
```
方法 2: 次のレジストリ キーを照会します: HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion 例:
```
C:\>reg query "HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion" /v BuildLabEx
```
または
```
Windows PowerShell
Copyright (C) 2016 Microsoft Corporation. All rights reserved.

PS C:\Users\Administrator> (Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\').BuildLabEx
14393.321.amd64fre.rs1_release_inmarket.161004-2338
```

基本イメージが使用しているバージョンは、Docker ハブのタグまたはイメージの説明で提供されるイメージのハッシュ テーブルで確認できます。  「[Windows 10 の更新履歴](https://support.microsoft.com/en-us/help/12387/windows-10-update-history)」のページでは、各ビルドとリビジョンがリリースされた日付がわかります。

### <a name="hyper-v-isolation-for-containers"></a>Hyper-V によるコンテナーの分離
Windows コンテナーは、Hyper-V による分離を使用して実行することも、使用せずに実行することもできます。  Hyper-V による分離を使用すると、最適化された VM によって、コンテナーの周囲にセキュリティ保護された境界が形成されます。  標準の Windows コンテナーが、コンテナーやホストとの間でカーネルを共有するのに対し、Hyper-V によって分離されたコンテナーは、専用の Windows カーネル インスタンスを利用します。  このため、コンテナー ホストとコンテナー イメージで異なるバージョンの OS を使用できます (下の互換性マトリクスを参照)。  

Hyper-V による分離を使ってコンテナーを実行するには、docker run コマンドに "--isolation=hyper-v" というタグを追加します。

### <a name="compatibility-matrix"></a>互換性マトリクス
2016 GA (10.0.14393.206) 以降の Windows Server ビルドでは、リビジョン番号に関わらず、Windows Server Core と Nano Server のいずれかの Windows Server 2016 GA イメージを、サポート対象の構成で実行できます。
Windows Server Version 1709 ホストで Windows Server 2016 ベースのコンテナーを実行することもできますが、この逆はサポートされていません。

Windows Update によって完全な機能、信頼性、セキュリティが確実に提供されるようにするには、すべてのシステムを最新バージョンに維持する必要があることを理解しておくことが重要です。  
