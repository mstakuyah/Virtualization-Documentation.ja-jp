# 手順 8: Hyper-V と Windows PowerShell の使用

Hyper-V の展開、仮想マシンの作成、仮想マシンの管理の基本を確認できたので、次は PowerShell でこれらの作業の大半を自動化する方法について説明します。

### Hyper-V コマンドの一覧を返す

1.  Windows の [スタート] ボタンをクリックし、「**PowerShell**」と入力します。
2.  次のコマンドを実行すると、Hyper-V PowerShell モジュールで利用できる PowerShell コマンドの検索可能な一覧が表示されます

 ```powershell
get-command –module hyper-v | out-gridview
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

2. 電源がオンになっている仮想マシンのみの一覧を返す場合、`get-vm` コマンドにフィルターを追加します。 フィルターは where-object コマンドを利用して追加できます。 フィルター処理の詳細については、[Where-Object の使用](https://technet.microsoft.com/en-us/library/ee177028.aspx)に関するドキュメントを参照してください。

 ```powershell
 get-vm | where {$_.State –eq ‘Running’}
 ```
3.  電源がオフになっているすべての仮想マシンを一覧表示するには、次のコマンドを実行します。 このコマンドは手順 2 のコマンドをコピーしたものですが、フィルターが「Running」から「Off」に変更されています。

 ```powershell
 get-vm | where {$_.State –eq ‘Off’}
 ```

### 仮想マシンの起動とシャットダウン

1. 特定の仮想マシンを起動するには、仮想マシンの名前を指定して次のコマンドを実行します。

 ```powershell
 Start-vm –Name <virtual machine name>
 ```

2. 現在電源がオフになっているすべての仮想マシンを起動するには、そのようなマシンの一覧を取得し、「start-vm」コマンドに対して一覧を渡します。

  ```powershell
 get-vm | where {$_.State –eq ‘Off’} | start-vm
  ```
3. 実行中のすべての仮想マシンをシャットダウンするには、次を実行します。

  ```powershell
 get-vm | where {$_.State –eq ‘Running’} | stop-vm
  ```

### VM チェックポイントを作成する

PowerShell を使用してチェックポイントを作成するには、`get-vm` コマンドを使用して仮想マシンを選択し、それを `checkpoint-vm` コマンドに渡します。 最後に、`-snapshotname` を使用してチェックポイントに名前を付けます。 完全なコマンドは次のようになります。

 ```powershell
 get-vm -Name <VM Name> | checkpoint-vm -snapshotname <name for snapshot>
 ```
### 新しい仮想マシンを作成する

次の例は、PowerShell Integrated Scripting Environment (ISE) で新しい仮想マシンを作成する方法を示すものです。 これは単純な例であり、拡張して PowerShell 機能やより高度な VM 展開を追加できます。

1. PowerShell ISE を開くには、[開始] をクリックし、「**PowerShell ISE**」と入力します。
2. 次のコードを実行し、仮想マシンを作成します。 New-VM コマンドの詳細については、[New-VM](https://technet.microsoft.com/en-us/library/hh848537.aspx) ドキュメントを参照してください。

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

## まとめと参照

このドキュメントでは、Hyper-V PowerShell モジュールについて知るための簡単な手順といくつかのサンプル シナリオを紹介しました。 Hyper-V PowerShell モジュールの詳細については、「[Windows PowerShell リファレンスの Hyper-V コマンドレット](https://technet.microsoft.com/%5Clibrary/Hh848559.aspx)」を参照してください。




