---
title: Nano Server での Windows コンテナーの展開
description: Nano Server での Windows コンテナーの展開
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 05/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: b82acdf9-042d-4b5c-8b67-1a8013fa1435
---

# コンテナー ホストの展開 - Nano Server

**この記事は暫定的な内容であり、変更される可能性があります。** 

Nano Server で Windows コンテナーの構成を開始するには、Nano Server を実行するシステムが必要です。また、このシステムとのリモート PowerShell 接続も必要です。

Nano Server の展開と接続の詳細については、「[Getting Started with Nano Server]( https://technet.microsoft.com/en-us/library/mt126167.aspx)」 (Nano Server の概要) を参照してください。

Nano Server の評価版については、[こちら](https://msdn.microsoft.com/en-us/virtualization/windowscontainers/nano_eula)を参照してください。

## コンテナー機能のインストール

Nano Server パッケージ管理プロバイダーをインストールします。

```none
Install-PackageProvider NanoServerPackage
```

パッケージ プロバイダーがインストールされたら、コンテナー機能をインストールします。

```none
Install-NanoServerPackage -Name Microsoft-NanoServer-Containers-Package
```

これらの機能がインストールされたら、Nano Server ホストを再起動する必要があります。

## Docker のインストール

Docker は Windows コンテナーで使用するために必要です。 Docker は、Docker エンジンと Docker クライアントで構成されます。 次の手順を使用して、Docker デーモンをインストールします。

Docker デーモンをダウンロードし、コンテナー ホストの `$env:SystemRoot\system32\` にコピーします。 Nano Server では現在、`Invoke-Webrequest` はサポートされていないため、リモート システムからこれを実行する必要があります。

```none
Invoke-WebRequest https://aka.ms/tp5/b/dockerd -OutFile .\dockerd.exe
```

Windows サービスとして Docker をインストールします。

```none
dockerd.exe --register-service
```

Docker サービスを開始します。

```none
Start-Service Docker
```

## コンテナーの基本イメージのインストール

基本 OS イメージは、任意の Windows Server または Hyper-V コンテナーのベースとして使用されます。 基本 OS イメージは、基となるオペレーティング システムとして Windows Server Core と Nano Server の両方で使用でき、コンテナー イメージ プロバイダーを使用してインストールすることができます。 Windows コンテナー イメージの詳細については、[コンテナー イメージの管理](../management/manage_images.md)に関するページを参照してください。

コンテナー イメージ プロバイダーは、次のコマンドを使用してインストールできます。

```none
Install-PackageProvider ContainerImage -Force
```

Nano Server の基本イメージをダウンロードしてインストールするには、次を実行します。

```none
Install-ContainerImage -Name NanoServer
```

**注** - 現時点では、Nano Server コンテナー ホストと互換性があるのは Nano Server 基本イメージのみです。

Docker サービスを再起動します。

```none
Restart-Service Docker
```

最後に、イメージに '最新' バージョンであることを示すタグを付ける必要があります。 これを行うために、次のコマンドを実行します。

```none
docker tag nanoserver:10.0.14300.1010 nanoserver:latest
```

## Hyper-V コンテナー ホスト

Hyper-V コンテナーを展開するには、Hyper-V ロールが必要になります。 Windows コンテナー ホスト自体が Hyper-V 仮想マシンの場合、Hyper-V ロールをインストールする前に、入れ子になった仮想化を有効にする必要があります。 入れ子になった仮想化の詳細については、「入れ子になった仮想化」を参照してください。

### 入れ子になった仮想化

次のスクリプトでは、コンテナー ホストの入れ子になった仮想化を構成します。 このスクリプトは、コンテナー ホストの仮想マシンをホストする Hyper-V マシンで実行します。 このスクリプトを実行する場合は、必ず、コンテナー ホストの仮想マシンを無効にしてください。

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

```none
Install-NanoServerPackage Microsoft-NanoServer-Compute-Package
```

## Nano Server の Docker の管理

**Docker デーモンを次のように準備します。**

最適な結果を得るために、リモート システムから Nano Server の Docker を管理します。 そのためには、以下の作業を行う必要があります。

Docker 接続用のコンテナー ホストにファイアウォール規則を作成します。 セキュリティで保護されていない接続の場合はポート `2375`、セキュリティで保護されている接続の場合はポート `2376` になります。

```none
netsh advfirewall firewall add rule name="Docker daemon " dir=in action=allow protocol=TCP localport=2376
```

TCP 経由で着信接続を受け入れるように、Docker デーモンを構成します。

まず、`c:\ProgramData\docker\config\daemon.json` で `daemon.json` ファイルを作成します。

```none
new-item -Type File c:\ProgramData\docker\config\daemon.json
```

次に、この JSON をファイルにコピーします。 これにより、TCP ポート 2375 経由で着信接続を受け入れるように、Docker デーモンが構成されます。 これはセキュリティで保護されていない接続であるため、お勧めできませんが、分離されたテストで使用することはできます。

```none
{
    "hosts": ["tcp://0.0.0.0:2375", "npipe://"]
}
```

次の例では、セキュリティで保護されたリモート接続を構成します。 TLS 証明書を作成して、適切な場所にコピーする必要があります。 詳細については、「[Docker Daemon on Windows](./docker_windows.md)」 (Windows 上の Docker デーモン) を参照してください。

```none
{
    "hosts": ["tcp://0.0.0.0:2376", "npipe://"],
    "tlsverify": true,
    "tlscacert": "C:\\ProgramData\\docker\\certs.d\\ca.pem",
    "tlscert": "C:\\ProgramData\\docker\\certs.d\\server-cert.pem",
    "tlskey": "C:\\ProgramData\\docker\\certs.d\\server-key.pem",
}
```

Docker サービスを再起動します。

```none
Restart-Service docker
```

**Docker クライアントを次のように準備します。**

リモート管理システムに Docker クライアントをダウンロードします。

```none
Invoke-WebRequest https://aka.ms/tp5/b/docker -OutFile $env:SystemRoot\system32\docker.exe
```

完了したら、`Docker -H` パラメーターを使用して Docker デーモンにアクセスすることができます。

```none
docker -H tcp://10.0.0.5:2376 run -it nanoserver cmd
```

`-H` パラメーター要件は、環境変数 `DOCKER_HOST` を作成して削除することができます。 その場合、次の PowerShell コマンドを使用することができます。

```none
$env:DOCKER_HOST = "tcp://<ipaddress of server:2376"
```

この変数を設定すると、コマンドは次のようになります。

```none
docker run -it nanoserver cmd
```

<!--HONumber=May16_HO5-->


