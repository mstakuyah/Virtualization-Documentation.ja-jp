---
title: Windows コンテナーの印刷スプーラ
description: Windows コンテナーの印刷スプーラサービスの現在の動作について説明します。
keywords: docker、コンテナー、プリンター、スプーラ
author: cwilhit
ms.openlocfilehash: e104a87046545b90d244783aafb62ad9d151e14b
ms.sourcegitcommit: 16744984ede5ec94cd265b6bff20aee2f782ca88
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/18/2020
ms.locfileid: "77439539"
---
# <a name="print-spooler-in-windows-containers"></a>Windows コンテナーの印刷スプーラ

印刷サービスに依存するアプリケーションは、Windows コンテナーで正常にコンテナー化できます。 プリンタサービスの機能を正常に有効にするためには、特別な要件を満たす必要があります。 このガイドでは、展開を適切に構成する方法について説明します。

> [!IMPORTANT]
> コンテナーで印刷サービスへのアクセスを正常に取得できますが、機能はフォームで制限されています。一部の印刷関連の操作が機能しない可能性があります。 たとえば、ホストへのプリンタードライバーのインストールに依存しているアプリ**は、コンテナー内からのドライバーのインストールがサポートされ**ていないため、コンテナー化できません。 サポートする必要のあるサポートされていない印刷機能がコンテナーで見つかった場合は、以下のフィードバックを開いてください。

## <a name="setup"></a>セットアップ

* このホストは、Windows Server 2019 または Windows 10 Pro/Enterprise の10月2018更新プログラム以降である必要があります。
* [Mcr.microsoft.com/windows](https://hub.docker.com/_/microsoft-windowsfamily-windows)イメージは、対象となる基本イメージである必要があります。 その他の Windows コンテナー基本イメージ (Nano Server、Windows Server Core など) には、プリントサーバーの役割はありません。

### <a name="hyper-v-isolation"></a>Hyper-V による分離

Hyper-v の分離を使用してコンテナーを実行することをお勧めします。 このモードで実行すると、印刷サービスへのアクセスで実行するコンテナーの数を指定できます。 ホストのスプーラサービスを変更する必要はありません。

次の PowerShell クエリを使用して、機能を確認できます。

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

### <a name="process-isolation"></a>プロセスの分離

プロセス分離コンテナーのカーネルの共有の性質により、現在の動作によって、ユーザーは、ホストとそのすべてのコンテナーの子に対して、プリンタースプーラサービスの**1 つのインスタンス**のみを実行するように制限されます。 ホストでプリンタスプーラが実行されている場合は、attemping の前にホスト上のサービスを停止して、ゲストでプリンタサービスを起動する必要があります。

> [!TIP]
> コンテナーを起動し、コンテナーとホストの両方でスプーラサービスに対してクエリを同時に実行すると、両方の状態が "実行中" として報告されます。 ただし、思い込まではありません。コンテナーは、使用可能なプリンターの一覧を照会できません。 ホストのスプーラサービスを実行することはできません。 

ホストでプリンタサービスが実行されているかどうかを確認するには、次の PowerShell でクエリを使用します。

```PowerShell
PS C:\Users\Administrator> Get-Service spooler

Status   Name               DisplayName
------   ----               -----------
Running  spooler            Print Spooler

PS C:\Users\Administrator>
```

ホストでスプーラサービスを停止するには、以下の PowerShell で次のコマンドを使用します。

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