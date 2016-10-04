---
title: "Nano Server での Windows コンテナーの展開"
description: "Nano Server での Windows コンテナーの展開"
keywords: "Docker, コンテナー"
author: neilpeterson
manager: timlt
ms.date: 09/28/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: b82acdf9-042d-4b5c-8b67-1a8013fa1435
translationtype: Human Translation
ms.sourcegitcommit: df9723e3a9d9ada778d01d43dcb36c99813dea8f
ms.openlocfilehash: 9af33e6bce21aa339109f060100b2c7ab3c1eb91

---

# コンテナー ホストの展開 - Nano Server

このドキュメントでは、Windows コンテナー機能を使用した基本的な Nano Server の展開を順番に説明します。 これは上級レベルのトピックであり、Windows および Windows コンテナーの概要を理解していることを前提としています。 Windows コンテナーの概要については、「[Windows コンテナー クイック スタート](../quick_start/quick_start.md)」を参照してください。

## Nano Server の準備

次のセクションでは、基本的な Nano Server の構成の展開について具体的に説明します。 Nano Server の展開オプションと構成オプションの詳細については、「Getting Started with Nano Server」 (Nano Server の概要) (https://technet.microsoft.com/en-us/library/mt126167.aspx) を参照してください。

### Nano Server VM を作成する

まず Nano Server 評価版 VHD を[ここ](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2016)からダウンロードします。 この VHD で仮想マシンを作成し、その仮想マシンを起動します。次に、Hyper-V 接続オプション、または使用中の仮想化プラットフォームに基づく同等の接続オプションを使用して仮想マシンに接続します。

### リモート PowerShell セッションを作成する

Nano Server は対話型のログオン機能を備えていないため、管理はすべて PowerShell を使用してリモート システムから実行されます。

Nano Server システムをリモート システムの信頼できるホストに追加します。 IP アドレスをこの Nano Server の IP アドレスで置き換えます。

```none
Set-Item WSMan:\localhost\Client\TrustedHosts 192.168.1.50 -Force
```

リモート PowerShell セッションを作成します。

```none
Enter-PSSession -ComputerName 192.168.1.50 -Credential ~\Administrator
```

上記の手順が完了すると、Nano Server システムではリモート PowerShell セッションの状態になります。 このドキュメントの残りの部分では、特に記載のない限り、リモート セッションから作業を行います。

### Windows の更新プログラムをインストールする

Windows コンテナー機能が動作するためには、重要な更新プログラムが必要です。 これらの更新プログラムをインストールするには、次のコマンドを実行します。

```none
$sess = New-CimInstance -Namespace root/Microsoft/Windows/WindowsUpdate -ClassName MSFT_WUOperationsSession
Invoke-CimMethod -InputObject $sess -MethodName ApplyApplicableUpdates
```

更新プログラムが適用されたら、システムを再起動します。

```none
Restart-Computer
```

バックアップが作成されたら、リモート PowerShell 接続を再び確立します。

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

Windows コンテナーを使用するには、Docker エンジンが必要です。 次の手順を使用して、Docker エンジンをインストールします。

Docker エンジンとクライアントを 1 つの zip アーカイブとしてダウンロードします。

```none
Invoke-WebRequest "https://download.docker.com/components/engine/windows-server/cs-1.12/docker.zip" -OutFile "$env:TEMP\docker.zip" -UseBasicParsing
```

この zip アーカイブを展開して Program Files に出力します。アーカイブの内容は既に docker ディレクトリに入っています。

```none
Expand-Archive -Path "$env:TEMP\docker.zip" -DestinationPath $env:ProgramFiles
```

Docker ディレクトリを Nano Server のシステム パスに追加します。

```none
# For quick use, does not require shell to be restarted.
$env:path += “;C:\program files\docker”

# For persistent use, will apply even after a reboot.
setx PATH $env:path /M
```

Windows サービスとして Docker をインストールします。

```none
dockerd --register-service
```

Docker サービスを開始します。

```none
Start-Service Docker
```

## コンテナーの基本イメージのインストール

基本 OS イメージは、任意の Windows Server または Hyper-V コンテナーのベースとして使用されます。 基本 OS イメージは、基となるオペレーティング システムとして Windows Server Core と Nano Server の両方で使用でき、`docker pull` を使用してインストールすることができます。 Docker コンテナー イメージの詳細については、「[Build your own images on docker.com](https://docs.docker.com/engine/tutorials/dockerimages/)」(docker.com で独自のイメージを構築する) を参照してください。

Windows Server と Nano Server の基本イメージをダウンロードしてインストールするには、次のコマンドを実行します。

```none
docker pull microsoft/nanoserver
```

```none
docker pull microsoft/windowsservercore
```

> Windows Containers OS Image 使用許諾契約書 (EULA) を参照してください。こちらの「[EULA](../Images_EULA.md)」に掲載されています。

## Nano Server の Docker の管理

最適な結果を得るために、またベスト プラクティスとして、リモート システムから Nano Server の Docker を管理します。 そのためには、以下の作業を行う必要があります。

### コンテナー ホストを用意する

Docker 接続用のコンテナー ホストにファイアウォール規則を作成します。 セキュリティで保護されていない接続の場合はポート `2375`、セキュリティで保護されている接続の場合はポート `2376` になります。

```none
netsh advfirewall firewall add rule name="Docker daemon " dir=in action=allow protocol=TCP localport=2375
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

作業するリモート システム上で、Docker クライアントをダウンロードします。

```none
Invoke-WebRequest "https://download.docker.com/components/engine/windows-server/cs-1.12/docker.zip" -OutFile "$env:TEMP\docker.zip" -UseBasicParsing
```

圧縮されたパッケージを解凍します。

```none
Expand-Archive -Path "$env:TEMP\docker.zip" -DestinationPath $env:ProgramFiles
```

次の 2 つのコマンドを実行して、Docker ディレクトリをシステム パスに追加します。

```none
# For quick use, does not require shell to be restarted.
$env:path += ";c:\program files\docker"

# For persistent use, will apply even after a reboot. 
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files\Docker", [EnvironmentVariableTarget]::Machine)
```

完了したら、`docker -H` パラメーターを使用してリモート Docker ホストにアクセスすることができます。

```none
docker -H tcp://<IPADDRESS>:2375 run -it microsoft/nanoserver cmd
```

`-H` パラメーター要件は、環境変数 `DOCKER_HOST` を作成して削除することができます。 その場合、次の PowerShell コマンドを使用することができます。

```none
$env:DOCKER_HOST = "tcp://<ipaddress of server>:2375"
```

この変数を設定すると、コマンドは次のようになります。

```none
docker run -it microsoft/nanoserver cmd
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



<!--HONumber=Sep16_HO5-->


