---
title: PowerShell ダイレクトでの Windows Virtual Machines の管理
description: PowerShell ダイレクトでの Windows Virtual Machines の管理
keywords: windows 10, hyper-v
author: scooley
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: fb228e06-e284-45c0-b6e6-e7b0217c3a49
---

# PowerShell ダイレクトでの Windows Virtual Machines の管理
 
PowerShell のダイレクトを使用すると、Windows 10 または Windows Server テクニカル プレビュー HYPER-V ホストから 10 の Windows または Windows Server に関するテクニカル プレビューの仮想マシンをリモートで管理します。 PowerShell ダイレクトでは、Hyper-V ホストまたは仮想マシンのいずれかにおけるネットワーク構成またはリモート管理設定に関係なく、仮想マシン内の PowerShell の管理ができます。 これにより、Hyper-V 管理者は管理タスクと構成タスクを簡単に自動化およびスクリプト化できるようになります。

**PowerShell ダイレクトを実行する方法:**  
* 対話型セッションとして -- Enter-PSSession を使用して対話型 PowerShell セッションを作成および終了するには、[ここをクリック](vmsession.md#create-and-exit-an-interactive-powershell-session)してください。
* 単一のコマンドまたはスクリプトを実行する一時使用セッションとして -- Invoke-Command を使用してスクリプトまたはコマンド実行するには[ここをクリック](vmsession.md#run-a-script-or-command-with-invoke-command)してください。
* 永続的なセッションとして (ビルド 14280 およびそれ以降) -- New-PSSession を使用して永続的なセッションを作成するには[ここをクリック](vmsession.md#copy-files-with-New-PSSession-and-Copy-Item)してください。  
続行するには、Copy-Item を使用してファイルを仮想マシンにコピーおよび仮想マシンからコピーし、接続を切断するには Remove-PSSession を使用します。

## 要件
**オペレーティング システムの要件:**
* ホスト: Windows 10、Windows Server Technical Preview 2、または Hyper-V を実行するそれ以降のバージョン。
* ゲスト/仮想マシン: Windows 10、Windows Server Technical Preview 2、またはそれ以降のバージョン。

以前の仮想マシンを管理している場合は、仮想マシン接続 (VMConnect) を使用するか、[仮想マシン用の仮想ネットワークを構成します](http://technet.microsoft.com/library/cc816585.aspx)。 

**構成に必要な条件:**    
* 仮想マシンがホスト上でローカルに実行されている必要があります。
* 仮想マシンが有効であり、少なくとも 1 つの構成済みユーザー プロファイルで実行されている必要があります。
* HYPER-V の管理者として、ホスト コンピューターには、ログインする必要があります。
* 仮想マシンの有効なユーザー資格情報を指定する必要があります。

-------------

## 対話型 PowerShell セッションの作成と終了

仮想マシンで PowerShell コマンドを実行するには、対話型セッションを開始するのが最も簡単な方法です。

セッションが始まると、入力したコマンドが、仮想マシン自体の PowerShell セッションに直接入力されたかのように、仮想マシンで実行されます。

**対話型セッションを開始するには:**

1. Hyper-V ホストで PowerShell を管理者として開きます。

2. 次のいずれかのコマンドを実行すると、仮想マシンの名前または GUID を使用して対話型セッションが作成されます。  
  
  ``` PowerShell
  Enter-PSSession -VMName <VMName>
  Enter-PSSession -VMId <VMId>
  ```
  
  メッセージが表示されたら仮想マシンの資格情報を指定します。

3. 仮想マシンでコマンドを実行します。
  
  PowerShell プロンプトのプレフィックスとして VMName が次のように表示されます。
  
  ``` 
  [VMName]: PS C:\ >
  ```
  
  仮想マシンで任意のコマンドが実行されます。  テストのために、`ipconfig` または `hostname` を実行して、これらのコマンドが仮想マシンで実行されていることを確認します。
  
4. 次のように完了したら、セッションを終了するには、次のコマンドを実行します。  
  
   ``` PowerShell
   Exit-PSSession 
   ``` 

> 注: セッションが接続されない場合は、考えられる原因の「[トラブルシューティング](vmsession.md#troubleshooting)」を参照してください。 

これらのコマンドレットの詳細については、「[Enter-PSSession](http://technet.microsoft.com/library/hh849707.aspx)」と「[Exit-PSSession](http://technet.microsoft.com/library/hh849743.aspx)」を参照してください。 

-------------

## Invoke-Command でのスクリプトまたはコマンドの実行

PowerShell ダイレクトと Invoke-Command コマンドの組み合わせは、1 つのコマンドまたは 1 つのスクリプトを仮想マシン上で実行する必要があるが、それらの実行以降は仮想マシンとの対話を続行する必要がないという場合に最適です。

**単一のコマンドを実行するには:**

1. Hyper-V ホストで PowerShell を管理者として開きます。

2. 次のいずれかのコマンドを実行すると、仮想マシンの名前または GUID を使用してセッションが作成されます。  
   
   ``` PowerShell
   Invoke-Command -VMName <VMName> -ScriptBlock { cmdlet } 
   Invoke-Command -VMId <VMId> -ScriptBlock { cmdlet }
   ```
   
   メッセージが表示されたら仮想マシンの資格情報を指定します。
   
   仮想マシンでコマンドが実行されます。コンソールへの出力がある場合は、その内容がコンソールに出力されます。  コマンドが実行されるとすぐに、接続が自動的に切断されます。
   
   
**スクリプトを実行するには:**

1. Hyper-V ホストで PowerShell を管理者として開きます。

2. 次のいずれかのコマンドを実行すると、仮想マシンの名前または GUID を使用してセッションが作成されます。  
   
   ``` PowerShell
   Invoke-Command -VMName <VMName> -FilePath C:\host\script_path\script.ps1 
   Invoke-Command -VMId <VMId> -FilePath C:\host\script_path\script.ps1 
   ```
   
   メッセージが表示されたら仮想マシンの資格情報を指定します。
   
   仮想マシンでスクリプトが実行されます。  コマンドが実行されるとすぐに、接続が自動的に切断されます。

このコマンドレットの詳細については、「[Invoke-Command](http://technet.microsoft.com/library/hh849719.aspx)」を参照してください。 

-------------

## New-PSSession および Copy-Item でファイルをコピーする

> **注:** PowerShell ダイレクトでは、Windows ビルド 14280 以降でのみ永続的なセッションをサポートしています。

永続的な PowerShell セッションは、1 つまたは複数のリモート コンピューターを対象にした動作を調整するスクリプトを記述する際に非常に便利です。  いったん作成された永続的なセッションは、それを削除するまで、バック グラウンドに保持されます。  すなわち、資格情報を渡さなくても`Invoke-Command` または `Enter-PSSession` を使用して、同じセッションを何度でも繰り返し参照することができます。

セッションは同じトークンにより状態を保持します。  永続的なセッションは存続するので、セッションで作成された変数またはセッションに渡された変数はいずれも複数回の呼び出しにわたり保持されます。 永続的なセッションでの作業で使用できるさまざまなツールがあります。  たとえば、[New-PSSession](https://technet.microsoft.com/en-us/library/hh849717.aspx) および [Copy-Item](https://technet.microsoft.com/en-us/library/hh849793.aspx) を使用して、ホストから仮想マシンへ、仮想マシンからホストへデータを移動することができます。

**セッションを作成し、ファイルをコピーするには:**  

1. Hyper-V ホストで PowerShell を管理者として開きます。

2. 次のいずれかのコマンドを実行すると、`New-PSSession` を使用して仮想マシンに対する永続的な PowerShell セッションが作成されます。
  
  ``` PowerShell
  $s = New-PSSession -VMName <VMName> -Credential (Get-Credential)
  $s = New-PSSession -VMId <VMId> -Credential (Get-Credential)
  ```
  
  メッセージが表示されたら仮想マシンの資格情報を指定します。
  
  > **警告:**  
   14500 より前のビルドにはバグがあります。  `-Credential` フラグで資格情報が明示的に指定されていない場合、ゲスト内のサービスはクラッシュするので、再起動する必要があります。  この問題が発生した場合、解決策については[こちら](vmsession.md#error-a-remote-session-might-have-ended)を参照してください。
  
3. 仮想マシンにファイルをコピーします。
  
  ホスト マシンから仮想マシンに `C:\host_path\data.txt` をコピーするには、次を実行します。
  
  ``` PowerShell
  Copy-Item -ToSession $s -Path C:\host_path\data.txt -Destination C:\guest_path\
  ```
  
4.  仮想マシンから (ホストに) ファイルをコピーします。 
   
   仮想マシンからホストに `C:\guest_path\data.txt` をコピーするには、次を実行します。
  
  ``` PowerShell
  Copy-Item -FromSession $s -Path C:\guest_path\data.txt -Destination C:\host_path\
  ```

5. `Remove-PSSession` を使用して永続的なセッションを停止します。
  
  ``` PowerShell 
  Remove-PSSession $s
  ```
  
-------------

## トラブルシューティング

PowerShell Direct を通じて表示される一般的なエラー メッセージが少しあります。  いくつかの最も一般的な原因、および問題を診断するためのツールは、次のとおりです。

### -VMName または -VMID パラメーターが存在しない
**問題:**  
`Enter-PSSession``Invoke-Command` または `New-PSSession` に `-VMName` パラメーターまたは `-VMId` パラメーターがありません。

**考えられる原因:**  
最も可能性の高い原因として、PowerShell ダイレクトがご使用のホスト オペレーティング システムでサポートされないことが挙げられます。

Windows ビルドは、次のコマンドを実行して確認できます。

``` PowerShell
[System.Environment]::OSVersion.Version
```

サポートされたビルドを実行していても、ご使用の PowerShell バージョンで PowerShell ダイレクトが実行されない可能もあります。  PowerShell ダイレクトと JEA については、メジャー バージョンが 5 以降である必要があります。

PowerShell バージョンのビルドは、次のコマンドを実行して確認できます。

``` PowerShell
$PSVersionTable.PSVersionTable
```


### エラー: リモート セッションが終了した可能性がある
> **注:**  
10240 ～ 12400 のホスト ビルドでの Enter-PSSession の場合、以下に示すエラーはすべて、"リモート セッションが終了した可能性がある" とレポートされます。

**エラー メッセージ:**
```
Enter-PSSession : An error has occurred which Windows PowerShell cannot handle. A remote session might have ended.
```

**考えられる原因:**
* 仮想マシンは存在しますが、まだ実行されていません。
* ゲスト OS が PowerShell Direct をサポートしていない (「[要件](#Requirements)」を参照してください)
* PowerShell がまだゲストで使用できない
  * オペレーティング システムが起動を完了していない
  * オペレーティング システムが正しく起動できない
  * ユーザー入力が必要な起動時のイベントがある

[GeT-VM](http://technet.microsoft.com/library/hh848479.aspx) コマンドレットを使用すれば、どの VM がホストで実行されているかを確認することができます。

**エラー メッセージ:**  
```
New-PSSession : An error has occurred which Windows PowerShell cannot handle. A remote session might have ended.
```

**考えられる原因:**
* 上記に一覧した理由のいずれかです (これらはすべて にも同様に適用されます)。 `New-PSSession`  
* 現在のビルドにバグがあります。`-Credential` で資格情報を明示的に渡す必要があります。  このエラーが発生すると、ゲスト オペレーティング システムでサービス全体がハングし、再起動する必要があります。  Enter-PSSession を使用すれば、セッションが引き続き使用できるかどうかを確認できます。

資格情報の問題を回避するには、VMConnect を使用して仮想マシンにログインし、PowerShell を開き、次の PowerShell を使用して vmicvmsession サービスを再開します。

``` PowerShell
Restart-Service -Name vmicvmsession
```

### エラー: パラメーター セットを解決できません。
**エラー メッセージ:**  
``` 
Enter-PSSession : Parameter set cannot be resolved using the specified named parameters.
```

**考えられる原因:**  
* `-RunAsAdministrator` 仮想マシンに接続するときに がサポートされません。
     
  Windows コンテナーに接続するときは、`-RunAsAdministrator` フラグを指定すると、明示的な資格情報がなくても管理者として接続できます。  仮想マシンはホストに暗黙の管理者アクセス権を付与しないため、資格情報を明示的に入力する必要があります。

仮想マシンに管理者の資格情報を渡すには、`-Credential` パラメーターを使用するか、要求されたときに手動で入力します。


### エラー: 資格情報が無効です。

**エラー メッセージ:**  
```
Enter-PSSession : The credential is invalid.
```

**考えられる原因:** 
* ゲストの資格情報を検証できない
  * 指定された資格情報が誤っていた。
  * ゲストにユーザー アカウントがない (前もって OS が起動されていない)
  * 管理者として接続する場合: 管理者がアクティブなユーザーとして設定されていない。  詳細については、[こちら](https://technet.microsoft.com/en-us/library/hh825104.aspx)をご覧ください。
  
### エラー: 入力 VMName パラメーターが仮想マシンに解決されません。

**エラー メッセージ:**  
```
Enter-PSSession : The input VMName parameter does not resolve to any virtual machine.
```

**考えられる原因:**  
* Hyper-V の管理者ではありません。  
* 仮想マシンが存在しません。

[Get-VM](http://technet.microsoft.com/library/hh848479.aspx) コマンドレットを使用して、使用している資格情報に Hyper-V 管理者の役割があることを確認し、ホスト上でローカルで実行されている、起動した VM を確認することができます。


-------------

## サンプルとユーザー ガイド

PowerShell ダイレクトでは JEA (Just Enough Administration) をサポートしています。  これを試すには、このユーザー ガイドをご覧ください。

[GitHub](https://github.com/Microsoft/Virtualization-Documentation/search?l=powershell&q=-VMName+OR+-VMGuid&type=Code&utf8=%E2%9C%93) のサンプルをご確認ください。

実際の環境で PowerShell Direct を使用する方法を示すさまざまな例や、PowerShell で Hyper-V スクリプトを記述するためのヒントとコツについては、「[PowerShell スニペット](../develop/powershell_snippets.md)」を参照してください。


<!--HONumber=May16_HO4-->


