---
title: &1033283708 Hyper-V と Windows PowerShell の使用
description: Hyper-V と Windows PowerShell の使用
keywords: windows 10, hyper-v
author: neilpeterson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: &1248108713 windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 6d1ae036-0841-4ba5-b7e0-733aad31e9a7
---

# Hyper-V と Windows PowerShell の使用

Hyper-V の展開、仮想マシンの作成、仮想マシンの管理の基本を確認できたので、次は PowerShell でこれらの作業の大半を自動化する方法について説明します。

### HYPER-V のコマンドの一覧を返します

1.  Windows の [スタート] ボタンをクリックし、「<g id="2" ctype="x-strong">PowerShell</g>」と入力します。
2.  次のコマンドを実行すると、Hyper-V PowerShell モジュールで利用できる PowerShell コマンドの検索可能な一覧が表示されます

 ```powershell
get-command -module hyper-v | out-gridview
 ```
  次のような一覧になります。

  <g id="1" ctype="x-linkText"></g>

3. 特定の PowerShell コマンドの詳細を確認するには、<g id="2" ctype="x-code">get-help</g> を使用します。 たとえば、次のコマンドを実行すると、<g id="2" ctype="x-code">get-vm</g> Hyper-V コマンドに関する情報が返されます。

  ```powershell
get-help get-vm
  ```
 コマンドを構築する方法、必須と任意のパラメーター、および使用できるエイリアスが出力されます。

 <g id="1" ctype="x-linkText"></g>


### 仮想マシンの一覧を返す

仮想マシンの一覧を返すには、<g id="2" ctype="x-code">get-vm</g> コマンドを使用します。

1. PowerShell で次のコマンドを実行します。

 ```powershell
get-vm
 ```
 次のような出力が表示されます。

 <g id="1" ctype="x-linkText"></g>

2. 電源がオンになっている仮想マシンのみの一覧を返す場合、<g id="2" ctype="x-code">get-vm</g> コマンドにフィルターを追加します。 フィルターは where-object コマンドを利用して追加できます。 フィルター処理の詳細については、<g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">Where-Object の使用</g><g id="2CapsExtId3" ctype="x-title"></g></g>に関するドキュメントをご覧ください。

 ```powershell
 get-vm | where {$_.State -eq ‘Running’}
 ```
3.  電源のすべての仮想マシンの一覧を表示する、状態を無効には、次のコマンドを実行します。 このコマンドは、'Running' から 'Off' に変更、フィルターを使用して、手順 2. のコマンドのコピーです。

 ```powershell
 get-vm | where {$_.State -eq ‘Off’}
 ```

### 起動し、仮想マシンをシャット ダウン

1. 特定のバーチャル マシンを開始するには、仮想マシンの名前で、次のコマンドを実行します。

 ```powershell
 Start-vm -Name <virtual machine name>
 ```

2. 現在電源がオフになっているすべての仮想マシンを起動するには、そのようなマシンの一覧を取得し、「start-vm」コマンドに対して一覧を渡します。

  ```powershell
 get-vm | where {$_.State -eq ‘Off’} | start-vm
  ```
3. 実行中のすべての仮想マシンをシャットダウンするには、次を実行します。

  ```powershell
 get-vm | where {$_.State -eq ‘Running’} | stop-vm
  ```

### VM チェックポイントを作成する

PowerShell を使用してチェックポイントを作成するには、<g id="2" ctype="x-code">get-vm</g> コマンドを使用して仮想マシンを選択し、それを <g id="4" ctype="x-code">checkpoint-vm</g> コマンドに渡します。 最後に、<g id="2" ctype="x-code">-snapshotname</g> を使用してチェックポイントに名前を付けます。 完全なコマンドは次のようになります。

 ```powershell
 get-vm -Name <VM Name> | checkpoint-vm -snapshotname <name for snapshot>
 ```
### 新しい仮想マシンを作成する

次の例は、PowerShell Integrated Scripting Environment (ISE) で新しい仮想マシンを作成する方法を示すものです。 これは単純な例であり、拡張して PowerShell 機能やより高度な VM 展開を追加できます。

1. PowerShell ISE を開くには、[開始] をクリックし、「<g id="2" ctype="x-strong">PowerShell ISE</g>」と入力します。
2. 次のコードを実行し、仮想マシンを作成します。 New-VM コマンドについて詳しくは、<g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">New-VM</g><g id="2CapsExtId3" ctype="x-title"></g></g> ドキュメントをご覧ください。

  ```powershell
 $VMName = "VMNAME"

 $VM = @{
     Name = $VMName 
     MemoryStartupBytes = 2147483648
     Generation = 2
     NewVHDPath = "C:\Virtual Machines\$VMName\$VMName.vhdx"
     NewVHDSizeBytes = 53687091200
     BootDevice = "VHD"
     Path = "C:\Virtual Machines\$VMName "
     SwitchName = (get-vmswitch).Name[0]
 }

 New-VM @VM
  ```

## ラップし、参照

このドキュメントはいくつかのサンプル シナリオと同様に、HYPER-V の PowerShell モジュールに、エクスプ ローラーにいくつかの簡単な手順を説明しました。 Hyper-V PowerShell モジュールの詳細については、「<g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">Windows PowerShell リファレンスの Hyper-V コマンドレット</g><g id="2CapsExtId3" ctype="x-title"></g></g>」をご覧ください。






<!--HONumber=May16_HO1-->


