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
ms.openlocfilehash: d897560061fae23fda6f88ebdad6dd804da9a8f1
ms.sourcegitcommit: c48dcfe43f73b96e0ebd661164b6dd164c775bfa
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/06/2019
ms.locfileid: "9610342"
---
# <a name="optimize-windows-dockerfiles"></a>Windows Dockerfile を最適化する

Docker ビルド処理および Docker クリップアートの両方を最適化するさまざまな方法があります。 この記事では、Docker ビルド プロセスのしくみと Windows コンテナーの画像を最適な方法で作成する方法について説明します。

## <a name="image-layers-in-docker-build"></a>Docker で画像のレイヤーを作成します。

Docker ビルドを最適化するには、前に、Docker での動作を作成する方法を知る必要があります。 Docker ビルド プロセスでは、Dockerfile が使用されて、アクション可能な各命令が 1 つずつ専用の一時的なコンテナーで実行されます。 その結果、アクション可能な各命令に対して新しいイメージ レイヤーが作成されます。

たとえば、次のサンプル Dockerfile 関数を使用して、`windowsservercore`ベースの OS イメージは、IIS をインストールし、単純な web サイトを作成します。

```dockerfile
# Sample Dockerfile

FROM windowsservercore
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

この Dockerfile すると、コンテナー OS イメージ、および秒を含む IIS と web サイトのいずれかの 2 つのレイヤーを使用してイメージ表示されるはずです。 ただし、実際の画像が多数のレイヤー、各レイヤーの前に、1 つが異なります。

明確にするを実行してみよう、`docker history`サンプル行った Dockerfile イメージに対してコマンドします。

```dockerfile
docker history iis

IMAGE               CREATED              CREATED BY                                      SIZE                COMMENT
f4caf476e909        16 seconds ago       cmd /S /C REM (nop) CMD ["cmd"]                 41.84 kB
f0e017e5b088        21 seconds ago       cmd /S /C echo "Hello World - Dockerfile" > c   6.816 MB
88438e174b7c        About a minute ago   cmd /S /C dism /online /enable-feature /all /   162.7 MB
6801d964fda5        4 months ago                                                         0 B
```

出力へのご協力この画像に 4 つのレイヤーが含まれている: 基本レイヤーと、Dockerfile でそれぞれの命令に割り当てられているレイヤーを 3 つ追加します。 最下位のレイヤー (この例では `6801d964fda5`) は、ベース OS イメージを表します。 1 つのレイヤーは、IIS のインストールします。 次のレイヤーには、新しい Web サイトなどが含まれます。

画像レイヤーを最小化、ビルドのパフォーマンスを最適化およびユーザー補助機能を通じて読みやすさを最適化する Dockerfiles を記述できます。 最終的に、同じイメージ ビルド タスクを実行するにはさまざまな方法があります。 Dockerfile の書式設定ビルド時に影響して、イメージを作成、オートメーション エクスペリエンスを向上する方法について説明します。

## <a name="optimize-image-size"></a>画像のサイズを最適化します。

領域の要件に応じて画像サイズできます重要 Docker コンテナーの画像を作成するとき。 コンテナー イメージは、レジストリとホストの間を移動され、エクスポートおよびインポートされて、最終的にはスペースを消費します。 このセクションでは、Windows のコンテナーの Docker ビルド処理中にイメージのサイズを小さくするために方法を説明します。

Dockerfile のベスト プラクティスについての詳細については、 [[Docker.com Dockerfiles を作成するためのベスト プラクティス](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/)を参照してください。

### <a name="group-related-actions"></a>関連するアクションをグループ化する

各`RUN`命令にコンテナーの画像をグループ化するのには、いずれかの操作で新しいレイヤーを作成する`RUN`命令が、Dockerfile 内のレイヤーの数を減らすことができます。 レイヤーの数を最小限に抑えてもイメージのサイズに大きな影響はありませんが、後の例で示すように、関連するアクションをグループ化するとイメージのサイズに大きな影響があります。

このセクションで次の 2 つの例、同じ操作を行う Dockerfiles と比較してみましょう。 ただし、1 つの Dockerfile には、関連するアクションをグループ化が他の操作ごとに 1 つの命令します。

次のグループ化されていない例 Dockerfile Python for Windows をダウンロードするには、それには、インストールし、インストールが終了したら、セットアップをダウンロードしたファイルを削除します。 この Dockerfile では、[アクションの指定独自`RUN`命令します。

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

2 番目の例では、まったく同じ操作を実行する Dockerfile です。 ただし、関連するすべてのアクションは、単一のグループ化されている`RUN`命令します。 各ステップが、 `RUN` 、Dockerfile の新しい行には、命令中に、'。 ' 文字を使用して、折り返さします。

```dockerfile
FROM windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

結果のイメージがの 1 つだけの強化、`RUN`命令します。

```dockerfile
docker history doc-example-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
69e44f37c748        54 seconds ago      cmd /S /C powershell.exe -Command   $ErrorAct   216.3 MB
```

### <a name="remove-excess-files"></a>余分なファイルを削除する

あるかどうか、その必要はありませんが、インストーラーなど、Dockerfile でファイルが使用されている、画像のサイズを小さくために削除することができます。 この処理は、イメージ レイヤーへのファイルのコピーが行われるのと同じステップで行う必要があります。 これを行うと、ファイルを下位レベルの画像のレイヤーに保持できなくなります。

Dockerfile 次の例では Python パッケージがダウンロード、実行すると、[削除します。 このすべてを 1 つの `RUN` 操作で行うので、作成されるイメージ レイヤーは 1 つです。

```dockerfile
FROM windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

## <a name="optimize-build-speed"></a>ビルド速度を最適化します。

### <a name="multiple-lines"></a>複数の行

操作 Docker ビルド速度を最適化するために複数の個々 の命令に分割できます。 複数`RUN`ごとに個別のレイヤーを作成するために、操作がキャッシュの有効性を向上`RUN`命令します。 同じ命令が既に Docker ビルドのさまざまな操作で、実行する場合は、このキャッシュされた操作 (画像 layer) が再利用が減ったり Docker ビルドの実行時のようになります。

次の例では、Apache と Visual Studio を再配布パッケージの両方がダウンロード、インストールされ、不要なファイルを削除して、[クリーンアップします。 これは、すべて単一`RUN`命令します。 これらの操作のいずれかの更新された場合、すべての操作を再実行されます。

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

結果のイメージがあり、2 つのレイヤー、基本 OS イメージのいずれか 1 つからすべての操作が含まれている`RUN`命令します。

```dockerfile
docker history doc-sample-1

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
9bdf3a21fd41        8 minutes ago       cmd /S /C powershell -Command     Invoke-WebR   205.8 MB
6801d964fda5        5 months ago                                                        0 B
```

3 つに同じアクション分割をここでは、`RUN`指示します。 この例では、各`RUN`コンテナー画像レイヤーの命令がキャッシュされ、以降の Dockerfile で再実行する必要が変更されたものだけを作成します。

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

4 つのレイヤーで構成される結果のイメージ基本 OS イメージと、3 つの 1 つのレイヤー`RUN`指示します。 各`RUN`命令の実行専用のレイヤーに、この Dockerfile のすべての後続の実行または」の手順には、さまざまな Dockerfile のと同じ設定はビルド時間を短縮する、キャッシュされた画像のレイヤーを使用します。

```dockerfile
docker history doc-sample-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
ddf43b1f3751        6 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   127.2 MB
d43abb81204a        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   66.46 MB
7a21073861a1        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   115.8 MB
6801d964fda5        5 months ago
```

手順の順序をどのようにするとはイメージのキャッシュを使用する場合に、次のセクションに表示されるように重要なできます。

### <a name="ordering-instructions"></a>手順の順序

Dockerfile は上から下に処理され、各命令はキャッシュされているレイヤーと比較されます。 キャッシュされたレイヤーがない命令が見つかると、その命令およびそれ以降のすべての命令が新しいコンテナー イメージ レイヤーで処理されます。 このため、命令の配置の順序が重要になります。 変化しない命令は、Dockerfile の前の方に配置します。 変化する可能性のある命令は、Dockerfile の後の方に配置します。 そうすることで、既存のキャッシュが役に立たなくなる可能性が減ります。

次の例では、Dockerfile 命令の順序に与える影響についてキャッシュの有効性を示します。 このシンプルな例 Dockerfile には、4 つの段落番号付きのフォルダーがあります。  

```dockerfile
FROM windowsservercore

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-3
RUN mkdir test-4
```

結果のイメージは、基本 OS イメージのいずれか、およびそれぞれの 5 つのレイヤーの`RUN`指示します。

```dockerfile
docker history doc-sample-1

IMAGE               CREATED              CREATED BY               SIZE                COMMENT
afba1a3def0a        38 seconds ago       cmd /S /C mkdir test-4   42.46 MB
86f1fe772d5c        49 seconds ago       cmd /S /C mkdir test-3   42.35 MB
68fda53ce682        About a minute ago   cmd /S /C mkdir test-2   6.745 MB
5e5aa8ba1bc2        About a minute ago   cmd /S /C mkdir test-1   7.12 MB
6801d964fda5        5 months ago                                  0 B
```

この次 Dockerfile 今すぐややが変更されている、3 番目に`RUN`命令が新しいファイルに変更します。 この Dockerfile に対して Docker ビルドを実行すると、前の例と同じ最初の 3 つの命令は、キャッシュされたイメージ レイヤーを使用します。 ただし、ため、変更された`RUN`命令がキャッシュされていない、変更命令およびそれ以降のすべての手順の新しいレイヤーが作成されます。

```dockerfile
FROM windowsservercore

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-5
RUN mkdir test-4
```

このセクションの最初の例に新しい画像の画像の Id を比較すると、下から上の最初の 3 つのレイヤーを共有するが固有では、第 4 と 5 のことがわかります。

```dockerfile
docker history doc-sample-2

IMAGE               CREATED             CREATED BY               SIZE                COMMENT
c92cc95632fb        28 seconds ago      cmd /S /C mkdir test-4   5.644 MB
2f05e6f5c523        37 seconds ago      cmd /S /C mkdir test-5   5.01 MB
68fda53ce682        3 minutes ago       cmd /S /C mkdir test-2   6.745 MB
5e5aa8ba1bc2        4 minutes ago       cmd /S /C mkdir test-1   7.12 MB
6801d964fda5        5 months ago                                 0 B
```

## <a name="cosmetic-optimization"></a>外観の最適化

### <a name="instruction-case"></a>命令ケース

Dockerfile 手順については、大文字小文字を区別されませんが、規則は、大文字に変換を使用します。 これにより、命令 call 関数と命令の操作を区別して読みやすさが向上します。 次の 2 つの例は、uncapitalized と大文字 Dockerfile を比較します。

Uncapitalized、Dockerfile は、次のようにします。

```dockerfile
# Sample Dockerfile

from windowsservercore
run dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
run echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
cmd [ "cmd" ]
```

大文字を使用して同じ Dockerfile は、次のようにします。

```dockerfile
# Sample Dockerfile

FROM windowsservercore
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

### <a name="line-wrapping"></a>行の折り返し

長い形式および複雑な操作は複数の行に円記号で区切られていることができます`\`文字。 次の Dockerfile は、Visual Studio の再頒布可能パッケージをインストールし、インストーラー ファイルを削除して、構成ファイルを作成します。 これら 3 つの操作すべてが 1 行で指定されています。

```dockerfile
FROM windowsservercore

RUN powershell -Command c:\vcredist_x86.exe /quiet ; Remove-Item c:\vcredist_x86.exe -Force ; New-Item c:\config.ini
```

コマンド分けることができますスラッシュをようにする各いずれかからの操作`RUN`命令を独立した行に指定します。

```dockerfile
FROM windowsservercore

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait ; \
    Remove-Item c:\vcredist_x86.exe -Force ; \
    New-Item c:\config.ini
```

## <a name="further-reading-and-references"></a>さらに閲覧と参照

[Windows 上の Dockerfile](manage-windows-dockerfile.md)

[Best practices for writing Dockerfiles on Docker.com (Dockerfile 作成のベスト プラクティス (Docker.com))](https://docs.docker.com/engine/reference/builder/)
