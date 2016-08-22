---
title: "Nano Server での Windows コンテナーの展開"
description: "Nano Server での Windows コンテナーの展開"
keywords: "Docker, コンテナー"
author: neilpeterson
manager: timlt
ms.date: 07/06/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: b82acdf9-042d-4b5c-8b67-1a8013fa1435
translationtype: Human Translation
ms.sourcegitcommit: fac57150de3ffd6c7d957dd628b937d5c41c1b35
ms.openlocfilehash: d2f19e96f06ba18ab7e23e62652f569265c6f43f

---

# コンテナー ホストの展開 - Nano Server

**この記事は暫定的な内容であり、変更される可能性があります。** 

このドキュメントでは、Windows コンテナー機能を使用した基本的な Nano Server の展開を順番に説明します。 これは上級レベルのトピックであり、Windows および Windows コンテナーの概要を理解していることを前提としています。 Windows コンテナーの概要については、「[Windows コンテナー クイック スタート](../quick_start/quick_start.md)」を参照してください。

## Nano Server の準備

次のセクションでは、基本的な Nano Server の構成の展開について具体的に説明します。 Nano Server の展開オプションと構成オプションの詳細については、「Getting Started with Nano Server」 (Nano Server の概要) (https://technet.microsoft.com/en-us/library/mt126167.aspx) を参照してください。

### Nano Server VM を作成する

まず Nano Server 評価版 VHD を[ここ](https://msdn.microsoft.com/en-us/virtualization/windowscontainers/nano_eula)からダウンロードします。 この VHD で仮想マシンを作成し、その仮想マシンを起動します。次に、Hyper-V 接続オプション、または使用中の仮想化プラットフォームに基づく同等の接続オプションを使用して仮想マシンに接続します。

次に、管理者のパスワードを設定する必要があります。 そのためには、Nano Server 回復コンソールで `F11` を押します。 これにより、[パスワードの変更] ダイアログ ボックスが表示されます。

### リモート PowerShell セッションを作成する

Nano Server は対話型のログオン機能を備えていないので、管理はすべてリモート PowerShell セッションから実行されます。 リモート セッションを作成するには、Nano Server 回復コンソールのネットワーキング セクションを使用してシステムの IP アドレスを取得し、リモート ホストで次のコマンドを実行します。 IPADDRESS を Nano Server システムの実際の IP アドレスに置き換えます。

Nano Server システムを信頼できるホストに追加します。

```none
set-item WSMan:\localhost\Client\TrustedHosts IPADDRESS -Force
```

リモート PowerShell セッションを作成します。

```none
Enter-PSSession -ComputerName IPADDRESS -Credential ~\Administrator
```

上記の手順が完了すると、Nano Server システムではリモート PowerShell セッションの状態になります。 このドキュメントの残りの部分では、特に記載のない限り、リモート セッションから作業を行います。


## コンテナー機能のインストール

Nano Server パッケージ管理プロバイダーを使用すると、役割や機能を Nano Server にインストールすることができます。 次のコマンドを使用してプロバイダーをインストールします。

```none
Install-PackageProvider NanoServerPackage
```

パッケージ プロバイダーがインストールされたら、コンテナー機能をインストールします。

```none
Install-NanoServerPackage -Name Microsoft-NanoServer-Containers-Package
```

コンテナー機能がインストールされたら、Nano Server ホストを再起動する必要があります。 

```none
Restart-Computer
```

バックアップが作成されたら、リモート PowerShell 接続を再び確立します。

## Docker のインストール

Docker は Windows コンテナーで使用するために必要です。 Docker は、Docker エンジンと Docker クライアントで構成されます。 次の手順を使用して、Docker エンジンおよびクライアントをインストールします。

Nano Server ホストに Docker 実行可能ファイル用のフォルダーを作成します。

```none
New-Item -Type Directory -Path $env:ProgramFiles'\docker\'
```

Docker エンジンおよびクライアントをダウンロードし、これらをコンテナー ホストの 'C:\Program Files\docker\' にコピーします。 

**注** - Nano Server では現在、`Invoke-WebRequest` がサポートされていないため、リモート システムからダウンロードを実行し、その内容を Nano Server ホストにコピーする必要があります。

```none
Invoke-WebRequest https://aka.ms/tp5/b/dockerd -OutFile .\dockerd.exe
```

Docker クライアントをダウンロードします。

```none
Invoke-WebRequest https://aka.ms/tp5/b/docker -OutFile .\docker.exe
```

Docker エンジンとクライアントがダウンロードされたら、Nano Server コンテナー ホストの 'C:\Program Files\docker\' フォルダーにコピーします。 着信 SMB 接続が許可されるように Nano Server ファイアウォールを構成する必要があります。 この構成は、PowerShell または Nano Server 回復コンソールのどちらでも実行できます。 

```none
Set-NetFirewallRule -Name FPS-SMB-In-TCP -Enabled True
```

これで、SMB ファイルの標準的なコピー方法を使用してファイルをコピーできるようになりました。

dockerd.exe ファイルをホストにコピーし場合は、次のコマンドを実行して Windows サービスとして Docker をインストールします。

```none
& $env:ProgramFiles'\docker\dockerd.exe' --register-service
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

Nano Server ベース イメージに最新のタグを付けます。

```none
& $env:ProgramFiles'\docker\docker.exe' tag nanoserver:10.0.14300.1016 nanoserver:latest
```

## Nano Server の Docker の管理

最適な結果を得るために、またベスト プラクティスとして、リモート システムから Nano Server の Docker を管理します。 そのためには、以下の作業を行う必要があります。

### コンテナー ホストを用意する

Docker 接続用のコンテナー ホストにファイアウォール規則を作成します。 セキュリティで保護されていない接続の場合はポート `2375`、セキュリティで保護されている接続の場合はポート `2376` になります。

```none
netsh advfirewall firewall add rule name="Docker daemon " dir=in action=allow protocol=TCP localport=2376
```

TCP 経由で着信接続を受け入れるように、Docker エンジンを構成します。

最初に、Nano Server ホストの `c:\ProgramData\docker\config\daemon.json` に `daemon.json` ファイルを作成します。

```none
new-item -Type File c:\ProgramData\docker\config\daemon.json
```

次のコマンドを実行して `daemon.json` ファイルに接続構成を追加します。 これにより、TCP ポート 2375 経由で着信接続を受け入れるように、Docker エンジンが構成されます。 これはセキュリティで保護されていない接続であるため、お勧めできませんが、分離されたテストで使用することはできます。 この接続をセキュリティで保護する方法の詳細については、「[Protect the Docker Daemon on Docker.com](https://docs.docker.com/engine/security/https/)」 (Docker.com で Docker デーモンを保護する) を参照してください。

```none
Add-Content 'c:\programdata\docker\config\daemon.json' '{ "hosts": ["tcp://0.0.0.0:2375", "npipe://"] }'
```

Docker サービスを再起動します。

```none
Restart-Service docker
```

### リモート クライアントを準備する

作業するリモート システム上で、Docker クライアントを保持するディレクトリを作成します。

```none
New-Item -Type Directory -Path 'C:\Program Files\docker\'
```

このディレクトリに Docker クライアントをダウンロードします。

```none
Invoke-WebRequest https://aka.ms/tp5/b/docker -OutFile "$env:ProgramFiles\docker\docker.exe"
```

Docker ディレクトリをシステム パスに追加します。

```none
$env:Path += ";$env:ProgramFiles\Docker"
```

変更されたパスが認識されるように、PowerShell またはコマンド セッションを再起動します。

完了したら、`docker -H` パラメーターを使用してリモート Docker ホストにアクセスすることができます。

```none
docker -H tcp://<IPADDRESS>:2375 run -it nanoserver cmd
```

`-H` パラメーター要件は、環境変数 `DOCKER_HOST` を作成して削除することができます。 その場合、次の PowerShell コマンドを使用することができます。

```none
$env:DOCKER_HOST = "tcp://<ipaddress of server>:2375"
```

この変数を設定すると、コマンドは次のようになります。

```none
docker run -it nanoserver cmd
```

## Hyper-V コンテナー ホスト

Hyper-V コンテナーを展開するには、コンテナー ホストで Hyper-V の役割が必要になります。 Hyper-V コンテナーの詳細については、「[Hyper-V コンテナー](../management/hyperv_container.md)」を参照してください。

Windows コンテナー ホスト自体が Hyper-V 仮想マシンの場合、入れ子になった仮想化を有効にする必要があります。 入れ子になった仮想化の詳細については、「[入れ子になった仮想化](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting)」を参照してください。


Nano Server コンテナー ホストに Hyper-V の役割をインストールします。

```none
Install-NanoServerPackage Microsoft-NanoServer-Compute-Package
```

Hyper-V の役割がインストールされたら、Nano Server ホストを再起動する必要があります。

```none
Restart-Computer
```


<!--HONumber=Aug16_HO3-->


