---
author: neilpeterson
---

# コンテナー イメージ

**この記事は暫定的な内容であり、変更される可能性があります。**

コンテナー イメージは、コンテナーを展開するために使用します。 これらのイメージには、オペレーティング システム、アプリケーション、およびすべてのアプリケーションの依存関係を含めることができます。 たとえば、Nano Server、IIS、および IIS で実行するアプリケーションによって事前に構成されたコンテナー イメージを開発できます。 このコンテナー イメージは、後で使用するためにコンテナー レジストリに保存したり、任意の Windows コンテナー ホスト (オンプレミス、クラウド、またはコンテナー サービスでも) に展開したり、新しいコンテナー イメージのベースとして使用したりすることができます。

2 つの種類のコンテナー イメージがあります。

- **ベース OS イメージ** - これらは Microsoft によって提供され、コア OS コンポーネントが含まれます。
- **コンテナー イメージ** - ベース OS イメージから派生したカスタム コンテナー イメージ。

## ベース OS イメージ

### イメージのインストール

コンテナー OS イメージは、ContainerProvider PowerShell モジュールを使用して、PowerShell 管理と Docker 管理の両方に対して検索およびインストールできます。 このモジュールを使用するには、先にインストールする必要があります。 モジュールをインストールするために、次のコマンドを使用できます。

```powershell
PS C:\> Install-PackageProvider ContainerProvider -Force
```

いったんインストールすると、`Find-ContainerImage` を使用してベース OS イメージの一覧を返せるようになります。

```powershell
PS C:\> Find-ContainerImage

Name                 Version                 Description
----                 -------                 -----------
NanoServer           10.0.10586.0            Container OS Image of Windows Server 2016 Techn...
WindowsServerCore    10.0.10586.0            Container OS Image of Windows Server 2016 Techn...
```

Nano Server ベースの OS イメージをダウンロードしてインストールするには、次を実行します。 `-version` パラメーターは省略可能です。 ベース OS イメージのバージョンを指定しない場合は、最新のバージョンがインストールされます。

```powershell
PS C:\> Install-ContainerImage -Name NanoServer -Version 10.0.10586.0

Downloaded in 0 hours, 0 minutes, 10 seconds.
```

このコマンドは、Windows Server Core ベース OS イメージも同様にダウンロードしてインストールします。 `-version` パラメーターは省略可能です。 ベース OS イメージのバージョンを指定しない場合は、最新のバージョンがインストールされます。

> **問題:** Save-ContainerImage コマンドレットと Install-ContainerImage コマンドレットが、PowerShell リモート セッションで WindowsServerCore コンテナー イメージの操作に失敗する場合があります。 **回避策:** リモート デスクトップを使用してコンピューターにログオンし、直接 Save-ContainerImage コマンドレットを使用します。

```powershell
PS C:\> Install-ContainerImage -Name WindowsServerCore -Version 10.0.10586.0

Downloaded in 0 hours, 2 minutes, 28 seconds.
```

`Get-ContainerImage` コマンドを使用して、イメージがインストールされていることを確認します。

```powershell
PS C:\> Get-ContainerImage

Name              Publisher    Version      IsOSImage
----              ---------    -------      ---------
NanoServer        CN=Microsoft 10.0.10586.0 True
WindowsServerCore CN=Microsoft 10.0.10586.0 True
```

> **Install-ContainerImage** は、PowerShell または Docker の管理されているコンテナーのいずれかで使用するベース OS イメージをインストールします。 ベース OS イメージがダウンロードされているのに `docker images` の実行時に表示されない場合は、[サービス] コントロール パネル アプレットを使用するか、またはコマンド 'sc docker stop' および 'sc docker start' を順番に使用して、Docker サービスを再起動します。

### オフライン インストール

インターネットに接続しないでベース OS イメージをインストールすることもできます。 これを行うには、インターネットに接続されたコンピューターにイメージをダウンロードし、対象のシステムにコピーしてから、`Install-ContainerOSImages` コマンドを使用してインポートします。

ベース OS イメージをダウンロードする前に、次のコマンドを実行して、コンテナー イメージ プロバイダーでシステムを準備します。

```powershell
PS C:\> Install-PackageProvider ContainerProvider -Force
```

PowerShell OneGet パッケージ マネージャーからイメージの一覧が返されます。

```powershell
PS C:\> Find-ContainerImage

Name                 Version                 Description
----                 -------                 -----------
NanoServer           10.0.10586.0            Container OS Image of Windows Server 2016 Techn...
WindowsServerCore    10.0.10586.0            Container OS Image of Windows Server 2016 Techn...
```

イメージをダウンロードするには、`Save-ContainerImage` コマンドを使用します。

```powershell
PS C:\> Save-ContainerImage -Name NanoServer -Destination c:\container-image\NanoServer.wim
```

これで、ダウンロードしたコンテナー イメージを別のコンテナー ホストにコピーし、`Install-ContainerOSImage` コマンドを使用してインストールできるようになりました。

```powershell
Install-ContainerOSImage -WimPath C:\container-image\NanoServer.wim -Force
```

### タグ イメージ

名前でコンテナー イメージを参照すると、Docker エンジンは最新バージョンのイメージを検索します。 最新のバージョンを特定できない場合は、次のエラーがスローされます。

```powershell
PS C:\> docker run -it windowsservercore cmd

Unable to find image 'windowsservercore:latest' locally
Pulling repository docker.io/library/windowsservercore
C:\Windows\system32\docker.exe: Error: image library/windowsservercore not found.
```

Windows Server Core または Nano Server のベース OS イメージのインストール後、これらのイメージには 'latest' バージョンのタグが付けられる必要があります。 これを行うには、`docker tag` コマンドを使用します。

`docker tag` の詳細については、[docker.com の「イメージのタグ付け、プッシュおよびプル」](https://docs.docker.com/mac/step_six/)をご参照ください。

```powershell
PS C:\> docker tag <image id> windowsservercore:latest
```

タグ付けすると、`docker images` の出力により同じイメージの 2 つのバージョンが表示されます。1 つはイメージのバージョンのタグが付けられたもの、もう 1 つは 'latest' のタグが付けられたものです。 これで、イメージを名前で参照できるようになりました。

```powershell
PS C:\> docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nanoserver          10.0.14289.1000     df03a4b28c50        2 days ago          783.2 MB
windowsservercore   10.0.14289.1000     290ab6758cec        2 days ago          9.148 GB
windowsservercore   latest              290ab6758cec        2 days ago          9.148 GB
```

### OS イメージのアンインストール

`Uninstall-ContainerOSImage` コマンドを使用して、ベース OS イメージをアンインストールすることができます。 次の例では、NanoServer のベース OS イメージをアンインストールします。

```powershell
Get-ContainerImage -Name NanoServer | Uninstall-ContainerOSImage
```

## コンテナー イメージの PowerShell

### イメージの一覧表示

`Get-ContainerImage` を実行すると、コンテナー ホスト上のイメージの一覧が返されます。 コンテナー イメージの種類は、`IsOSImage` プロパティを使用する場合に区別されます。

```powershell
PS C:\> Get-ContainerImage

Name                    Publisher       Version         IsOSImage
----                    ---------       -------         ---------
NanoServer              CN=Microsoft    10.0.10586.0    True
WindowsServerCore       CN=Microsoft    10.0.10586.0    True
WindowsServerCoreIIS    CN=Demo         1.0.0.0         False
```

### 新しいイメージの作成

既存のコンテナーから新しいコンテナー イメージを作成できます。 これを行うには、`New-ContainerImage` コマンドを使用します。

```powershell
PS C:\> New-ContainerImage -Container $container -Publisher Demo -Name DemoImage -Version 1.0
```

### イメージの削除

いずれかのコンテナーに、それが停止状態であっても、イメージへの依存関係がある場合は、コンテナー イメージを削除できません。

PowerShell で 1 つのイメージを削除します。

```powershell
PS C:\> Get-ContainerImage -Name newimage | Remove-ContainerImage -Force
```

### イメージの依存関係

新しいイメージが作成されると、それは作成元のイメージに依存するようになります。 この依存関係は、`Get-ContainerImage` コマンドを使用して表示できます。 親イメージが表示されない場合、これはイメージがベース OS イメージであることを示しています。

```powershell
PS C:\> Get-ContainerImage | select Name, ParentImage

Name              ParentImage
----              -----------
NanoServerIIS     ContainerImage (Name = 'NanoServer') [Publisher = 'CN=Microsoft', Version = '10.0.10586.0']
NanoServer
WindowsServerCore
```

### イメージ リポジトリの移動

`New-ContainerImage` コマンドを使用して新しいコンテナー イメージを作成すると、このイメージは既定の場所である 'C:\ProgramData\Microsoft\Windows\Hyper-V\Container Image Store' に保存されます。 `Move-ContainerImageRepository` コマンドを使用して、このリポジトリを移動することができます。 次の例では新しいコンテナー イメージのリポジトリを場所 'c:\container-images' に作成します。

```powershell
Move-ContainerImageRepository -Path c:\container-images
```
> `Move-ContainerImageRepository` コマンドで使用されるパスが、コマンドの実行時に既に存在していてはなりません。

## コンテナー イメージの Docker

### イメージの一覧表示

```powershell
C:\> docker images

REPOSITORY             TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
windowsservercoreiis   latest              ca40b33453f8        About a minute ago   44.88 MB
windowsservercore      10.0.10586.0        6801d964fda5        2 weeks ago          0 B
nanoserver             10.0.10586.0        8572198a60f1        2 weeks ago          0 B
```

### 新しいイメージの作成

既存のコンテナーから新しいコンテナー イメージを作成できます。 これを行うには、`docker commit` コマンドを使用します。 次の例では、‘windowsservercoreiis’ という名前で新しいコンテナー イメージを作成します。

```powershell
C:\> docker commit 475059caef8f windowsservercoreiis

ca40b33453f803bb2a5737d4d5dd2f887d2b2ad06b55ca681a96de8432b5999d
```

### イメージの削除

いずれかのコンテナーに、それが停止状態であっても、イメージへの依存関係がある場合は、コンテナー イメージを削除できません。

docker でイメージを削除する場合、イメージをイメージ名または id で参照できます。

```powershell
C:\> docker rmi windowsservercoreiis

Untagged: windowsservercoreiis:latest
Deleted: ca40b33453f803bb2a5737d4d5dd2f887d2b2ad06b55ca681a96de8432b5999d
```

### イメージの依存関係

Docker によってイメージの依存関係を表示するには、`docker history` コマンドを使用できます。

```powershell
C:\> docker history windowsservercoreiis

IMAGE               CREATED             CREATED BY          SIZE                COMMENT
2236b49aaaef        3 minutes ago       cmd                 171.2 MB
6801d964fda5        2 weeks ago                             0 B
```

### Docker Hub

Docker Hub レジストリには、事前構築されたイメージが格納され、これをコンテナー ホストにダウンロードできます。 これらのイメージがダウンロードされていると、これらを Windows コンテナー アプリケーションのベースとして使用することができます。

Docker Hub から使用できるイメージの一覧を表示するには、`docker search` コマンドを使用します。 注: Windows Serve Core または Nano Server のベース OS イメージをインストールしてからでないと、これらの依存するコンテナー イメージを Docker Hub からプルできません。

> "nano-" で始まるイメージは、Nano Server のベース OS イメージに依存しています。

```powershell
C:\> docker search *

NAME                    DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
microsoft/aspnet        ASP.NET 5 framework installed in a Windows...   1         [OK]       [OK]
microsoft/django        Django installed in a Windows Server Core ...   1                    [OK]
microsoft/dotnet35      .NET 3.5 Runtime installed in a Windows Se...   1         [OK]       [OK]
microsoft/golang        Go Programming Language installed in a Win...   1                    [OK]
microsoft/httpd         Apache httpd installed in a Windows Server...   1                    [OK]
microsoft/iis           Internet Information Services (IIS) instal...   1         [OK]       [OK]
microsoft/mongodb       MongoDB installed in a Windows Server Core...   1                    [OK]
microsoft/mysql         MySQL installed in a Windows Server Core b...   1                    [OK]
microsoft/nginx         Nginx installed in a Windows Server Core b...   1                    [OK]
microsoft/node          Node installed in a Windows Server Core ba...   1                    [OK]
microsoft/php           PHP running on Internet Information Servic...   1                    [OK]
microsoft/python        Python installed in a Windows Server Core ...   1                    [OK]
microsoft/rails         Ruby on Rails installed in a Windows Serve...   1                    [OK]
microsoft/redis         Redis installed in a Windows Server Core b...   1                    [OK]
microsoft/ruby          Ruby installed in a Windows Server Core ba...   1                    [OK]
microsoft/sqlite        SQLite installed in a Windows Server Core ...   1                    [OK]
microsoft/nano-golang   Go Programming Language installed in a Nan...   1                    [OK]
microsoft/nano-httpd    Apache httpd installed in a Nano Server ba...   1                    [OK]
microsoft/nano-iis      Internet Information Services (IIS) instal...   1         [OK]       [OK]
microsoft/nano-mysql    MySQL installed in a Nano Server based con...   1                    [OK]
microsoft/nano-nginx    Nginx installed in a Nano Server based con...   1                    [OK]
microsoft/nano-node     Node installed in a Nano Server based cont...   1                    [OK]
microsoft/nano-python   Python installed in a Nano Server based co...   1                    [OK]
microsoft/nano-rails    Ruby on Rails installed in a Nano Server b...   1                    [OK]
microsoft/nano-redis    Redis installed in a Nano Server based con...   1                    [OK]
microsoft/nano-ruby     Ruby installed in a Nano Server based cont...   1                    [OK]
```

Docker Hub からイメージをダウンロードするには、`docker pull` を使用します。

```powershell
C:\> docker pull microsoft/aspnet

Using default tag: latest
latest: Pulling from microsoft/aspnet
f9e8a4cc8f6c: Pull complete

b71a5b8be5a2: Download complete
```

`docker images` を実行すると、イメージが表示されるようになります。

```powershell
C:\> docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
microsoft/aspnet    latest              b3842ee505e5        5 hours ago         101.7 MB
windowsservercore   10.0.10586.0        6801d964fda5        2 weeks ago         0 B
windowsservercore   latest              6801d964fda5        2 weeks ago         0 B
```






<!--HONumber=Mar16_HO3-->


