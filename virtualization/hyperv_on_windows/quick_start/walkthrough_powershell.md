---
title: "Hyper-V と Windows PowerShell の使用"
description: "Hyper-V と Windows PowerShell の使用"
keywords: windows 10, hyper-v
author: neilpeterson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 6d1ae036-0841-4ba5-b7e0-733aad31e9a7
translationtype: Human Translation
ms.sourcegitcommit: e14ede0a2b13de08cea0a955b37a21a150fb88cf
ms.openlocfilehash: a8e567b6447aa73f14825b7054d977d2b003a726

---

# Hyper-V と Windows PowerShell の使用

Hyper-V の展開、仮想マシンの作成、仮想マシンの管理の基本を確認できたので、次は PowerShell でこれらの作業の大半を自動化する方法について説明します。

### HYPER-V のコマンドの一覧を返します

1.  Windows の [スタート] ボタンをクリックし、「**PowerShell**」と入力します。
2.  次のコマンドを実行すると、Hyper-V PowerShell モジュールで利用できる PowerShell コマンドの検索可能な一覧が表示されます

 ```powershell
get-command -module hyper-v | out-gridview
```
  次のような一覧になります。

  ![](media\command_grid.png)

3. 特定の PowerShell コマンドの詳細を確認するには、`get-help` を使用します。 たとえば、次のコマンドを実行すると、`get-vm` Hyper-V コマンドに関する情報が返されます。

  ```powershell
get-help get-vm
```
 コマンドを構築する方法、必須と任意のパラメーター、および使用できるエイリアスが出力されます。

 ![](media\get_help.png)


### 仮想マシンの一覧を返す

仮想マシンの一覧を返すには、`get-vm` コマンドを使用します。

1. PowerShell で次のコマンドを実行します。
 
 ```powershell
get-vm
```
 次のような出力が表示されます。

 ![](media\get_vm.png)

2. 電源がオンになっている仮想マシンのみの一覧を返す場合、`get-vm` コマンドにフィルターを追加します。 フィルターは where-object コマンドを利用して追加できます。 フィルター処理の詳細については、[Where-Object の使用](https://technet.microsoft.com/en-us/library/ee177028.aspx)に関するドキュメントをご覧ください。   

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

PowerShell を使用してチェックポイントを作成するには、`get-vm` コマンドを使用して仮想マシンを選択し、それを `checkpoint-vm` コマンドに渡します。 最後に、`-snapshotname` を使用してチェックポイントに名前を付けます。 完全なコマンドは次のようになります。

 ```powershell
 get-vm -Name <VM Name> | checkpoint-vm -snapshotname <name for snapshot>
 ```
### 新しい仮想マシンを作成する

次の例は、PowerShell Integrated Scripting Environment (ISE) で新しい仮想マシンを作成する方法を示すものです。 これは単純な例であり、拡張して PowerShell 機能やより高度な VM 展開を追加できます。

1. PowerShell ISE を開くには、[開始] をクリックし、「**PowerShell ISE**」と入力します。
2. 次のコードを実行し、仮想マシンを作成します。 New-VM コマンドについて詳しくは、[New-VM](https://technet.microsoft.com/en-us/library/hh848537.aspx) ドキュメントをご覧ください。

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

このドキュメントはいくつかのサンプル シナリオと同様に、HYPER-V の PowerShell モジュールに、エクスプ ローラーにいくつかの簡単な手順を説明しました。 Hyper-V PowerShell モジュールの詳細については、[Windows PowerShell リファレンスの Hyper-V コマンドレット](https://technet.microsoft.com/%5Clibrary/Hh848559.aspx)をご覧ください。  
 


<!--HONumber=Jun16_HO4-->


