---
title: Windows コンテナーの印刷スプーラー
description: Windows コンテナーでの印刷スプーラーサービスの現在の動作について説明します。
keywords: docker、コンテナー、printer、spooler
author: cwilhit
ms.openlocfilehash: e104a87046545b90d244783aafb62ad9d151e14b
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/07/2019
ms.locfileid: "9999099"
---
# <a name="print-spooler-in-windows-containers"></a>Windows コンテナーの印刷スプーラー

印刷サービスに依存するアプリケーションは、Windows コンテナーで正常にコンテナーにすることができます。 プリンターサービス機能を正常に有効にするには、特別な要件が満たされている必要があります。 このガイドでは、展開を適切に構成する方法について説明します。

> [!IMPORTANT]
> コンテナーで正常に印刷サービスにアクセスできるようになると、機能はフォームに制限されます。一部の印刷関連の操作が機能しないことがあります。 たとえば、ホストへのプリンタードライバーのインストールに依存しているアプリ**は、コンテナー内からのドライバーのインストールがサポートされ**ていないため、コンテナーに含めることができません。 コンテナーでサポートする必要のない印刷機能を見つけた場合は、以下のフィードバックをお寄せください。

## <a name="setup"></a>Setup

* ホストは、Windows Server 2019 または Windows 10 Pro/Enterprise 2018 10 月の更新プログラム以降である必要があります。
* [Mcr.microsoft.com/windows](https://hub.docker.com/_/microsoft-windowsfamily-windows)の画像はターゲットベースの画像である必要があります。 その他の Windows コンテナーベースの画像 (Nano Server や Windows Server Core など) には、印刷サーバーの役割はありません。

### <a name="hyper-v-isolation"></a>Hyper-V による分離

コンテナーは、Hyper-v 分離を使って実行することをお勧めします。 このモードで実行すると、印刷サービスへのアクセスを実行するためのコンテナーをいくつでも作成できます。 ホストのスプーラーサービスを変更する必要はありません。

機能は、次の PowerShell クエリで確認できます。

```PowerShell
PS C:\Users\Administrator> docker run -it --isolation hyperv mcr.microsoft.com/windows:1809 powershell.exe
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

PS C:\> Get-Service spooler

Status   Name               DisplayName
------   ----               -----------
Running  spooler            Print Spooler


PS C:\> Get-Printer

Name                           ComputerName    Type         DriverName                PortName        Shared   Published
----                           ------------    ----         ----------                --------        ------   --------
Microsoft XPS Document Writer                  Local        Microsoft XPS Document... PORTPROMPT:     False    False
Microsoft Print to PDF                         Local        Microsoft Print To PDF    PORTPROMPT:     False    False
Fax                                            Local        Microsoft Shared Fax D... SHRFAX:         False    False


PS C:\>
```

### <a name="process-isolation"></a>プロセス分離

プロセス分離コンテナーの共有されたカーネルの性質により、現在の動作によって、ホストとすべてのコンテナーの子で、ユーザーはプリンタースプーラーサービスの**1 つのインスタンス**しか実行できません。 ホストでプリンタースプーラーが実行されている場合は、attemping の前にそのホストのサービスを停止してからゲストのプリンターサービスを起動する必要があります。

> [!TIP]
> コンテナーとホストの両方で、コンテナーを起動し、スプーラーサービスのクエリを同時に実行すると、両方の状態が "実行中" と報告されます。 ただし、deceived はできません。コンテナーは、使用可能なプリンターの一覧を照会できません。 ホストのスプーラーサービスを実行しないでください。 

ホストがプリンターサービスを実行しているかどうかを確認するには、次の PowerShell のクエリを使用します。

```PowerShell
PS C:\Users\Administrator> Get-Service spooler

Status   Name               DisplayName
------   ----               -----------
Running  spooler            Print Spooler

PS C:\Users\Administrator>
```

ホストのスプーラーサービスを停止するには、次の PowerShell で次のコマンドを使用します。

```PowerShell
Stop-Service spooler
Set-Service spooler -StartupType Disabled
```

コンテナーを起動し、プリンターへのアクセスを確認します。

```PowerShell
PS C:\Users\Administrator> docker run -it --isolation process mcr.microsoft.com/windows:1809 powershell.exe
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.


PS C:\> Get-Service spooler

Status   Name               DisplayName
------   ----               -----------
Running  spooler            Print Spooler


PS C:\> Get-Printer

Name                           ComputerName    Type         DriverName                PortName        Shared   Published
----                           ------------    ----         ----------                --------        ------   --------
Microsoft XPS Document Writer                  Local        Microsoft XPS Document... PORTPROMPT:     False    False
Microsoft Print to PDF                         Local        Microsoft Print To PDF    PORTPROMPT:     False    False
Fax                                            Local        Microsoft Shared Fax D... SHRFAX:         False    False


PS C:\>
```