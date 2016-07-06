---
title: "Windows Server に Windows コンテナーを展開する"
description: "Windows Server に Windows コンテナーを展開する"
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 05/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: ba4eb594-0cdb-4148-81ac-a83b4bc337bc
translationtype: Human Translation
ms.sourcegitcommit: eae45c2c81c7edc94d963da69dcdee2b6f08f37d
ms.openlocfilehash: cbbff2bf4a68ee348bcc33979ef4469daf54a8a7

---

# コンテナー ホストの展開 - Windows Server

**この記事は暫定的な内容であり、変更される可能性があります。**

Windows コンテナー ホストの展開手順は、オペレーティング システムやホスト システムの種類 (物理または仮想) によって異なります。 このドキュメントでは、物理または仮想システム上で、Windows Server 2016 または Windows Server Core 2016 のいずれかに対して Windows コンテナー ホストを展開する方法を詳しく説明します。

## Azure イメージ 

完全に構成された Windows Server イメージは Azure で利用できます。 このイメージを利用するには、下のボタンをクリックして仮想マシンをデプロイします。 このテンプレートを使用して Windows コンテナー システムを Azure にデプロイする場合は、このドキュメントの以降の項を省略できます。

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2FVirtualization-Documentation%2Fmaster%2Fwindows-server-container-tools%2Fcontainers-azure-template%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

## コンテナー機能のインストール

コンテナー機能は、Windows コンテナーを使用する前に有効にする必要があります。 そのためには、管理者特権の PowerShell セッションで次のコマンドを実行します。

```none
Install-WindowsFeature containers
```

機能のインストールが完了したら、コンピューターを再起動します。

```none
Restart-Computer -Force
```

## Docker のインストール

Docker は Windows コンテナーで使用するために必要です。 Docker は、Docker エンジンと Docker クライアントで構成されます。 この演習では、両方をインストールします。

Docker の実行可能ファイル用のフォルダーを作成します。

```none
New-Item -Type Directory -Path 'C:\Program Files\docker\'
```

Docker デーモンをダウンロードします。

```none
Invoke-WebRequest https://aka.ms/tp5/b/dockerd -OutFile $env:ProgramFiles\docker\dockerd.exe
```

Docker クライアントをダウンロードします。

```none
Invoke-WebRequest https://aka.ms/tp5/b/docker -OutFile $env:ProgramFiles\docker\docker.exe
```

Docker ディレクトリをシステム パスに追加します。

```none
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files\Docker", [EnvironmentVariableTarget]::Machine)
```

変更されたパスが認識されるように、PowerShell セッションを再起動します。

Windows サービスとして Docker をインストールするには、以下を実行します。

```none
dockerd --register-service
```

インストールされたら、サービスを開始することができます。

```none
Start-Service Docker
```

## コンテナーの基本イメージのインストール

コンテナーを展開する前に、コンテナーの基本 OS イメージをダウンロードする必要があります。 次の例では、Windows Server Core のベース OS イメージをダウンロードします。 これと同じ手順を実行して、Nano Server 基本イメージをインストールすることができます。 Windows コンテナー イメージの詳細については、[コンテナー イメージの管理](../management/manage_images.md)に関するページを参照してください。

まず、コンテナー イメージ パッケージ プロバイダーをインストールします。

```none
Install-PackageProvider ContainerImage -Force
```

次に、Windows Server Core のイメージをインストールします。 このプロセスには時間がかかる場合があるため、しばらく待ち、ダウンロードが完了したら作業に戻ってください。

```none
Install-ContainerImage -Name WindowsServerCore    
```

基本イメージがインストールされたら、Docker サービスを再起動する必要があります。

```none
Restart-Service docker
```

最後に、イメージに '最新' バージョンであることを示すタグを付ける必要があります。 これを行うために、次のコマンドを実行します。

```none
docker tag windowsservercore:10.0.14300.1000 windowsservercore:latest
```

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

PowerShell を使用してHyper-V 機能を有効にするには、管理者特権の PowerShell セッションで次のコマンドを実行します。

```none
Install-WindowsFeature hyper-v
```



<!--HONumber=Jun16_HO5-->


