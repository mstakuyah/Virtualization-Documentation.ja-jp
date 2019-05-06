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
ms.openlocfilehash: 9ff6256ab9708533f72e9b3210f8a5fd32f4048a
ms.sourcegitcommit: c48dcfe43f73b96e0ebd661164b6dd164c775bfa
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/06/2019
ms.locfileid: "9610272"
---
# <a name="dockerfile-on-windows"></a>Windows 上の Dockerfile

Docker エンジンには、コンテナーの画像の作成を自動化するツールが含まれています。 コンテナーの画像を実行して手動で作成できますが、`docker commit`コマンドなど、多くの利点には、画像の自動作成のプロセスを採用します。

- コンテナー イメージをコードとして保存。
- メンテナンスやアップグレードの目的による、コンテナー イメージの迅速かつ正確な再作成。
- コンテナー イメージと開発サイクル間の継続的な統合。

この自動処理を行う Docker コンポーネントは、Dockerfile であり、`docker build` コマンドです。

[Dockerfile は、新しいコンテナー イメージを作成するために必要な指示が含まれているテキスト ファイルです。 たとえば、ベースとして使用する既存のイメージの指定、イメージの作成プロセス時に実行されるコマンド、コンテナー イメージの新しいインスタンスが展開されるときに実行されるコマンドなどの命令が含まれます。

Docker ビルドを Dockerfile を利用して、イメージの作成手順を実行する Docker エンジン コマンドです。

Windows コンテナーが Dockerfiles を使用して、それぞれの基本的な構文を理解する方法と、最も一般的な Dockerfile 指示が、このトピックでは表示されます。

このドキュメントでは、コンテナーの画像とコンテナーの画像のレイヤーの概念について説明します。 画像と画像の階層化の詳細を確認する場合は、[画像のクイック スタート ガイド](../quick-start/quick-start-images.md)を参照してください。

完全な Dockerfiles 見て、 [Dockerfile リファレンス](https://docs.docker.com/engine/reference/builder/)を参照してください。

## <a name="basic-syntax"></a>基本構文

ごく基本的なフォームでは、Dockerfile はとても単純です。 次の例では、IIS を含み、’hello world’ サイトを含む新しいイメージを作成します。 この例に含まれるコマンド (`#` で示されます) については、各手順で説明します。 この記事の以降のセクションでは、Dockerfile 構文規則と、Dockerfile 命令について詳しく説明します。

>[!NOTE]
>拡張子なしの Dockerfile を作成する必要があります。 Windows の場合は、任意のエディターでファイルを作成し、"Dockerfile"(引用符も含めます) 表記法で保存します。

```dockerfile
# Sample Dockerfile

# Indicates that the windowsservercore image will be used as the base image.
FROM microsoft/windowsservercore

# Metadata indicating an image maintainer.
LABEL maintainer="jshelton@contoso.com"

# Uses dism.exe to install the IIS role.
RUN dism.exe /online /enable-feature /all /featurename:iis-webserver /NoRestart

# Creates an HTML file and adds content to this file.
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html

# Sets a command or process that will run each time a container is run from the new image.
CMD [ "cmd" ]
```

Dockerfiles for Windows の他の例については、 [windows 版の Dockerfile リポジトリ](https://github.com/Microsoft/Virtualization-Documentation/tree/master/windows-container-samples)を参照してください。

## <a name="instructions"></a>手順

Dockerfile 手順については、Docker エンジン コンテナー イメージを作成するために必要な手順を提供します。 次の手順は、1 つずつと順番に実行されます。 次の例は、Dockerfiles で頻繁に使用する手順です。 Dockerfile 手順の一覧については、 [Dockerfile リファレンス](https://docs.docker.com/engine/reference/builder/)を参照してください。

### <a name="from"></a>FROM

`FROM` 命令では、新しいイメージ作成プロセス中に使用されるコンテナー イメージを設定します。 たとえば、命令 `FROM microsoft/windowsservercore` を使用すると、結果のイメージは Windows Server Core ベース OS イメージから派生し、依存します。 指定したイメージが、Docker ビルド プロセスが実行されているシステムに存在しない場合、Docker エンジンは、パブリックまたはプライベートのイメージ レジストリからイメージのダウンロードを試行します。

FROM 命令の形式では、次のようなとおりです。

```dockerfile
FROM <image>
```

[差出人] コマンドの例を示します。

```dockerfile
FROM microsoft/windowsservercore
```

詳細については、[差出人のリファレンス](https://docs.docker.com/engine/reference/builder/#from)を参照してください。

### <a name="run"></a>RUN

`RUN` 命令は、コマンドを実行し、新しいコンテナー イメージにキャプチャするように指定します。 これらのコマンドには、ソフトウェアのインストール、ファイルとディレクトリの作成、環境構成の作成などの項目が含まれます。

実行命令は、次のようになります。

```dockerfile
# exec form

RUN ["<executable>", "<param 1>", "<param 2>"]

# shell form

RUN <command>
```

方法で実行およびシェル形式の違いは、`RUN`命令を実行します。 exec フォームを使用すると、指定したプログラムが明示的に実行されます。

実行フォームの例を示します。

```dockerfile
FROM microsoft/windowsservercore

RUN ["powershell", "New-Item", "c:/test"]
```

結果のイメージが実行される、`powershell New-Item c:/test`コマンド。

```dockerfile
docker history doc-exe-method

IMAGE               CREATED             CREATED BY                    SIZE                COMMENT
b3452b13e472        2 minutes ago       powershell New-Item c:/test   30.76 MB
```

比較するは、次の例は、シェル フォームで同じ操作を実行します。

```dockerfile
FROM microsoft/windowsservercore

RUN powershell New-Item c:\test
```

結果のイメージがの実行命令`cmd /S /C powershell New-Item c:\test`します。

```dockerfile
docker history doc-shell-method

IMAGE               CREATED             CREATED BY                              SIZE                COMMENT
062a543374fc        19 seconds ago      cmd /S /C powershell New-Item c:\test   30.76 MB
```

### <a name="considerations-for-using-run-with-windows"></a>Windows で実行を使用するための考慮事項

Windows では、exec 形式で `RUN` 命令を使用する場合、バックスラッシュをエスケープする必要があります。

```dockerfile
RUN ["powershell", "New-Item", "c:\\test"]
```

目的のプログラムでは、Windows インストーラーが、抽出をセットアップする必要があります、 `/x:<directory>` (無音) の実際のインストール手順を起動する前にフラグを設定します。 その他の操作を行う前に終了] コマンドを使用するも待つ必要があります。 それ以外の場合、プロセスは、何もインストールせずに途中で終了します。 詳細については、次の例を参照してください。

#### <a name="examples-of-using-run-with-windows"></a>Windows での使用例を行う

Dockerfile 次の例では、IIS のインストール、コンテナーの画像に DISM を使用します。

```dockerfile
RUN dism.exe /online /enable-feature /all /featurename:iis-webserver /NoRestart
```

この例では、Visual Studio 再頒布可能パッケージをインストールします。 `Start-Process` `-Wait`インストーラーを実行するパラメーターを使用します。 これにより、Dockerfile 内の次の命令移動する前に、インストールが完了したことができます。

```dockerfile
RUN powershell.exe -Command Start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait
```

実行命令の詳細については、[参照を実行](https://docs.docker.com/engine/reference/builder/#run)するを参照してください。

### <a name="copy"></a>コピー

`COPY`命令をファイル システムのコンテナーのファイルとディレクトリをコピーします。 ファイルとディレクトリは、[Dockerfile に対する相対的なパスでなければなりません。

`COPY`命令の形式は次のようなについて説明します。

```dockerfile
COPY <source> <destination>
```

ソースまたは宛先には、空白文字が含まれている場合は、パスを次の例に示すように角かっこや二重引用符で囲みます。

```dockerfile
COPY ["<source>", "<destination>"]
```

#### <a name="considerations-for-using-copy-with-windows"></a>Windows でのコピーを使用するための考慮事項

Windows では、変換先形式でスラッシュを使用する必要があります。 たとえば、これらは有効な`COPY`手順。

```dockerfile
COPY test1.txt /temp/
COPY test1.txt c:/temp/
```

その一方で、円記号では、次の形式は動作しません。

```dockerfile
COPY test1.txt c:\temp\
```

#### <a name="examples-of-using-copy-with-windows"></a>Windows での使用例をコピーします。

次の例では、ソース ディレクトリの内容を追加するディレクトリ`sqllite`、コンテナーの画像にします。

```dockerfile
COPY source /sqlite/
```

次の例に構成で始まるすべてのファイルを追加、`c:\temp`コンテナーの画像のディレクトリ。

```dockerfile
COPY config* c:/temp/
```

に関する詳細について、`COPY`命令を[コピーを参照](https://docs.docker.com/engine/reference/builder/#copy)してください。

### <a name="add"></a>ADD

追加の命令が [コピーの命令のようながさらに多くの機能を備えたです。 `ADD` 命令は、ホストのファイルがコンテナー イメージにコピーされるだけでなく、リモートの場所にあるファイルを URL の仕様に対してコピーします。

`ADD`命令の形式は次のようなについて説明します。

```dockerfile
ADD <source> <destination>
```

ソースまたは宛先が空白を含める場合は、パスを角かっこや二重引用符で囲みます。

```dockerfile
ADD ["<source>", "<destination>"]
```

#### <a name="considerations-for-running-add-with-windows"></a>Windows を実行するための考慮事項を追加します。

Windows では、変換先形式でスラッシュを使用する必要があります。 たとえば、これらは有効な`ADD`手順。

```dockerfile
ADD test1.txt /temp/
ADD test1.txt c:/temp/
```

その一方で、円記号では、次の形式は動作しません。

```dockerfile
ADD test1.txt c:\temp\
```

さらに、Linux の `ADD` 動作では、コンピューター上の圧縮されたパッケージを展開できます。 この機能は、Windows ではこの機能を使用できません。

#### <a name="examples-of-using-add-with-windows"></a>Windows での使用例を追加します。

次の例では、ソース ディレクトリの内容を追加するディレクトリ`sqllite`、コンテナーの画像にします。

```dockerfile
ADD source /sqlite/
```

次の例に"config"で始まるすべてのファイルを追加、`c:\temp`コンテナーの画像のディレクトリです。

```dockerfile
ADD config* c:/temp/
```

次の例をダウンロード Python for Windows に、`c:\temp`コンテナーの画像のディレクトリです。

```dockerfile
ADD https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe /temp/python-3.5.1.exe
```

詳細については、`ADD`命令、[参照を追加](https://docs.docker.com/engine/reference/builder/#add)するを参照してください。

### <a name="workdir"></a>WORKDIR

`WORKDIR` 命令では、`RUN`、`CMD`、コンテナー イメージのインスタンスを実行する作業ディレクトリなど、他の Dockerfile 命令の作業ディレクトリを設定します。

`WORKDIR`命令の形式は次のようなについて説明します。

```dockerfile
WORKDIR <path to working directory>
```

#### <a name="considerations-for-using-workdir-with-windows"></a>Windows で WORKDIR を使用するための考慮事項

Windows では、作業ディレクトリにバックスラッシュが含まれる場合、エスケープする必要があります。

```dockerfile
WORKDIR c:\\windows
```

**例**

```dockerfile
WORKDIR c:\\Apache24\\bin
```

詳細については、`WORKDIR`命令[WORKDIR リファレンス](https://docs.docker.com/engine/reference/builder/#workdir)を参照してください。

### <a name="cmd"></a>CMD

`CMD` 命令では、コンテナー イメージのインスタンスを展開するときに実行する既定のコマンドを設定します。 たとえば、コンテナーが NGINX web サーバーをホストする場合は、`CMD`のようなコマンドと、web サーバーを開始する手順があります。`nginx.exe`します。 Dockerfile に複数の `CMD` 命令を指定すると、最後の命令のみが評価されます。

`CMD`命令の形式は次のようなについて説明します。

```dockerfile
# exec form

CMD ["<executable", "<param>"]

# shell form

CMD <command>
```

#### <a name="considerations-for-using-cmd-with-windows"></a>Windows でコマンドを使用するための考慮事項

Windows では、`CMD` 命令に指定されたファイル パスにはフォワード スラッシュを使用し、バックスラッシュをエスケープする (`\\`) 必要があります。 有効では、次の`CMD`手順。

```dockerfile
# exec form

CMD ["c:\\Apache24\\bin\\httpd.exe", "-w"]

# shell form

CMD c:\\Apache24\\bin\\httpd.exe -w
```

ただし、適切なスラッシュせずに、次の形式は機能しません。

```dockerfile
CMD c:\Apache24\bin\httpd.exe -w
```

詳細については、`CMD`命令、 [「cmd」を参照](https://docs.docker.com/engine/reference/builder/#cmd)してください。

## <a name="escape-character"></a>エスケープ文字

多くの場合、Dockerfile 命令を複数の行にわたる必要があります。 これを行うには、エスケープ文字を使用できます。 既定の Dockerfile のエスケープ文字はバックスラッシュ `\` です。 ただし、バック スラッシュは Windows のファイル パスの区切り文字でもあるので使い方、複数行に問題が発生します。 この問題を回避するのにには、既定のエスケープ文字を変更するのには、パーサー ディレクティブを使用することができます。 詳細については、パーサー ディレクティブ、[パーサー ディレクティブ](https://docs.docker.com/engine/reference/builder/#parser-directives)を参照してください。

次の例では、既定のエスケープ文字を使用して複数の行に表示される 1 つの実行命令が表示されます。

```dockerfile
FROM microsoft/windowsservercore

RUN powershell.exe -Command \
    $ErrorActionPreference = 'Stop'; \
    wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
    Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
    Remove-Item c:\python-3.5.1.exe -Force
```

エスケープ文字を変更するには、エスケープ パーサー ディレクティブを Dockerfile の最初の行に配置します。 これは、次の例では表示できます。

>[!NOTE]
>2 つの値は、エスケープ文字として使用することができます:`\`と`` ` ``します。

```dockerfile
# escape=`

FROM microsoft/windowsservercore

RUN powershell.exe -Command `
    $ErrorActionPreference = 'Stop'; `
    wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; `
    Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; `
    Remove-Item c:\python-3.5.1.exe -Force
```

詳細については、エスケープ パーサー ディレクティブ、[エスケープ パーサー ディレクティブ](https://docs.docker.com/engine/reference/builder/#escape)を参照してください。

## <a name="powershell-in-dockerfile"></a>Dockerfile の PowerShell

### <a name="powershell-cmdlets"></a>PowerShell コマンドレット

PowerShell コマンドレットを実行するには、Dockerfile での`RUN`操作します。

```dockerfile
FROM microsoft/windowsservercore

RUN powershell -command Expand-Archive -Path c:\apache.zip -DestinationPath c:\
```

### <a name="rest-calls"></a>他の通話

PowerShell の`Invoke-WebRequest`web サービスからの情報またはファイルを収集するとき、コマンドレットが役に立ちます。 たとえば、Python を含むイメージを作成する場合は、設定できます`$ProgressPreference`に`SilentlyContinue`次の例のように、高速なダウンロードを達成するためにします。

```dockerfile
FROM microsoft/windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  $ProgressPreference = 'SilentlyContinue'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

>[!NOTE]
>`Invoke-WebRequest` Nano サーバーでも動作します。

イメージの作成プロセス中にファイルをダウンロードするために PowerShell を使用するもう 1 つの方法として、.NET WebClient ライブラリを使う方法があります。 この方法で、ダウンロードのパフォーマンスが向上します。 次の例では、WebClient ライブラリを使用して、Python ソフトウェアをダウンロードしています。

```dockerfile
FROM microsoft/windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  (New-Object System.Net.WebClient).DownloadFile('https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe','c:\python-3.5.1.exe') ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

>[!NOTE]
>Nano サーバーは、[WebClient を現在サポートしていません。

### <a name="powershell-scripts"></a>Powershell スクリプト

場合によっては、イメージの作成処理を使用しているコンテナーにスクリプトをコピーし、コンテナー内からスクリプトを実行すると便利なポイントがあります。

>[!NOTE]
>任意の画像レイヤーのキャッシュを制限する、Dockerfile の読みやすさを縮小します。

この例では、`ADD` 命令を使用して、ビルド コンピューターからコンテナーにスクリプトをコピーします。 このスクリプトは、RUN 命令を使用して実行されます。

```dockerfile
FROM microsoft/windowsservercore
ADD script.ps1 /windows/temp/script.ps1
RUN powershell.exe -executionpolicy bypass c:\windows\temp\script.ps1
```

## <a name="docker-build"></a>Docker ビルド

実行することが、Dockerfile が作成され、ディスクに保存されている、`docker build`新しいイメージを作成します。 `docker build` コマンドには、いくつかの省略可能なパラメーターと、Dockerfile のパスを指定できます。 Docker 作成の詳細については、すべてのリストを含むオプションを作成、[参照を作成](https://docs.docker.com/engine/reference/commandline/build/#build)するを参照してください。

書式設定、`docker build`コマンドは次のようなについて説明します。

```dockerfile
docker build [OPTIONS] PATH
```

たとえば、次のコマンドが"iis"という名前のイメージを作成します。

```dockerfile
docker build -t iis .
```

ビルド処理が開始されると、出力が状態を示す、スロー エラーを返します。

```dockerfile
C:\> docker build -t iis .

Sending build context to Docker daemon 2.048 kB
Step 1 : FROM micrsoft/windowsservercore
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

結果は、この例では"iis"と呼ばれる新しいコンテナー イメージ

```dockerfile
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
iis                 latest              e2aafdfbe392        About a minute ago   207.8 MB
windowsservercore   latest              6801d964fda5        4 months ago         0 B
```

## <a name="further-reading-and-references"></a>さらに閲覧と参照

- [Windows 用ビルド Dockerfiles と Docker を最適化します。](optimize-windows-dockerfile.md)
- [Dockerfile リファレンス](https://docs.docker.com/engine/reference/builder/)
