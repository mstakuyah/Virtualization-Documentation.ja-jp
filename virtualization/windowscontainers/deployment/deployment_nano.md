---
title: "Nano Server での Windows コンテナーの展開"
description: "Nano Server での Windows コンテナーの展開"
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 06/17/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: b82acdf9-042d-4b5c-8b67-1a8013fa1435
ms.sourcegitcommit: 1ba6af300d0a3eba3fc6d27598044f983a4c9168
ms.openlocfilehash: f2790186aa641378b1981a1f946665ca46fdbd73

---

# コンテナー ホストの展開 - Nano Server

**この記事は暫定的な内容であり、変更される可能性があります。** 

Nano Server で Windows コンテナーの構成を開始するには、Nano Server を実行するシステムが必要です。また、このシステムとのリモート PowerShell 接続も必要です。 Nano Server の展開と接続の詳細については、「[Getting Started with Nano Server]( https://technet.microsoft.com/en-us/library/mt126167.aspx)」 (Nano Server の概要) を参照してください。

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

コンテナー機能がインストールされたら、Nano Server ホストを再起動する必要があります。

```none
Restart-Computer
```

## Docker のインストール

Docker は Windows コンテナーで使用するために必要です。 Docker は、Docker エンジンと Docker クライアントで構成されます。 次の手順を使用して、Docker デーモンおよびクライアントをインストールします。

Nano Server ホストに Docker 実行可能ファイル用のフォルダーを作成します。

```none
New-Item -Type Directory -Path 'C:\Program Files\docker\'
```

Docker デーモンおよびクライアントをダウンロードし、これらをコンテナー ホストの 'C:\Program Files\docker\' にコピーします。 

**注** - Nano Server では現在、`Invoke-WebRequest` はサポートされていないため、リモート システムからダウンロードを実行する必要があります。

```none
Invoke-WebRequest https://aka.ms/tp5/b/dockerd -OutFile .\dockerd.exe
```

Docker クライアントをダウンロードします。

```none
Invoke-WebRequest https://aka.ms/tp5/b/docker -OutFile .\docker.exe
```

Docker デーモンおよびクライアントをダウンロードして Nano Server コンテナー ホストにコピーしたら、次のコマンドをホストで実行して Docker を Windows サービスとしてインストールします。

```none
& 'C:\Program Files\docker\dockerd.exe' --register-service
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
& 'C:\Program Files\docker\docker.exe' tag nanoserver:10.0.14300.1016 nanoserver:latest
```

## Nano Server の Docker の管理

最適な結果を得るために、またベスト プラクティスとして、リモート システムから Nano Server の Docker を管理します。 そのためには、以下の作業を行う必要があります。

**Docker デーモンを次のように準備します。**

Docker 接続用のコンテナー ホストにファイアウォール規則を作成します。 セキュリティで保護されていない接続の場合はポート `2375`、セキュリティで保護されている接続の場合はポート `2376` になります。

```none
netsh advfirewall firewall add rule name="Docker daemon " dir=in action=allow protocol=TCP localport=2376
```

TCP 経由で着信接続を受け入れるように、Docker デーモンを構成します。

まず、`c:\ProgramData\docker\config\daemon.json` で `daemon.json` ファイルを作成します。

```none
new-item -Type File c:\ProgramData\docker\config\daemon.json
```

次に、この JSON を構成ファイルにコピーします。 これにより、TCP ポート 2375 経由で着信接続を受け入れるように、Docker デーモンが構成されます。 これはセキュリティで保護されていない接続であるため、お勧めできませんが、分離されたテストで使用することはできます。

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

Docker クライアントを格納するディレクトリを作成します。

```none
New-Item -Type Directory -Path 'C:\Program Files\docker\'
```

リモート管理システムに Docker クライアントをダウンロードします。

```none
Invoke-WebRequest https://aka.ms/tp5/b/docker -OutFile "C:\Program Files\docker\docker.exe"
```

Docker ディレクトリをシステム パスに追加します。

```none
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files\Docker", [EnvironmentVariableTarget]::Machine)
```

変更されたパスが認識されるように、PowerShell またはコマンド セッションを再起動します。

完了したら、`docker -H` パラメーターを使用してリモート Docker ホストにアクセスすることができます。

```none
docker -H tcp://10.0.0.5:2375 run -it nanoserver cmd
```

`-H` パラメーター要件は、環境変数 `DOCKER_HOST` を作成して削除することができます。 その場合、次の PowerShell コマンドを使用することができます。

```none
$env:DOCKER_HOST = "tcp://<ipaddress of server:2375"
```

この変数を設定すると、コマンドは次のようになります。

```none
docker run -it nanoserver cmd
```

## Hyper-V コンテナー ホスト

Hyper-V コンテナーを展開するには、Hyper-V ロールが必要になります。 Hyper-V コンテナーの詳細については、「[Hyper-V コンテナー](../management/hyperv_container.md)」を参照してください。

Windows コンテナー ホスト自体が Hyper-V 仮想マシンの場合、入れ子になった仮想化を有効にする必要があります。 入れ子になった仮想化の詳細については、「[入れ子になった仮想化](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting)」を参照してください。


Hyper-V の役割をインストールします。

```none
Install-NanoServerPackage Microsoft-NanoServer-Compute-Package
```

Hyper-V の役割がインストールされたら、Nano Server ホストを再起動する必要があります。

```none
Restart-Computer
```






<!--HONumber=Jun16_HO3-->


