---
title: Windows Server に Windows コンテナーを展開する
description: Windows Server に Windows コンテナーを展開する
keywords: Docker, コンテナー
author: enderb-ms
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: ba4eb594-0cdb-4148-81ac-a83b4bc337bc
ms.openlocfilehash: 701112cac9c3f6d647fe5fb70309350fd0d07161
ms.sourcegitcommit: d69ed13d505e96f514f456cdae0f93dab4fd3746
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/03/2018
ms.locfileid: "4340850"
---
# <a name="container-host-deployment---windows-server"></a>コンテナー ホストの展開 - Windows Server

Windows コンテナー ホストの展開手順は、オペレーティング システムやホスト システムの種類 (物理または仮想) によって異なります。 このドキュメントでは、物理または仮想システム上で、Windows Server 2016 または Windows Server Core 2016 のいずれかに対して Windows コンテナー ホストを展開する方法を詳しく説明します。

## <a name="install-docker"></a>Docker のインストール

Docker は Windows コンテナーで使用するために必要です。 Docker は、Docker エンジンと Docker クライアントで構成されます。 

Docker をインストールするには、[OneGet プロバイダー PowerShell モジュール](https://github.com/OneGet/MicrosoftDockerProvider)を使用します。 プロバイダーは、コンピューターでコンテナー機能を有効にして、Docker をインストールします。これには、再起動が必要です。 

管理者特権の PowerShell セッションを開き、次のコマンドを実行します。

OneGet PowerShell モジュールをインストールします。

```PowerShell
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
```

OneGet を使用して最新バージョンの Docker をインストールします。

```PowerShell
Install-Package -Name docker -ProviderName DockerMsftProvider
```

インストールが完了したら、コンピューターを再起動します。

```PowerShell
Restart-Computer -Force
```

## <a name="install-a-specific-version-of-docker"></a>Docker の特定のバージョンをインストールします。

発生している 2 つのチャンネル Docker EE の Windows Server を使用します。

* `17.06` -Docker Enterprise Edition (Docker エンジン、ポインター UCP、DTR) を使っている場合は、このバージョンを使用します。 `17.06` 既定値です。
* `18.03` -Docker EE エンジンがのみを使用している場合は、このバージョンを使用します。

特定のバージョンをインストールするのには、使用、`RequiredVersion`フラグを設定します。

```PowerShell
Install-Package -Name docker -ProviderName DockerMsftProvider -Force -RequiredVersion 18.03
```

特定の Docker EE バージョンをインストールすると、更新プログラムをインストール済みの DockerMsftProvider モジュールが必要があります。 更新する場合。

```PowerShell
Update-Module DockerMsftProvider
```

## <a name="update-docker"></a>Docker を更新します。

後でチャンネルに、以前のチャネルから Docker EE エンジンを更新する必要がある場合は両方を使用する`-Update`と`-RequiredVersion`フラグ。

```PowerShell
Install-Package -Name docker -ProviderName DockerMsftProvider -Update -Force -RequiredVersion 18.03
```

## <a name="install-base-container-images"></a>コンテナーの基本イメージのインストール

Windows コンテナーを使用する前に、基本イメージをインストールする必要があります。 基本イメージは、コンテナー オペレーティング システムとして Windows Server Core と Nano Server の両方で使用できます。 Docker コンテナー イメージの詳細については、「[Build your own images on docker.com](https://docs.docker.com/engine/tutorials/dockerimages/)」(docker.com で独自のイメージを構築する) を参照してください。

Windows Server 2019 のリリースでは、Microsoft 供給コンテナーの画像は、Microsoft コンテナー レジストリと呼ばれる新しいレジストリに移動します。 Microsoft が発行したコンテナーの画像は、Docker ハブで検出する続行する必要があります。 新しいコンテナー画像の Windows Server 2019 とだけでなく、する発行、MCR からそれらを抽出できるようになります。 古いコンテナー画像は、Windows Server 2019 する前に発行] の Docker のレジストリから引き出すを続行する必要があります。

### <a name="windows-server-2019-and-newer"></a>Windows Server 2019 と新しい

次の実行 ' Windows Server Core' 基本イメージをインストールするには。

```PowerShell
docker pull mcr.microsoft.com/windows/servercore:ltsc2019
```

次の実行 'Nano サーバー' 基本イメージをインストールするには。

```PowerShell
docker pull mcr.microsoft.com/windows/nanoserver:1809
```

### <a name="windows-server-2016-versions-1607-1803"></a>Windows Server 2016 (1607 1803 のバージョン)

Windows Server Core 基本イメージをインストールするには、次のコマンドを実行します。

```PowerShell
docker pull microsoft/windowsservercore
```

Nano Server 基本イメージをインストールするには、次のコマンドを実行します。

```PowerShell
docker pull microsoft/nanoserver
```

> Windows Containers OS Image 使用許諾契約書 (EULA) を参照してください。こちらの「[EULA](../images-eula.md)」に掲載されています。

## <a name="hyper-v-container-host"></a>Hyper-V コンテナー ホスト

Hyper-V コンテナーを実行するには、Hyper-V ロールが必要になります。 Windows コンテナー ホスト自体が Hyper-V 仮想マシンの場合、Hyper-V ロールをインストールする前に、入れ子になった仮想化を有効にする必要があります。 入れ子になった仮想化の詳細については、「[入れ子になった仮想化]( https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting)」を参照してください。

### <a name="nested-virtualization"></a>入れ子になった仮想化

次のスクリプトでは、コンテナー ホストの入れ子になった仮想化を構成します。 このスクリプトは親 Hyper-V マシンで実行されます。 このスクリプトを実行する場合は、必ず、コンテナー ホストの仮想マシンを無効にしてください。

```PowerShell
#replace with the virtual machine name
$vm = "<virtual-machine>"

#configure virtual processor
Set-VMProcessor -VMName $vm -ExposeVirtualizationExtensions $true -Count 2

#disable dynamic memory
Set-VMMemory -VMName $vm -DynamicMemoryEnabled $false

#enable mac spoofing
Get-VMNetworkAdapter -VMName $vm | Set-VMNetworkAdapter -MacAddressSpoofing On
```

### <a name="enable-the-hyper-v-role"></a>Hyper-V ロールの有効化

PowerShell を使用してHyper-V 機能を有効にするには、管理者特権の PowerShell セッションで次のコマンドを実行します。

```PowerShell
Install-WindowsFeature hyper-v
```
