---
title: "Windows Server に Windows コンテナーを展開する"
description: "Windows Server に Windows コンテナーを展開する"
keywords: "Docker, コンテナー"
author: enderb-ms
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: ba4eb594-0cdb-4148-81ac-a83b4bc337bc
ms.openlocfilehash: 1db66d5dfe0f0060c56625e396ecff3bb72e3189
ms.sourcegitcommit: 456485f36ed2d412cd708aed671d5a917b934bbe
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/08/2017
---
# <a name="container-host-deployment---windows-server"></a>コンテナー ホストの展開 - Windows Server

Windows コンテナー ホストの展開手順は、オペレーティング システムやホスト システムの種類 (物理または仮想) によって異なります。 このドキュメントでは、物理または仮想システム上で、Windows Server 2016 または Windows Server Core 2016 のいずれかに対して Windows コンテナー ホストを展開する方法を詳しく説明します。

## <a name="install-docker"></a>Docker のインストール

Docker は Windows コンテナーで使用するために必要です。 Docker は、Docker エンジンと Docker クライアントで構成されます。 

Docker をインストールするには、[OneGet プロバイダー PowerShell モジュール](https://github.com/OneGet/MicrosoftDockerProvider)を使用します。 プロバイダーは、コンピューターでコンテナー機能を有効にして、Docker をインストールします。これには、再起動が必要です。 

管理者特権の PowerShell セッションを開き、次のコマンドを実行します。

OneGet PowerShell モジュールをインストールします。

```
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
```

OneGet を使用して最新バージョンの Docker をインストールします。

```
Install-Package -Name docker -ProviderName DockerMsftProvider
```

インストールが完了したら、コンピューターを再起動します。

```
Restart-Computer -Force
```

## <a name="install-base-container-images"></a>コンテナーの基本イメージのインストール

Windows コンテナーを使用する前に、基本イメージをインストールする必要があります。 基本イメージは、コンテナー オペレーティング システムとして Windows Server Core と Nano Server の両方で使用できます。 Docker コンテナー イメージの詳細については、「[Build your own images on docker.com](https://docs.docker.com/engine/tutorials/dockerimages/)」(docker.com で独自のイメージを構築する) を参照してください。

Windows Server Core 基本イメージをインストールするには、次のコマンドを実行します。

```
docker pull microsoft/windowsservercore
```

Nano Server 基本イメージをインストールするには、次のコマンドを実行します。

```
docker pull microsoft/nanoserver
```

> Windows Containers OS Image 使用許諾契約書 (EULA) を参照してください。こちらの「[EULA](../images-eula.md)」に掲載されています。

## <a name="hyper-v-container-host"></a>Hyper-V コンテナー ホスト

Hyper-V コンテナーを実行するには、Hyper-V ロールが必要になります。 Windows コンテナー ホスト自体が Hyper-V 仮想マシンの場合、Hyper-V ロールをインストールする前に、入れ子になった仮想化を有効にする必要があります。 入れ子になった仮想化の詳細については、「[入れ子になった仮想化]( https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting)」を参照してください。

### <a name="nested-virtualization"></a>入れ子になった仮想化

次のスクリプトでは、コンテナー ホストの入れ子になった仮想化を構成します。 このスクリプトは親 Hyper-V マシンで実行されます。 このスクリプトを実行する場合は、必ず、コンテナー ホストの仮想マシンを無効にしてください。

```
#replace with the virtual machine name
$vm = "<virtual-machine>"

#configure virtual processor
Set-VMProcessor -VMName $vm -ExposeVirtualizationExtensions $true -Count 2

#disable dynamic memory
Set-VMMemory $vm -DynamicMemoryEnabled $false

#enable mac spoofing
Get-VMNetworkAdapter -VMName $vm | Set-VMNetworkAdapter -MacAddressSpoofing On
```

### <a name="enable-the-hyper-v-role"></a>Hyper-V ロールの有効化

PowerShell を使用してHyper-V 機能を有効にするには、管理者特権の PowerShell セッションで次のコマンドを実行します。

```
Install-WindowsFeature hyper-v
```
