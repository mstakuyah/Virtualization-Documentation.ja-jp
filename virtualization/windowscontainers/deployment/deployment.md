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
translationtype: Human Translation
ms.sourcegitcommit: ffdf89b0ae346197b9ae631ee5260e0565261c55
ms.openlocfilehash: 1392cb13798c96f91b7daa036e529c97e560edbb

---

# コンテナー ホストの展開 - Windows Server

Windows コンテナー ホストの展開手順は、オペレーティング システムやホスト システムの種類 (物理または仮想) によって異なります。 このドキュメントでは、物理または仮想システム上で、Windows Server 2016 または Windows Server Core 2016 のいずれかに対して Windows コンテナー ホストを展開する方法を詳しく説明します。

## Docker のインストール

Docker は Windows コンテナーで使用するために必要です。 Docker は、Docker エンジンと Docker クライアントで構成されます。 

Docker をインストールするには、[OneGet プロバイダー PowerShell モジュール](https://github.com/oneget/oneget)を使用します。 プロバイダーは、コンピューターでコンテナー機能を有効にして、Docker をインストールします。これには、再起動が必要です。 

管理者特権の PowerShell セッションを開き、次のコマンドを実行します。

最初に、OneGet PowerShell モジュールをインストールします。

```none
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
```

次に、OneGet を使用して最新バージョンの Docker をインストールします。

```none
Install-Package -Name docker -ProviderName DockerMsftProvider
```

インストールが完了したら、コンピューターを再起動します。

```none
Restart-Computer -Force
```

## コンテナーの基本イメージのインストール

Windows コンテナーを使用する前に、基本イメージをインストールする必要があります。 基本イメージは、基になるオペレーティング システムとして Windows Server Core と Nano Server の両方で使用できます。 Docker コンテナー イメージの詳細については、「[Build your own images on docker.com](https://docs.docker.com/engine/tutorials/dockerimages/)」(docker.com で独自のイメージを構築する) を参照してください。

Windows Server Core 基本イメージをインストールするには、次のコマンドを実行します。

```none
docker pull microsoft/windowsservercore
```

Nano Server 基本イメージをインストールするには、次のコマンドを実行します。

```none
docker pull microsoft/nanoserver
```

> Windows Containers OS Image 使用許諾契約書 (EULA) を参照してください。こちらの「[EULA](../Images_EULA.md)」に掲載されています。

## Hyper-V コンテナー ホスト

Hyper-V コンテナーを実行するには、Hyper-V ロールが必要になります。 Windows コンテナー ホスト自体が Hyper-V 仮想マシンの場合、Hyper-V ロールをインストールする前に、入れ子になった仮想化を有効にする必要があります。 入れ子になった仮想化の詳細については、「[入れ子になった仮想化]( https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting)」を参照してください。

### 入れ子になった仮想化

次のスクリプトでは、コンテナー ホストの入れ子になった仮想化を構成します。 このスクリプトは親 Hyper-V マシンで実行されます。 このスクリプトを実行する場合は、必ず、コンテナー ホストの仮想マシンを無効にしてください。

```none
#replace with the virtual machine name
$vm = "<virtual-machine>"

#configure virtual processor
Set-VMProcessor -VMName $vm -ExposeVirtualizationExtensions $true -Count 2

#disable dynamic memory
Set-VMMemory $vm -DynamicMemoryEnabled $false

#enable mac spoofing
Get-VMNetworkAdapter -VMName $vm | Set-VMNetworkAdapter -MacAddressSpoofing On
```

### Hyper-V ロールの有効化

PowerShell を使用して Hyper-V 機能を有効にするには、管理者特権の PowerShell セッションで次のコマンドを実行します。

```none
Install-WindowsFeature hyper-v
```



<!--HONumber=Oct16_HO4-->


