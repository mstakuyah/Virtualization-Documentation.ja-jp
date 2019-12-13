---
title: Dockerfile コンテナーと Windows コンテナー
description: Windows コンテナー用に Dockerfile を作成します。
keywords: Docker, コンテナー
author: PatrickLang
ms.date: 05/03/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 75fed138-9239-4da9-bce4-4f2e2ad469a1
ms.openlocfilehash: 9fef74c029dc3efc220b1f9924d2695cdbaa61be
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909662"
---
# <a name="dockerfile-on-windows"></a>Windows 上の Dockerfile

Docker エンジンには、コンテナーイメージの作成を自動化するツールが含まれています。 `docker commit` コマンドを実行してコンテナーイメージを手動で作成することもできますが、自動イメージ作成プロセスを採用すると、次のような多くの利点があります。

- コンテナー イメージをコードとして保存。
- メンテナンスやアップグレードの目的による、コンテナー イメージの迅速かつ正確な再作成。
- コンテナー イメージと開発サイクル間の継続的な統合。

この自動処理を行う Docker コンポーネントは、Dockerfile であり、`docker build` コマンドです。

Dockerfile は、新しいコンテナーイメージを作成するために必要な命令を含むテキストファイルです。 たとえば、ベースとして使用する既存のイメージの指定、イメージの作成プロセス時に実行されるコマンド、コンテナー イメージの新しいインスタンスが展開されるときに実行されるコマンドなどの命令が含まれます。

Docker build は、Dockerfile を使用してイメージ作成プロセスをトリガーする Docker エンジンコマンドです。

このトピックでは、Windows コンテナーで Dockerfiles を使用する方法、基本構文を理解する方法、および最も一般的な Dockerfiles 命令について説明します。

このドキュメントでは、コンテナーイメージとコンテナーイメージレイヤーの概念について説明します。 イメージとイメージレイヤーの詳細については、「[コンテナーの基本イメージ](../manage-containers/container-base-images.md)」を参照してください。

Dockerfiles の詳細については、 [Dockerfiles リファレンス](https://docs.docker.com/engine/reference/builder/)を参照してください。

## <a name="basic-syntax"></a>基本構文

ごく基本的なフォームでは、Dockerfile はとても単純です。 次の例では、IIS を含み、’hello world’ サイトを含む新しいイメージを作成します。 この例に含まれるコマンド (`#` で示されます) については、各手順で説明します。 この記事の以降のセクションでは、Dockerfile 構文規則と、Dockerfile 命令について詳しく説明します。

>[!NOTE]
>Dockerfile は、拡張子を付けずに作成する必要があります。 Windows でこれを行うには、任意のエディターを使用してファイルを作成し、"Dockerfile" という表記 (引用符を含む) を使用してファイルを保存します。

```dockerfile
# Sample Dockerfile

# Indicates that the windowsservercore image will be used as the base image.
FROM mcr.microsoft.com/windows/servercore:ltsc2019

# Metadata indicating an image maintainer.
LABEL maintainer="jshelton@contoso.com"

# Uses dism.exe to install the IIS role.
RUN dism.exe /online /enable-feature /all /featurename:iis-webserver /NoRestart

# Creates an HTML file and adds content to this file.
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html

# Sets a command or process that will run each time a container is run from the new image.
CMD [ "cmd" ]
```

Windows 用の Dockerfiles のその他の例については、 [windows リポジトリの Dockerfiles](https://github.com/Microsoft/Virtualization-Documentation/tree/master/windows-container-samples)を参照してください。

## <a name="instructions"></a>手順

Dockerfile 命令は、コンテナーイメージを作成するために必要な命令を Docker エンジンに提供します。 これらの手順は、1つずつ順番に実行されます。 次の例は、Dockerfiles で最もよく使用される手順です。 Dockerfile 命令の完全な一覧については、 [dockerfile リファレンス](https://docs.docker.com/engine/reference/builder/)を参照してください。

### <a name="from"></a>FROM

`FROM` 命令では、新しいイメージ作成プロセス中に使用されるコンテナー イメージを設定します。 たとえば、命令 `FROM mcr.microsoft.com/windows/servercore` を使用すると、結果のイメージは Windows Server Core ベース OS イメージから派生し、依存します。 指定したイメージが、Docker ビルド プロセスが実行されているシステムに存在しない場合、Docker エンジンは、パブリックまたはプライベートのイメージ レジストリからイメージのダウンロードを試行します。

FROM 命令の形式は次のようになります。

```dockerfile
FROM <image>
```

FROM コマンドの例を次に示します。

Microsoft Container Registry (MCR) から ltsc2019 バージョンの windows server core をダウンロードするには、次のようにします。
```
FROM mcr.microsoft.com/windows/servercore:ltsc2019
```

詳細については、「 [FROM reference](https://docs.docker.com/engine/reference/builder/#from)」を参照してください。

### <a name="run"></a>RUN

`RUN` 命令は、コマンドを実行し、新しいコンテナー イメージにキャプチャするように指定します。 これらのコマンドには、ソフトウェアのインストール、ファイルとディレクトリの作成、環境構成の作成などの項目が含まれます。

RUN 命令は次のようになります。

```dockerfile
# exec form

RUN ["<executable>", "<param 1>", "<param 2>"]

# shell form

RUN <command>
```

Exec 形式とシェル形式の違いは、`RUN` 命令の実行方法です。 exec フォームを使用すると、指定したプログラムが明示的に実行されます。

Exec フォームの例を次に示します。

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN ["powershell", "New-Item", "c:/test"]
```

結果のイメージは、`powershell New-Item c:/test` コマンドを実行します。

```dockerfile
docker history doc-exe-method

IMAGE               CREATED             CREATED BY                    SIZE                COMMENT
b3452b13e472        2 minutes ago       powershell New-Item c:/test   30.76 MB
```

これに対し、次の例では、シェル形式で同じ操作を実行します。

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell New-Item c:\test
```

結果のイメージには、`cmd /S /C powershell New-Item c:\test`の実行命令があります。

```dockerfile
docker history doc-shell-method

IMAGE               CREATED             CREATED BY                              SIZE                COMMENT
062a543374fc        19 seconds ago      cmd /S /C powershell New-Item c:\test   30.76 MB
```

### <a name="considerations-for-using-run-with-windows"></a>Windows での実行の使用に関する考慮事項

Windows では、exec 形式で `RUN` 命令を使用する場合、バックスラッシュをエスケープする必要があります。

```dockerfile
RUN ["powershell", "New-Item", "c:\\test"]
```

ターゲットプログラムが Windows インストーラーの場合は、`/x:<directory>` フラグを使用してセットアップを抽出してから、実際の (サイレント) インストール手順を実行する必要があります。 また、コマンドが終了するまで待ってから、他の操作を実行する必要があります。 それ以外の場合、プロセスは何もインストールせずに途中で終了します。 詳細については、次の例を参照してください。

#### <a name="examples-of-using-run-with-windows"></a>Windows での実行の使用例

次の Dockerfile の例では、DISM を使用してコンテナーイメージに IIS をインストールしています。

```dockerfile
RUN dism.exe /online /enable-feature /all /featurename:iis-webserver /NoRestart
```

この例では、Visual Studio 再頒布可能パッケージをインストールします。 `Start-Process` と `-Wait` パラメーターを使用してインストーラーを実行します。 これにより、Dockerfile の次の手順に進む前に、インストールが完了します。

```dockerfile
RUN powershell.exe -Command Start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait
```

実行命令の詳細については、「 [run reference](https://docs.docker.com/engine/reference/builder/#run)」を参照してください。

### <a name="copy"></a>コピー

`COPY` 命令は、ファイルとディレクトリをコンテナーのファイルシステムにコピーします。 ファイルとディレクトリは、Dockerfile を基準とした相対パスにある必要があります。

`COPY` 命令の形式は次のようになります。

```dockerfile
COPY <source> <destination>
```

変換元または変換先のいずれかに空白が含まれている場合は、次の例に示すように、角かっこと二重引用符でパスを囲みます。

```dockerfile
COPY ["<source>", "<destination>"]
```

#### <a name="considerations-for-using-copy-with-windows"></a>Windows でのコピーの使用に関する考慮事項

Windows では、変換先形式でスラッシュを使用する必要があります。 たとえば、次のような `COPY` の手順があります。

```dockerfile
COPY test1.txt /temp/
COPY test1.txt c:/temp/
```

一方、円記号付きの次の形式は機能しません。

```dockerfile
COPY test1.txt c:\temp\
```

#### <a name="examples-of-using-copy-with-windows"></a>COPY を Windows で使用する例

次の例では、ソースディレクトリの内容を、コンテナーイメージ内の `sqllite` という名前のディレクトリに追加します。

```dockerfile
COPY source /sqlite/
```

次の例では、config で始まるすべてのファイルを、コンテナーイメージの `c:\temp` ディレクトリに追加します。

```dockerfile
COPY config* c:/temp/
```

`COPY` 命令の詳細については、「[コピーの参照](https://docs.docker.com/engine/reference/builder/#copy)」を参照してください。

### <a name="add"></a>ADD

ADD 命令はコピー命令に似ていますが、さらに多くの機能があります。 `ADD` 命令は、ホストのファイルがコンテナー イメージにコピーされるだけでなく、リモートの場所にあるファイルを URL の仕様に対してコピーします。

`ADD` 命令の形式は次のようになります。

```dockerfile
ADD <source> <destination>
```

コピー元またはコピー先に空白が含まれている場合は、角かっこと二重引用符でパスを囲みます。

```dockerfile
ADD ["<source>", "<destination>"]
```

#### <a name="considerations-for-running-add-with-windows"></a>Windows での追加の実行に関する考慮事項

Windows では、変換先形式でスラッシュを使用する必要があります。 たとえば、次のような `ADD` の手順があります。

```dockerfile
ADD test1.txt /temp/
ADD test1.txt c:/temp/
```

一方、円記号付きの次の形式は機能しません。

```dockerfile
ADD test1.txt c:\temp\
```

さらに、Linux の `ADD` 動作では、コンピューター上の圧縮されたパッケージを展開できます。 この機能は、Windows ではこの機能を使用できません。

#### <a name="examples-of-using-add-with-windows"></a>Windows での ADD の使用例

次の例では、ソースディレクトリの内容を、コンテナーイメージ内の `sqllite` という名前のディレクトリに追加します。

```dockerfile
ADD source /sqlite/
```

次の例では、"config" で始まるすべてのファイルを、コンテナーイメージの `c:\temp` ディレクトリに追加します。

```dockerfile
ADD config* c:/temp/
```

次の例では、Windows 用の Python を、コンテナーイメージの `c:\temp` ディレクトリにダウンロードします。

```dockerfile
ADD https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe /temp/python-3.5.1.exe
```

`ADD` 命令の詳細については、「 [ADD reference](https://docs.docker.com/engine/reference/builder/#add)」を参照してください。

### <a name="workdir"></a>WORKDIR

`WORKDIR` 命令では、`RUN`、`CMD`、コンテナー イメージのインスタンスを実行する作業ディレクトリなど、他の Dockerfile 命令の作業ディレクトリを設定します。

`WORKDIR` 命令の形式は次のようになります。

```dockerfile
WORKDIR <path to working directory>
```

#### <a name="considerations-for-using-workdir-with-windows"></a>Windows で作業ディレクトリを使用する場合の考慮事項

Windows では、作業ディレクトリにバックスラッシュが含まれる場合、エスケープする必要があります。

```dockerfile
WORKDIR c:\\windows
```

**使用例**

```dockerfile
WORKDIR c:\\Apache24\\bin
```

`WORKDIR` 命令の詳細については、「[ワークディレクトリリファレンス](https://docs.docker.com/engine/reference/builder/#workdir)」を参照してください。

### <a name="cmd"></a>CMD

`CMD` 命令では、コンテナー イメージのインスタンスを展開するときに実行する既定のコマンドを設定します。 たとえば、コンテナーが NGINX web サーバーをホストする場合、`nginx.exe`のようなコマンドを使用して web サーバーを起動する指示が `CMD` に含まれることがあります。 Dockerfile に複数の `CMD` 命令を指定すると、最後の命令のみが評価されます。

`CMD` 命令の形式は次のようになります。

```dockerfile
# exec form

CMD ["<executable", "<param>"]

# shell form

CMD <command>
```

#### <a name="considerations-for-using-cmd-with-windows"></a>Windows での CMD の使用に関する考慮事項

Windows では、`CMD` 命令に指定されたファイル パスにはフォワード スラッシュを使用し、バックスラッシュをエスケープする (`\\`) 必要があります。 `CMD` の有効な手順は次のとおりです。

```dockerfile
# exec form

CMD ["c:\\Apache24\\bin\\httpd.exe", "-w"]

# shell form

CMD c:\\Apache24\\bin\\httpd.exe -w
```

ただし、適切なスラッシュを使用しない次の形式は機能しません。

```dockerfile
CMD c:\Apache24\bin\httpd.exe -w
```

`CMD` 命令の詳細については、「 [CMD reference](https://docs.docker.com/engine/reference/builder/#cmd)」を参照してください。

## <a name="escape-character"></a>[エスケープ文字]

多くの場合、Dockerfile 命令は複数の行にまたがる必要があります。 これを行うには、エスケープ文字を使用します。 既定の Dockerfile のエスケープ文字はバックスラッシュ `\` です。 ただし、バックスラッシュは Windows のファイルパスの区切り文字でもあるため、複数の行にまたがるために使用すると、問題が発生する可能性があります。 この問題を回避するには、パーサーディレクティブを使用して、既定のエスケープ文字を変更します。 パーサーディレクティブの詳細については、「[パーサーディレクティブ](https://docs.docker.com/engine/reference/builder/#parser-directives)」を参照してください。

次の例は、既定のエスケープ文字を使用して複数の行にまたがる1つの実行命令を示しています。

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command \
    $ErrorActionPreference = 'Stop'; \
    wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
    Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
    Remove-Item c:\python-3.5.1.exe -Force
```

エスケープ文字を変更するには、エスケープ パーサー ディレクティブを Dockerfile の最初の行に配置します。 これについては、次の例を参照してください。

>[!NOTE]
>エスケープ文字として使用できる値は、`\` と `` ` ``の2つだけです。

```dockerfile
# escape=`

FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command `
    $ErrorActionPreference = 'Stop'; `
    wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; `
    Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; `
    Remove-Item c:\python-3.5.1.exe -Force
```

エスケープパーサーディレクティブの詳細については、「[エスケープパーサーディレクティブ](https://docs.docker.com/engine/reference/builder/#escape)」を参照してください。

## <a name="powershell-in-dockerfile"></a>Dockerfile の PowerShell

### <a name="powershell-cmdlets"></a>PowerShell コマンドレット

PowerShell コマンドレットは、`RUN` 操作を使用して Dockerfile で実行できます。

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell -command Expand-Archive -Path c:\apache.zip -DestinationPath c:\
```

### <a name="rest-calls"></a>REST 呼び出し

PowerShell の `Invoke-WebRequest` コマンドレットは、web サービスから情報やファイルを収集するときに役立ちます。 たとえば、Python を含むイメージをビルドする場合は、次の例に示すように、`$ProgressPreference` を `SilentlyContinue` に設定すると、ダウンロードが高速になります。

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  $ProgressPreference = 'SilentlyContinue'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

>[!NOTE]
>`Invoke-WebRequest` Nano Server でも機能します。

イメージの作成プロセス中にファイルをダウンロードするために PowerShell を使用するもう 1 つの方法として、.NET WebClient ライブラリを使う方法があります。 この方法で、ダウンロードのパフォーマンスが向上します。 次の例では、WebClient ライブラリを使用して、Python ソフトウェアをダウンロードしています。

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  (New-Object System.Net.WebClient).DownloadFile('https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe','c:\python-3.5.1.exe') ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

>[!NOTE]
>現在、Nano Server では WebClient はサポートされていません。

### <a name="powershell-scripts"></a>Powershell スクリプト

場合によっては、イメージの作成プロセス中に使用するコンテナーにスクリプトをコピーしてから、コンテナー内からスクリプトを実行すると便利な場合があります。

>[!NOTE]
>これにより、イメージレイヤーのキャッシュが制限され、Dockerfile の読みやすさが低下します。

この例では、`ADD` 命令を使用して、ビルド コンピューターからコンテナーにスクリプトをコピーします。 このスクリプトは、RUN 命令を使用して実行されます。

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019
ADD script.ps1 /windows/temp/script.ps1
RUN powershell.exe -executionpolicy bypass c:\windows\temp\script.ps1
```

## <a name="docker-build"></a>Docker ビルド

Dockerfile を作成してディスクに保存すると、`docker build` を実行して新しいイメージを作成できます。 `docker build` コマンドには、いくつかの省略可能なパラメーターと、Dockerfile のパスを指定できます。 すべてのビルドオプションの一覧を含む、Docker ビルドの完全なドキュメントについては、「[ビルドリファレンス](https://docs.docker.com/engine/reference/commandline/build/#build)」を参照してください。

`docker build` コマンドの形式は次のようになります。

```dockerfile
docker build [OPTIONS] PATH
```

たとえば、次のコマンドでは、"iis" という名前のイメージが作成されます。

```dockerfile
docker build -t iis .
```

ビルドプロセスが開始されると、出力は状態を示し、スローされたエラーを返します。

```dockerfile
C:\> docker build -t iis .

Sending build context to Docker daemon 2.048 kB
Step 1 : FROM mcr.microsoft.com/windows/servercore:ltsc2019
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

その結果、新しいコンテナーイメージが作成されます。この例では、"iis" という名前が付けられています。

```dockerfile
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
iis                 latest              e2aafdfbe392        About a minute ago   207.8 MB
windowsservercore   latest              6801d964fda5        4 months ago         0 B
```

## <a name="further-reading-and-references"></a>参考資料と参照

- [Windows 用の Dockerfiles と Docker ビルドを最適化する](optimize-windows-dockerfile.md)
- [Dockerfile リファレンス](https://docs.docker.com/engine/reference/builder/)
