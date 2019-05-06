---
title: Windows で Docker を構成する
description: Windows で Docker を構成する
keywords: Docker, コンテナー
author: PatrickLang
ms.date: 05/03/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 6885400c-5623-4cde-8012-f6a00019fafa
ms.openlocfilehash: 354469199f3c7e886760e8a391edccde067986af
ms.sourcegitcommit: c48dcfe43f73b96e0ebd661164b6dd164c775bfa
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/06/2019
ms.locfileid: "9610292"
---
# <a name="docker-engine-on-windows"></a>Windows 上の Docker エンジン

Docker エンジンとクライアントは、Windows に含まれていないし、インストールされ、個別に構成する必要があります。 また、Docker エンジンは多くのカスタム構成に対応できます。 たとえば、デーモンで受信要求を受け入れる方法、既定のネットワークキング オプション、デバッグ/ログの設定の構成などです。 Windows でこのような構成を指定するには、構成ファイルまたは Windows サービス コントロール マネージャーを使用します。 このドキュメントは、インストールして、Docker エンジンを構成する方法の詳細も頻繁に使われる構成の例をいくつか用意されています。

## <a name="install-docker"></a>Docker のインストール

Windows コンテナを操作するために Docker する必要があります。 Docker は、Docker エンジン (dockerd.exe) と Docker クライアント (docker.exe) で構成されます。 インストールされているすべてのアイテムを取得する最も簡単な方法が、クイック スタート ガイドで、すべてのアイテムを取得するために役立つを設定して、最初のコンテナーを実行します。

- [Windows Server 2019 の Windows コンテナー](../quick-start/quick-start-windows-server.md)
- [Windows 10 の Windows コンテナー](../quick-start/quick-start-windows-10.md)

スクリプトのインストール[Docker EE をインストールするスクリプトを使用する](https://docs.docker.com/install/windows/docker-ee/#use-a-script-to-install-docker-ee)を参照してください。

Docker を使用するには、コンテナー イメージをインストールする必要があります。 詳細については、[画像を使用するためのクイック スタート ガイド](../quick-start/quick-start-images.md)を参照してください。

## <a name="configure-docker-with-a-configuration-file"></a>構成ファイルと Docker を構成します。

Windows で Docker エンジンを構成するには、構成ファイルを使用する方法がお勧めです。 構成ファイルは、'C:\ProgramData\Docker\config\daemon.json' にあります。 存在しない場合は、このファイルを作成できます。

>[!NOTE]
>すべての利用可能な Docker 構成オプションは、windows Docker に適用されます。 次の例では、適用の設定オプションを示します。 Docker エンジン構成の詳細については、 [Docker デーモン構成ファイル](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file)を参照してください。

```json
{
    "authorization-plugins": [],
    "dns": [],
    "dns-opts": [],
    "dns-search": [],
    "exec-opts": [],
    "storage-driver": "",
    "storage-opts": [],
    "labels": [],
    "log-driver": "", 
    "mtu": 0,
    "pidfile": "",
    "data-root": "",
    "cluster-store": "",
    "cluster-advertise": "",
    "debug": true,
    "hosts": [],
    "log-level": "",
    "tlsverify": true,
    "tlscacert": "",
    "tlscert": "",
    "tlskey": "",
    "group": "",
    "default-ulimits": {},
    "bridge": "",
    "fixed-cidr": "",
    "raw-logs": false,
    "registry-mirrors": [],
    "insecure-registries": [],
    "disable-legacy-registry": false
}
```

のみ構成ファイルに必要な変更を追加する必要があります。 たとえば、次の例では、ポート 2375 での着信接続を受け入れるには、Docker エンジンを構成します。 その他のすべての構成オプションには、既定値が使用されます。

```json
{
    "hosts": ["tcp://0.0.0.0:2375"]
}
```

同様に、次の例では、代替パスに画像およびコンテナーを常に Docker デーモンを構成します。 既定では指定しない場合、`c:\programdata\docker`します。

```json
{    
    "data-root": "d:\\docker"
}
```

次の例では、のみポート 2376 経由でセキュリティで保護された接続を許可する Docker デーモンを構成します。

```json
{
    "hosts": ["tcp://0.0.0.0:2376", "npipe://"],
    "tlsverify": true,
    "tlscacert": "C:\\ProgramData\\docker\\certs.d\\ca.pem",
    "tlscert": "C:\\ProgramData\\docker\\certs.d\\server-cert.pem",
    "tlskey": "C:\\ProgramData\\docker\\certs.d\\server-key.pem",
}
```

## <a name="configure-docker-on-the-docker-service"></a>Docker Docker サービスを構成します。

Docker サービスを変更することによって、Docker エンジンを構成することも`sc config`します。 この方法の場合、Docker エンジンのフラグは Docker サービスに直接設定されます。 コマンド プロンプト (PowerShell ではなく cmd.exe) で、次のコマンドを実行します。

```cmd
sc config docker binpath= "\"C:\Program Files\docker\dockerd.exe\" --run-service -H tcp://0.0.0.0:2375"
```

>[!NOTE]
>Daemon.json ファイルが既に含まれている場合は、このコマンドを実行する必要はありませんが、`"hosts": ["tcp://0.0.0.0:2375"]`入力します。

## <a name="common-configuration"></a>一般的な構成

次に、一般的な Docker 構成の構成ファイル例を示します。 これらの構成を 1 つの構成ファイルにまとめることができます。

### <a name="default-network-creation"></a>既定のネットワークの作成

既定 NAT ネットワークを作成していないように Docker エンジンを構成するには、次の構成を使用します。

```json
{
    "bridge" : "none"
}
```

詳細については、「[コンテナーのネットワーク](../container-networking/network-drivers-topologies.md)」を参照してください。

### <a name="set-docker-security-group"></a>Docker セキュリティ グループを設定します。

Docker ホストにサインインして Docker コマンドを実行しているローカルにすると、名前付きパイプでこれらのコマンドが実行されます。 既定では、Administrators グループのメンバーのみが、名前付きパイプ経由で Docker エンジンにアクセスできます。 このアクセス権を持つセキュリティ グループを指定するには、`group` フラグを使用します。

```json
{
    "group" : "docker"
}
```

## <a name="proxy-configuration"></a>プロキシの構成

`docker search` と `docker pull` のプロキシ情報を設定するには、`HTTP_PROXY` または `HTTPS_PROXY` という名前と、プロキシ情報の値を使用して Windows 環境変数を作成します。 そのためには、PowerShell で次のようなコマンドを実行します。

```powershell
[Environment]::SetEnvironmentVariable("HTTP_PROXY", "http://username:password@proxy:port/", [EnvironmentVariableTarget]::Machine)
```

変数が設定されたら、Docker サービスを再起動します。

```powershell
Restart-Service docker
```

詳細については、 [Windows Docker.com 構成ファイル](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file)を参照してください。

## <a name="how-to-uninstall-docker"></a>Docker をアンインストールする方法

このセクションでは、Windows 10 または Windows Server 2016 システムから Docker システム コンポーネントの完全なクリーンアップを行い、Docker をアンインストールする方法を説明します。

>[!NOTE]
>管理者特権の PowerShell セッションから次の手順では、すべてのコマンドを実行する必要があります。

### <a name="prepare-your-system-for-dockers-removal"></a>Docker の削除、システムを準備します。

Docker をアンインストールする前に、システムでコンテナーが実行されていないことを確認します。

コンテナーを実行するためのチェックには、次のコマンドレットを実行します。

```powershell
# Leave swarm mode (this will automatically stop and remove services and overlay networks)
docker swarm leave --force

# Stop all running containers
docker ps --quiet | ForEach-Object {docker stop $_}
```

Docker を削除する前に、システムからすべてのコンテナー、コンテナーの画像、ネットワーク、およびボリュームを削除することをお勧めします。 次のコマンドレットを実行して、これを行うことができます。

```powershell
docker system prune --volumes --all
```

### <a name="uninstall-docker"></a>Docker のアンインストール

次に、実際に Docker をアンインストールする必要があります。

Windows 10 で Docker をアンインストールするには

- [**設定** > Windows 10 のコンピューター上の**アプリ**
- [**アプリの & 機能**、 **Docker for Windows**を見つける
- **Windows 版の Docker**に > **をアンインストールします。**

Windows Server 2016 で Docker をアンインストールするには。

管理者特権 PowerShell セッションをからには、次の例のように Docker モジュールと対応するパッケージ管理プロバイダーをシステムから削除するのには、**アンインストール パッケージ**と**アンインストール モジュール**のコマンドレットを使用します。

```powershell
Uninstall-Package -Name docker -ProviderName DockerMsftProvider
Uninstall-Module -Name DockerMsftProvider
```

>[!TIP]
>Docker をインストールするために使用するパッケージ プロバイダーを見つけてできます。 `PS C:\> Get-PackageProvider -Name *Docker*`

### <a name="clean-up-docker-data-and-system-components"></a>Docker データとシステム コンポーネントをクリーンアップします。

Docker をアンインストールした後、ため、Docker が削除した後、システムで、構成してもされません Docker の既定のネットワークを削除する必要があります。 次のコマンドレットを実行して、これを行うことができます。

```powershell
Get-HNSNetwork | Remove-HNSNetwork
```

システムから Docker のプログラムのデータを削除するのには、次のコマンドレットを実行します。

```powershell
Remove-Item "C:\ProgramData\Docker" -Recurse
```

Windows 上の Docker/コンテナーに関連付けられている Windows オプション機能も必要に応じて削除します。

これには、Docker がインストールされているときに、Windows 10 または Windows Server 2016 で自動的に有効に、「コンテナー」機能が含まれます。 また、"Hyper-V" 機能も含まれる可能性があります。これは、Windows 10 では Docker のインストール時に自動的に有効になりますが、Windows Server 2016 では明示的に有効にする必要がある機能です。

>[!IMPORTANT]
>[[HYPER-V 機能](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/about/)は、コンテナーだけでなくできるようにする一般的な仮想化機能です。 HYPER-V 機能を無効にする前に、システムでは、その他に HYPER-V を必要と仮想化されたコンポーネントがないになっていることを確認します。

Windows 10 の Windows の機能を削除するには。

- **コントロール パネル]** を [ > **プログラム** > **プログラムと機能** > **Windows の機能のオンまたはオフに**します。
- 機能または無効に機能の名前を検索する: この場合は**コンテナー**および (オプション) **HYPER-V**します。
- 無効に機能の名前の横にあるボックスをオフにします。
- **"OK"** を選択します。

Windows Server 2016 で Windows の機能を削除するには。

管理者特権 PowerShell セッションを**コンテナー**と (省略可能) システムから**HYPER-V**機能を無効にするのには、次のコマンドレットを実行します。

```powershell
Remove-WindowsFeature Containers
Remove-WindowsFeature Hyper-V
```

### <a name="reboot-your-system"></a>コンピューターを再起動します。

アンインストールとクリーンアップを完了するには、するには、管理者特権 PowerShell セッション、コンピューターを再起動してから、次のコマンドレットを実行します。

```powershell
Restart-Computer -Force
```
