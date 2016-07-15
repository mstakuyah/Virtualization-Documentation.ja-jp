---
title: "Dockerfile コンテナーと Windows コンテナー"
description: "Windows コンテナー用に Dockerfile を作成します。"
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 05/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 75fed138-9239-4da9-bce4-4f2e2ad469a1
translationtype: Human Translation
ms.sourcegitcommit: daf82c943f9e19ec68e37207bba69fb0bf46f46f
ms.openlocfilehash: ace5fd12856cdcff3a380eb35e4982c4c1ce4c5a

---

# Windows 上の Dockerfile

**この記事は暫定的な内容であり、変更される可能性があります。** 

Docker エンジンには、コンテナー イメージの作成を自動化するツールが含まれています。 `docker commit` コマンドを使用して手動でコンテナー イメージを作成することもできますが、イメージの自動作成プロセスを使用すると、次のような多くの利点があります。

- コンテナー イメージをコードとして保存。
- メンテナンスやアップグレードの目的による、コンテナー イメージの迅速かつ正確な再作成。
- コンテナー イメージと開発サイクル間の継続的な統合。

この自動処理を行う Docker コンポーネントは、Dockerfile であり、`docker build` コマンドです。

- **Dockerfile** - 新しいコンテナー イメージを作成するために必要な命令を含むテキスト ファイル。 たとえば、ベースとして使用する既存のイメージの指定、イメージの作成プロセス時に実行されるコマンド、コンテナー イメージの新しいインスタンスが展開されるときに実行されるコマンドなどの命令が含まれます。
- **Docker ビルド** - Dockerfile を使用する Docker エンジン コマンド。イメージ作成プロセスをトリガーします。

このドキュメントでは、Dockerfile と Windows コンテナーの使用方法について説明します。また、構文、一般的に使用される Dockerfile 命令の詳細についても説明します。 

このドキュメントでは、コンテナー イメージとコンテナー イメージ レイヤーの概念について説明します。 イメージとイメージの階層化の詳細については、「[コンテナー イメージ](../management/manage_images.md)」を参照してください。 

Dockerfile の詳細については、[docker.com の Dockerfile リファレンス]( https://docs.docker.com/engine/reference/builder/)を参照してください。

## Dockerfile の概要

### 基本構文

ごく基本的なフォームでは、Dockerfile はとても単純です。 次の例では、IIS を含み、’hello world’ サイトを含む新しいイメージを作成します。 この例に含まれるコマンド (`#` で示されます) については、各手順で説明します。 この記事の以降のセクションでは、Dockerfile 構文規則と、Dockerfile 命令について詳しく説明します。

```none
# Sample Dockerfile

# Indicates that the windowsservercore image will be used as the base image.
FROM windowsservercore

# Metadata indicating an image maintainer.
MAINTAINER jshelton@contoso.com

# Uses dism.exe to install the IIS role.
RUN dism.exe /online /enable-feature /all /featurename:iis-webserver /NoRestart

# Creates an html file and adds content to this file.
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html

# Sets a command or process that will run each time a container is run from the new image.
CMD [ "cmd" ]
```

Windows 用 Dockerfile のその他の例については、「[Dockerfile for Windows Repository (Windows リポジトリの Dockerfile)] (https://github.com/Microsoft/Virtualization-Documentation/tree/master/windows-container-samples)」を参照してください。

## 手順

Dockerfile 命令は、Docker エンジンに対して、コンテナー イメージを作成するために必要な手順を示します。 次の命令は、順番に、1 つずつ実行されます。 Dockerfile の基本的な命令の詳細を次に示します。 Dockerfile の命令の詳細な一覧については、[Docker.com の Dockerfile リファレンス] (https://docs.docker.com/engine/reference/builder/) を参照してください。

### FROM

`FROM` 命令。新しいイメージ作成プロセス中に使用されるコンテナー イメージを設定します。 たとえば、命令 `FROM windowsservercore` を使用すると、結果のイメージは Windows Server Core ベース OS イメージから派生し、依存します。 指定したイメージが、Docker ビルド プロセスが実行されているシステムに存在しない場合、Docker エンジンは、パブリックまたはプライベートのイメージ レジストリからイメージのダウンロードを試行します。

**Format (形式)**

FROM 命令の形式は、次のとおりです。 

```
FROM <image>
```

**例**

```
FROM windowsservercore
```

FROM 命令の詳細については、[Docker.com の FROM リファレンス]( https://docs.docker.com/engine/reference/builder/#from)を参照してください。 

### RUN

`RUN` 命令は、コマンドを実行し、新しいコンテナー イメージにキャプチャするように指定します。 これらのコマンドには、ソフトウェアのインストール、ファイルとディレクトリの作成、環境構成の作成などの項目が含まれます。

**Format (形式)**

RUN 命令の形式は、次のとおりです。 

```none
# exec form

RUN ["<executable", "<param 1>", "<param 2>"]

# shell form

RUN <command>
```

exec とシェル フォーム間の違いは、`RUN` 命令の実行方法です。 exec フォームを使用すると、指定したプログラムが明示的に実行されます。 

次の例では、exec フォームを使用しています。

```none
FROM windowsservercore

RUN ["powershell", "New-Item", "c:/test"]
```

結果のイメージを確認して、実行されたコマンドは `powershell new-item c:/test` です。

```none
docker history doc-exe-method

IMAGE               CREATED             CREATED BY                    SIZE                COMMENT
b3452b13e472        2 minutes ago       powershell New-Item c:/test   30.76 MB
```

対照的に、次の例では、同じ操作を実行していますが、ここではシェル フォームを使用しています。

```none
FROM windowsservercore

RUN powershell new-item c:\test
```

これは、`cmd /S /C powershell new-item c:\test` の実行命令の結果です。 

```none
docker history doc-shell-method

IMAGE               CREATED             CREATED BY                              SIZE                COMMENT
062a543374fc        19 seconds ago      cmd /S /C powershell new-item c:\test   30.76 MB
```

**Windows に関する考慮事項**
 
Windows では、exec 形式で `RUN` 命令を使用する場合、バックスラッシュをエスケープする必要があります。

```none
RUN ["powershell", "New-Item", "c:\\test"]
```

ターゲット プログラムが Windows インストーラーである場合は、追加の手順として `/x:<directory>` フラグを使用したセットアップの抽出を行ってから、実際の (サイレント) インストール プロシージャを起動する必要があります。 さらに、コマンドが終了するまで待機する必要があります。 そうしないと、プロセスは何もインストールせずに途中で終了してしまいます。 詳細については、次の例を参照してください。

**例**

この例では、DISM を使用してコンテナー イメージに IIS をインストールします。
```none
RUN dism.exe /online /enable-feature /all /featurename:iis-webserver /NoRestart
```

この例では、Visual Studio 再頒布可能パッケージをインストールします。
```none
RUN powershell.exe -Command c:\vcredist_x86.exe /quiet
``` 

この例では、.NET Framework 4.5.2 Developer Pack をインストールするために、まずそれの抽出を行ってから、実際のインストーラーを起動しています。 
```none
RUN start /wait C:\temp\NDP452-KB2901951-x86-x64-DevPack.exe /q /x:C:\temp\NDP452DevPackSetupDir && \
    start /wait C:\temp\NDP452DevPackSetupDir\Setup.exe /norestart /q /log %TEMP%\ndp452_install_log.txt && \
    rmdir /s /q C:\temp\NDP452DevPackSetupDir
```

RUN 命令の詳細については、[Docker.com の RUN リファレンス]( https://docs.docker.com/engine/reference/builder/#run)を参照してください。 

### コピー

`COPY` 命令は、ファイルとディレクトリを、コンテナーのファイルシステムにコピーします。 ファイルとディレクトリは、Dockerfile の相対パスで示す必要があります。

**Format (形式)**

`COPY` 命令の形式は、次のとおりです。 

```none
COPY <source> <destination>
``` 

送信元または送信先のいずれかに空白が含まれる場合は、パスを角かっこと二重引用符で囲みます。
 
```none
COPY ["<source>" "<destination>"]
```

**Windows に関する考慮事項**
 
Windows では、変換先形式でスラッシュを使用する必要があります。 たとえば、次に有効な `COPY` 命令を示します。

```none
COPY test1.txt /temp/
COPY test1.txt c:/temp/
```

ただし、次の命令は動作しません。

```none
COPY test1.txt c:\temp\
```

**例**

この例では、ソース ディレクトリのコンテンツを、コンテナー イメージの `sqllite` というディレクトリに追加します。
```none
COPY source /sqlite/
```

この例では、config から始まるすべてのファイルを、コンテナー イメージの `c:\temp` ディレクトリに追加します。
```none
COPY config* c:/temp/
```

`COPY` 命令の詳細については、[Docker.com の COPY リファレンス]( https://docs.docker.com/engine/reference/builder/#copy)をご覧ください。

### 追加

ADD 命令は、COPY 命令とよく似ていますが、ADD 命令にはさらに他の機能があります。 `ADD` 命令は、ホストのファイルがコンテナー イメージにコピーされるだけでなく、リモートの場所にあるファイルを URL の仕様に対してコピーします。

**Format (形式)**

`ADD` 命令の形式は、次のとおりです。 

```none
ADD <source> <destination>
``` 

送信元または送信先のいずれかに空白が含まれる場合は、パスを角かっこと二重引用符で囲みます。
 
```none
ADD ["<source>" "<destination>"]
```

**Windows に関する考慮事項**
 
Windows では、変換先形式でスラッシュを使用する必要があります。 たとえば、次に有効な `ADD` 命令を示します。

```none
ADD test1.txt /temp/
ADD test1.txt c:/temp/
```

ただし、次の命令は動作しません。

```none
ADD test1.txt c:\temp\
```

さらに、Linux の `ADD` 動作では、コンピューター上の圧縮されたパッケージを展開できます。 この機能は、Windows ではこの機能を使用できません。

**例**

この例では、ソース ディレクトリのコンテンツを、コンテナー イメージの `sqllite` というディレクトリに追加します。
```none
ADD source /sqlite/
```

この例では、config から始まるすべてのファイルを、コンテナー イメージの `c:\temp` ディレクトリに追加します。
```none
ADD config* c:/temp/
```

この例では、Windows 用 Python を、コンテナー イメージの `c:\temp` ディレクトリにダウンロードします。
```none
ADD https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe /temp/python-3.5.1.exe
```

`ADD` 命令の詳細については、[Docker.com の ADD リファレンス]( https://docs.docker.com/engine/reference/builder/#add)を参照してください。 

### WORKDIR

`WORKDIR` 命令では、`RUN`、`CMD`、コンテナー イメージのインスタンスを実行する作業ディレクトリなど、他の Dockerfile 命令の作業ディレクトリを設定します。

**Format (形式)**

`WORKDIR` 命令の形式は、次のとおりです。 

```none
WORKDIR <path to working directory>
``` 

**Windows に関する考慮事項**

Windows では、作業ディレクトリにバックスラッシュが含まれる場合、エスケープする必要があります。

```none
WORKDIR c:\\windows
```

**例**

```none
WORKDIR c:\\Apache24\\bin
```

`WORKDIR` 命令の詳細については、[Docker.com の WORKDIR リファレンス]( https://docs.docker.com/engine/reference/builder/#workdir)を参照してください。 

### CMD

`CMD` 命令には、コンテナー イメージのインスタンスを展開するときに実行する既定のコマンドを設定します。 たとえば、コンテナーが NGINX Web サーバーをホストする場合、`CMD` には、`nginx.exe` など、Web サーバーを起動する命令を含めることができます。 Dockerfile に複数の `CMD` 命令を指定すると、最後の命令のみが評価されます。

**Format (形式)**

`CMD` 命令の形式は、次のとおりです。 

```none
# exec form

CMD ["<executable", "<param>"]

# shell form

CMD <command>
```

**Windows に関する考慮事項**

Windows では、`CMD` 命令に指定されたファイル パスにはフォワード スラッシュを使用し、バックスラッシュをエスケープする (`\\`) 必要があります。 たとえば、次に有効な `CMD` 命令を示します。

```none
# exec form

CMD ["c:\\Apache24\\bin\\httpd.exe", "-w"]

# shell form

CMD c:\\Apache24\\bin\\httpd.exe -w
```
ただし、次の命令は動作しません。

```none
CMD c:\Apache24\bin\httpd.exe -w
```

`CMD` 命令の詳細については、[Docker.com の CMD リファレンス]( https://docs.docker.com/engine/reference/builder/#cmd)を参照してください。 

## エスケープ文字

多くの場合、Dockerfile の命令は複数の行にまたがる必要があります。このためには、エスケープ文字を使用します。 既定の Dockerfile のエスケープ文字はバックスラッシュ `\` です。 バックスラッシュは Windows でのファイル パス区切り文字でもあるため、問題が生じる場合があります。 パーサー ディレクティブを使用して、既定のエスケープ文字を変更することもできます。 パーサー ディレクティブについて詳しくは、[Docker.com のパーサー ディレクティブに関する記事]( https://docs.docker.com/engine/reference/builder/#parser-directives)を参照してください。

次の例は、既定のエスケープ文字を使用した複数の行にまたがる単一の RUN 命令を示しています。

```none
FROM windowsservercore

RUN powershell.exe -Command \
    $ErrorActionPreference = 'Stop'; \
    wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
    Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
    Remove-Item c:\python-3.5.1.exe -Force
```

エスケープ文字を変更するには、エスケープ パーサー ディレクティブを Dockerfile の最初の行に配置します。 次のようになります。

> エスケープ文字として使用できる値は `\` と `` ` `` の 2 つだけであることに注意してください。

```none
# escape=`

FROM windowsservercore

RUN powershell.exe -Command `
    $ErrorActionPreference = 'Stop'; `
    wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; `
    Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; `
    Remove-Item c:\python-3.5.1.exe -Force
```

エスケープ パーサー ディレクティブについて詳しくは、[Docker.com のエスケープ パーサー ディレクティブに関する記事]( https://docs.docker.com/engine/reference/builder/#escape)を参照してください。

## Dockerfile の PowerShell

### PowerShell コマンド

PowerShell コマンドは、Dockerfile で `RUN` 操作を使用して実行できます。 

```none
FROM windowsservercore

RUN powershell -command Expand-Archive -Path c:\apache.zip -DestinationPath c:\
```

### REST 呼び出し

PowerShell と `Invoke-WebRequest` コマンドは、Web サービスから情報やファイルを収集するときに便利です。 たとえば、Python を含むイメージを構築する場合、次の例を使用できます。

```none
FROM windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

> Invoke-WebRequest は、Nano Server で現在サポートされていません

イメージの作成プロセス中にファイルをダウンロードするために PowerShell を使用するもう 1 つのオプションは、.NET WebClient ライブラリを使用することです。 この方法で、ダウンロードのパフォーマンスが向上します。 次の例では、WebClient ライブラリを使用して、Python ソフトウェアをダウンロードしています。

```none
FROM windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  (New-Object System.Net.WebClient).DownloadFile('https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe','c:\python-3.5.1.exe') ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

> WebClient は、現在 Nano Server でサポートされていません

### PowerShell スクリプト

場合によっては、イメージの作成プロセス中に使用されるコンテナーにスクリプトをコピーしてから、コンテナー内から実行する方法が便利なことがあります。 注: この方法では、イメージ レイヤー キャッシュが制限され、Dockerfile の読みやすさが低下します。

この例では、`ADD` 命令を使用して、ビルド コンピューターからコンテナーにスクリプトをコピーします。 このスクリプトは、RUN 命令を使用して実行されます。

```
FROM windowsservercore
ADD script.ps1 /windows/temp/script.ps1
RUN powershell.exe -executionpolicy bypass c:\windows\temp\script.ps1
```

## Docker のビルド 

Dockerfile が作成され、ディスクに保存されると、`docker build` を実行して新しいイメージを作成できます。 `docker build` コマンドには、いくつかの省略可能なパラメーターと、Dockerfile のパスを指定できます。 すべてのビルド オプションの一覧を含め、Docker のビルドの詳細については、[Docker.com のビルドに関するページ](https://docs.docker.com/engine/reference/commandline/build/#build)をご覧ください。

```none
Docker build [OPTIONS] PATH
```
たとえば、次のコマンドでは、’iis’ というイメージが作成されます。

```none
docker build -t iis .
```

ビルド プロセスが開始されると、出力に状態が示され、スローされたエラーがある場合はそのエラーが返されます。

```none
C:\> docker build -t iis .

Sending build context to Docker daemon 2.048 kB
Step 1 : FROM windowsservercore
 ---> 6801d964fda5

Step 2 : RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
 ---> Running in ae8759fb47db

Deployment Image Servicing and Management tool
Version: 10.0.10586.0

Image Version: 10.0.10586.0

Enabling feature(s)
The operation completed successfully.

 ---> 4cd675d35444
Removing intermediate container ae8759fb47db

Step 3 : RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
 ---> Running in 9a26b8bcaa3a
 ---> e2aafdfbe392
Removing intermediate container 9a26b8bcaa3a

Successfully built e2aafdfbe392
```

結果は、新しいコンテナー イメージになります。この例では、’iis’ というイメージです。

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
iis                 latest              e2aafdfbe392        About a minute ago   207.8 MB
windowsservercore   latest              6801d964fda5        4 months ago         0 B
```

## 参考資料

[Optimize Dockerfiles and Docker build for Windows (Windows の Dockerfile と Docker ビルドの最適化)] (./optimize_windows_dockerfile.md)

[Docker.com の Dockerfile リファレンス](https://docs.docker.com/engine/reference/builder/)



<!--HONumber=Jul16_HO1-->


