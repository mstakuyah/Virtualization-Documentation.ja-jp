---
title: Windows Server に Windows コンテナーを展開する
description: Windows Server に Windows コンテナーを展開する
keywords: Docker, コンテナー
author: taylorb-microsoft
ms.date: 09/09/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: ba4eb594-0cdb-4148-81ac-a83b4bc337bc
ms.openlocfilehash: 9899a2d76bfa1fe312e3bd983f60d09d77c272e9
ms.sourcegitcommit: ac923217ee2f74f08df2b71c2a4c57b694f0d7c3
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/06/2020
ms.locfileid: "78853913"
---
# <a name="container-host-deployment-windows-server"></a>コンテナーホストの展開: Windows Server

Windows コンテナー ホストの展開手順は、オペレーティング システムやホスト システムの種類 (物理または仮想) によって異なります。 このドキュメントでは、物理または仮想システム上で、Windows Server 2016 または Windows Server Core 2016 のいずれかに対して Windows コンテナー ホストを展開する方法を詳しく説明します。

## <a name="install-docker"></a>Docker のインストール

Docker は Windows コンテナーで使用するために必要です。 Docker は、docker エンジンと Docker クライアントで構成されています。

Docker をインストールするには、 [Oneget Provider PowerShell モジュール](https://github.com/OneGet/MicrosoftDockerProvider)を使用します。 プロバイダーは、コンピューターでコンテナー機能を有効にし、Docker をインストールします。これにより、再起動が必要になります。

管理者特権の PowerShell セッションを開き、次のコマンドレットを実行します。

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

## <a name="install-a-specific-version-of-docker"></a>Docker の特定のバージョンをインストールする

現在、Docker EE for Windows Server で使用できるチャネルは2つあります。

* `17.06`-Docker Enterprise Edition (Docker エンジン、UCP、DTR) を使用している場合は、このバージョンを使用します。 既定値は `17.06` です。
* `18.03`-Docker EE エンジンのみを実行している場合は、このバージョンを使用します。

特定のバージョンをインストールするには、`RequiredVersion` フラグを使用します。

```PowerShell
Install-Package -Name docker -ProviderName DockerMsftProvider -Force -RequiredVersion 18.03
```

特定の Docker EE バージョンをインストールするには、以前にインストールした DockerMsftProvider モジュールの更新が必要になる場合があります。 更新するには:

```PowerShell
Update-Module DockerMsftProvider
```

## <a name="update-docker"></a>Docker の更新

以前のチャネルから後のチャネルに Docker EE エンジンを更新する必要がある場合は、`-Update` と `-RequiredVersion` の両方のフラグを使用します。

```PowerShell
Install-Package -Name docker -ProviderName DockerMsftProvider -Update -Force -RequiredVersion 18.03
```

## <a name="install-base-container-images"></a>基本コンテナーイメージのインストール

Windows コンテナーを使用する前に、基本イメージをインストールする必要があります。 基本イメージは、コンテナー オペレーティング システムとして Windows Server Core と Nano Server の両方で使用できます。 Docker コンテナー イメージの詳細については、「[Build your own images on docker.com](https://docs.docker.com/engine/tutorials/dockerimages/)」(docker.com で独自のイメージを構築する) を参照してください。

> [!TIP]
> 2018年5月から、一貫性のある信頼できる取得エクスペリエンスが提供されるようになり、ほとんどすべての Microsoft ソースコンテナーイメージは、 [_Docker Hub_](https://hub.docker.com/publishers/microsoftowner)を介して現在の検出プロセスを維持しながら、microsoft Container Registry _mcr.microsoft.com_から提供されます。

### <a name="windows-server-2019-and-newer"></a>Windows Server 2019 以降

"Windows Server Core" 基本イメージをインストールするには、次のように実行します。

```PowerShell
docker pull mcr.microsoft.com/windows/servercore:ltsc2019
```

' Nano Server ' 基本イメージをインストールするには、次のように実行します。

```PowerShell
docker pull mcr.microsoft.com/windows/nanoserver:1809
```

### <a name="windows-server-2016-versions-1607-1803"></a>Windows Server 2016 (バージョン 1607-1803)

Windows Server Core 基本イメージをインストールするには、次のコマンドを実行します。

```PowerShell
docker pull mcr.microsoft.com/windows/servercore:1607
```

Nano Server 基本イメージをインストールするには、次のコマンドを実行します。

```PowerShell
docker pull mcr.microsoft.com/windows/nanoserver:1803
```

> Windows コンテナー OS イメージの使用許諾契約書 ( [eula](../images-eula.md)) を参照してください。

## <a name="hyper-v-isolation-host"></a>Hyper-v 分離ホスト

Hyper-v の分離を実行するには、Hyper-v の役割が必要です。 Windows コンテナー ホスト自体が Hyper-V 仮想マシンの場合、Hyper-V ロールをインストールする前に、入れ子になった仮想化を有効にする必要があります。 入れ子になった仮想化の詳細については、「[入れ子になった仮想化](https://docs.microsoft.com/virtualization/hyper-v-on-windows/user-guide/nested-virtualization)」を参照してください。

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

PowerShell を使用して Hyper-v 機能を有効にするには、管理者特権の PowerShell セッションで次のコマンドレットを実行します。

```PowerShell
Install-WindowsFeature hyper-v
```
