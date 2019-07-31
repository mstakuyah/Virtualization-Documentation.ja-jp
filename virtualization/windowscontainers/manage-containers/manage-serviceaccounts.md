---
title: Windows コンテナーのグループ管理サービスアカウント
description: Windows コンテナーのグループ管理サービスアカウント
keywords: docker、コンテナー、active directory、gmsa
author: rpsqrd
ms.date: 06/12/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: b908a35f63b2f25da3fb19c0f96b55fe3e513350
ms.sourcegitcommit: c4a3f88d1663dd19336bfd4ede0368cb18550ac7
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/31/2019
ms.locfileid: "9883175"
---
# <a name="group-managed-service-accounts-for-windows-containers"></a>Windows コンテナーのグループ管理サービスアカウント

Windows ベースのネットワークでは、一般に Active Directory (AD) を使用して、ユーザー、コンピューター、その他のネットワークリソース間の認証と承認を容易にします。 エンタープライズアプリケーションの開発者は、アプリが、ドメインに参加しているサーバーで動作するように設計されているため、統合された Windows 認証を利用できるようになっています。これにより、ユーザーや他のサービスが、自動的に透過的にサインインすることが簡単になります。id を持つアプリケーション。

Windows コンテナーはドメインに参加することはできませんが、Active Directory ドメイン id を使って、さまざまな認証シナリオをサポートすることができます。

これを実現するために、[グループ管理サービスアカウント](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview)(gMSA) で実行するように windows コンテナーを構成できます。これは、windows Server 2012 で導入された特別な種類のサービスアカウントであり、複数のコンピューターで id を共有する必要はありません。を選び、パスワードを確認します。

GMSA を使ってコンテナーを実行すると、コンテナーホストは Active Directory ドメインコントローラーから gMSA パスワードを取得し、それをコンテナーインスタンスに提供します。 コンピューターアカウント (システム) がネットワークリソースにアクセスする必要がある場合、コンテナーは gMSA の資格情報を使用します。

この記事では、Windows コンテナーで Active Directory グループ管理サービスアカウントの使用を開始する方法について説明します。

## <a name="prerequisites"></a>前提条件

グループ管理サービスアカウントを使用して Windows コンテナーを実行するには、次のものが必要です。

- Windows Server 2012 以降を実行している1つ以上のドメインコントローラーを持つ Active Directory ドメイン。 GMSAs を使用するためのフォレストまたはドメインの機能レベルの要件はありませんが、gMSA パスワードは Windows Server 2012 以降を実行しているドメインコントローラーでのみ配布できます。 詳細については、「 [gMSAs の Active Directory の要件](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/getting-started-with-group-managed-service-accounts#BKMK_gMSA_Req)」を参照してください。
- GMSA アカウントを作成するためのアクセス許可。 GMSA アカウントを作成するには、ドメイン管理者であるか、または "*グループの作成*" という権限の作成を委任されたアカウントを使用している必要があります。
- CredentialSpec PowerShell モジュールをダウンロードするには、インターネットにアクセスしてください。 接続されていない環境で作業している場合は、インターネットアクセスのあるコンピューターに[モジュールを保存](https://docs.microsoft.com/powershell/module/powershellget/save-module?view=powershell-5.1)して、開発用コンピューターまたはコンテナーのホストにコピーできます。

## <a name="one-time-preparation-of-active-directory"></a>Active Directory の1回限りの準備

ドメインで gMSA をまだ作成していない場合は、キー配布サービス (KDS) のルートキーを生成する必要があります。 KDS は、承認されたホストへの gMSA パスワードの作成、回転、解放を担当します。 コンテナーホストが gMSA を使ってコンテナーを実行する必要がある場合は、現在のパスワードを取得するために KDS に連絡します。

KDS ルートキーが既に作成されているかどうかを確認するには、次の PowerShell コマンドレットを、ドメインコントローラーまたはドメインメンバーのドメイン管理者として実行します。これには、AD PowerShell ツールがインストールされています。

```powershell
Get-KdsRootKey
```

コマンドがキー ID を返す場合は、すべて設定されています。また、「[グループ管理サービスアカウントを作成](#create-a-group-managed-service-account)する」セクションに進むこともできます。 それ以外の場合は、続行して KDS ルートキーを作成します。

複数のドメインコントローラーを含む運用環境またはテスト環境では、KDS ルートキーを作成するために、PowerShell の次のコマンドレットをドメイン管理者として実行します。

```powershell
# For production environments
Add-KdsRootKey -EffectiveImmediately
```

このコマンドは、キーをすぐに有効にすることを意味しますが、KDS ルートキーが複製され、すべてのドメインコントローラーで使用できるようになるまで10時間待つ必要があります。

ドメインにドメインコントローラーが1つしかない場合は、10時間前に有効にするようにキーを設定することで、プロセスを迅速化できます。

>[!IMPORTANT]
>この手法は運用環境では使わないでください。

```powershell
# For single-DC test environments ONLY
Add-KdsRootKey -EffectiveTime (Get-Date).AddHours(-10)
```

## <a name="create-a-group-managed-service-account"></a>グループ管理サービスアカウントを作成する

統合 Windows 認証を使うすべてのコンテナーには、少なくとも1つの gMSA が必要です。 プライマリ gMSA は、システムまたはネットワークサービスとして実行されているアプリがネットワーク上のリソースにアクセスするたびに使用されます。 GMSA の名前は、コンテナーに割り当てられているホスト名に関係なく、ネットワーク上のコンテナーの名前になります。 コンテナーコンピューターアカウントとは別の id としてコンテナーでサービスまたはアプリケーションを実行する場合は、コンテナーを追加の gMSAs 構成することもできます。

GMSA を作成する場合は、複数の異なるコンピューターで同時に使用できる共有 id も作成します。 GMSA パスワードへのアクセスは、Active Directory のアクセス制御リストで保護されています。 パスワードへのアクセスを制限するには、各 gMSA account のセキュリティグループを作成し、関連するコンテナーホストをセキュリティグループに追加することをお勧めします。

また、コンテナーではサービスプリンシパル名 (SPN) が自動的に登録されないため、少なくとも gMSA アカウント用のホスト SPN を手動で作成する必要があります。

通常、ホストまたは http SPN は、gMSA アカウントと同じ名前を使って登録されますが、クライアントがコンテナー内のアプリケーションにアクセスするときに、ロードバランサーの背後または gMSA 名とは異なる DNS 名を使用している場合は、別のサービス名の使用が必要になることがあります。

たとえば、gMSA アカウントが "WebApp01" という名前で、ユーザーがそのサイト`mysite.contoso.com`にアクセスする場合は、gMSA `http/mysite.contoso.com`アカウントで SPN を登録する必要があります。

アプリケーションによっては、固有のプロトコル用に追加の Spn が必要になることがあります。 たとえば、SQL Server には`MSSQLSvc/hostname` SPN が必要です。

次の表では、gMSA を作成するために必要な属性を示します。

|gMSA プロパティ | 必須の値 | 例 |
|--------------|----------------|--------|
|名前 | 任意の有効なアカウント名。 | `WebApp01` |
|DnsHostName | アカウント名に追加されたドメイン名。 | `WebApp01.contoso.com` |
|ServicePrincipalNames | 少なくともホストの SPN を設定し、必要に応じて他のプロトコルを追加します。 | `'host/WebApp01', 'host/WebApp01.contoso.com'` |
|PrincipalsAllowedToRetrieveManagedPassword | コンテナーホストが含まれているセキュリティグループ。 | `WebApp01Hosts` |

GMSA の名前を決めたら、PowerShell で次のコマンドレットを実行して、セキュリティグループと gMSA を作成します。

> [!TIP]
> 以下のコマンドを実行するには、**ドメイン管理者**セキュリティグループに属しているか、または、 **GroupManagedServiceAccount オブジェクトの作成**アクセス許可を委任されているアカウントを使用する必要があります。
> [新しい-ADServiceAccount](https://docs.microsoft.com/powershell/module/addsadministration/new-adserviceaccount?view=win10-ps)コマンドレットは、[リモートサーバー管理ツール](https://aka.ms/rsat)の AD PowerShell ツールに含まれています。

```powershell
# Replace 'WebApp01' and 'contoso.com' with your own gMSA and domain names, respectively

# To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
# To install the AD module on Windows 10 version 1809 or later, run Install-WindowsCapability -Online 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
# To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

# Create the security group
New-ADGroup -Name "WebApp01 Authorized Hosts" -SamAccountName "WebApp01Hosts" -GroupScope DomainLocal

# Create the gMSA
New-ADServiceAccount -Name "WebApp01" -DnsHostName "WebApp01.contoso.com" -ServicePrincipalNames "host/WebApp01", "host/WebApp01.contoso.com" -PrincipalsAllowedToRetrieveManagedPassword "WebApp01Hosts"

# Add your container hosts to the security group
Add-ADGroupMember -Identity "WebApp01Hosts" -Members "ContainerHost01", "ContainerHost02", "ContainerHost03"
```

開発環境、テスト環境、および運用環境用に、個別の gMSA アカウントを作成することをお勧めします。

## <a name="prepare-your-container-host"></a>コンテナーホストを準備する

GMSA で Windows コンテナーを実行する各コンテナーホストは、ドメインに参加しており、gMSA パスワードを取得するためのアクセス権を持っている必要があります。

1. Active Directory ドメインにコンピューターを参加します。
2. ホストが gMSA パスワードへのアクセスを制御するセキュリティグループに属していることを確認します。
3. 新しいグループメンバーシップが取得されるように、コンピューターを再起動します。
4. Windows [Server 用](https://docs.docker.com/install/windows/docker-ee/)の windows 10 または docker[用の docker デスクトップ](https://docs.docker.com/docker-for-windows/install/)をセットアップします。
5. 勧めホストが gMSA アカウントを使用できることを確認するには、 [ADServiceAccount](https://docs.microsoft.com/powershell/module/activedirectory/test-adserviceaccount)を実行します。 コマンドが**False**を返す場合は、診断手順については「[トラブルシューティング](#troubleshooting)」をご覧ください。

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Install-WindowsCapability -Online 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
    # To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

    Test-ADServiceAccount WebApp01
    ```

## <a name="create-a-credential-spec"></a>資格情報の仕様を作成する

資格情報スペックファイルは、コンテナーで使用する gMSA account に関するメタデータが含まれた JSON ドキュメントです。 Id 構成をコンテナーイメージと区別することで、コンテナーで使う gMSA を変更することができます。資格情報スペックファイルを交換するだけで、コードの変更は必要ありません。

資格情報スペックファイルは、ドメインに参加しているコンテナーホスト上で[Credentialspec PowerShell モジュール](https://aka.ms/credspec)を使用して作成されます。
ファイルを作成したら、他のコンテナーホストまたはコンテナーオーケストレータにコピーできます。
コンテナーホストはコンテナーの代わりに gMSA を取得するため、資格情報スペックファイルには、gMSA password などのシークレットは含まれません。

Docker は、Docker データディレクトリの**Credentialspecs**ディレクトリで credential spec ファイルを探します。 既定のインストールでは、に`C:\ProgramData\Docker\CredentialSpecs`このフォルダーが表示されます。

コンテナーホスト上に資格情報スペックファイルを作成するには、次の操作を行います。

1. RSAT AD PowerShell ツールをインストールする
    - Windows Server の場合は、 **Install-add-windowsfeature**を実行します。
    - Windows 10 バージョン1809以降の場合は、インストール-ウィンドウを実行します。 Windows 10 では、 **"0.0.1.0"** のような機能があります。
    - Windows 10 の以前のバージョンについ<https://aka.ms/rsat>ては、を参照してください。
2. 次のコマンドレットを実行して、 [Credentialspec PowerShell モジュール](https://aka.ms/credspec)の最新バージョンをインストールします。

    ```powershell
    Install-Module CredentialSpec
    ```

    コンテナーホストでインターネットにアクセスできない場合は、インターネット`Save-Module CredentialSpec`に接続されたコンピューターで実行し、モジュールフォルダー `C:\Program Files\WindowsPowerShell\Modules`またはコンテナーホスト`$env:PSModulePath`上の別の場所にコピーします。

3. 新しい資格情報スペックファイルを作成するには、次のコマンドレットを実行します。

    ```powershell
    New-CredentialSpec -AccountName WebApp01
    ```

    既定では、コマンドレットは、指定された gMSA 名をコンテナーのコンピューターアカウントとして使用して、資格を作成します。 ファイルは、gMSA ドメインとファイル名のアカウント名を使用して、Docker CredentialSpecs ディレクトリに保存されます。

    コンテナーでサービスまたはプロセスをセカンダリ gMSA として実行している場合は、追加の gMSA アカウントを含む資格情報の仕様を作成できます。 そのためには、 `-AdditionalAccounts`パラメーターを使用します。

    ```powershell
    New-CredentialSpec -AccountName WebApp01 -AdditionalAccounts LogAgentSvc, OtherSvc
    ```

    サポートされているパラメーターの完全な`Get-Help New-CredentialSpec`一覧については、を実行します。

4. 次のコマンドレットを使用して、すべての資格情報のスペックと完全なパスの一覧を表示できます。

    ```powershell
    Get-CredentialSpec
    ```

## <a name="configure-your-application-to-use-the-gmsa"></a>GMSA を使用するようにアプリケーションを構成する

一般的な構成では、コンテナーには1つの gMSA アカウントしかありません。これは、コンテナーコンピューターアカウントがネットワークリソースへの認証を試みるときに使われます。 つまり、gMSA id を使用する必要がある場合は、アプリを**ローカルシステム**または**ネットワークサービス**として実行する必要があります。

### <a name="run-an-iis-app-pool-as-network-service"></a>IIS アプリプールをネットワークサービスとして実行する

コンテナーで IIS web サイトをホストしている場合、gMSA を利用するために必要な操作は、アプリプール id を**Network Service**に設定することだけです。 次のコマンドを追加して、Dockerfile でその操作を行うことができます。

```dockerfile
RUN %windir%\system32\inetsrv\appcmd.exe set AppPool DefaultAppPool -processModel.identityType:NetworkService
```

以前に IIS アプリプールに対して静的ユーザー資格情報を使用したことがある場合は、それらの資格情報の代わりとして gMSA を検討してください。 Dev、テスト、および運用環境の間で gMSA を変更することができ、IIS はコンテナーイメージを変更せずに、自動的に現在の id を取得します。

### <a name="run-a-windows-service-as-network-service"></a>Windows サービスをネットワークサービスとして実行する

コンテナーアプリが Windows サービスとして実行されている場合は、Dockerfile で**Network service**として実行するようにサービスを設定することができます。

```dockerfile
RUN sc.exe config "YourServiceName" obj= "NT AUTHORITY\NETWORK SERVICE" password= ""
```

### <a name="run-arbitrary-console-apps-as-network-service"></a>ネットワークサービスとして任意のコンソールアプリを実行する

IIS または Service Manager でホストされていない一般的なコンソールアプリの場合、コンテナーを**ネットワークサービス**として実行すると、アプリは自動的に gMSA コンテキストを継承するため、最も簡単です。 この機能は、Windows Server バージョン1709で利用できます。

既定でネットワークサービスとして実行するには、次の行を Dockerfile に追加します。

```dockerfile
USER "NT AUTHORITY\NETWORK SERVICE"
```

また、を使用`docker exec`して、コンテナーにネットワークサービスとして接続することもできます。 これは、コンテナーが通常ネットワークサービスとして実行されていない場合に、実行中のコンテナーでの接続の問題をトラブルシューティングする場合に特に役立ちます。

```powershell
# Opens an interactive PowerShell console in the container (id = 85d) as the Network Service account
docker exec -it --user "NT AUTHORITY\NETWORK SERVICE" 85d powershell
```

## <a name="run-a-container-with-a-gmsa"></a>GMSA でコンテナーを実行する

GMSA を使ってコンテナーを実行するには、次のように`--security-opt` [docker run](https://docs.docker.com/engine/reference/run)のパラメーターに credential spec ファイルを指定します。

```powershell
# For Windows Server 2016, change the image name to mcr.microsoft.com/windows/servercore:ltsc2016
docker run --security-opt "credentialspec=file://contoso_webapp01.json" --hostname webapp01 -it mcr.microsoft.com/windows/servercore:ltsc2019 powershell
```

>[!IMPORTANT]
>Windows Server 2016 バージョン1709および1803では、コンテナーの hostname は gMSA short 名と一致する必要があります。

前の例では、gMSA SAM アカウント名は "webapp01" であるため、コンテナーの hostname は "webapp01" とも呼ばれます。

Windows Server 2019 以降では、hostname フィールドは必須ではありませんが、明示的に別の名前を指定した場合でも、ホスト名ではなく、gMSA 名でコンテナーが識別されます。

GMSA が正常に動作しているかどうかを確認するには、コンテナーで次のコマンドレットを実行します。

```powershell
# Replace contoso.com with your own domain
PS C:\> nltest /sc_verify:contoso.com

Flags: b0 HAS_IP  HAS_TIMESERV
Trusted DC Name \\dc01.contoso.com
Trusted DC Connection Status Status = 0 0x0 NERR_Success
Trust Verification Status = 0 0x0 NERR_Success
The command completed successfully
```

[信頼された DC 接続状態] と [信頼`NERR_Success`の確認状態] がない場合は、「[トラブルシューティング](#troubleshooting)」を参照して、問題のデバッグ方法に関するヒントを確認してください。

次のコマンドを実行し、クライアント名を確認して、コンテナー内から gMSA identity を確認できます。

```powershell
PS C:\> klist get krbtgt

Current LogonId is 0:0xaa79ef8
A ticket to krbtgt has been retrieved successfully.

Cached Tickets: (2)

#0>     Client: webapp01$ @ CONTOSO.COM
        Server: krbtgt/webapp01 @ CONTOSO.COM
        KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96
        Ticket Flags 0x40a10000 -> forwardable renewable pre_authent name_canonicalize
        Start Time: 3/21/2019 4:17:53 (local)
        End Time:   3/21/2019 14:17:53 (local)
        Renew Time: 3/28/2019 4:17:42 (local)
        Session Key Type: AES-256-CTS-HMAC-SHA1-96
        Cache Flags: 0
        Kdc Called: dc01.contoso.com

[...]
```

PowerShell または別のコンソールアプリを gMSA アカウントとして開くには、通常の ContainerAdministrator (または ContainerUser for NanoServer) アカウントではなく、ネットワークサービスアカウントでコンテナーを実行するように要求できます。

```powershell
# NOTE: you can only run as Network Service or SYSTEM on Windows Server 1709 and later
docker run --security-opt "credentialspec=file://contoso_webapp01.json" --hostname webapp01 --user "NT AUTHORITY\NETWORK SERVICE" -it mcr.microsoft.com/windows/servercore:ltsc2019 powershell
```

ネットワークサービスとして実行している場合は、ドメインコントローラーで SYSVOL に接続して、gMSA としてネットワーク認証をテストすることができます。

```powershell
# This command should succeed if you're successfully running as the gMSA
PS C:\> dir \\contoso.com\SYSVOL


    Directory: \\contoso.com\sysvol


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d----l        2/27/2019   8:09 PM                contoso.com
```

## <a name="orchestrate-containers-with-gmsa"></a>GMSA でコンテナーを調整する

運用環境では、多くの場合、アプリとサービスを展開して管理するためにコンテナーオーケストレータを使用します。 各 orchestrator には独自の管理パラダイムがあり、Windows コンテナープラットフォームに渡す資格情報の仕様を受け入れる必要があります。

GMSAs でコンテナーをオーケストレーションとして使用している場合は、次のことを確認します。

> [!div class="checklist"]
> * "ドメインは参加しています" というコンテナーを実行するようにスケジュールできるすべてのコンテナーホスト
> * コンテナーホストは、コンテナーで使用されるすべての gMSAs パスワードを取得するためのアクセス権を持っています。
> * 資格情報スペックファイルは、orchestrator で処理する方法に応じて、orchestrator に作成され、orchestrator にアップロードされるか、すべてのコンテナーホストにコピーされます。
> * コンテナーネットワークを使うと、コンテナーは Active Directory ドメインコントローラーと通信して gMSA チケットを取得することができます。

### <a name="how-to-use-gmsa-with-service-fabric"></a>Service Fabric で gMSA を使用する方法

アプリケーションマニフェストで資格情報の仕様の場所を指定した場合、Service Fabric は、gMSA を使用して Windows コンテナーの実行をサポートします。 Service Fabric で特定できるように、資格情報スペックファイルを作成し、各ホスト上の Docker データディレクトリの**Credentialspecs**サブディレクトリに配置する必要があります。 [Credentialspec PowerShell モジュール](https://aka.ms/credspec)の一部である、credentialspec コマンドレットを実行して、資格情報の仕様が適切な場所にあるかどうかを確認することができます。 ****

アプリケーションの構成方法の詳細については、「[クイックスタート: windows コンテナーをサービスファブリックに展開する](https://docs.microsoft.com/azure/service-fabric/service-fabric-quickstart-containers)」と「[サービスファブリックで実行されている Windows コンテナーの gMSA を設定](https://docs.microsoft.com/azure/service-fabric/service-fabric-setup-gmsa-for-windows-containers)する」を参照してください。

### <a name="how-to-use-gmsa-with-docker-swarm"></a>Docker 群れで gMSA を使用する方法

Docker 群れで管理されるコンテナーと共に gMSA を使用するには、次の`--credential-spec`パラメーターを使用して、 [docker サービスの create](https://docs.docker.com/engine/reference/commandline/service_create/)コマンドを実行します。

```powershell
docker service create --credential-spec "file://contoso_webapp01.json" --hostname "WebApp01" <image name>
```

Docker サービスでの資格情報の仕様の使い方の詳細については、 [docker の群れの例](https://docs.docker.com/engine/reference/commandline/service_create/#provide-credential-specs-for-managed-service-accounts-windows-only)を参照してください。

### <a name="how-to-use-gmsa-with-kubernetes"></a>Kubernetes で gMSA を使用する方法

Kubernetes の gMSAs の Windows コンテナーのスケジュール設定は、Kubernetes 1.14 のアルファ機能として利用できます。 この機能に関する最新情報については、「 [Windows ポッドとコンテナー用に gMSA を構成](https://kubernetes.io/docs/tasks/configure-pod-container/configure-gmsa)する」と「Kubernetes の配布でテストする方法」を参照してください。

## <a name="example-uses"></a>使用例

### <a name="sql-connection-strings"></a>SQL 接続文字列

サービスがローカル システムまたはコンテナー内のネットワーク サービスとして実行されている場合は、Microsoft SQL Server に接続するために Windows 統合認証を使用できます。

次に示すのは、コンテナー id を使って SQL Server に対して認証する接続文字列の例です。

```sql
Server=sql.contoso.com;Database=MusicStore;Integrated Security=True;MultipleActiveResultSets=True;Connect Timeout=30
```

Microsoft SQL Server では、ドメインと gMSA の名前を使用し、後に $ を付けてログインを作成します。 ログインを作成したら、データベースのユーザーに追加して、適切なアクセス許可を与えることができます。

例:

```sql
CREATE LOGIN "DEMO\WebApplication1$"
    FROM WINDOWS
    WITH DEFAULT_DATABASE = "MusicStore"
GO

USE MusicStore
GO
CREATE USER WebApplication1 FOR LOGIN "DEMO\WebApplication1$"
GO

EXEC sp_addrolemember 'db_datareader', 'WebApplication1'
EXEC sp_addrolemember 'db_datawriter', 'WebApplication1'
```

動作を確認するには、「Containerization [](https://youtu.be/cZHPz80I-3s?t=2672)へのパスを 2016 Ignite にする」の「パスをコンテナーに変換する」を参照してください。

## <a name="troubleshooting"></a>トラブルシューティング

### <a name="known-issues"></a>既知の問題

#### <a name="container-hostname-must-match-the-gmsa-name-for-windows-server-2016-and-windows-10-versions-1709-and-1803"></a>コンテナーのホスト名は、Windows Server 2016 および Windows 10 のバージョン1709および1803の gMSA 名と一致する必要があります。

Windows Server 2016、バージョン1709、または1803を実行している場合、コンテナーの hostname は gMSA SAM アカウント名と一致する必要があります。

ホスト名が gMSA 名と一致しない場合、受信 NTLM 認証要求と名前/SID 翻訳 (ASP.NET メンバーシップロールプロバイダーなどの多くのライブラリで使用) は失敗します。 Kerberos は、hostname と gMSA name が一致しない場合でも、通常どおり機能します。

この制限は Windows Server 2019 で修正されました。この制限は、割り当てられたホスト名に関係なく、コンテナーが常にネットワーク上で gMSA 名を使用するようになっています。

#### <a name="using-a-gmsa-with-more-than-one-container-simultaneously-leads-to-intermittent-failures-on-windows-server-2016-and-windows-10-versions-1709-and-1803"></a>複数のコンテナーで gMSA を同時に使用すると、Windows Server 2016 および Windows 10 バージョン1709および1803で断続的なエラーが発生する場合があります。

すべてのコンテナーは同じホスト名を使う必要があるため、第2の問題は Windows Server 2019 および Windows 10 バージョン1809より前のバージョンの Windows に影響します。 複数のコンテナーに同じ id と hostname が割り当てられている場合、2つのコンテナーが同じドメインコントローラーと同時に対話するときに競合状態が発生することがあります。 別のコンテナーと同じドメインコントローラーとの通信が行われると、同じ id を使用する前のコンテナーとの通信がキャンセルされます。 これにより、認証エラーが断続的に発生したり、コンテナー内で実行`nltest /sc_verify:contoso.com`したときに信頼エラーとして観察されることがあります。

Windows Server 2019 の動作を変更して、コンピューター名からコンテナー id を分離することで、複数のコンテナーで同じ gMSA を同時に使うことができるようになりました。

#### <a name="you-cant-use-gmsas-with-hyper-v-isolated-containers-on-windows-10-versions-1703-1709-and-1803"></a>Windows 10 バージョン1703、1709、および1803では、Hyper-v として gMSAs 使用できません。

Windows 10 および Windows Server バージョン1703、1709、1803で、Hyper-v 分離コンテナーで gMSA を使用しようとすると、コンテナーの初期化がハングするか、失敗します。

このバグは、Windows Server 2019 および Windows 10 バージョン1809で修正されました。 また、Windows Server 2016 および Windows 10 バージョン1607では、"Hyper-v" を使って Hyper-v 分離コンテナーを実行することもできます。

### <a name="general-troubleshooting-guidance"></a>一般的なトラブルシューティングガイダンス

GMSA でコンテナーを実行しているときにエラーが発生した場合は、次の手順に従って根本原因を特定することができます。

#### <a name="make-sure-the-host-can-use-the-gmsa"></a>ホストで gMSA を使用できることを確認します。

1. ホストがドメインに参加していて、ドメインコントローラーに接続できることを確認します。
2. RSAT から AD PowerShell ツールをインストールし、[テスト-ADServiceAccount](https://docs.microsoft.com/powershell/module/activedirectory/test-adserviceaccount)を実行して、コンピューターに gMSA を取得するためのアクセス許可があるかどうかを確認します。 コマンドレットで**False**が返される場合、コンピューターには gMSA パスワードへのアクセス権がありません。

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Install-WindowsCapability -Online 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
    # To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

    Test-ADServiceAccount WebApp01
    ```

3. **テスト-ADServiceAccount**が**False**を返す場合は、ホストが gMSA パスワードにアクセスできるセキュリティグループに属していることを確認します。

    ```powershell
    # Get the current computer's group membership
    Get-ADComputer $env:computername | Get-ADPrincipalGroupMembership | Select-Object DistinguishedName

    # Get the groups allowed to retrieve the gMSA password
    # Change "WebApp01" for your own gMSA name
    (Get-ADServiceAccount WebApp01 -Properties PrincipalsAllowedToRetrieveManagedPassword).PrincipalsAllowedToRetrieveManagedPassword
    ```

4. 使用しているホストが、gMSA パスワードを取得するために承認されているセキュリティグループに属しているが、まだ失敗する**テスト-ADServiceAccount**の場合は、コンピューターを再起動して、現在のグループメンバーシップを反映する新しいチケットを取得することが必要な場合があります。

#### <a name="check-the-credential-spec-file"></a>資格情報スペックファイルを確認する

1. コンピューター上のすべての資格情報の仕様を検索するには、 [Credentialspec PowerShell モジュール](https://aka.ms/credspec)から**Get-credentialspec**を実行します。 資格情報の仕様は、Docker ルートディレクトリの "CredentialSpecs" ディレクトリに保存されている必要があります。 Docker ルートディレクトリは、docker info を実行して見つけることができ**ます-f "{{.DockerRootDir}} "**。
2. CredentialSpec ファイルを開き、次のフィールドが正しく入力されていることを確認します。
    - **Sid**: gMSA アカウントの sid
    - **Machineaccountname**: GMSA SAM アカウント名 (完全なドメイン名またはドル記号は含めません)
    - **Dnstreename**: Active Directory フォレストの FQDN
    - **DnsName**: gMSA が属しているドメインの FQDN
    - **NetBiosName**: gMSA が属しているドメインの NETBIOS 名
    - **Groupmanagedserviceaccounts/Name**: gMSA SAM アカウント名 (完全なドメイン名またはドル記号は含めません)
    - **Groupmanagedserviceaccounts/Scope**: ドメイン FQDN と NETBIOS 用の1つのエントリ

    入力は、次のような完全な資格情報仕様の例のようになります。

    ```json
    {
        "CmsPlugins": [
            "ActiveDirectory"
        ],
        "DomainJoinConfig": {
            "Sid": "S-1-5-21-702590844-1001920913-2680819671",
            "MachineAccountName": "webapp01",
            "Guid": "56d9b66c-d746-4f87-bd26-26760cfdca2e",
            "DnsTreeName": "contoso.com",
            "DnsName": "contoso.com",
            "NetBiosName": "CONTOSO"
        },
        "ActiveDirectoryConfig": {
            "GroupManagedServiceAccounts": [
                {
                    "Name": "webapp01",
                    "Scope": "contoso.com"
                },
                {
                    "Name": "webapp01",
                    "Scope": "CONTOSO"
                }
            ]
        }
    }
    ```

3. オーケストレーションソリューションの資格情報スペシフィケーションファイルへのパスが正しいことを確認します。 Docker を使っている場合は、コンテナーの run コマンドに`--security-opt="credentialspec=file://NAME.json"`"name" という名前が含まれていることを確認**** してください。 この名前は、Docker ルートディレクトリの [CredentialSpecs] フォルダーを基準としたフラットなファイル名です。

#### <a name="check-the-firewall-configuration"></a>ファイアウォールの構成を確認する

コンテナーまたはホストネットワークで厳密なファイアウォールポリシーを使用している場合、Active Directory ドメインコントローラーまたは DNS サーバーへの必要な接続がブロックされることがあります。

| プロトコルとポート | 目的 |
|-------------------|---------|
| TCP および UDP 53 | DNS |
| TCP および UDP 88 | Kerberos |
| TCP 139 | ネット |
| TCP および UDP 389 | LDAP |
| TCP 636 | LDAP SSL |

コンテナーからドメインコントローラーに送信されるトラフィックの種類に応じて、追加のポートへのアクセスを許可することが必要な場合があります。
Active Directory で使用されるすべてのポートの一覧については、「 [Active directory と active Directory ドメインサービスのポートの要件](https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd772723(v=ws.10)#communication-to-domain-controllers)」を参照してください。

#### <a name="check-the-container"></a>コンテナーを確認する

1. Windows Server 2019 または Windows 10 バージョン1809より前のバージョンの Windows を実行している場合は、コンテナーのホスト名が gMSA 名と一致する必要があります。 パラメーターが`--hostname` gMSA short 名 ("webapp01.contoso.com" ではなく "webapp01" など) と一致することを確認します。

2. コンテナーネットワーク構成を確認して、コンテナーが gMSA のドメインのドメインコントローラーを解決してアクセスできることを確認します。 コンテナーの DNS サーバーが正しく構成されていないと、id の問題が発生する原因となります。

3. コンテナーのドメインへの有効な接続があるかどうかを確認するに`docker exec`は、コンテナー (または同等のもの) で次のコマンドレットを実行します。

    ```powershell
    nltest /sc_verify:contoso.com
    ```

    GMSA が利用可能で`NERR_SUCCESS`あり、ネットワーク接続によってコンテナーがドメインと通信できる場合は、信頼の確認が返されます。 失敗した場合は、ホストとコンテナーのネットワーク構成を確認します。 どちらの場合も、ドメインコントローラーと通信できる必要があります。

4. アプリが gMSA を[使用するように構成](#configure-your-application-to-use-the-gmsa)されていることを確認します。 GMSA を使用しても、コンテナー内のユーザーアカウントは変更されません。 代わりに、システムアカウントは、他のネットワークリソースと通信するときに gMSA を使います。 つまり、gMSA id を使用するには、アプリがネットワークサービスまたはローカルシステムとして実行されている必要があります。

    > [!TIP]
    > コンテナー内の`whoami`現在のユーザーコンテキストを特定するために別のツールを実行するか、別のツールを使用すると、gMSA 名自体は表示されません。 これは、ドメイン id ではなく、ローカルユーザーとしてコンテナーに常にサインインしているためです。 GMSA は、コンピューターアカウントがネットワークリソースと通信するたびに使用されます。これは、アプリがネットワークサービスまたはローカルシステムとして実行する必要がある理由です。

5. 最後に、コンテナーが正しく構成されているように見えても、ユーザーまたは他のサービスがコンテナー内のアプリに自動的に認証できない場合は、gMSA アカウントの Spn を確認します。 クライアントは、アプリケーションに到達したときの名前で gMSA account を検索します。 これは、たとえば、クライアントがロード`host`バランサーまたは別の DNS 名を使用してアプリに接続している場合に、gMSA に追加の spn が必要になることを意味します。

## <a name="additional-resources"></a>その他の資料

- [グループ管理サービスアカウントの概要](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview)
