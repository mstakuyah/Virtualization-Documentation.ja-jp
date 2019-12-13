---
title: Windows コンテナーの gMSAs のトラブルシューティング
description: Windows コンテナーのグループの管理されたサービスアカウント (gMSAs) のトラブルシューティングを行う方法について説明します。
keywords: docker、コンテナー、active directory、gmsa、グループの管理されたサービスアカウント、グループの管理されたサービスアカウント、トラブルシューティング、トラブルシューティング
author: rpsqrd
ms.date: 10/03/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 89f255e307c2a48fd743d5abd1a49bba7703aaf3
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910242"
---
# <a name="troubleshoot-gmsas-for-windows-containers"></a>Windows コンテナーの gMSAs のトラブルシューティング

## <a name="known-issues"></a>既知の問題

### <a name="container-hostname-must-match-the-gmsa-name-for-windows-server-2016-and-windows-10-versions-1709-and-1803"></a>コンテナーのホスト名は、Windows Server 2016 と Windows 10、バージョン1709および1803の gMSA 名と一致する必要があります

Windows Server 2016 (バージョン1709または 1803) を実行している場合は、コンテナーのホスト名と gMSA SAM アカウント名が一致している必要があります。

ホスト名が gMSA 名と一致しない場合、受信 NTLM 認証要求と名前/SID 変換 (ASP.NET メンバーシップロールプロバイダーなど、多くのライブラリで使用されます) は失敗します。 ホスト名と gMSA 名が一致しない場合でも、Kerberos は引き続き正常に機能します。

この制限は、割り当てられたホスト名に関係なく、コンテナーが常にネットワーク上で gMSA 名を使用するようになった Windows Server 2019 で修正されました。

### <a name="using-a-gmsa-with-more-than-one-container-simultaneously-leads-to-intermittent-failures-on-windows-server-2016-and-windows-10-versions-1709-and-1803"></a>複数のコンテナーで gMSA を同時に使用すると、Windows Server 2016 と Windows 10、バージョン1709、および1803で断続的なエラーが発生します。

すべてのコンテナーで同じホスト名を使用する必要があるため、2つ目の問題は、Windows Server 2019 および Windows 10 version 1809 より前の Windows のバージョンに影響します。 複数のコンテナーに同じ id とホスト名が割り当てられている場合、2つのコンテナーが同じドメインコントローラーと同時に通信するときに競合状態が発生する可能性があります。 別のコンテナーが同じドメインコントローラーと通信すると、同じ id を使用する以前のコンテナーとの通信が取り消されます。 このため、断続的な認証エラーが発生し、コンテナー内で `nltest /sc_verify:contoso.com` を実行すると、信頼エラーとして確認されることがあります。

Windows Server 2019 の動作を変更して、コンテナー id をコンピューター名と分離し、複数のコンテナーで同じ gMSA を同時に使用できるようにしました。

### <a name="you-cant-use-gmsas-with-hyper-v-isolated-containers-on-windows-10-versions-1703-1709-and-1803"></a>Windows 10 バージョン1703、1709、および1803では、gMSAs を Hyper-v 分離コンテナーと共に使用することはできません。

Windows 10 および Windows Server バージョン1703、1709、および1803で Hyper-v 分離コンテナーと共に gMSA を使用しようとすると、コンテナーの初期化が停止または失敗します。

このバグは、Windows Server 2019 および Windows 10 version 1809 で修正されました。 また、Windows Server 2016 および Windows 10 version 1607 で gMSAs を使用して、Hyper-v 分離コンテナーを実行することもできます。

## <a name="general-troubleshooting-guidance"></a>一般的なトラブルシューティング ガイダンス

GMSA を使用してコンテナーを実行するときにエラーが発生した場合は、次の手順に従って、根本的な原因を特定することができます。

### <a name="make-sure-the-host-can-use-the-gmsa"></a>ホストで gMSA を使用できることを確認します。

1. ホストがドメインに参加していて、ドメインコントローラーに接続できることを確認します。
2. RSAT から AD PowerShell ツールをインストールし、[テスト ADServiceAccount](https://docs.microsoft.com/powershell/module/activedirectory/test-adserviceaccount)を実行して、コンピューターが gMSA を取得するためのアクセス許可を持っているかどうかを確認します。 コマンドレットから**False**が返された場合、コンピューターは gMSA パスワードにアクセスできません。

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Add-WindowsCapability -Online -Name 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
    # To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

    Test-ADServiceAccount WebApp01
    ```

3. **テスト ADServiceAccount**が**False**を返す場合は、gMSA パスワードにアクセスできるセキュリティグループにホストが属していることを確認します。

    ```powershell
    # Get the current computer's group membership
    Get-ADComputer $env:computername | Get-ADPrincipalGroupMembership | Select-Object DistinguishedName

    # Get the groups allowed to retrieve the gMSA password
    # Change "WebApp01" for your own gMSA name
    (Get-ADServiceAccount WebApp01 -Properties PrincipalsAllowedToRetrieveManagedPassword).PrincipalsAllowedToRetrieveManagedPassword
    ```

4. GMSA パスワードの取得が承認されていても、まだ失敗している**テスト ADServiceAccount**のセキュリティグループにホストが属している場合は、コンピューターを再起動して、現在のグループメンバーシップを反映する新しいチケットを取得する必要があります。

#### <a name="check-the-credential-spec-file"></a>資格情報の仕様ファイルを確認する

1. [Credentialspec PowerShell モジュール](https://aka.ms/credspec)から**Get credentialspec**を実行して、コンピューター上のすべての資格情報の仕様を検索します。 資格情報の仕様は、Docker ルートディレクトリの下の "CredentialSpecs" ディレクトリに格納されている必要があります。 Docker**情報-f "{{を実行すると、docker ルートディレクトリを見つけることができます。DockerRootDir}} "**
2. CredentialSpec ファイルを開き、次のフィールドが正しく入力されていることを確認します。
    - **Sid**: gMSA アカウントの sid
    - **Machineaccountname**: GMSA SAM アカウント名 (完全なドメイン名またはドル記号を含まない)
    - **Dnstreename**: Active Directory フォレストの FQDN
    - **DnsName**: gMSA が属しているドメインの FQDN
    - **NetBiosName**: gMSA が属しているドメインの NETBIOS 名
    - **Groupmanagedserviceaccounts/Name**: gMSA SAM アカウント名 (完全なドメイン名またはドル記号は含まない)
    - **Groupmanagedserviceaccounts/Scope**: ドメイン FQDN と NETBIOS 用の1つのエントリ

    入力内容は、完全な資格情報の仕様の次の例のようになります。

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

3. 資格情報仕様ファイルへのパスが、オーケストレーションソリューションに対して正しいことを確認します。 Docker を使用している場合は、container run コマンドに `--security-opt="credentialspec=file://NAME.json"`が含まれていることを確認してください。ここで、"NAME. json" は、 **Get-CredentialSpec**によって出力される名前に置き換えられます。 名前は、Docker ルートディレクトリの下にある CredentialSpecs フォルダーに対して相対的なフラットファイル名です。

### <a name="check-the-firewall-configuration"></a>ファイアウォールの構成を確認する

コンテナーまたはホストネットワークで厳格なファイアウォールポリシーを使用している場合は、Active Directory ドメインコントローラーまたは DNS サーバーへの必要な接続をブロックする可能性があります。

| プロトコルおよびポート | 目的 |
|-------------------|---------|
| TCP および UDP 53 | DNS |
| TCP および UDP 88 | Kerberos |
| TCP 139 | NetLogon |
| TCP および UDP 389 | LDAP |
| TCP 636 | LDAP SSL |

コンテナーがドメインコントローラーに送信するトラフィックの種類によっては、追加のポートへのアクセスを許可する必要がある場合があります。
Active Directory によって使用されるポートの完全な一覧については[、「Active Directory および Active Directory Domain Services のポート要件](https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd772723(v=ws.10)#communication-to-domain-controllers)」を参照してください。

### <a name="check-the-container"></a>コンテナーを確認する

1. Windows Server 2019 または Windows 10 version 1809 より前のバージョンの Windows を実行している場合は、コンテナーのホスト名が gMSA 名と一致している必要があります。 `--hostname` パラメーターが gMSA short 名 ("webapp01.contoso.com" の代わりに "webapp01" など) と一致していることを確認します。

2. コンテナーのネットワーク構成を調べて、コンテナーが解決し、gMSA のドメインのドメインコントローラーにアクセスできることを確認します。 コンテナーに正しく構成されていない DNS サーバーは、id の問題の一般的な原因です。

3. コンテナーで次のコマンドレットを実行して、コンテナーにドメインへの有効な接続があるかどうかを確認します (`docker exec` またはそれと同等のものを使用します)。

    ```powershell
    nltest /sc_verify:contoso.com
    ```

    GMSA が使用可能であり、ネットワーク接続によってコンテナーがドメインと通信できる場合は、信頼の検証によって `NERR_SUCCESS` が返されます。 失敗した場合は、ホストとコンテナーのネットワーク構成を確認します。 どちらも、ドメインコントローラーと通信できる必要があります。

4. コンテナーが有効な Kerberos チケット保証チケット (TGT) を取得できるかどうかを確認します。

    ```powershell
    klist get krbtgt
    ```

    このコマンドは、"krbtgt へのチケットが正常に取得されました" を返し、チケットを取得するために使用されるドメインコントローラーの一覧を表示します。 TGT を取得できても、前の手順の `nltest` が失敗した場合は、gMSA アカウントが正しく構成されていないことを示している可能性があります。 詳細について[は、「gMSA アカウントの確認](#check-the-gmsa-account)」を参照してください。

    コンテナー内の TGT を取得できない場合は、DNS またはネットワーク接続の問題を示している可能性があります。 コンテナーがドメイン DNS 名を使用してドメインコントローラーを解決できること、およびドメインコントローラーがコンテナーからルーティング可能であることを確認します。

5. [GMSA を使用するように](gmsa-configure-app.md)アプリが構成されていることを確認します。 GMSA を使用する場合、コンテナー内のユーザーアカウントは変更されません。 システムアカウントでは、他のネットワークリソースと通信するときに gMSA が使用されます。 つまり、アプリは、gMSA id を利用するために、ネットワークサービスまたはローカルシステムとして実行する必要があります。

    > [!TIP]
    > `whoami` を実行するか、別のツールを使用してコンテナー内の現在のユーザーコンテキストを識別すると、gMSA 名自体は表示されません。 これは、常にドメイン id ではなくローカルユーザーとしてコンテナーにサインインするためです。 GMSA は、コンピューターアカウントがネットワークリソースと通信するときに常に使用されます。これは、アプリケーションを Network Service または Local System として実行する必要があるためです。

### <a name="check-the-gmsa-account"></a>GMSA アカウントを確認する

1. コンテナーが正しく構成されているように見えても、ユーザーまたは他のサービスがコンテナー化されたアプリに対して自動的に認証できない場合は、gMSA アカウントの Spn を確認します。 クライアントは、アプリケーションに到着した名前で gMSA アカウントを検索します。 これは、たとえば、クライアントがロードバランサーまたは別の DNS 名を使用してアプリに接続する場合に、gMSA に追加の `host` Spn が必要になる可能性があることを意味します。

2. GMSA ホストとコンテナーホストが同じ Active Directory ドメインに属していることを確認します。 GMSA が別のドメインに属している場合、コンテナーホストは gMSA パスワードを取得できません。

3. GMSA と同じ名前のアカウントがドメイン内に1つだけ存在することを確認します。 gMSA オブジェクトには、SAM アカウント名にドル記号 ($) が付加されているので、gMSA には "myaccount $" という名前を付け、関連付けられていないユーザーアカウントを同じドメイン内で "myaccount" という名前にすることができます。 ドメインコントローラーまたはアプリケーションが名前で gMSA を検索する必要がある場合に、問題が発生する可能性があります。 次のコマンドを使用して、同じ名前のオブジェクトに対して AD を検索できます。

    ```powershell
    # Replace "GMSANAMEHERE" with your gMSA account name (no trailing dollar sign)
    Get-ADObject -Filter 'sAMAccountName -like "GMSANAMEHERE*"'
    ```

4. GMSA アカウントで制約なしの委任を有効にした場合は、 [UserAccountControl 属性](https://support.microsoft.com/en-us/help/305144/how-to-use-useraccountcontrol-to-manipulate-user-account-properties)の `WORKSTATION_TRUST_ACCOUNT` フラグが有効になっていることを確認してください。 このフラグは、アプリが SID に対して名前を解決する必要がある場合と同様に、コンテナー内の NETLOGON がドメインコントローラーと通信する場合に必要です。 次のコマンドを使用して、フラグが正しく構成されているかどうかを確認できます。

    ```powershell
    $gMSA = Get-ADServiceAccount -Identity 'yourGmsaName' -Properties UserAccountControl
    ($gMSA.UserAccountControl -band 0x1000) -eq 0x1000
    ```

    上記のコマンドで `False`が返された場合は、次のコマンドを使用して、gMSA アカウントの UserAccountControl プロパティに `WORKSTATION_TRUST_ACCOUNT` フラグを追加します。 このコマンドは、UserAccountControl プロパティから `NORMAL_ACCOUNT`、`INTERDOMAIN_TRUST_ACCOUNT`、および `SERVER_TRUST_ACCOUNT` フラグもクリアします。

    ```powershell
    Set-ADObject -Identity $gMSA -Replace @{ userAccountControl = ($gmsa.userAccountControl -band 0x7FFFC5FF) -bor 0x1000 }
    ```
