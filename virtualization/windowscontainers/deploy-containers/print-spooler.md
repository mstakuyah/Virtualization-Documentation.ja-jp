---
title: Windows のコンテナーに印刷スプーラー
description: Windows のコンテナーに印刷スプーラー サービスの現在の作業の動作を説明します。
keywords: docker、コンテナー、プリンター、スプーラー
author: cwilhit
ms.openlocfilehash: 48130bc6a826a45dfa49d0a3b4600d227f34704e
ms.sourcegitcommit: 3c81b0efd1ac2c4c93d58f16edae1044c9a5ad55
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/05/2019
ms.locfileid: "9284576"
---
# <a name="print-spooler-in-windows-containers"></a>Windows のコンテナーに印刷スプーラー

印刷サービスに依存アプリケーションは、Windows のコンテナーが正常にコンテナー詰めことができます。 正常にプリンターのサービスの機能を有効にするために満たす必要がある特別な要件があります。 このガイドでは、適切に展開を構成する方法について説明します。

> [!IMPORTANT]
> 印刷へのアクセスを取得するサービスの正常にコンテナーの動作、中に、フォームの機能が制限されます。印刷に関連するいくつかの操作が機能しないことができます。 たとえば、アプリのホストにプリンター ドライバーのインストールに依存することはできませんはコンテナー詰めため **、コンテナー内からドライバーのインストールがサポートされていません**。 コンテナーでサポートされるサポートされていない印刷機能を見つけた場合は、次のフィードバックを開いてください。

## <a name="setup"></a>Setup

* ホストは、Windows Server 2019 または Windows 10 Pro/エンタープライズ 2018 の年 10 月の更新以降。
* [Mcr.microsoft.com/windows](https://hub.docker.com/_/microsoft-windowsfamily-windows)イメージを対象となる基本イメージとなります。 その他の Windows コンテナー基本などの画像 (Nano Server と Windows Server Core) を印刷サーバーの役割にすることはありません。

### <a name="hyper-v-isolation"></a>Hyper-V による分離

HYPER-V 分離コンテナーを実行していることをお勧めします。 このモードで実行すると、印刷サービスにアクセスして必要な分だけのコンテナーを持つことができます。 ホスト上のスプーラー サービスを変更する必要はありません。

次の PowerShell クエリを使用して機能を確認できます。

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

共有カーネル性プロセス分離コンテナーが原因では、現在の動作は、ホストとそのコンテナーのすべての子にわたって、プリンター スプーラー サービスの**1 つのインスタンス**を実行しているユーザーを制限します。 ホストが実行されているプリンター スプーラーおり、ゲストで、プリンターのサービスを起動しようとした前に、ホストでサービスを停止する必要があります。

> [!TIP]
> コンテナーを起動しますコンテナーとホストの両方でスプーラー サービスを同時にクエリを実行すると、どちらも"実行中"としての状態を報告します。 コンテナーを利用可能なプリンターの一覧についてはクエリを実行することにすることはできません - deceived する必要はありませんが、します。 ホストのスプーラー サービスは実行されませんする必要があります。 

ホストで、プリンターのサービスを実行しているかどうかを確認するには、[PowerShell の下で、クエリを使用します。

```PowerShell
PS C:\Users\Administrator> Get-Service spooler

Status   Name               DisplayName
------   ----               -----------
Running  spooler            Print Spooler

PS C:\Users\Administrator>
```

ホスト上、スプーラー サービスを停止するには、次の PowerShell で次のコマンドを使用します。

```PowerShell
Stop-Service spooler
Set-Service spooler -StartupType Disabled
```

コンテナーを起動し、[プリンターへのアクセスを確認します。

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