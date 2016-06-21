---
title: Windows での Docker の展開
description: Windows での Docker の展開
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 05/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bdfa4545-2291-4827-8165-2d6c98d72d37
---

# Docker と Windows

**この記事は暫定的な内容であり、変更される可能性があります。** 

Docker エンジンは Windows に含まれていないため、個別にインポートして構成する必要があります。 Windows で Docker エンジンを実行するために使用する手順は、Linux で実行する場合の手順とは異なります。 このドキュメントでは、Windows Server 2016、Nano Server、および Windows クライアントで Docker エンジンをインストールして構成する手順について説明します。 Docker エンジンとコマンド ライン インターフェイスは最近 2 つのファイルに分割されたことにも注意してください。 このドキュメントには、両方をインストールするための手順が含まれています。

Docker と Docker ツールセットの詳細については、[Docker.com](https://www.docker.com/) を参照してください。 

> Docker を使用して Windows コンテナーの作成と管理を行うには、先に Windows コンテナー機能を有効にする必要があります。 この機能を有効にする手順については、[コンテナー ホストの展開に関するガイド](./docker_windows.md)を参照してください。

## Windows Server 2016

### Docker デーモンのインストール <!--1-->

`https://aka.ms/tp5/dockerd` から dockerd.exe をダウンロードし、コンテナー ホストの System32 ディレクトリ内に配置します。

```none
wget https://aka.ms/tp5/dockerd -OutFile $env:SystemRoot\system32\dockerd.exe
```

`c:\programdata\docker` という名前のディレクトリを作成します。 このディレクトリ内に、`runDockerDaemon.cmd` という名前のファイルを作成します。

```none
New-Item -ItemType File -Path C:\ProgramData\Docker\runDockerDaemon.cmd -Force
```

次のテキストを `runDockerDaemon.cmd` ファイルにコピーします。

```none
@echo off
set certs=%ProgramData%\docker\certs.d

if exist %ProgramData%\docker (goto :run)
mkdir %ProgramData%\docker

:run
if exist %certs%\server-cert.pem (if exist %ProgramData%\docker\tag.txt (goto :secure))

if not exist %systemroot%\system32\dockerd.exe (goto :legacy)

dockerd -H npipe:// 
goto :eof

:legacy
docker daemon -H npipe:// 
goto :eof

:secure
if not exist %systemroot%\system32\dockerd.exe (goto :legacysecure)
dockerd -H npipe:// -H 0.0.0.0:2376 --tlsverify --tlscacert=%certs%\ca.pem --tlscert=%certs%\server-cert.pem --tlskey=%certs%\server-key.pem
goto :eof

:legacysecure
docker daemon -H npipe:// -H 0.0.0.0:2376 --tlsverify --tlscacert=%certs%\ca.pem --tlscert=%certs%\server-cert.pem --tlskey=%certs%\server-key.pem
```
[https://nssm.cc/release/nssm-2.24.zip](https://nssm.cc/release/nssm-2.24.zip) から nssm.exe をダウンロードします。

```none
wget https://nssm.cc/release/nssm-2.24.zip -OutFile $env:ALLUSERSPROFILE\nssm.zip
```

圧縮されたパッケージを解凍します。

```none
Expand-Archive -Path $env:ALLUSERSPROFILE\nssm.zip $env:ALLUSERSPROFILE
```

`nssm-2.24\win64\nssm.exe` を `c:\windows\system32` ディレクトリにコピーします。

```none
Copy-Item $env:ALLUSERSPROFILE\nssm-2.24\win64\nssm.exe $env:SystemRoot\system32
```
`nssm install` を実行し、Docker サービスを構成します。

```none
start-process nssm install
```

次のデータを NSSM サービスのインストーラー内の対応するフィールドに入力します。

[アプリケーション] タブ:

**パス:** C:\Windows\System32\cmd.exe

**スタートアップ ディレクトリ:** C:\Windows\System32

**引数:** /s /c C:\ProgramData\docker\runDockerDaemon.cmd < nul

**サービス名:** Docker

![](media/nssm1.png)

[詳細] タブ

**表示名:** Docker

**説明:** Docker デーモンは、コンテナーの管理機能を docker クライアントに提供します。

![](media/nssm2.png)

[IO] タブ:

**出力 (StdOut):** C:\ProgramData\docker\daemon.log

**エラー (StdErr):** C:\ProgramData\docker\daemon.log

![](media/nssm3.png)

完了したら、`Install Service` ボタンをクリックします。

これで Docker デーモンが Windows サービスとして構成されました。

### ファイアウォール <!--1-->

リモート Docker 管理を有効にする場合は、TCP ポート 2376 を開く必要もあります。

```none
netsh advfirewall firewall add rule name="Docker daemon " dir=in action=allow protocol=TCP localport=2376
```

### Docker の削除 <!--1-->

次のコマンドでは、Docker サービスを削除します。

```none
sc.exe delete Docker
```

### Docker CLI のインストール

`https://aka.ms/tp5/docker` から docker.exe をダウンロードし、コンテナー ホストの System32 ディレクトリまたは、Docker コマンドを実行する他のシステムに配置します。

```none
wget https://aka.ms/tp5/docker -OutFile $env:SystemRoot\system32\docker.exe
```

## Nano Server

### Docker のインストール <!--2-->

`https://aka.ms/tp5/dockerd` から dockerd.exe をダウンロードし、Nano Server コンテナー ホストの `windows\system32` フォルダーにコピーします。

`c:\programdata\docker` という名前のディレクトリを作成します。 このディレクトリ内に、`runDockerDaemon.cmd` という名前のファイルを作成します。

```none
New-Item -ItemType File -Path C:\ProgramData\Docker\runDockerDaemon.cmd -Force
```

次のテキストを `runDockerDaemon.cmd` ファイルにコピーします。

```none
@echo off
set certs=%ProgramData%\docker\certs.d

if exist %ProgramData%\docker (goto :run)
mkdir %ProgramData%\docker

:run
if exist %certs%\server-cert.pem (if exist %ProgramData%\docker\tag.txt (goto :secure))

if not exist %systemroot%\system32\dockerd.exe (goto :legacy)

dockerd -H npipe:// 
goto :eof

:legacy
docker daemon -H npipe:// 
goto :eof

:secure
if not exist %systemroot%\system32\dockerd.exe (goto :legacysecure)
dockerd -H npipe:// -H 0.0.0.0:2376 --tlsverify --tlscacert=%certs%\ca.pem --tlscert=%certs%\server-cert.pem --tlskey=%certs%\server-key.pem
goto :eof

:legacysecure
docker daemon -H npipe:// -H 0.0.0.0:2376 --tlsverify --tlscacert=%certs%\ca.pem --tlscert=%certs%\server-cert.pem --tlskey=%certs%\server-key.pem
```

次のスクリプトを使用して、Windows の起動時に Docker デーモンを起動するようにスケジュールされたタスクを作成することができます。

```none
# Creates a scheduled task to start docker.exe at computer start up.

$dockerData = "$($env:ProgramData)\docker"
$dockerDaemonScript = "$dockerData\runDockerDaemon.cmd"
$dockerLog = "$dockerData\daemon.log"
$action = New-ScheduledTaskAction -Execute "cmd.exe" -Argument "/c $dockerDaemonScript > $dockerLog 2>&1" -WorkingDirectory $dockerData
$trigger = New-ScheduledTaskTrigger -AtStartup
$settings = New-ScheduledTaskSettingsSet -Priority 5
Register-ScheduledTask -TaskName Docker -Action $action -Trigger $trigger -Settings $settings -User SYSTEM -RunLevel Highest | Out-Null
Start-ScheduledTask -TaskName Docker 
```

### ファイアウォール <!--2-->

リモート Docker 管理を有効にする場合は、TCP ポート 2376 を開く必要もあります。

```none
netsh advfirewall firewall add rule name="Docker daemon " dir=in action=allow protocol=TCP localport=2376
```

### 対話型 Nano セッション

Nano Server は、リモート PowerShell セッションを介して管理されます。 Nano Server のリモート管理の詳細については、[Nano Server の概要]( https://technet.microsoft.com/en-us/library/mt126167.aspx#bkmk_ManageRemote)に関するページを参照してください。

'docker attach' などの一部の docker 操作は、このリモート PowerShell セッションを介して実行することはできません。 この回避方法として、一般的なベスト プラクティスは、セキュリティで保護された TCP 接続を介してリモート クライアントから Docker を管理することです。

これを行うには、TCP ポートでリッスンするように Docker デーモンが構成されており、Docker コマンド ライン インターフェイスがリモート クライアント マシンで使用可能であることを確認します。 構成されている場合は、-H パラメーターを指定して docker コマンドをホストに発行することができます。 リモート システムからの Docker デーモンへのアクセスの詳細については、[Docker.com のデーモン ソケット オプション](https://docs.docker.com/engine/reference/commandline/daemon/#daemon-socket-option)に関するページを参照してください。

リモートでコンテナーを展開し、対話型セッションを入力するには、次のコマンドを実行します。

```none
docker -H tcp://<ipaddress of server>:2376 run -it nanoserver cmd
```

-H パラメーター要件は、環境変数 DOCKER_HOST を作成して削除することができます。 その場合、次の PowerShell コマンドを使用することができます。

```none
$env:DOCKER_HOST = "tcp://<ipaddress of server:2376"
```

この変数を設定すると、コマンドは次のようになります。

```none
docker run -it nanoserver cmd
```

### Docker の削除 <!--2-->

Nano Server から docker デーモンと cli を削除するには、Windows\system32 ディレクトリから `docker.exe` を削除します。

```none
Remove-Item $env:SystemRoot\system32\docker.exe
``` 

Docker のスケジュールされたタスクの登録を解除するには、次を実行します。

```none
Get-ScheduledTask -TaskName Docker | UnRegister-ScheduledTask
```

### Docker CLI のインストール

`https://aka.ms/tp5/docker` から docker.exe をダウンロードして、Nano Server コンテナー ホストの windows\system32 フォルダーにコピーします。

```none
wget https://aka.ms/tp5/docker -OutFile $env:SystemRoot\system32\docker.exe
```

## Docker のスタートアップの構成

Docker デーモンでは、複数のスタートアップ オプションを使用できます。 このセクションでは、Windows の Docker デーモンに関するものをいくつか詳しく説明します。 すべてのデーモン オプションの詳細については、[docker.com の Docker デーモンに関するドキュメント]( https://docs.docker.com/engine/reference/commandline/daemon/)を参照してください。

### TCP ポートのリッスン

名前付きパイプを介してローカルで、または TCP 接続を介してリモートで着信接続をリッスンするように Docker デーモンを構成することができます。 既定のスタートアップ動作では、名前付きパイプのみでリッスンするため、リモート接続を防ぐことができます。

```none
docker daemon -D
```

これを、以下のスタートアップ コマンドを使用して、セキュリティで保護された着信接続をリッスンするように変更することができます。 接続のセキュリティ保護の詳細については、[docker.com のセキュリティ構成に関するドキュメント](https://docs.docker.com/engine/security/https/)を参照してください。

```none
docker daemon -D -H npipe:// -H tcp://0.0.0.0:2376 --tlsverify --tlscacert=%certs%\ca.pem --tlscert=%certs%\server-cert.pem --tlskey=%certs%\server-key.pem
``` 

### 名前付きパイプへのアクセス

コンテナー ホストでローカルで実行される Docker コマンドは、名前付きパイプを介して受信されます。 これらのコマンドを実行するには、管理アクセス権が必要です。 名前付きパイプへのアクセス権を持つグループを指定することもできます。 以下の例では、`docker` という名前の Windows グループにこのアクセス権が与えられています。

```none
dockerd -H npipe:// -G docker
```  


### 既定のランタイム

Windows コンテナーには、Windows Server と Hyper-V という 2 つの異なるランタイム型があります。 Docker デーモンは既定で Windows Server ランタイムを使用するに構成されていますが、これは変更できます。 既定のランタイムとして Hyper-V を設定するには、Docker デーモンを初期化する際に '—exec-opt isolation=hyperv' を指定します。

```none
docker daemon -D —exec-opt isolation=hyperv
```



<!--HONumber=May16_HO5-->


