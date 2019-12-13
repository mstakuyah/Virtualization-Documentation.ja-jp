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
ms.openlocfilehash: c84a6652b5918238ee8ef6e1fa7a9b2aa596aefd
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910162"
---
# <a name="docker-engine-on-windows"></a>Windows 上の Docker エンジン

Docker エンジンとクライアントは Windows に含まれていないため、個別にインストールして構成する必要があります。 また、Docker エンジンは多くのカスタム構成に対応できます。 たとえば、デーモンで受信要求を受け入れる方法、既定のネットワークキング オプション、デバッグ/ログの設定の構成などです。 Windows でこのような構成を指定するには、構成ファイルまたは Windows サービス コントロール マネージャーを使用します。 このドキュメントでは、Docker エンジンをインストールして構成する方法について詳しく説明し、一般的に使用される構成の例をいくつか紹介します。

## <a name="install-docker"></a>Docker のインストール

Windows コンテナーを操作するには、Docker が必要です。 Docker は、Docker エンジン (dockerd.exe) と Docker クライアント (docker.exe) で構成されます。 すべてをインストールするための最も簡単な方法は、クイックスタートガイドを参照してください。このガイドは、すべての設定を取得し、最初のコンテナーを実行するのに役立ちます。

- [Docker のインストール](../quick-start/set-up-environment.md)

スクリプト化されたインストールについては、「[スクリプトを使用した DOCKER EE のインストール](https://docs.docker.com/install/windows/docker-ee/#use-a-script-to-install-docker-ee)」を参照してください。

Docker を使用するには、コンテナーイメージをインストールする必要があります。 詳細については、[コンテナーの基本イメージに関するドキュメント](../manage-containers/container-base-images.md)を参照してください。

## <a name="configure-docker-with-a-configuration-file"></a>構成ファイルを使用した Docker の構成

Windows で Docker エンジンを構成するには、構成ファイルを使用する方法がお勧めです。 構成ファイルは、'C:\ProgramData\Docker\config\daemon.json' にあります。 このファイルがまだ存在しない場合は、作成できます。

>[!NOTE]
>使用可能なすべての Docker 構成オプションが Windows 上の Docker に適用されるわけではありません。 次の例は、適用される構成オプションを示しています。 Docker エンジンの構成の詳細については、「 [docker デーモン構成ファイル](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file)」を参照してください。

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

必要な構成の変更を構成ファイルに追加するだけです。 たとえば、次のサンプルでは、ポート2375で受信接続を受け入れるように Docker エンジンを構成しています。 その他のすべての構成オプションには、既定値が使用されます。

```json
{
    "hosts": ["tcp://0.0.0.0:2375"]
}
```

同様に、次のサンプルでは、イメージとコンテナーを代替パスに保持するように Docker デーモンを構成します。 指定しない場合、既定値は `c:\programdata\docker`になります。

```json
{    
    "data-root": "d:\\docker"
}
```

次のサンプルでは、ポート2376経由のセキュリティで保護された接続のみを受け入れるように Docker デーモンを構成します。

```json
{
    "hosts": ["tcp://0.0.0.0:2376", "npipe://"],
    "tlsverify": true,
    "tlscacert": "C:\\ProgramData\\docker\\certs.d\\ca.pem",
    "tlscert": "C:\\ProgramData\\docker\\certs.d\\server-cert.pem",
    "tlskey": "C:\\ProgramData\\docker\\certs.d\\server-key.pem",
}
```

## <a name="configure-docker-on-the-docker-service"></a>Docker サービスで Docker を構成する

Docker エンジンは、`sc config`で Docker サービスを変更することによって構成することもできます。 この方法の場合、Docker エンジンのフラグは Docker サービスに直接設定されます。 コマンド プロンプト (PowerShell ではなく cmd.exe) で、次のコマンドを実行します。

```cmd
sc config docker binpath= "\"C:\Program Files\docker\dockerd.exe\" --run-service -H tcp://0.0.0.0:2375"
```

>[!NOTE]
>このコマンドは、daemon.log ファイルに既に `"hosts": ["tcp://0.0.0.0:2375"]` エントリが含まれている場合には実行する必要はありません。

## <a name="common-configuration"></a>共通の構成

次に、一般的な Docker 構成の構成ファイル例を示します。 これらの構成を 1 つの構成ファイルにまとめることができます。

### <a name="default-network-creation"></a>既定のネットワークの作成

既定の NAT ネットワークを作成しないように Docker エンジンを構成するには、次の構成を使用します。

```json
{
    "bridge" : "none"
}
```

詳細については、「[コンテナーのネットワーク](../container-networking/network-drivers-topologies.md)」を参照してください。

### <a name="set-docker-security-group"></a>Docker セキュリティグループの設定

Docker ホストにサインインし、Docker コマンドをローカルで実行している場合、これらのコマンドは名前付きパイプを介して実行されます。 既定では、Administrators グループのメンバーのみが、名前付きパイプ経由で Docker エンジンにアクセスできます。 このアクセス権を持つセキュリティ グループを指定するには、`group` フラグを使用します。

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

詳細については、 [Docker.com の「Windows 構成ファイル](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file)」を参照してください。

## <a name="how-to-uninstall-docker"></a>Docker をアンインストールする方法

このセクションでは、Docker をアンインストールし、Windows 10 または Windows Server 2016 システムから Docker システムコンポーネントの完全なクリーンアップを実行する方法について説明します。

>[!NOTE]
>この手順では、管理者特権の PowerShell セッションからすべてのコマンドを実行する必要があります。

### <a name="prepare-your-system-for-dockers-removal"></a>Docker の削除用にシステムを準備する

Docker をアンインストールする前に、コンテナーがシステム上で実行されていないことを確認してください。

次のコマンドレットを実行して、実行中のコンテナーを確認します。

```powershell
# Leave swarm mode (this will automatically stop and remove services and overlay networks)
docker swarm leave --force

# Stop all running containers
docker ps --quiet | ForEach-Object {docker stop $_}
```

Docker を削除する前に、すべてのコンテナー、コンテナーイメージ、ネットワーク、およびボリュームをシステムから削除することをお勧めします。 これを行うには、次のコマンドレットを実行します。

```powershell
docker system prune --volumes --all
```

### <a name="uninstall-docker"></a>Docker のアンインストール

次に、Docker を実際にアンインストールする必要があります。

Windows 10 で Docker をアンインストールするには

- Windows 10 コンピューターの **[設定]**  >  **[アプリ]** にアクセス
- **アプリ & 機能** で、検索 **Docker for Windows**
- **Docker for Windows**にアクセス > **アンインストール**

Windows Server 2016 で Docker をアンインストールするには:

管理者特権の PowerShell セッションから、次の例に示すように、 **Uninstall パッケージ**と**uninstall-Module**コマンドレットを使用して、Docker モジュールとそれに対応する Package Management プロバイダーをシステムから削除します。

```powershell
Uninstall-Package -Name docker -ProviderName DockerMsftProvider
Uninstall-Module -Name DockerMsftProvider
```

>[!TIP]
>Docker のインストールに使用したパッケージプロバイダーは、を使用して見つけることができ `PS C:\> Get-PackageProvider -Name *Docker*`

### <a name="clean-up-docker-data-and-system-components"></a>Docker データとシステムコンポーネントをクリーンアップする

Docker をアンインストールした後は、docker の既定のネットワークを削除する必要があります。これにより、Docker が使用されなくなった後も、構成がシステムに残りません。 これを行うには、次のコマンドレットを実行します。

```powershell
Get-HNSNetwork | Remove-HNSNetwork
```

次のコマンドレットを実行して、Docker のプログラムデータをシステムから削除します。

```powershell
Remove-Item "C:\ProgramData\Docker" -Recurse
```

Windows の Docker/コンテナーに関連付けられている Windows のオプション機能を削除することもできます。

これには、Docker をインストールすると、Windows 10 または Windows Server 2016 で自動的に有効になる "コンテナー" 機能が含まれます。 また、"Hyper-V" 機能も含まれる可能性があります。これは、Windows 10 では Docker のインストール時に自動的に有効になりますが、Windows Server 2016 では明示的に有効にする必要がある機能です。

>[!IMPORTANT]
>[Hyper-v 機能](https://docs.microsoft.com/virtualization/hyper-v-on-windows/about/)は、コンテナーだけではなく、一般的な仮想化機能です。 Hyper-v 機能を無効にする前に、Hyper-v を必要とする他の仮想化コンポーネントがシステムに存在しないことを確認してください。

Windows 10 の Windows の機能を削除するには:

- **Windows の機能の有効化または無効化** > には、[**コントロールパネル]** の [**プログラム > プログラム** **と機能**] > ます。
- 無効にする機能の名前 (この場合は、**コンテナー**と (必要に応じて **)) を検索します**。
- 無効にする機能の名前の横にあるチェックボックスをオフにします。
- **[OK]** を選択します。

Windows Server 2016 で Windows の機能を削除するには:

管理者特権の PowerShell セッションから、次のコマンドレットを実行して、**コンテナー**と (必要に応じて) システムの**hyper-v**機能を無効にします。

```powershell
Remove-WindowsFeature Containers
Remove-WindowsFeature Hyper-V
```

### <a name="reboot-your-system"></a>システムを再起動する

アンインストールとクリーンアップを完了するには、管理者特権の PowerShell セッションから次のコマンドレットを実行して、システムを再起動します。

```powershell
Restart-Computer -Force
```
