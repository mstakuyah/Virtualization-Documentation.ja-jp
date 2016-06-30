---
author: enderb-ms
redirect_url: https://github.com/Microsoft/Virtualization-Documentation/tree/master/windows-container-samples
translationtype: Human Translation
ms.sourcegitcommit: 2bc48278e48915577149ae6c1ae1f4cb9c52cc8c
ms.openlocfilehash: dde80a4599bd099b5361b016b8547169589d4c8e

---


# .NET 3.5 Server Core コンテナー イメージの作成

このガイドでは、.NET Framework 3.5 を含んだ Windows Server Core コンテナーの作成について詳しく説明します。 この演習を開始する前に、Windows Server 2016 の .iso ファイル (Windows Server 2016 メディア) が必要です。

## メディアの準備

.NET 3.5 対応のコンテナー イメージを作成する前に、コンテナー内で使用するための準備を .NET 3.5 パッケージに対して行う必要があります。 この例では、Windows Server 2016 メディアからコンテナー ホストに `Microsoft-windows-netfx3-ondemand-package.cab` ファイルをコピーします。

コンテナー ホストに、`dotnet3.5\source` というディレクトリを作成します。

```powershell
New-Item -ItemType Directory c:\dotnet3.5\source
```

`Microsoft-windows-netfx3-ondemand-package.cab` ファイルをこのディレクトリにコピーします。 このファイルは、Windows Server 2016 メディアの sources\sxs フォルダーにあります。

```powershell
$file = "d:\sources\sxs\Microsoft-windows-netfx3-ondemand-package.cab"
Copy-Item -Path $file -Destination c:\dotnet3.5\source
``` 
    
または、コンテナーのホストが Hyper-V の仮想マシンで実行されており、クイック スタート スクリプトを使って展開されている場合は、次のコマンドを実行することもできます。 このコマンドは、コンテナーのホスト上ではなく、Hyper-V ホスト上で実行する必要があります。 

```powershell
$vm = "<Container Host VM Name>"
$iso = "$((Get-VMHost).VirtualHardDiskPath)".TrimEnd("\") + "\WindowsServerTP4.iso"
Mount-DiskImage -ImagePath $iso
$ISOImage = Get-DiskImage -ImagePath $iso | Get-Volume
$ISODrive = "$([string]$iSOImage.DriveLetter):"
Get-VM -Name $vm | Enable-VMIntegrationService -Name "Guest Service Interface"
Copy-VMFile -Name $vm -SourcePath "$iSODrive\sources\sxs\microsoft-windows-netfx3-ondemand-package.cab" -DestinationPath "c:\dotnet3.5\source\microsoft-windows-netfx3-ondemand-package.cab" -FileSource Host -CreateFullPath
Dismount-DiskImage -ImagePath $iso
```

.NET Framework 3.5 の追加先となるコンテナーのイメージを作成する準備が整いました。 この作業は、PowerShell または Docker を使って実行できます。 以降、その両方の例を紹介しています。

## イメージの作成 - PowerShell

PowerShell を使って新しいイメージを作成するには、コンテナーを作成し、必要な変更をすべて反映して、新しいイメージにキャプチャします。

Windows Server Core の基本イメージから新しいコンテナーを作成します。

```powershell
New-Container -Name dotnet35 -ContainerImageName windowsservercore -SwitchName "Virtual Switch"
```

この新しいコンテナーに共有フォルダーを作成します。 新しいコンテナー内で .NET 3.5 の cab ファイルへのアクセスを確保するために、このフォルダーが使用されます。  次のコマンドを実行するときは、コンテナーを停止する必要があります。

```powershell
Add-ContainerSharedFolder -ContainerName dotnet35 -SourcePath C:\dotnet3.5\source -DestinationPath c:\sxs
```

コンテナーを起動し、次のコマンドを実行して .NET 3.5 をインストールします。

```powershell
Start-Container dotnet35
Invoke-Command -ContainerName dotnet35 -ScriptBlock {Add-WindowsFeature -Name NET-Framework-Core -Source c:\sxs} -RunAsAdministrator
```

インストールが完了したら、コンテナーを停止します。

```powershell
Stop-Container dotnet35
```

このコンテナーからイメージを作成するには、コンテナーのホストで次のコマンドを実行します。

```powershell
New-ContainerImage -ContainerName dotnet35 -Name dotnet35 -Publisher Demo -Version 1.0
```

`Get-ContainerImages` を実行して、新しいイメージを表示します。 このイメージを使用して、.NET Framework 3.5 がプレインストールされたコンテナーを実行することができます。

```powershell
Get-ContainerImages
```

## イメージの作成 - Docker
 
Docker を使って新しいイメージを作成するには、新しいイメージの作成手順に従って dockerfile を作成します。 この dockerfile を実行すると、新しいコンテナー イメージが作成されます。 以降のコマンドは、コンテナーのホスト VM で実行します。

dockerfile を作成し、メモ帳で開きます。

```powershell
New-Item C:\dotnet3.5\dockerfile -Force
Notepad C:\dotnet3.5\dockerfile
```

次のテキストを dockerfile にコピーして保存します。

```powershell
FROM windowsservercore
ADD source /sxs
RUN powershell -Command "& {Add-WindowsFeature -Name NET-Framework-Core -Source c:\sxs}"
```

`docker build` を実行します。これにより、dockerfile を使用して新しいコンテナー イメージが作成されます。

```powershell
Docker build -t dotnet35 C:\dotnet3.5\
```

`docker images` を実行して、新しいイメージを表示します。 このイメージを使用して、.NET Framework 3.5 がプレインストールされたコンテナーを実行することができます。

```powershell
docker images
```



<!--HONumber=Jun16_HO4-->


