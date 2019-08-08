---
title: Windows Server に Windows コンテナーを展開する
description: Windows Server に Windows コンテナーを展開する
keywords: Docker, コンテナー
author: taylorb-microsoft
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: ba4eb594-0cdb-4148-81ac-a83b4bc337bc
ms.openlocfilehash: e045539b189eb8cd1594da0784ab0c88e848c948
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/07/2019
ms.locfileid: "9998779"
---
# <a name="container-host-deployment-windows-server"></a>コンテナーホストの展開: Windows Server

Windows コンテナー ホストの展開手順は、オペレーティング システムやホスト システムの種類 (物理または仮想) によって異なります。 このドキュメントでは、物理または仮想システム上で、Windows Server 2016 または Windows Server Core 2016 のいずれかに対して Windows コンテナー ホストを展開する方法を詳しく説明します。

## <a name="install-docker"></a>Docker のインストール

Docker は Windows コンテナーで使用するために必要です。 Docker は、Docker エンジンと Docker クライアントで構成されます。

Docker をインストールするには、 [Oneget Provider PowerShell モジュール](https://github.com/OneGet/MicrosoftDockerProvider)を使います。 プロバイダーは、コンピューター上のコンテナー機能を有効にし、Docker をインストールします。これにより、再起動が必要になります。

管理者特権の PowerShell セッションを開いて、次のコマンドレットを実行します。

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

## <a name="install-a-specific-version-of-docker"></a>特定のバージョンの Docker をインストールする

現在、Windows Server 向けの Docker EE には、次の2つのチャネルが用意されています。

* `17.06` -Docker Enterprise Edition (Docker Engine、UCP、DTR) を使っている場合は、このバージョンを使用します。 `17.06` は既定です。
* `18.03` -Docker EE エンジンだけを実行している場合は、このバージョンを使用します。

特定のバージョンをインストールするには`RequiredVersion` 、次のフラグを使用します。

```PowerShell
Install-Package -Name docker -ProviderName DockerMsftProvider -Force -RequiredVersion 18.03
```

特定の Docker EE バージョンをインストールするには、以前にインストールされた DockerMsftProvider モジュールの更新が必要になることがあります。 更新するには:

```PowerShell
Update-Module DockerMsftProvider
```

## <a name="update-docker"></a>Docker の更新

以前のチャネルから後のチャネルに向けて Docker EE エンジンを更新する必要がある場合`-Update`は`-RequiredVersion` 、とフラグの両方を使用します。

```PowerShell
Install-Package -Name docker -ProviderName DockerMsftProvider -Update -Force -RequiredVersion 18.03
```

## <a name="install-base-container-images"></a>ベースコンテナーのイメージをインストールする

Windows コンテナーを使用する前に、基本イメージをインストールする必要があります。 基本イメージは、コンテナー オペレーティング システムとして Windows Server Core と Nano Server の両方で使用できます。 Docker コンテナー イメージの詳細については、「[Build your own images on docker.com](https://docs.docker.com/engine/tutorials/dockerimages/)」(docker.com で独自のイメージを構築する) を参照してください。

Windows Server 2019 のリリースでは、Microsoft が供給しているコンテナーのイメージは、Microsoft Container レジストリという新しいレジストリに移動します。 Microsoft によって公開されたコンテナーイメージは、引き続き Docker Hub から検出される必要があります。 Windows Server 2019 以降で公開されている新しいコンテナーイメージについては、MCR から引き出すことを検討してください。 Windows Server 2019 より前に公開されている古いコンテナーイメージの場合は、引き続き Docker のレジストリからそのイメージを取得する必要があります。

### <a name="windows-server-2019-and-newer"></a>Windows Server 2019 以降

「Windows Server Core」のベース画像をインストールするには、次の操作を実行します。

```PowerShell
docker pull mcr.microsoft.com/windows/servercore:ltsc2019
```

"Nano Server" のベースイメージをインストールするには、次の操作を実行します。

```PowerShell
docker pull mcr.microsoft.com/windows/nanoserver:1809
```

### <a name="windows-server-2016-versions-1607-1803"></a>Windows Server 2016 (バージョン 1607-1803)

Windows Server Core 基本イメージをインストールするには、次のコマンドを実行します。

```PowerShell
docker pull microsoft/windowsservercore
```

Nano Server 基本イメージをインストールするには、次のコマンドを実行します。

```PowerShell
docker pull microsoft/nanoserver
```

> Windows コンテナ OS イメージ EULA (ここに記載されています) については、「 [eula](../images-eula.md)」を参照してください。

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
