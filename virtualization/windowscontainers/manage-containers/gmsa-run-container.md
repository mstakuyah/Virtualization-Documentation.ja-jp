---
title: GMSA でコンテナーを実行する
description: グループ管理サービスアカウント (gMSA) を使用して Windows コンテナーを実行する方法について説明します。
keywords: docker、コンテナー、active directory、gmsa、グループ管理サービスアカウント、グループ管理サービスアカウント
author: rpsqrd
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 52625517748356251aa41115caebd7801ec3cdaf
ms.sourcegitcommit: 22dcc1400dff44fb85591adf0fc443360ea92856
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/12/2019
ms.locfileid: "10209864"
---
# <a name="run-a-container-with-a-gmsa"></a>GMSA でコンテナーを実行する

グループ管理サービスアカウント (gMSA) を使用してコンテナーを実行するには、次のよう`--security-opt`に[docker run](https://docs.docker.com/engine/reference/run)のパラメーターに credential spec ファイルを指定します。

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

信頼された DC 接続状態と信頼の確認状態`NERR_Success`が見つからない場合は、[トラブルシューティングの手順](gmsa-troubleshooting.md#check-the-container)に従って問題をデバッグします。

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

## <a name="next-steps"></a>次のステップ

コンテナーの実行に加えて、gMSAs 次のように使うこともできます。

- [アプリを構成する](gmsa-configure-app.md)
- [オーケストレーションコンテナー](gmsa-orchestrate-containers.md)

セットアップ時に問題が発生した場合は、可能な解決策については、[トラブルシューティングガイド](gmsa-troubleshooting.md)を確認してください。
