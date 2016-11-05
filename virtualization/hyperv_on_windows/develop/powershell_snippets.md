---
title: "PowerShell スニペット"
description: "PowerShell スニペット"
keywords: "windows 10、hyper-v"
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: dc33c703-c5bc-434e-893b-0c0976b7cb88
translationtype: Human Translation
ms.sourcegitcommit: ffdf89b0ae346197b9ae631ee5260e0565261c55
ms.openlocfilehash: ff356c8dad96b5705c6d698480c6f435cedd7bd9

---

# PowerShell スニペット

PowerShell は、Hyper-V 用の優れたスクリプティング、自動化、および管理のツールです。  ここでは、PowerShell のいくつかの優れた機能を紹介します。

Hyper-V 管理はすべて管理者が実行する必要があります。したがって、すべてのスクリプトおよびスニペットは、Hyper-V 管理者アカウントから管理者として実行することが前提となります。

適切なアクセス許可があるかどうかが不明な場合は、「`Get-VM`」と入力し、エラーなしで実行できれば作業を開始できます。


## PowerShell ダイレクト ツール
このセクションのすべてのスクリプトおよびスニペットは、次の基本事項に依存しています。

**要件**:  
*  PowerShell ダイレクト。  Windows 10 ゲストおよびホスト OS。

**共通の変数**:  
`$VMName` -- これは VMName を含む文字列です。  次を使用して利用可能な VM の一覧をご覧ください。 `Get-VM`  
`$cred` -- ゲスト OS の資格情報。  次を使用して設定できます。 `$cred = Get-Credential`  

### ゲストが起動したかどうかを確認する

Hyper-V マネージャーは、ゲスト オペレーティング システムに対する可視性を提供しないので、ゲスト OS が起動したかどうかがわかりにくいことがよくあります。

以下のコードは、同じ機能を 2 とおりの方法 (コード スニペットと PowerShell 関数) で示したものです。

スニペット:  
``` PowerShell
if((Invoke-Command -VMName $VMName -Credential $cred {"Test"}) -ne "Test"){Write-Host "Not Booted"} else {Write-Host "Booted"}
```  

関数:  
``` PowerShell
function waitForPSDirect([string]$VMName, $cred){
   Write-Output "[$($VMName)]:: Waiting for PowerShell Direct (using $($cred.username))"
   while ((Invoke-Command -VMName $VMName -Credential $cred {"Test"} -ea SilentlyContinue) -ne "Test") {Sleep -Seconds 1}}
```

**結果**  
わかりやすいメッセージを出力し、VM への接続が成功するまでロックします。  
通知なしに成功します。

### ゲストがネットワークを得るまでロックするスクリプト
PowerShell ダイレクトでは、仮想マシンが IP アドレスを受け取る前に、仮想マシン内の PowerShell セッションに接続することができます。

``` PowerShell
# Wait for DHCP
while ((Get-NetIPAddress | ? AddressFamily -eq IPv4 | ? IPAddress -ne 127.0.0.1).SuffixOrigin -ne "Dhcp") {sleep -Milliseconds 10}
```

**結果**: DHCP リースが受信されるまでロックします。  このスクリプトは、特定のサブネットまたは IP アドレスを探すわけではないので、使用しているネットワーク構成に関係なく機能します。  
通知なしに成功します。

## PowerShell を使用した資格情報の管理
Hyper-V スクリプトでは、多くの場合、1 つまたは複数の仮想マシン、Hyper-V ホスト、またはその両方の資格情報を処理する必要があります。

PowerShell ダイレクトまたは標準の PowerShell リモート処理を使用すると、次の複数の方法でこの操作を実行できます。

1. 1 つ目の (そして最も単純な) 方法としては、ホストとゲスト、またはローカル ホストとリモート ホストで同じユーザー資格情報を有効にしておきます。  
  Microsoft アカウントでログインしているか、またはドメイン環境にいる場合、この方法は非常に簡単です。  
  このシナリオでは、`Invoke-Command -VMName "test" {get-process}` を実行するだけで済みます。

2. PowerShell に、資格情報を求めるプロンプトを表示させる  
  資格情報が一致しない場合、資格情報を求めるプロンプトが自動的に表示されて、仮想マシンの適切な資格情報を指定できます。

3. 資格情報を再利用するために変数に格納します。
  次のように単純なコマンドを実行します。  
  ``` PowerShell
  $localCred = Get-Credential
   ```
  その後、次のようなコマンドを実行します。
  ``` PowerShell
  Invoke-Command -VMName "test" -Credential $localCred  {get-process} 
  ```
  つまり、スクリプト /PowerShell セッションごとに 1 回のみプロンプトが表示されます。

4. 資格情報をスクリプトにコーディングします。  **実際のワークロードやシステムに対しては、この操作を行わないでください。**
 > 警告: _実稼働システムではこの操作を行わないでください。実際のパスワードを使用してこの操作を行わないでください。_
  
  次のようなコードを使用して、PSCredential オブジェクトを作成できます。  
  ``` PowerShell
  $localCred = New-Object -typename System.Management.Automation.PSCredential -argumentlist "Administrator", (ConvertTo-SecureString "P@ssw0rd" -AsPlainText -Force) 
  ```
  安全性がきわめて低いものの、テストには有用です。  これで、このセッションではプロンプトは表示されなくなります。 




<!--HONumber=Oct16_HO4-->


