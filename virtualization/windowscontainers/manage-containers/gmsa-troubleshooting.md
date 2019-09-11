---
title: Windows コンテナー用の gMSAs トラブルシューティング
description: Windows コンテナーのグループ管理サービスアカウント (gMSAs) のトラブルシューティングを行う方法について説明します。
keywords: docker、コンテナー、active directory、gmsa、グループ管理サービスアカウント、グループ管理サービスアカウント、トラブルシューティング、トラブルシューティング
author: Heidilohr
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 00a0d9b1367da55b7669fc26a3eca303272967ab
ms.sourcegitcommit: 5d4b6823b82838cb3b574da3cd98315cdbb95ce2
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/11/2019
ms.locfileid: "10079744"
---
# <a name="troubleshoot-gmsas-for-windows-containers"></a>Windows コンテナー用の gMSAs トラブルシューティング

## <a name="known-issues"></a>既知の問題

### <a name="container-hostname-must-match-the-gmsa-name-for-windows-server-2016-and-windows-10-versions-1709-and-1803"></a>コンテナーのホスト名は、Windows Server 2016 および Windows 10 のバージョン1709および1803の gMSA 名と一致する必要があります。

Windows Server 2016、バージョン1709、または1803を実行している場合、コンテナーの hostname は gMSA SAM アカウント名と一致する必要があります。

ホスト名が gMSA 名と一致しない場合、受信 NTLM 認証要求と名前/SID 翻訳 (ASP.NET メンバーシップロールプロバイダーなどの多くのライブラリで使用) は失敗します。 Kerberos は、hostname と gMSA name が一致しない場合でも、通常どおり機能します。

この制限は Windows Server 2019 で修正されました。この制限は、割り当てられたホスト名に関係なく、コンテナーが常にネットワーク上で gMSA 名を使用するようになっています。

### <a name="using-a-gmsa-with-more-than-one-container-simultaneously-leads-to-intermittent-failures-on-windows-server-2016-and-windows-10-versions-1709-and-1803"></a>複数のコンテナーで gMSA を同時に使用すると、Windows Server 2016 および Windows 10 バージョン1709および1803で断続的なエラーが発生する場合があります。

すべてのコンテナーは同じホスト名を使う必要があるため、第2の問題は Windows Server 2019 および Windows 10 バージョン1809より前のバージョンの Windows に影響します。 複数のコンテナーに同じ id と hostname が割り当てられている場合、2つのコンテナーが同じドメインコントローラーと同時に対話するときに競合状態が発生することがあります。 別のコンテナーと同じドメインコントローラーとの通信が行われると、同じ id を使用する前のコンテナーとの通信がキャンセルされます。 これにより、認証エラーが断続的に発生したり、コンテナー内で実行`nltest /sc_verify:contoso.com`したときに信頼エラーとして観察されることがあります。

Windows Server 2019 の動作を変更して、コンピューター名からコンテナー id を分離することで、複数のコンテナーで同じ gMSA を同時に使うことができるようになりました。

### <a name="you-cant-use-gmsas-with-hyper-v-isolated-containers-on-windows-10-versions-1703-1709-and-1803"></a>Windows 10 バージョン1703、1709、および1803では、Hyper-v として gMSAs 使用できません。

Windows 10 および Windows Server バージョン1703、1709、1803で、Hyper-v 分離コンテナーで gMSA を使用しようとすると、コンテナーの初期化がハングするか、失敗します。

このバグは、Windows Server 2019 および Windows 10 バージョン1809で修正されました。 また、Windows Server 2016 および Windows 10 バージョン1607では、"Hyper-v" を使って Hyper-v 分離コンテナーを実行することもできます。

## <a name="general-troubleshooting-guidance"></a>一般的なトラブルシューティングガイダンス

GMSA でコンテナーを実行しているときにエラーが発生した場合は、次の手順に従って根本原因を特定することができます。

### <a name="make-sure-the-host-can-use-the-gmsa"></a>ホストで gMSA を使用できることを確認します。

1. ホストがドメインに参加していて、ドメインコントローラーに接続できることを確認します。
2. RSAT から AD PowerShell ツールをインストールし、[テスト-ADServiceAccount](https://docs.microsoft.com/powershell/module/activedirectory/test-adserviceaccount)を実行して、コンピューターに gMSA を取得するためのアクセス許可があるかどうかを確認します。 コマンドレットで**False**が返される場合、コンピューターには gMSA パスワードへのアクセス権がありません。

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Add-WindowsCapability -Online -Name 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
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

3. オーケストレーションソリューションの資格情報スペシフィケーションファイルへのパスが正しいことを確認します。 Docker を使っている場合は、コンテナーの run コマンドに`--security-opt="credentialspec=file://NAME.json"`"name" という名前が含まれていることを確認**してください。** この名前は、Docker ルートディレクトリの [CredentialSpecs] フォルダーを基準としたフラットなファイル名です。

### <a name="check-the-firewall-configuration"></a>ファイアウォールの構成を確認する

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

### <a name="check-the-container"></a>コンテナーを確認する

1. Windows Server 2019 または Windows 10 バージョン1809より前のバージョンの Windows を実行している場合は、コンテナーのホスト名が gMSA 名と一致する必要があります。 パラメーターが`--hostname` gMSA short 名 ("webapp01.contoso.com" ではなく "webapp01" など) と一致することを確認します。

2. コンテナーネットワーク構成を確認して、コンテナーが gMSA のドメインのドメインコントローラーを解決してアクセスできることを確認します。 コンテナーの DNS サーバーが正しく構成されていないと、id の問題が発生する原因となります。

3. コンテナーのドメインへの有効な接続があるかどうかを確認するに`docker exec`は、コンテナー (または同等のもの) で次のコマンドレットを実行します。

    ```powershell
    nltest /sc_verify:contoso.com
    ```

    GMSA が利用可能で`NERR_SUCCESS`あり、ネットワーク接続によってコンテナーがドメインと通信できる場合は、信頼の確認が返されます。 失敗した場合は、ホストとコンテナーのネットワーク構成を確認します。 どちらの場合も、ドメインコントローラーと通信できる必要があります。

4. アプリが gMSA を[使用するように構成](gmsa-configure-app.md)されていることを確認します。 GMSA を使用しても、コンテナー内のユーザーアカウントは変更されません。 代わりに、システムアカウントは、他のネットワークリソースと通信するときに gMSA を使います。 つまり、gMSA id を使用するには、アプリがネットワークサービスまたはローカルシステムとして実行されている必要があります。

    > [!TIP]
    > コンテナー内の`whoami`現在のユーザーコンテキストを特定するために別のツールを実行するか、別のツールを使用すると、gMSA 名自体は表示されません。 これは、ドメイン id ではなく、ローカルユーザーとしてコンテナーに常にサインインしているためです。 GMSA は、コンピューターアカウントがネットワークリソースと通信するたびに使用されます。これは、アプリがネットワークサービスまたはローカルシステムとして実行する必要がある理由です。

5. 最後に、コンテナーが正しく構成されているように見えても、ユーザーまたは他のサービスがコンテナー内のアプリに自動的に認証できない場合は、gMSA アカウントの Spn を確認します。 クライアントは、アプリケーションに到達したときの名前で gMSA account を検索します。 これは、たとえば、クライアントがロード`host`バランサーまたは別の DNS 名を使用してアプリに接続している場合に、gMSA に追加の spn が必要になることを意味します。
