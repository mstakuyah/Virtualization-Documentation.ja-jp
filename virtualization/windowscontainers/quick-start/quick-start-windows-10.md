---
title: "Windows 10 の Windows コンテナー"
description: "コンテナー展開のクイック スタート"
keywords: "Docker, コンテナー"
author: enderb-ms
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
translationtype: Human Translation
ms.sourcegitcommit: 54eff4bb74ac9f4dc870d6046654bf918eac9bb5
ms.openlocfilehash: 8aba9f2ef619b79e7459d7fd55bd27cf107621b3

---

# Windows 10 の Windows コンテナー

この演習では、Windows 10 Professional または Enterprise (Anniversary Edition) の Windows コンテナー機能の基本的な展開と使用について段階的に確認します。 完了後、コンテナー ロールがインストールされ、シンプルな Hyper-V コンテナーが展開されます。 このクイック スタートを始める前に、コンテナーの基本的な概念と用語を理解しておいてください。 情報は[クイック スタートの概要](./index.md)にあります。

このクイック スタートは Windows 10 の Hyper-V コンテナーのみに適用されます。 このページの左側の目次に追加のクイック スタート文書があります。

**前提条件:**

- Windows 10 Anniversary Edition (Professional または Enterprise) を実行する 1 台の物理コンピューター システム。   
- このクイック スタートは Windows 10 仮想マシンで実行できますが、入れ子の仮想化を有効にする必要があります。 詳細については、「[Nested Virtualization Guide](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting)」 (入れ子になった仮想化ガイド) を参照してください。

> Windows コンテナーが機能するには、重要な更新プログラムがインストールされている必要があります。 
> 使用している OS のバージョンを確認するには、`winver.exe` を実行し、表示されたバージョンを「[Windows 10 の更新履歴](https://support.microsoft.com/en-us/help/12387/windows-10-update-history)」と比較してください。 
> 14393.222 以降であることを確認してから、次に進んでください。

## 1.コンテナー機能のインストール

コンテナー機能は、Windows コンテナーを使用する前に有効にする必要があります。 そのためには、PowerShell セッションで、**管理者特権**で次のコマンドを実行します。

`Enable-WindowsOptionalFeature` は存在しないというエラー メッセージが表示される場合は、PowerShell を管理者として実行していることを再度確認します。

```none
Enable-WindowsOptionalFeature -Online -FeatureName containers -All
```

Windows 10 は Hyper-V コンテナーにのみ対応しているため、Hyper-V 機能も有効にする必要があります。 PowerShell を使用して Hyper-V 機能を有効にするには、管理者特権の PowerShell セッションで次のコマンドを実行します。

```none
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
```

インストールが完了したら、コンピューターを再起動します。

```none
Restart-Computer -Force
```

> 以前に Windows 10 で Hyper-V コンテナーと Technical Preview 5 コンテナー基本イメージを使用していた場合、忘れずに OpLocks を有効にし直してください。 次のコマンドを実行します。  `Set-ItemProperty -Path 'HKLM:SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\Containers' -Name VSmbDisableOplocks -Type DWord -Value 0 -Force`

## 2.Docker のインストール

Docker は Windows コンテナーで使用するために必要です。 Docker は、Docker エンジンと Docker クライアントで構成されます。 この演習では、両方をインストールします。 これを行うために、次のコマンドを実行します。

Docker エンジンとクライアントを 1 つの zip アーカイブとしてダウンロードします。

```none
Invoke-WebRequest "https://test.docker.com/builds/Windows/x86_64/docker-1.13.0-rc4.zip" -OutFile "$env:TEMP\docker-1.13.0-rc4.zip" -UseBasicParsing
```

この zip アーカイブを展開して Program Files に出力します。アーカイブの内容は既に docker ディレクトリに入っています。

```none
Expand-Archive -Path "$env:TEMP\docker-1.13.0-rc4.zip" -DestinationPath $env:ProgramFiles
```

Docker ディレクトリをシステム パスに追加します。

```none
# Add path to this PowerShell session immediately
$env:path += ";$env:ProgramFiles\Docker"

# For persistent use after a reboot
$existingMachinePath = [Environment]::GetEnvironmentVariable("Path",[System.EnvironmentVariableTarget]::Machine)
[Environment]::SetEnvironmentVariable("Path", $existingMachinePath + ";$env:ProgramFiles\Docker", [EnvironmentVariableTarget]::Machine)
```

Windows サービスとして Docker をインストールするには、以下を実行します。

```none
dockerd --register-service
```

インストールされたら、サービスを開始することができます。

```none
Start-Service Docker
```

## 3.コンテナーの基本イメージのインストール

Windows コンテナーは、テンプレートまたはイメージから展開されます。 コンテナーを展開する前に、コンテナーの基本 OS イメージをダウンロードする必要があります。 次のコマンドは、Nano Server のベース イメージをダウンロードします。

Nano Server ベース イメージをダウンロードします。

```none
docker pull microsoft/nanoserver
```

イメージの pull が完了したら、`docker images` を実行するとインストール済みのイメージのリストが返されます。この場合は Nano Server のイメージです。

```none
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
```

> Windows Containers OS Image 使用許諾契約書 (EULA) を参照してください。こちらの「[EULA](../images-eula.md)」に掲載されています。

## 4.最初のコンテナーの展開

この簡単な例では、"Hello World" というコンテナー イメージを作成して展開します。 最適なパフォーマンスでご利用いただくために、管理者特権で起動した Windows CMD シェルまたは PowerShell で以下のコマンドを実行します。

> Windows PowerShell ISE は、コンテナーとの対話型セッションには機能しません。 コンテナーが実行されている場合でも、ハングしているように見えます。

まず、`nanoserver` イメージから対話型セッションでコンテナーを起動します。 コンテナーが起動すると、コンテナーからコマンド シェルが表示されます。  

```none
docker run -it microsoft/nanoserver cmd
```

コンテナー内で、"Hello World" という簡単なスクリプトを作成することにします。

```none
powershell.exe Add-Content C:\helloworld.ps1 'Write-Host "Hello World"'
```   

スクリプトが完成したら、コンテナーを終了します。

```none
exit
```

次に、変更したコンテナーから新しいコンテナー イメージを作成します。 コンテナーの一覧を表示するために、次を実行します。該当するコンテナー ID をメモします。

```none
docker ps -a
```

次のコマンドを実行して、"HelloWorld" というイメージを新たに作成します。 <containerid> を目的のコンテナーの ID に置き換えます。

```none
docker commit <containerid> helloworld
```

操作を完了すると、hello world スクリプトを含むカスタム イメージが作成されます。 これは、次のコマンドで確認できます。

```none
docker images
```

最後に、コンテナーを実行するには、`docker run` コマンドを使用します。

```none
docker run --rm helloworld powershell c:\helloworld.ps1
```

`docker run` コマンドを実行すると、結果的に、Hyper-V コンテナーが "Hello World" イメージから作成され、サンプル スクリプト "Hello World" が実行されます (出力がシェルにエコーされます)。次に、コンテナーが停止し、削除されます。
後続の Windows 10 とコンテナーのクイック スタートでは、Windows 10 のコンテナーでアプリケーションを作成し、展開する方法について説明します。

## 次の手順

[Windows Server の Windows コンテナー](./quick-start-windows-server.md)



<!--HONumber=Jan17_HO3-->


