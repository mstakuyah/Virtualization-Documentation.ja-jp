---
title: Windows のコンテナーのグループの管理サービス アカウントします。
description: Windows のコンテナーのグループの管理サービス アカウントします。
keywords: docker、コンテナー、active directory、gmsa
author: rpsqrd
ms.date: 03/22/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 17c4089c98a74ea5937bac5d0eb4d4f1749aecf7
ms.sourcegitcommit: b8afbfb63c33a491d7bad44d8d5962e6a60cb566
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/22/2019
ms.locfileid: "9257448"
---
# <a name="group-managed-service-accounts-for-windows-containers"></a>Windows のコンテナーのグループの管理サービス アカウントします。

Windows ベースのネットワークでは、認証とユーザー、コンピューター、およびその他のネットワーク リソース間での承認を容易にするために Active Directory (AD) を使用する一般的なはできます。 企業アプリケーションの開発者 AD 統合して、簡単な自動的および透過的にログインするには、ユーザーと他のサービスの統合の Windows 認証を活用する、ドメインに参加しているサーバーで実行するユーザーのアプリケーションを設計します。その id を持つアプリケーションです。

Windows コンテナーが参加しているドメインではできないは、さまざまな認証シナリオをサポートする Active Directory ドメイン id を使っていることもできます。
[グループ管理サービスのアカウント](https://docs.microsoft.com/en-us/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview)(gMSA) - で実行するための Windows コンテナーを構成するために特殊なサービスのアカウントを使用せずに id を共有する複数のコンピューターを許可するように設計された Windows Server 2012 の導入そのパスワードを知っています。
GMSA とコンテナーを実行してコンテナー ホストに Active Directory ドメイン コント ローラーから gMSA パスワードを取得しますコンテナーのインスタンスに与えられます。
そのコンピューター アカウント (システム) は、リソースのネットワークにアクセスする必要があるとき、コンテナーは gMSA 資格情報を使用します。

この記事では、Windows のコンテナーの Active Directory グループの管理サービスのアカウントの使用を開始する方法について説明します。

## <a name="prerequisites"></a>前提条件

サービス アカウントの管理] グループで、Windows のコンテナーを実行するのには、次の必要があります。

**Windows Server 2012 以降を実行している 1 つ以上のドメイン コント ローラーの Active Directory ドメイン。**
GMSAs] を使用するフォレストまたはドメインの機能レベル要件はありませんが、gMSA パスワードできるだけでは、Windows Server 2012 を実行しているドメイン コント ローラー分散以降。
詳細については、 [gMSAs の Active Directory の要件](https://docs.microsoft.com/en-us/windows-server/security/group-managed-service-accounts/getting-started-with-group-managed-service-accounts#BKMK_gMSA_Req)を参照してください。



**GMSA アカウントを作成するのにはアクセスを許可します。**
GMSA アカウントを作成するには、ドメイン管理者であるかされたアカウントを使用する必要があります*msDS GroupManagedServiceAccount オブジェクトの作成*の権限を委任します。



**CredentialSpec PowerShell モジュールをダウンロードするのには、インターネットにアクセスします。**
未接続の環境で作業している場合は、インターネットでコンピューター上の[モジュールを保存](https://docs.microsoft.com/en-us/powershell/module/powershellget/save-module?view=powershell-5.1)することができます。 にアクセスして、開発コンピューターまたはコンテナーのホストにコピーします。

## <a name="one-time-preparation-of-active-directory"></a>Active Directory の 1 回限りの準備

自分のドメインで、gMSA を作成していない場合は、キー配布サービス ルート キーを生成する可能性が高い必要があります。
KDS は、作成、回転、および承認されたホスト gMSA パスワードを解除します。
コンテナーのホストの gMSA を使用して、コンテナーを実行する場合、現在のパスワードを取得する KDS に連絡します。

場合は、KDS ルート キーを確認するのには既に作成されて、インストールされている AD PowerShell ツールを使用してドメイン コント ローラーまたはドメインのメンバーで*ドメインの管理者*として、次の PowerShell コマンドを実行します。

```powershell
Get-KdsRootKey
```

コマンドがキーが返される場合、ID すべてを設定していて、[サービス アカウントの管理グループを作成する](#create-a-group-managed-service-account)] セクションまでスキップできます。
それ以外の場合、KDS ルート キーを作成するのには続行します。

運用環境または複数のドメイン コント ローラーのテスト環境では、KDS ルート キーを作成するのには、*ドメインの管理者*として PowerShell で次のコマンドを実行します。
コマンドは、キー効果がすぐにわかるように、KDS ルート キーが複製ドメイン コント ローラーすべてで使用可能な 10 時間かかるにする必要があります。

```powershell
# For production environments
Add-KdsRootKey -EffectiveImmediately
```

自分のドメインで 1 つのドメイン コント ローラーをのみがある場合、キーを有効にする前の 10 の時間を設定してプロセスを迅速ことができます。
実運用環境では、この方法を使わないでください。

```powershell
# For single-DC test environments ONLY
Add-KdsRootKey -EffectiveTime (Get-Date).AddHours(-10)
```

## <a name="create-a-group-managed-service-account"></a>グループの管理サービスのアカウントを作成します。

統合 Windows 認証を使用するすべてのコンテナーには、少なくとも 1 つの gMSA 必要があります。
プライマリ gMSA は、アプリの*システム*や*ネットワーク サービス*として実行されているネットワーク上のリソースにアクセスするときに使用されます。
[GMSA の名前には、コンテナーに割り当てられているホスト名に関係なく、ネットワーク上にコンテナーの名前になります。
コンテナーはコンテナーにコンテナーのコンピューターのアカウントから、別のユーザーとしてサービスまたはアプリケーションを実行する場合に、gMSAs の追加] を構成することもできます。

作成すると、gMSA、多くの異なるコンピューター間で同時に使用できる共有 id を作成します。
Active Directory アクセス制御リストで、gMSA パスワードへのアクセスが保護されています。
各 gMSA アカウントのセキュリティ グループを作成し、パスワードへのアクセスを制限するのには、セキュリティ グループに関連するコンテナー ホストを追加するをお勧めします。

最後に、コンテナーはどのサービス プリンシパル名 (SPN) を自動的に登録しないので、少なくとも「ホスト」SPN gMSA アカウントを手動で作成する必要があります。
通常、ホストまたは http SPN が登録されている gMSA のアカウントと同じ名前を使用して、クライアント負荷分散装置または gMSA 名とは異なる DNS 名からコンテナー化アプリケーションにアクセスする場合は、別のサービスの名前を使用する必要があります。
たとえば、gMSA アカウント*WebApp01*が、ユーザーが*mysite.contoso.com*のサイトにアクセスする必要がありますを登録する、 `http/mysite.contoso.com` gMSA アカウントの SPN します。
一部のアプリケーションが必要になる追加の Spn の一意のプロトコルの場合: たとえば、SQL Server が「MSSQLSvc/ホスト名」SPN が必要です。

次の表では、gMSA を作成するときに必要な属性が一覧表示します。

gMSA プロパティ | 必要な値 | 例
--------------|----------------|--------
名前 | 任意の有効なアカウント名。 | `WebApp01`
DnsHostName | ドメイン名は、アカウント名に追加します。 | `WebApp01.contoso.com`
ServicePrincipalNames | 少なくともホスト SPN を設定する、必要に応じて、その他のプロトコルを追加します。 | `'host/WebApp01', 'host/WebApp01.contoso.com'`
PrincipalsAllowedToRetrieveManagedPassword | コンテナーのホストに含まれているセキュリティ グループ。 | `WebApp01Hosts`

1 回、gMSA を呼び出し、gMSA、セキュリティ グループを作成するのには PowerShell で次のコマンドを実行する予定があるかがわかります。

> [!TIP]
> 使用する必要があります**Domain Admins**セキュリティ グループに属しているされたアカウントが次のコマンドを実行する**msDS GroupManagedServiceAccount オブジェクトの作成**の権限を委任します。
> [新規 ADServiceAccount](https://docs.microsoft.com/en-us/powershell/module/addsadministration/new-adserviceaccount?view=win10-ps)コマンドレットは、[リモート サーバー管理ツール](https://aka.ms/rsat)から AD PowerShell ツールの一部です。

```powershell
# Replace 'WebApp01' and 'contoso.com' with your own gMSA and domain names, respectively

# To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
# To install the AD module on Windows 10 version 1809 or later, run Install-WindowsCapability -Online 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
# To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

# Create the security group
New-ADGroup -Name "WebApp01 Authorized Hosts" -SamAccountName "WebApp01Hosts" -Scope DomainLocal

# Create the gMSA
New-ADServiceAccount -Name "WebApp01" -DnsHostName "WebApp01.contoso.com" -ServicePrincipalNames "host/WebApp01", "host/WebApp01.contoso.com" -PrincipalsAllowedToRetrieveManagedPassword "WebApp01Hosts"

# Add your container hosts to the security group
Add-ADGroupMember -Identity "WebApp01Hosts" -Members "ContainerHost01", "ContainerHost02", "ContainerHost03"
```

開発、テスト、運用環境の別の gMSA アカウントを作成することをお勧めします。

## <a name="prepare-your-container-host"></a>コンテナー ホストを準備します。

各コンテナーのホストを gMSA と Windows コンテナーを実行するには、ドメインの結合し、gMSA パスワードを取得するアクセスがあります。

1.  お使いのコンピューターに Active Directory ドメインに参加します。
2.  自分のホスト gMSA パスワードへのアクセスを制御できるセキュリティ グループに属していることを確認します。
3.  その新しいグループのメンバーシップを取得してコンピューターを再起動します。
4.  [Windows 10 用の Docker デスクトップ](https://docs.docker.com/docker-for-windows/install/)または[Windows Server の Docker](https://docs.docker.com/install/windows/docker-ee/)を設定します。
5.  (推奨)ホストは[テスト ADServiceAccount](https://docs.microsoft.com/en-us/powershell/module/activedirectory/test-adserviceaccount)を実行して gMSA アカウントを使用することを確認します。 コマンドが**False**が返される場合は、診断手順については、[[トラブルシューティング](#troubleshooting)] セクションを参照してください。

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Install-WindowsCapability -Online 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
    # To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

    Test-ADServiceAccount WebApp01
    ```

## <a name="create-a-credential-spec"></a>資格情報の仕様を作成します。

資格情報の仕様ファイルは、目的コンテナーを使用する gMSA アカウントに関するメタデータが含まれている JSON ドキュメントです。
コンテナーの画像とは別のユーザー設定を維持、して仕様ファイルの資格情報のコード変更は不要を取り替えるだけで、コンテナーで使用される gMSA を簡単に変更することができます。

資格情報の仕様ファイルを作成するには、ドメインに参加しているコンテナーのホストに[CredentialSpec PowerShell モジュール](https://aka.ms/credspec)の使用されます。
ファイルを作成した後は、他のコンテナー ホストしたり、コンテナー オーケストレータ コピーできます。
資格情報の仕様ファイルは gMSA パスワードなどの任意の機密情報が含まれないコンテナー ホストしますコンテナーの代理として、gMSA を取得するためです。

Docker データ ディレクトリに**CredentialSpecs**ディレクトリの資格情報の仕様ファイルを検索する docker 想定しています。
既定のインストールには、このフォルダーが表示されます`C:\ProgramData\Docker\CredentialSpecs`します。

コンテナー ホストの資格情報の仕様ファイルを作成するのには、次の手順を実行します。
1.  RSAT AD PowerShell ツールをインストールします。
    -   Windows server を実行します。 `Install-WindowsFeature RSAT-AD-PowerShell`
    -   Windows 10、1809 以降のバージョンの実行します。 `Install-WindowsCapability -Online 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'`
    -   以前のバージョンの Windows 10 では、次を参照してください。https://aka.ms/rsat
2.  [CredentialSpec PowerShell モジュール](https://aka.ms/credspec)の最新バージョンをインストールします。

    ```powershell
    Install-Module CredentialSpec
    ```

    コンテナー ホストのインターネットへのアクセスを持っていない場合は、実行`Save-Module CredentialSpec`インターネットに接続されているコンピューターでモジュール フォルダーにコピーして`C:\Program Files\WindowsPowerShell\Modules`または別の場所で`$env:PSModulePath`コンテナーのホストにします。

3.  新しい資格情報の仕様ファイルを作成します。

    ```powershell
    New-CredentialSpec -AccountName WebApp01
    ```

    既定では、コマンドレットはコンテナーのコンピューター アカウントとして指定された gMSA 名を使用して cred 仕様を作成します。
    ファイルは、ファイル名の gMSA ドメインとアカウント名を使用して、Docker CredentialSpecs ディレクトリに保存されます。

    コンテナー内の第 2 gMSA としてサービスやプロセスを使用している場合は、gMSA の他のアカウントを含む資格情報の仕様を作成することができます。
    そのため、使用して、`-AdditionalAccounts`パラメーター。

    ```powershell
    New-CredentialSpec -AccountName WebApp01 -AdditionalAccounts LogAgentSvc, OtherSvc
    ```

    サポートされるパラメーターの一覧は、実行し、`Get-Help New-CredentialSpec`します。

4.  すべての資格情報の仕様と、次のコマンドで、完全なパスの一覧を表示することができます。

    ```powershell
    Get-CredentialSpec
    ```

## <a name="configuring-your-application-to-use-the-gmsa"></a>[GMSA を使用するアプリケーションを構成します。

一般的な構成は、コンテナーをネットワーク リソースへの認証しようとしているコンテナー コンピューター アカウントときに使用される gMSA アカウントを 1 つのみ指定します。
つまり、アプリが gMSA id を使用することが必要な場合は、**ローカル システム**や**ネットワーク サービス**として実行する必要があります。

### <a name="run-an-iis-app-pool-as-network-service"></a>ネットワーク サービスとして IIS アプリケーション プールを実行します。

コンテナーの IIS web サイトをホストしている場合は、**ネットワーク サービス**にアプリケーション プール id を設定、gMSA を活用するために必要なされています。
設定は、次のコマンドを追加して、Dockerfile で行うことができます。

```dockerfile
RUN (Get-IISAppPool DefaultAppPool).ProcessModel.IdentityType = "NetworkService"
```

IIS アプリケーション プールの静的なユーザーの資格情報を使用していた場合は、これらの資格情報の代わりとして、gMSA を検討してください。
開発、テスト、および運用環境間で gMSA を変更して、IIS はコンテナーの画像を変更せずに現在のユーザーを自動的に選択します。

### <a name="run-a-windows-service-as-network-service"></a>ネットワーク サービスとして Windows サービスを実行します。

コンテナー化アプリは、Windows サービスとして実行する場合、Dockerfile の**ネットワーク サービス**として実行するサービスを設定できます。

```dockerfile
RUN cmd /c 'sc.exe config "YourServiceName" obj= "NT AUTHORITY\NETWORK SERVICE" password= ""'
```

### <a name="run-arbitrary-console-apps-as-network-service"></a>ネットワーク サービスとして任意コンソール アプリを実行します。

IIS またはサービス マネージャーでホストされていない汎用コンソールのアプリをアプリが自動的に gMSA コンテキストを継承できるように、**ネットワーク サービス**としてしますコンテナーを実行する最も簡単なことが多くの場合です。
この機能は、Windows Server 1709 のバージョンで使用可能です。

既定では、ネットワーク サービスとして実行して、Dockerfile には、次の行を追加します。

```dockerfile
USER "NT AUTHORITY\NETWORK SERVICE"
```

接続することも、コンテナーにネットワーク サービスとしてを 1 回限りに`docker exec`します。
コンテナーが通常の方法でネットワーク サービスとして実行できないときに実行されているコンテナーで接続の問題のトラブルシューティングをしている場合に便利です。

```powershell
# Opens an interactive PowerShell console in the container (id = 85d) as the Network Service account
docker exec -it --user "NT AUTHORITY\NETWORK SERVICE" 85d powershell
```

## <a name="run-a-container-with-a-gmsa"></a>GMSA とコンテナーを実行します。

実行するにはコンテナーを gMSA で提供、資格情報の仕様ファイルを`--security-opt`パラメーターの[docker を実行](https://docs.docker.com/engine/reference/run)します。

```powershell
# For Windows Server 2016, change the image name to mcr.microsoft.com/windows/servercore:ltsc2016
docker run --security-opt "credentialspec=file://contoso_webapp01.json" --hostname webapp01 -it mcr.microsoft.com/windows/servercore:ltsc2019 powershell
```

Windows Server 2016 では、[1709 および 1803、しますコンテナー*と一致する必要があります*、gMSA のホスト名の短い名前。
上記の例では、gMSA SAM アカウント名は"webapp01"コンテナー ホスト名が同じに設定します。
Windows Server 2019 と、後で、ホスト名] フィールドは必須ではありませんが、コンテナーが識別されます自体 gMSA 名、ホスト名に置き換えてくださいによって明示的に別の名前を指定する場合でもします。

確認するには、gMSA 正しく動作している場合に、コンテナー内で、次のコマンドを実行します。

```
# Replace contoso.com with your own domain
PS C:\> nltest /sc_verify:contoso.com

Flags: b0 HAS_IP  HAS_TIMESERV
Trusted DC Name \\dc01.contoso.com
Trusted DC Connection Status Status = 0 0x0 NERR_Success
Trust Verification Status = 0 0x0 NERR_Success
The command completed successfully
```

*信頼できる DC 状態*を*確認状態のセキュリティ*が*NERR_Success*にない場合は、問題をデバッグする方法に関するヒントの[トラブルシューティング](#troubleshooting)」をご覧ください。

次のコマンドを実行しているクライアントの名前を確認して、コンテナー内から gMSA id を確認できます。

```
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

PowerShell または gMSA アカウントとして別のコンソール アプリを開くには、ネットワーク サービスのアカウントが標準の ContainerAdministrator (または NanoServer の ContainerUser) ではなくアカウントで実行するためにコンテナーをもらうことができます。

```powershell
# NOTE: you can only run as SYSTEM on Windows Server 1709 and later
docker run --security-opt "credentialspec=file://contoso_webapp01.json" --hostname webapp01 --user "NT AUTHORITY\NETWORK SERVICE" -it mcr.microsoft.com/windows/servercore:ltsc2019 powershell
```

ネットワーク サービスとして実行すると、gMSA としてドメイン コント ローラーの SYSVOL に接続しようとしてネットワークの認証をテストすることができます。

```
# This command should succeed if you're successfully running as the gMSA
PS C:\> dir \\contoso.com\SYSVOL


    Directory: \\contoso.com\sysvol


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d----l        2/27/2019   8:09 PM                contoso.com
```

## <a name="orchestrating-containers-with-gmsa"></a>GMSA にコンテナーの調整

運用環境では、展開し、アプリとサービスを管理する多くの場合、コンテナー オーケストレータを使用します。
各オーケストレータ専用管理パラダイム、ユーザーの責任において、Windows コンテナー プラットフォームに資格情報の仕様を承諾します。

GMSAs にコンテナーを調整している、ことを確認します。
> [!div class="checklist"]
> * GMSAs とコンテナーを実行するのには、スケジュールするすべてのコンテナー ホストが参加しているドメインです。
> * コンテナーで使用されているすべての gMSAs のパスワードを取得しますコンテナーのホストにアクセスします。
> * 資格情報の仕様ファイルを作成し、オーケストレータにアップロードまたは優先的オーケストレータにそれらの処理方法に応じて、コンテナーのすべてのホストにコピーします。
> * コンテナーのネットワーク gMSA チケットを取得する Active Directory ドメイン コント ローラーとの通信にコンテナーを許可します。

### <a name="using-gmsa-with-service-fabric"></a>布地へのサービスと gMSA を使用します。

布地へのサービスには、アプリケーション マニフェストで資格情報の仕様場所を指定すると、gMSA で実行中の Windows コンテナーがサポートしています。
資格情報の仕様ファイルを作成して、布地へのサービスが見つけられるように、各ホスト Docker データ ディレクトリの**CredentialSpecs**サブディレクトリに配置する必要があります。
`Get-CredentialSpec` 、資格情報の仕様が正しい場所に、かを確認、 [CredentialSpec PowerShell モジュール](https://aka.ms/credspec)の一部のコマンドレットを使用することができます。

表示[クイック スタート: 布地へのサービスを Windows コンテナーを展開するには](https://docs.microsoft.com/en-us/azure/service-fabric/service-fabric-quickstart-containers)してアプリケーションを構成する方法の詳細について[布地へのサービスで実行されている Windows コンテナーの gMSA を設定](https://docs.microsoft.com/en-us/azure/service-fabric/service-fabric-setup-gmsa-for-windows-containers)します。

### <a name="using-gmsa-with-docker-swarm"></a>Docker 選ばと gMSA を使用します。

使用するには、gMSA Docker 選ばによって管理されているコンテナーがコマンドを使用して、 [docker サービスを作成する](https://docs.docker.com/engine/reference/commandline/service_create/)と、`--credential-spec`パラメーター。

```powershell
docker service create --credential-spec "file://contoso_webapp01.json" --hostname "WebApp01" <image name>
```

資格情報の定義を使って Docker サービスの詳細については、 [Docker 選ばの例](https://docs.docker.com/engine/reference/commandline/service_create/#provide-credential-specs-for-managed-service-accounts-windows-only)を参照してください。

### <a name="using-gmsa-with-kubernetes"></a>Kubernetes と gMSA を使用します。

スケジュール Kubernetes で gMSAs と Windows コンテナーに対するサポートは、Kubernetes 1.14 における α サポートされます。
この機能に関する最新の情報と、Kubernetes 配布でテストする方法についての[Windows グループ管理サービスのアカウント](https://github.com/kubernetes/enhancements/blob/master/keps/sig-windows/20181221-windows-group-managed-service-accounts-for-container-identity.md)を確認してください。

## <a name="example-uses"></a>使用例

### <a name="sql-connection-strings"></a>SQL 接続文字列
サービスがローカル システムまたはコンテナー内のネットワーク サービスとして実行されている場合は、Microsoft SQL Server に接続するために Windows 統合認証を使用できます。

SQL Server を認証するためにコンテナーの id を使用する接続文字列の例を示します。

```
Server=sql.contoso.com;Database=MusicStore;Integrated Security=True;MultipleActiveResultSets=True;Connect Timeout=30
```

Microsoft SQL Server では、ドメインと gMSA の名前を使用し、後に $ を付けてログインを作成します。
ログインが作成されると、データベース上のユーザーに追加され、適切なアクセス許可を与えることができます。

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

実際の動作を確認するには、Microsoft Ignite 2016 のセッション ”Walk the Path to Containerization - transforming workloads into containers” (コンテナー化への道: ワークロードをコンテナーに変換する) の[録画デモ](https://youtu.be/cZHPz80I-3s?t=2672)をご覧ください。

## <a name="troubleshooting"></a>トラブルシューティング

### <a name="known-issues"></a>既知の問題

**コンテナーのホスト名の WS2016、1709、および 1803 gMSA 名前と一致する必要があります。**

Windows Server 2016 を使用している場合 1709、または 1803、コンテナーのホスト名必要がありますと一致する gMSA SAM アカウント名。
ホスト名 gMSA 名前が一致しない場合に受信 NTLM 認証要求と (ASP.NET ロール メンバーシップなど、多くのライブラリで使用) 名前/SID 変換は失敗します。
Kerberos は引き続きホスト名と一致しない場合でも、正常に機能します。

この制限は、Windows Server 2019、場所、コンテナー今すぐ常に使用 gMSA 名割り当てられているホスト名に関係なく、ネットワーク上で修正されました。

**1 つ以上のコンテナーと同時に、gMSA を使用して、WS2016、1709 および 1803 で断続的なエラーにつながります。**

同じホスト名を使用するすべてのコンテナーを必要とする以前の制限] の結果としては、2 番目の問題には、バージョンの Windows Server 2019 よりも前の Windows と Windows 10 版 1809 に反映されます。
複数のコンテナーが割り当てられるは、同じ id とホスト名] と 2 つのコンテナーが同時に同じドメイン コント ローラーに問い合わせること競合状態可能性があります。
別のコンテナーには、同じドメイン コント ローラーが話してから、同じ id を使用して、以前のコンテナーが通信、キャンセルされます。
これは、断続的な認証の失敗を招くされ、実行すると、セキュリティ エラーとして観測場合があることができます`nltest /sc_verify:contoso.com`しますコンテナー内します。

同時に同じ gMSA を使用する複数のコンテナーができるようにするコンピューター名からコンテナーの id の別の Windows Server 2019 の動作を変更します。

**Windows バージョン 1703、1709、および 1803 HYPER-V 分離コンテナーが gMSAs を使うことはできません。**

コンテナー初期化はハングするか、使用 1703、1709 1803、Windows 10 および Windows Server のバージョンで HYPER-V 分離コンテナーには、gMSA しようとしたときに失敗します。

Windows Server 2019 と Windows 10、1809 のバージョンでこの問題が修正されました。 Windows Server 2016 と、Windows 10 版 1607 gMSAs に HYPER-V 分離コンテナーを実行することもできます。

### <a name="general-troubleshooting-guidance"></a>一般的なトラブルシューティング ガイダンス

場合は、gMSA でコンテナーを実行した場合、エラーが発生している、次の手順根本原因を特定できます。

**ホスト、gMSA が使用できるかどうかを確認します。**

1.  ホストがドメインの結合し、ドメイン コント ローラーにアクセスできることを確認します。
2.  RSAT から AD PowerShell ツールをインストールして、コンピューターに取得、gMSA へのアクセスが含まれている[テスト ADServiceAccount](https://docs.microsoft.com/en-us/powershell/module/activedirectory/test-adserviceaccount)を実行します。 コマンドレットが**False**が返される場合、コンピューターがない gMSA パスワードにアクセスします。

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Install-WindowsCapability -Online 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
    # To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

    Test-ADServiceAccount WebApp01
    ```
3.  テスト ADServiceAccount が**False**が返される場合は、gMSA パスワードを取得するアクセス権を持つセキュリティ グループに所属していることを確認します。

    ```powershell
    # Get the current computer's group membership
    Get-ADComputer $env:computername | Get-ADPrincipalGroupMembership | Select-Object DistinguishedName

    # Get the groups allowed to retrieve the gMSA password
    # Change "WebApp01" for your own gMSA name
    (Get-ADServiceAccount WebApp01 -Properties PrincipalsAllowedToRetrieveManagedPassword).PrincipalsAllowedToRetrieveManagedPassword
    ```

4.  ホスト gMSA パスワードを取得する権限を与えられているセキュリティ グループに属しているテスト ADServiceAccount が失敗しても場合は、現在、グループ メンバーシップを反映した新しいチケットを取得するには、コンピューターを再起動する必要があります。

**資格情報の仕様ファイルをチェックします。**

1.  実行`Get-CredentialSpec`を探し[CredentialSpec PowerShell モジュール](https://aka.ms/credspec)からすべての資格情報マシンで定義します。 Docker ルート ディレクトリに"CredentialSpecs"ディレクトリには、資格情報の定義を保存する必要があります。 実行して、[Docker ルート ディレクトリを検索できます`docker info -f "{{.DockerRootDir}}"`します。
2.  CredentialSpec ファイルを開き、次のフィールドが正しく入力したかどうかを確認します。
    -   **Sid**: gMSA アカウントの SID
    -   **MachineAccountName**: gMSA SAM アカウント名 (完全なドメイン名または含めないでドル記号)
    -   **DnsTreeName**: AD フォレストの FQDN
    -   **DnsName**: gMSA が所属しているドメインの FQDN
    -   **あります**:、gMSA が所属しているドメインの NETBIOS 名。
    -   **GroupManagedServiceAccounts/名前**: gMSA SAM アカウント名 (完全なドメイン名または含めないでドル記号)
    -   **GroupManagedServiceAccounts/スコープ**: NETBIOS のいずれかのドメインの FQDN を 1 つのエントリ

    完全な資格情報の仕様の下にある、完全な例を参照してください。

    ```json
    {
    "CmsPlugins":[
        "ActiveDirectory"
    ],
    "DomainJoinConfig":{
        "Sid":"S-1-5-21-702590844-1001920913-2680819671",
        "MachineAccountName":"webapp01",
        "Guid":"56d9b66c-d746-4f87-bd26-26760cfdca2e",
        "DnsTreeName":"contoso.com",
        "DnsName":"contoso.com",
        "NetBiosName":"CONTOSO"
    },
    "ActiveDirectoryConfig":{
        "GroupManagedServiceAccounts":[
        {
            "Name":"webapp01",
            "Scope":"contoso.com"
        },
        {
            "Name":"webapp01",
            "Scope":"CONTOSO"
        }
        ]
    }
    }
    ```

3.  資格情報の仕様ファイルへのパスがオーケストレーション ソリューションの正しいことを確認します。 Docker を使用している場合のコマンドを実行しますコンテナーが含まれています確認`--security-opt="credentialspec=file://NAME.json"`、名前の出力に**Get CredentialSpec**で"NAME.json"が置き換えられます。 名前は、[Docker ルート ディレクトリ] CredentialSpecs フォルダーに対する相対的なフラット ファイルの名前です。

**コンテナーを確認します。**

1.  Windows Server 2019 よりも前のウィンドウまたは Windows 10、バージョン、1809 のバージョンを使用している場合、コンテナー ホスト名 gMSA 名前と一致する必要があります。 確認、`--hostname`パラメーター gMSA 短い名前に一致 ("webapp01"などのないドメイン コンポーネント"webapp01.contoso.com")。

2.  しますコンテナーを確認するのには、コンテナー ネットワーク構成が解決および gMSA のドメインのドメイン コント ローラーにアクセスを確認してください。 コンテナー内の正しく構成されていない DNS サーバーは、id 問題の一般的な原因です。

3.  確認しますコンテナーの有効な接続ドメインにコンテナー内で、次のコマンドを実行して (を使用して`docker exec`または同等の)。

    ```powershell
    nltest /sc_verify:contoso.com
    ```

    場合は、gMSA が利用可能であり、ネットワーク接続により、ドメインを信頼の検証は NERR_SUCCESS を返す必要があります。
    失敗した場合は、ホストとコンテナーのネットワークの構成を確認する - 両方のドメイン コント ローラーと通信できるようにする必要があります。

4.  アプリは、 [[gMSA を使うように構成](#configuring-your-application-to-use-the-gmsa)することを確認します。 コンテナー内のユーザー アカウントを gMSA を使用する: 他のネットワーク リソースと通信するときに、システム アカウントが、gMSA を使用する代わりにときに、も変更されません。 そのため、アプリは、として gMSA id を活用するには、ネットワーク サービスまたはローカル システムを実行する必要があります。

    > [!TIP]
    > 実行する場合は、`whoami`または別のツールを使用して、コンテナー内のユーザーの現在のコンテキストを特定して gMSA 名前自体は表示されません。
    > ローカル ユーザー - ドメイン id ではなく、常にコンテナーにログインするためです。
    > アプリは、ネットワーク サービスとローカル システムを実行する必要がありますネットワーク リソースに述べてときにいつでもコンピューター アカウントを使用して、gMSA 表示されます。

5.  最後に、コンテナーが正しく構成されているようユーザーまたは他のサービスが、コンテナー化アプリに自動的に認証を行うことがない場合は、gMSA アカウントで Spn を確認します。 クライアントは、アプリケーション届くその名前が gMSA アカウントを見つけます。 その他必要があります`host`の場合、負荷分散装置またはその他の DNS 名を使用して、アプリにクライアントの接続などの gMSA Spn します。

## <a name="additional-resources"></a>その他の資料
-   [グループの管理サービス アカウントの概要](https://docs.microsoft.com/en-us/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview)
