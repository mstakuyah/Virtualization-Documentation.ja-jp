---
title: "Windows 10 の Windows コンテナー"
description: "コンテナー展開のクイック スタート"
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 06/28/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
translationtype: Human Translation
ms.sourcegitcommit: 3fc388632dee4d714ab5a7869fa852c079c11910
ms.openlocfilehash: e2d86c6f1aba07c7c40d1f932b3884c99bfd8f0a

---

# Windows 10 の Windows コンテナー

**この記事は暫定的な内容であり、変更される可能性があります。** 

この演習では、Windows 10 の Windows コンテナー機能の基本的な展開と使用について段階的に確認します (Insider ビルド 14352 以降)。 完了後、コンテナー ロールがインストールされ、シンプルな Hyper-V コンテナーが展開されます。 このクイック スタートを始める前に、コンテナーの基本的な概念と用語を理解しておいてください。 情報は[クイック スタートの概要](./quick_start.md)にあります。 

このクイック スタートは Windows 10 の Hyper-V コンテナーのみに適用されます。 このページの左側の目次に追加のクイック スタート文書があります。

**前提条件:**

- [Windows 10 Insider リリース](https://insider.windows.com/)で実行されている 1 台の物理コンピューター システム。   
- このクイック スタートは Windows 10 仮想マシンで実行できますが、入れ子の仮想化を有効にする必要があります。 詳細については、「[Nested Virtualization Guide](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting)」 (入れ子になった仮想化ガイド) を参照してください。

## 1.コンテナー機能のインストール

コンテナー機能は、Windows コンテナーを使用する前に有効にする必要があります。 そのためには、管理者特権の PowerShell セッションで次のコマンドを実行します。 

```none
Enable-WindowsOptionalFeature -Online -FeatureName containers -All
```

Windows 10 は Hyper-V コンテナーにのみ対応しているため、Hyper-V 機能も有効にする必要があります。 PowerShell を使用してHyper-V 機能を有効にするには、管理者特権の PowerShell セッションで次のコマンドを実行します。

```none
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
```

インストールが完了したら、コンピューターを再起動します。

```none
Restart-Computer -Force
```

## 2.Docker のインストール

Docker は Windows コンテナーで使用するために必要です。 Docker は、Docker エンジンと Docker クライアントで構成されます。 この演習では、両方をインストールします。 これを行うために、次のコマンドを実行します。 

Docker の実行可能ファイル用のフォルダーを作成します。

```none
New-Item -Type Directory -Path 'C:\Program Files\docker\'
```

Docker デーモンをダウンロードします。

```none
Invoke-WebRequest https://aka.ms/tp5/b/dockerd -OutFile $env:ProgramFiles\docker\dockerd.exe
```

Docker クライアントをダウンロードします。

```none
Invoke-WebRequest https://aka.ms/tp5/b/docker -OutFile $env:ProgramFiles\docker\docker.exe
```

Docker ディレクトリをシステム パスに追加します。

```none
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files\Docker", [EnvironmentVariableTarget]::Machine)
```

変更されたパスが認識されるように、PowerShell セッションを再起動します。

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
    
現在の PowerShell プロセスの PowerShell 実行ポリシーを設定します。 これは現在の PowerShell セッションで実行されているスクリプトにのみ適用されますが、実行ポリシーを変更するときは注意が必要です。

```none
Set-ExecutionPolicy Bypass -scope Process
```

コンテナー イメージ パッケージ プロバイダーをインストールします。

```none  
Install-PackageProvider ContainerImage -Force
```

次に、Nano Server イメージをインストールします。

```none
Install-ContainerImage -Name NanoServer
```

基本イメージがインストールされたら、Docker サービスを再起動する必要があります。

```none
Restart-Service docker
```

この段階では、`docker images` を実行すると、インストールされたイメージの一覧が返されます。この場合、Nano Server のイメージです。

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nanoserver          10.0.14300.1016     3f5112ddd185        3 weeks ago         810.2 MB
```

続行前に、このイメージに '最新' バージョンであることを示すタグを付ける必要があります。 これを行うために、次のコマンドを実行します。

```none
docker tag nanoserver:10.0.14300.1016 nanoserver:latest
```

Windows コンテナー イメージの詳細については、[コンテナー イメージの管理](../management/manage_images.md)に関するページを参照してください。

## 4.最初のコンテナーの展開

この単純な例のために、.NET Core イメージが事前に作成されています。 `docker pull` コマンドを利用し、このイメージをダウンロードします。

実行すると、コンテナーが開始されます。シンプルな .NET Core アプリケーションが実行されます。そして、コンテナーが終了となります。 

```none
docker pull microsoft/sample-dotnet
```

この状態は `docker images` コマンドで確認できます。

```none
docker images

REPOSITORY               TAG                 IMAGE ID            CREATED             SIZE
microsoft/sample-dotnet  latest              28da49c3bff4        41 hours ago        918.3 MB
nanoserver               10.0.14300.1016     3f5112ddd185        3 weeks ago         810.2 MB
nanoserver               latest              3f5112ddd185        3 weeks ago         810.2 MB
```

`docker run` コマンドでコンテナーを実行します。 次の例では、`--rm` パラメーターが指定されています。このパラメーターは、コンテナーが実行されなくなったら、コンテナーを削除するように Docker エンジンに指示します。 

Docker Run コマンドの詳細については、Docker.com の「[Docker Run リファレンス]( https://docs.docker.com/engine/reference/run/)」をご覧ください。

```none
docker run --isolation=hyperv --rm microsoft/sample-dotnet
```

**注** - タイムアウト イベントを示すエラーがスローされる場合は、次の PowerShell スクリプトを実行して操作をやり直してください。

```none
Set-ItemProperty -Path 'HKLM:SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\Containers' -Name VSmbDisableOplocks -Type DWord -Value 1 -Force
```

`docker run` コマンドを実行すると、結果的に、Hyper-V コンテナーが sample-dotnet イメージから作成され、サンプル アプリケーションが実行されます (出力がシェルにエコーされます)。それから、コンテナーが停止し、削除されます。 後続の Windows 10 とコンテナーのクイック スタートでは、Windows 10 のコンテナーでアプリケーションを作成し、展開する方法について説明します。

## 次の手順

[Windows Server の Windows コンテナー](./quick_start_windows_server.md)





<!--HONumber=Jun16_HO4-->


