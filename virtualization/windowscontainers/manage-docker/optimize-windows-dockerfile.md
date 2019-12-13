---
title: Windows Dockerfile を最適化する
description: Windows コンテナー用に Dockerfile を最適化します。
keywords: Docker, コンテナー
author: cwilhit
ms.date: 05/03/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb2848ca-683e-4361-a750-0d1d14ec8031
ms.openlocfilehash: ae633c7ba5d9672335addcc582988fc47c13ed79
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910152"
---
# <a name="optimize-windows-dockerfiles"></a>Windows Dockerfile を最適化する

Docker のビルドプロセスと作成された Docker イメージの両方を最適化するには、さまざまな方法があります。 この記事では、Docker のビルドプロセスのしくみと、Windows コンテナーのイメージを最適に作成する方法について説明します。

## <a name="image-layers-in-docker-build"></a>Docker ビルドのイメージレイヤー

Docker ビルドを最適化するには、Docker のビルドのしくみを理解しておく必要があります。 Docker ビルド プロセスでは、Dockerfile が使用されて、アクション可能な各命令が 1 つずつ専用の一時的なコンテナーで実行されます。 その結果、アクション可能な各命令に対して新しいイメージ レイヤーが作成されます。

たとえば、次のサンプル Dockerfile は、`mcr.microsoft.com/windows/servercore:ltsc2019` ベース OS イメージを使用し、IIS をインストールして、単純な web サイトを作成します。

```dockerfile
# Sample Dockerfile

FROM mcr.microsoft.com/windows/servercore:ltsc2019
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

この Dockerfile では、2つのレイヤーを持つイメージが生成されることが予想されます。1つはコンテナー OS イメージ用で、もう1つは IIS と web サイトで構成されます。 ただし、実際のイメージには多くのレイヤーがあり、各レイヤーは前のレイヤーに依存しています。

これをわかりやすくするために、サンプルの Dockerfile が作成されたイメージに対して `docker history` コマンドを実行してみましょう。

```dockerfile
docker history iis

IMAGE               CREATED              CREATED BY                                      SIZE                COMMENT
f4caf476e909        16 seconds ago       cmd /S /C REM (nop) CMD ["cmd"]                 41.84 kB
f0e017e5b088        21 seconds ago       cmd /S /C echo "Hello World - Dockerfile" > c   6.816 MB
88438e174b7c        About a minute ago   cmd /S /C dism /online /enable-feature /all /   162.7 MB
6801d964fda5        4 months ago                                                         0 B
```

出力には、このイメージに4つのレイヤーがあることが示されています。基本レイヤーと、Dockerfile の各命令にマップされる3つの追加レイヤーです。 最下位のレイヤー (この例では `6801d964fda5`) は、ベース OS イメージを表します。 1つは IIS のインストールです。 次のレイヤーには、新しい Web サイトなどが含まれます。

Dockerfiles は、イメージレイヤーを最小化し、ビルドのパフォーマンスを最適化し、読みやすさによってアクセシビリティを最適化するために記述できます。 最終的に、同じイメージ ビルド タスクを実行するにはさまざまな方法があります。 Dockerfile の形式がビルド時間にどのように影響するかを理解し、それによって作成されるイメージによってオートメーションエクスペリエンスが向上します。

## <a name="optimize-image-size"></a>イメージサイズの最適化

領域の要件によっては、Docker コンテナーイメージを作成するときにイメージのサイズが重要な要素になることがあります。 コンテナー イメージは、レジストリとホストの間を移動され、エクスポートおよびインポートされて、最終的にはスペースを消費します。 このセクションでは、Windows コンテナーの Docker ビルドプロセス中にイメージサイズを最小限に抑える方法について説明します。

Dockerfile のベストプラクティスの詳細については、「 [Docker.com で Dockerfile を作成するためのベストプラクティス](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/)」を参照してください。

### <a name="group-related-actions"></a>関連するアクションをグループ化する

各 `RUN` 命令によってコンテナーイメージに新しいレイヤーが作成されるため、アクションを1つの `RUN` 命令にグループ化すると、Dockerfile 内のレイヤーの数を減らすことができます。 レイヤーの数を最小限に抑えてもイメージのサイズに大きな影響はありませんが、後の例で示すように、関連するアクションをグループ化するとイメージのサイズに大きな影響があります。

このセクションでは、同じ処理を実行する2つの Dockerfiles の例を比較します。 ただし、1つの Dockerfile にはアクションごとに1つの命令があり、もう一方の Dockerfile には、関連するアクションがグループ化されています。

次のグループ化されていない例 Dockerfile は、Windows 用の Python をダウンロードしてインストールし、インストールが完了したらダウンロードしたセットアップファイルを削除します。 この Dockerfile では、各アクションに独自の `RUN` 命令が指定されています。

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command Invoke-WebRequest "https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe" -OutFile c:\python-3.5.1.exe
RUN powershell.exe -Command Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait
RUN powershell.exe -Command Remove-Item c:\python-3.5.1.exe -Force
```

結果として作成されるイメージは、`RUN` 命令ごとに 1 つずつ、3 つの追加レイヤーで構成されます。

```dockerfile
docker history doc-example-1

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
a395ca26777f        15 seconds ago      cmd /S /C powershell.exe -Command Remove-Item   24.56 MB
6c137f466d28        28 seconds ago      cmd /S /C powershell.exe -Command Start-Proce   178.6 MB
957147160e8d        3 minutes ago       cmd /S /C powershell.exe -Command Invoke-WebR   125.7 MB
```

2番目の例は、まったく同じ操作を実行する Dockerfile です。 ただし、すべての関連するアクションは、単一の `RUN` 命令の下にグループ化されています。 `RUN` 命令の各手順は、Dockerfile の新しい行にあります。一方、'\\' 文字は、行の折り返しに使用されます。

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

結果のイメージには、`RUN` 命令に対して追加されたレイヤーが1つだけあります。

```dockerfile
docker history doc-example-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
69e44f37c748        54 seconds ago      cmd /S /C powershell.exe -Command   $ErrorAct   216.3 MB
```

### <a name="remove-excess-files"></a>余分なファイルを削除する

使用後に必要のない、Dockerfile 内のファイルがある場合は、それを削除してイメージのサイズを小さくすることができます。 この処理は、イメージ レイヤーへのファイルのコピーが行われるのと同じステップで行う必要があります。 これにより、ファイルが下位レベルのイメージレイヤーに保持されなくなります。

次の Dockerfile の例では、Python パッケージがダウンロードされ、実行された後、削除されます。 このすべてを 1 つの `RUN` 操作で行うので、作成されるイメージ レイヤーは 1 つです。

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

## <a name="optimize-build-speed"></a>ビルド速度の最適化

### <a name="multiple-lines"></a>複数行

操作を複数の個別の命令に分割して、Docker のビルド速度を最適化することができます。 複数の `RUN` 操作では、`RUN` 命令ごとに個々のレイヤーが作成されるため、キャッシュの有効性が向上します。 同じ命令が別の Docker ビルド操作で既に実行されている場合、このキャッシュされた操作 (イメージレイヤー) が再利用されるため、Docker ビルドランタイムが減少します。

次の例では、Apache と Visual Studio の再配布パッケージの両方がダウンロードされ、インストールされた後、不要になったファイルを削除することによってクリーンアップされます。 これはすべて、1つの `RUN` 命令を使用して行われます。 これらのアクションのいずれかが更新されると、すべてのアクションが再実行されます。

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell -Command \

  # Download software ; \
    
  wget https://www.apachelounge.com/download/VC11/binaries/httpd-2.4.18-win32-VC11.zip -OutFile c:\apache.zip ; \
  wget "https://download.microsoft.com/download/1/6/B/16B06F60-3B20-4FF2-B699-5E9B7962F9AE/VSU_4/vcredist_x86.exe" -OutFile c:\vcredist.exe ; \
  wget -Uri http://windows.php.net/downloads/releases/php-5.5.33-Win32-VC11-x86.zip -OutFile c:\php.zip ; \

  # Install Software ; \
    
  Expand-Archive -Path c:\php.zip -DestinationPath c:\php ; \
  Expand-Archive -Path c:\apache.zip -DestinationPath c:\ ; \
  start-Process c:\vcredist.exe -ArgumentList '/quiet' -Wait ; \
    
  # Remove unneeded files ; \
     
  Remove-Item c:\apache.zip -Force; \
  Remove-Item c:\vcredist.exe -Force; \
  Remove-Item c:\php.zip
```

生成されるイメージには2つのレイヤーがあります。1つは基本 OS イメージ用で、もう1つは1つの `RUN` 命令からのすべての操作が含まれています。

```dockerfile
docker history doc-sample-1

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
9bdf3a21fd41        8 minutes ago       cmd /S /C powershell -Command     Invoke-WebR   205.8 MB
6801d964fda5        5 months ago                                                        0 B
```

比較して、ここでは、同じ操作を3つの `RUN` 命令に分割します。 この場合、各 `RUN` 命令はコンテナーイメージレイヤーにキャッシュされ、以降の Dockerfile ビルドでは変更されたものだけを再実行する必要があります。

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    wget https://www.apachelounge.com/download/VC11/binaries/httpd-2.4.18-win32-VC11.zip -OutFile c:\apache.zip ; \
    Expand-Archive -Path c:\apache.zip -DestinationPath c:\ ; \
    Remove-Item c:\apache.zip -Force

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    wget "https://download.microsoft.com/download/1/6/B/16B06F60-3B20-4FF2-B699-5E9B7962F9AE/VSU_4/vcredist_x86.exe" -OutFile c:\vcredist.exe ; \
    start-Process c:\vcredist.exe -ArgumentList '/quiet' -Wait ; \
    Remove-Item c:\vcredist.exe -Force

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    wget http://windows.php.net/downloads/releases/php-5.5.33-Win32-VC11-x86.zip -OutFile c:\php.zip ; \
    Expand-Archive -Path c:\php.zip -DestinationPath c:\php ; \
    Remove-Item c:\php.zip -Force
```

結果のイメージは、4つのレイヤーで構成されます。基本 OS イメージのための1つのレイヤーと3つの `RUN` 命令。 各 `RUN` 命令は独自のレイヤーで実行されるため、この Dockerfile の後続の実行、または異なる Dockerfile 内の同じ命令セットが、キャッシュされたイメージレイヤーを使用してビルド時間を短縮します。

```dockerfile
docker history doc-sample-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
ddf43b1f3751        6 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   127.2 MB
d43abb81204a        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   66.46 MB
7a21073861a1        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   115.8 MB
6801d964fda5        5 months ago
```

イメージキャッシュを使用する場合は、次のセクションで説明するように、命令の順序を付けることが重要です。

### <a name="ordering-instructions"></a>命令の順序

Dockerfile は上から下に処理され、各命令はキャッシュされているレイヤーと比較されます。 キャッシュされたレイヤーがない命令が見つかると、その命令およびそれ以降のすべての命令が新しいコンテナー イメージ レイヤーで処理されます。 このため、命令の配置の順序が重要になります。 変化しない命令は、Dockerfile の前の方に配置します。 変化する可能性のある命令は、Dockerfile の後の方に配置します。 そうすることで、既存のキャッシュが役に立たなくなる可能性が減ります。

次の例は、Dockerfile 命令の順序付けがキャッシュの有効性に与える影響を示しています。 この単純な Dockerfile の例では、4つの番号付きフォルダーがあります。  

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-3
RUN mkdir test-4
```

生成されるイメージには5つのレイヤーがあります。1つは基本 OS イメージ用で、もう1つは `RUN` 命令です。

```dockerfile
docker history doc-sample-1

IMAGE               CREATED              CREATED BY               SIZE                COMMENT
afba1a3def0a        38 seconds ago       cmd /S /C mkdir test-4   42.46 MB
86f1fe772d5c        49 seconds ago       cmd /S /C mkdir test-3   42.35 MB
68fda53ce682        About a minute ago   cmd /S /C mkdir test-2   6.745 MB
5e5aa8ba1bc2        About a minute ago   cmd /S /C mkdir test-1   7.12 MB
6801d964fda5        5 months ago                                  0 B
```

次の Dockerfile が少し変更され、3番目の `RUN` 命令が新しいファイルに変更されました。 この Dockerfile に対して Docker ビルドを実行すると、前の例と同じ最初の 3 つの命令は、キャッシュされたイメージ レイヤーを使用します。 ただし、変更された `RUN` 命令はキャッシュされないため、変更された命令とそれ以降のすべての命令に対して新しいレイヤーが作成されます。

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-5
RUN mkdir test-4
```

新しいイメージのイメージ Id を、このセクションの最初の例で使用したものと比較すると、下から上にある最初の3つのレイヤーが共有されますが、4番目と5番目のレイヤーは一意です。

```dockerfile
docker history doc-sample-2

IMAGE               CREATED             CREATED BY               SIZE                COMMENT
c92cc95632fb        28 seconds ago      cmd /S /C mkdir test-4   5.644 MB
2f05e6f5c523        37 seconds ago      cmd /S /C mkdir test-5   5.01 MB
68fda53ce682        3 minutes ago       cmd /S /C mkdir test-2   6.745 MB
5e5aa8ba1bc2        4 minutes ago       cmd /S /C mkdir test-1   7.12 MB
6801d964fda5        5 months ago                                 0 B
```

## <a name="cosmetic-optimization"></a>コスメティックの最適化

### <a name="instruction-case"></a>命令ケース

Dockerfile 命令では大文字と小文字は区別されませんが、大文字を使用します。 これにより、命令呼び出しと命令操作を区別することで読みやすさが向上します。 次の2つの例は、初期化されていない Dockerfile を比較しています。

次に示すのは、呼び出されていない Dockerfile です。

```dockerfile
# Sample Dockerfile

from mcr.microsoft.com/windows/servercore:ltsc2019
run dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
run echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
cmd [ "cmd" ]
```

次に示すのは、大文字を使用する同じ Dockerfile です。

```dockerfile
# Sample Dockerfile

FROM mcr.microsoft.com/windows/servercore:ltsc2019
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

### <a name="line-wrapping"></a>行の折り返し

長い複雑な操作は、円記号 `\` 文字で複数行に分けることができます。 次の Dockerfile は、Visual Studio の再頒布可能パッケージをインストールし、インストーラー ファイルを削除して、構成ファイルを作成します。 これら 3 つの操作すべてが 1 行で指定されています。

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell -Command c:\vcredist_x86.exe /quiet ; Remove-Item c:\vcredist_x86.exe -Force ; New-Item c:\config.ini
```

コマンドは円記号で分割できます。これにより、1つの `RUN` 命令の各操作がそれぞれの行に指定されます。

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait ; \
    Remove-Item c:\vcredist_x86.exe -Force ; \
    New-Item c:\config.ini
```

## <a name="further-reading-and-references"></a>参考資料と参照

[Windows 上の dockerfile](manage-windows-dockerfile.md)

[Docker.com で Dockerfiles を作成するためのベストプラクティス](https://docs.docker.com/engine/reference/builder/)
