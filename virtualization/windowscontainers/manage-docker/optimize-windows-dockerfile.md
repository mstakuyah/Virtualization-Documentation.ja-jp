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
ms.openlocfilehash: 056ab87189e8e423df5758be0f622a43b92c9056
ms.sourcegitcommit: c4a3f88d1663dd19336bfd4ede0368cb18550ac7
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/31/2019
ms.locfileid: "9882955"
---
# <a name="optimize-windows-dockerfiles"></a>Windows Dockerfile を最適化する

Docker のビルドプロセスと生成される Docker 画像の両方を最適化するには、さまざまな方法があります。 この記事では、Docker のビルドプロセスのしくみと、Windows コンテナーの画像を最適に作成する方法について説明します。

## <a name="image-layers-in-docker-build"></a>Docker ビルドのイメージレイヤー

Docker のビルドを最適化するには、Docker のビルドのしくみを理解する必要があります。 Docker ビルド プロセスでは、Dockerfile が使用されて、アクション可能な各命令が 1 つずつ専用の一時的なコンテナーで実行されます。 その結果、アクション可能な各命令に対して新しいイメージ レイヤーが作成されます。

たとえば、次のサンプル Dockerfile は、 `windowsservercore`ベース OS イメージを使用し、IIS をインストールして、単純な web サイトを作成します。

```dockerfile
# Sample Dockerfile

FROM windowsservercore
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

この Dockerfile では、2つのレイヤー、コンテナ OS イメージ用、もう1つは IIS と web サイトを含む画像を生成することが想定されている場合があります。 ただし、実際の画像には多くのレイヤーがあり、各レイヤーは、その前にあるものに基づいています。

わかりやすくするために、サンプルの Dockerfile の画像に対してコマンドを`docker history`実行していきましょう。

```dockerfile
docker history iis

IMAGE               CREATED              CREATED BY                                      SIZE                COMMENT
f4caf476e909        16 seconds ago       cmd /S /C REM (nop) CMD ["cmd"]                 41.84 kB
f0e017e5b088        21 seconds ago       cmd /S /C echo "Hello World - Dockerfile" > c   6.816 MB
88438e174b7c        About a minute ago   cmd /S /C dism /online /enable-feature /all /   162.7 MB
6801d964fda5        4 months ago                                                         0 B
```

この画像には、基本レイヤーと、Dockerfile の各命令にマップされる3つの追加レイヤーの4つのレイヤーが含まれていることが出力されています。 最下位のレイヤー (この例では `6801d964fda5`) は、ベース OS イメージを表します。 1つのレイヤーが IIS にインストールされています。 次のレイヤーには、新しい Web サイトなどが含まれます。

Dockerfiles は、画像レイヤーの最小化、ビルドのパフォーマンスの最適化、アクセシビリティの最適化を行うことができます。 最終的に、同じイメージ ビルド タスクを実行するにはさまざまな方法があります。 Dockerfile の形式がビルド時にどのように影響するかについて説明します。これにより、作成した画像によってオートメーションの操作性が向上します。

## <a name="optimize-image-size"></a>イメージサイズの最適化

必要な領域に応じて、イメージサイズは、Docker コンテナイメージを構築するときに重要な要素となります。 コンテナー イメージは、レジストリとホストの間を移動され、エクスポートおよびインポートされて、最終的にはスペースを消費します。 このセクションでは、Windows コンテナーの Docker のビルドプロセス中に画像のサイズを最小化する方法について説明します。

Dockerfile のベストプラクティスの詳細については、「 [Docker.com での Dockerfile の作成に関するベストプラクティス](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/)」を参照してください。

### <a name="group-related-actions"></a>関連するアクションをグループ化する

各`RUN`命令は、コンテナーイメージに新しいレイヤーを作成するため、アクションを`RUN` 1 つの命令にグループ化することで、dockerfile のレイヤーの数を減らすことができます。 レイヤーの数を最小限に抑えてもイメージのサイズに大きな影響はありませんが、後の例で示すように、関連するアクションをグループ化するとイメージのサイズに大きな影響があります。

このセクションでは、同じ操作を実行する2つの例の Dockerfiles を比較します。 ただし、1つの Dockerfile は操作ごとに1つの命令を持ちますが、もう1つの Dockerfile には関連するアクションがグループ化されています。

次のグループ化されていない例の Dockerfile は、Windows 用の Python をダウンロードしてインストールし、インストールが完了するとダウンロードしたセットアップファイルを削除します。 この Dockerfile では、各アクションに固有`RUN`の命令が与えられます。

```dockerfile
FROM windowsservercore

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

2番目の例は、まったく同じ操作を実行する Dockerfile です。 ただし、すべての関連アクションは1つ`RUN`の命令の下にグループ化されています。 `RUN`命令の各手順は Dockerfile の新しい行にありますが、' \ \ ' 文字は行の折り返しに使用されます。

```dockerfile
FROM windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

結果と`RUN`して得られる画像の追加レイヤーは、この命令につき1つだけです。

```dockerfile
docker history doc-example-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
69e44f37c748        54 seconds ago      cmd /S /C powershell.exe -Command   $ErrorAct   216.3 MB
```

### <a name="remove-excess-files"></a>余分なファイルを削除する

インストーラーなどのファイルが Dockerfile に含まれている場合、そのファイルを使用した後に必要ではない場合は、そのファイルを削除して画像のサイズを小さくすることができます。 この処理は、イメージ レイヤーへのファイルのコピーが行われるのと同じステップで行う必要があります。 こうすることにより、ファイルが下位レベルのイメージレイヤーで永続化されなくなります。

次の Dockerfile の例では、Python パッケージがダウンロードされ、実行された後、削除されています。 このすべてを 1 つの `RUN` 操作で行うので、作成されるイメージ レイヤーは 1 つです。

```dockerfile
FROM windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

## <a name="optimize-build-speed"></a>ビルド速度を最適化する

### <a name="multiple-lines"></a>複数行

操作を複数の個別の命令に分割して、Docker のビルド速度を最適化することができます。 各`RUN` `RUN`命令で個別のレイヤーが作成されるため、複数の操作でキャッシュの有効性が向上します。 同じ命令が別の Docker ビルド操作で既に実行されている場合は、このキャッシュ操作 (イメージレイヤー) が再利用され、その結果、Docker build runtime が減少します。

次の例では、Apache と Visual Studio の両方のパッケージを再頒布して、不要になったファイルを削除することによってパッケージを再配布します。 これはすべて1つ`RUN`の命令で実行されます。 これらの操作のいずれかが更新されると、すべての操作が再実行されます。

```dockerfile
FROM windowsservercore

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

結果として得られるイメージには、ベース OS イメージ用の2つのレイヤーと、1 `RUN`つの命令からすべての操作が含まれているものがあります。

```dockerfile
docker history doc-sample-1

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
9bdf3a21fd41        8 minutes ago       cmd /S /C powershell -Command     Invoke-WebR   205.8 MB
6801d964fda5        5 months ago                                                        0 B
```

ここでは、3つ`RUN`の手順に分かれた同じ操作を次に示します。 この場合、各`RUN`命令はコンテナーのイメージレイヤーにキャッシュされ、それ以降の Dockerfile ビルドで再実行する必要があるのは、変更されたものだけです。

```dockerfile
FROM windowsservercore

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

結果の画像は、4つのレイヤーで構成されます。ベース OS イメージの1つのレイヤーと3つ`RUN`の手順のそれぞれ。 各`RUN`命令はそれぞれ専用のレイヤーで実行されるため、以降のこの dockerfile または異なる dockerfile の同じセットの命令は、キャッシュされた画像レイヤーを使用し、ビルド時間を短縮します。

```dockerfile
docker history doc-sample-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
ddf43b1f3751        6 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   127.2 MB
d43abb81204a        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   66.46 MB
7a21073861a1        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   115.8 MB
6801d964fda5        5 months ago
```

手順の順序は、次のセクションで説明するように、イメージキャッシュを操作するときに重要です。

### <a name="ordering-instructions"></a>注文手順

Dockerfile は上から下に処理され、各命令はキャッシュされているレイヤーと比較されます。 キャッシュされたレイヤーがない命令が見つかると、その命令およびそれ以降のすべての命令が新しいコンテナー イメージ レイヤーで処理されます。 このため、命令の配置の順序が重要になります。 変化しない命令は、Dockerfile の前の方に配置します。 変化する可能性のある命令は、Dockerfile の後の方に配置します。 そうすることで、既存のキャッシュが役に立たなくなる可能性が減ります。

次の例は、Dockerfile 命令の順序がキャッシュの有効性に与える影響を示しています。 この簡単な例の Dockerfile には、4つの番号付きフォルダーがあります。  

```dockerfile
FROM windowsservercore

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-3
RUN mkdir test-4
```

生成された画像には、ベース OS イメージ用と各`RUN`命令の5つのレイヤーがあります。

```dockerfile
docker history doc-sample-1

IMAGE               CREATED              CREATED BY               SIZE                COMMENT
afba1a3def0a        38 seconds ago       cmd /S /C mkdir test-4   42.46 MB
86f1fe772d5c        49 seconds ago       cmd /S /C mkdir test-3   42.35 MB
68fda53ce682        About a minute ago   cmd /S /C mkdir test-2   6.745 MB
5e5aa8ba1bc2        About a minute ago   cmd /S /C mkdir test-1   7.12 MB
6801d964fda5        5 months ago                                  0 B
```

この次の Dockerfile が少し変更され、3番`RUN`目の命令が新しいファイルに変更されました。 この Dockerfile に対して Docker ビルドを実行すると、前の例と同じ最初の 3 つの命令は、キャッシュされたイメージ レイヤーを使用します。 ただし、変更さ`RUN`れた命令はキャッシュされないため、変更された命令と後続のすべての命令に対して新しいレイヤーが作成されます。

```dockerfile
FROM windowsservercore

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-5
RUN mkdir test-4
```

このセクションの最初の例で、新しい画像のイメージ Id を比較すると、下から上の3つのレイヤーは共有されますが、4番目と5番目のレイヤーは一意であることがわかります。

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

### <a name="instruction-case"></a>指示ケース

Dockerfile 命令では大文字と小文字は区別されませんが、大文字と小文字は区別されます。 これにより、命令の呼び出しと命令の操作が区別され、読みやすさが向上します。 次の2つの例は、空になった、大文字の Dockerfile を比較したものです。

次に示すのは、空になった Dockerkerfile です。

```dockerfile
# Sample Dockerfile

from windowsservercore
run dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
run echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
cmd [ "cmd" ]
```

次に、大文字を使用した同じ Dockerfile を示します。

```dockerfile
# Sample Dockerfile

FROM windowsservercore
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

### <a name="line-wrapping"></a>行の折り返し

長整数型と複雑な操作は、バックスラッシュ`\`文字で複数の行に分けることができます。 次の Dockerfile は、Visual Studio の再頒布可能パッケージをインストールし、インストーラー ファイルを削除して、構成ファイルを作成します。 これら 3 つの操作すべてが 1 行で指定されています。

```dockerfile
FROM windowsservercore

RUN powershell -Command c:\vcredist_x86.exe /quiet ; Remove-Item c:\vcredist_x86.exe -Force ; New-Item c:\config.ini
```

コマンドはバックスラッシュで分割して、1つ`RUN`の命令からの各操作がそれぞれの行で指定されるようにすることができます。

```dockerfile
FROM windowsservercore

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait ; \
    Remove-Item c:\vcredist_x86.exe -Force ; \
    New-Item c:\config.ini
```

## <a name="further-reading-and-references"></a>その他の閲覧と参照

[Windows 上の Dockerfile](manage-windows-dockerfile.md)

[Best practices for writing Dockerfiles on Docker.com (Dockerfile 作成のベスト プラクティス (Docker.com))](https://docs.docker.com/engine/reference/builder/)
