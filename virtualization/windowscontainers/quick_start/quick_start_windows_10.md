---
title: "Windows 10 の Windows コンテナー"
description: "コンテナー展開のクイック スタート"
keywords: "Docker, コンテナー"
author: neilpeterson
manager: timlt
ms.date: 07/13/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
translationtype: Human Translation
ms.sourcegitcommit: 6c7ce9f1767c6c6391cc6d33a553216bd815ff72
ms.openlocfilehash: bd93f5a73268b552710304d7da568e1497239679

---

# Windows 10 の Windows コンテナー

**この記事は暫定的な内容であり、変更される可能性があります。** 

この演習では、Windows 10 の Windows コンテナー機能の基本的な展開と使用について段階的に確認します (Insider ビルド 14372 以降)。 完了後、コンテナー ロールがインストールされ、シンプルな Hyper-V コンテナーが展開されます。 このクイック スタートを始める前に、コンテナーの基本的な概念と用語を理解しておいてください。 情報は[クイック スタートの概要](./quick_start.md)にあります。 

このクイック スタートは Windows 10 の Hyper-V コンテナーのみに適用されます。 このページの左側の目次に追加のクイック スタート文書があります。

**前提条件:**

- [Windows 10 Insider リリース](https://insider.windows.com/)で実行されている 1 台の物理コンピューター システム。   
- このクイック スタートは Windows 10 仮想マシンで実行できますが、入れ子の仮想化を有効にする必要があります。 詳細については、「[Nested Virtualization Guide](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting)」 (入れ子になった仮想化ガイド) を参照してください。

## 1.コンテナー機能のインストール

コンテナー機能は、Windows コンテナーを使用する前に有効にする必要があります。 そのためには、管理者特権の PowerShell セッションで次のコマンドを実行します。 

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

バックアップを作成したら、次のコマンドを使用して、Windows コンテナーのテクニカル プレビューで既知の問題を解決します。  

 ```none
Set-ItemProperty -Path 'HKLM:SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\Containers' -Name VSmbDisableOplocks -Type DWord -Value 1 -Force
```

## 2.Docker のインストール

Docker は Windows コンテナーで使用するために必要です。 Docker は、Docker エンジンと Docker クライアントで構成されます。 この演習では、両方をインストールします。 これを行うために、次のコマンドを実行します。 

Docker エンジンとクライアントを 1 つの zip アーカイブとしてダウンロードします。

```none
Invoke-WebRequest "https://get.docker.com/builds/Windows/x86_64/docker-1.12.0.zip" -OutFile "$env:TEMP\docker-1.12.0.zip" -UseBasicParsing
```

この zip アーカイブを展開して Program Files に出力します。アーカイブの内容は既に docker ディレクトリに入っています。

```none
Expand-Archive -Path "$env:TEMP\docker-1.12.0.zip" -DestinationPath $env:ProgramFiles
```

Docker ディレクトリをシステム パスに追加します。

```none
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";$env:ProgramFiles\docker\", [EnvironmentVariableTarget]::Machine)
```

変更されたパスが認識されるように、PowerShell セッションを再起動します。

Windows サービスとして Docker をインストールするには、以下を実行します。

```none
& $env:ProgramFiles\docker\dockerd.exe --register-service
```

インストールされたら、サービスを開始することができます。

```none
Start-Service Docker
```

## 3.コンテナーの基本イメージのインストール

Windows コンテナーは、テンプレートまたはイメージから展開されます。 コンテナーを展開する前に、コンテナーの基本 OS イメージをダウンロードする必要があります。 次のコマンドは、Nano Server のベース イメージをダウンロードします。
    
> この手順は、14372 より番号が大きい Windows Insider ビルドに適用されますが、'Docker Pull' が機能するまでの一時的なものとなります。

Nano Server ベース イメージをダウンロードします。 

```none
Start-BitsTransfer https://aka.ms/tp5/6b/docker/nanoserver -Destination nanoserver.tar.gz
```

基本イメージをインストールします。

```none  
docker load -i nanoserver.tar.gz
```

この段階では、`docker images` を実行すると、インストールされたイメージの一覧が返されます。この場合、Nano Server のイメージです。

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nanoserver          10.0.14300.1030     3f5112ddd185        3 weeks ago         810.2 MB
```

続行前に、このイメージに '最新' バージョンであることを示すタグを付ける必要があります。 これを行うために、次のコマンドを実行します。

```none
docker tag microsoft/nanoserver:10.0.14300.1030 nanoserver:latest
```

Windows コンテナー イメージの詳細については、[コンテナー イメージの管理](../management/manage_images.md)に関するページを参照してください。

## 4.最初のコンテナーの展開

この簡単な例では、"Hello World" というコンテナー イメージを作成して展開します。 最適なパフォーマンスでご利用いただくために、管理者特権で起動した Windows CMD シェルで以下のコマンドを実行します。

まず、`nanoserver` イメージから対話型セッションでコンテナーを起動します。 コンテナーが起動すると、コンテナーからコマンド シェルが表示されます。  

```none
docker run -it nanoserver cmd
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

`docker run` コマンドを実行すると、結果的に、Hyper-V コンテナーが "Hello World" イメージから作成され、サンプル スクリプト "Hello World" が実行されます (出力がシェルにエコーされます)。次に、コンテナーが停止し、削除されます。 後続の Windows 10 とコンテナーのクイック スタートでは、Windows 10 のコンテナーでアプリケーションを作成し、展開する方法について説明します。

## 次の手順

[Windows Server の Windows コンテナー](./quick_start_windows_server.md)





<!--HONumber=Aug16_HO1-->


