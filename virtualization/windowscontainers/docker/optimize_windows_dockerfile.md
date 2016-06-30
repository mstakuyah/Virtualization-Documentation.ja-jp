---
title: "Windows Dockerfile を最適化する"
description: "Windows コンテナー用に Dockerfile を最適化します。"
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 05/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb2848ca-683e-4361-a750-0d1d14ec8031
translationtype: Human Translation
ms.sourcegitcommit: cc216f56acd5e547d05a48beea57450ba5fcb28b
ms.openlocfilehash: 4822ff2f0248b2d7752299ea55b08e3499e2e2f7

---
# Windows Dockerfile を最適化する

**この記事は暫定的な内容であり、変更される可能性があります。** 

Docker のビルド プロセスと結果の Docker イメージの両方を最適化するには、複数の方法を使用できます。 このドキュメントでは、Docker ビルド プロセスのしくみについて詳しく説明し、Windows コンテナーでの最適なイメージの作成に使用できるいくつかの方法を示します。

## Docker のビルド

### イメージ レイヤー

Docker のビルドの最適化を調べる前に、Docker のビルドがどのように動作するのかを理解しておくことが重要です。 Docker ビルド プロセスでは、Dockerfile が使用されて、アクション可能な各命令が 1 つずつ専用の一時的なコンテナーで実行されます。 その結果、アクション可能な各命令に対して新しいイメージ レイヤーが作成されます。 

次の Dockerfile を見てください。 この例では、`windowsservercore` ベース OS イメージが使用されて、IIS がインストールされ、簡単な Web サイトが作成されます。

```none
# Sample Dockerfile

FROM windowsservercore
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

この Dockerfile からは、コンテナー OS イメージ用と IIS および Web サイト用の 2 つのレイヤーで構成されるイメージが作成されるように思うかもしれません。しかしそうではありません。 新しいイメージは、相互に独立した複数のレイヤーで構築されます。 これを視覚化するには、新しいイメージに対して `docker history` コマンドを実行します。 イメージは 4 つのレイヤーで構成されます。1 つの基本レイヤーと 3 つの追加レイヤー (Dockerfile の命令ごとに 1 つ) です。

```none
docker history iis

IMAGE               CREATED              CREATED BY                                      SIZE                COMMENT
f4caf476e909        16 seconds ago       cmd /S /C REM (nop) CMD ["cmd"]                 41.84 kB
f0e017e5b088        21 seconds ago       cmd /S /C echo "Hello World - Dockerfile" > c   6.816 MB
88438e174b7c        About a minute ago   cmd /S /C dism /online /enable-feature /all /   162.7 MB
6801d964fda5        4 months ago                                                         0 B
```

各レイヤーは、Dockerfile の命令に対応します。 最下位のレイヤー (この例では `6801d964fda5`) は、ベース OS イメージを表します。 1 つ上のレイヤーでは、IIS のインストールを確認できます。 次のレイヤーには、新しい Web サイトなどが含まれます。

イメージ レイヤーが最小になり、ビルドのパフォーマンスが最高になり、読みやすさなどの表面的なことが最適になるように、Dockerfile を記述できます。 最終的に、同じイメージ ビルド タスクを実行するにはさまざまな方法があります。 Dockerfile の形式がビルド時間と結果のイメージに与える影響を理解すると、オートメーションのエクスペリエンスが向上します。 

## イメージのサイズを最適化する

Docker コンテナー イメージを構築するときは、イメージのサイズが重要な要因になる場合があります。 コンテナー イメージは、レジストリとホストの間を移動され、エクスポートおよびインポートされて、最終的にはスペースを消費します。 Docker ビルド プロセスの間にイメージのサイズを最小限に抑えるには、複数の方法を使用できます。 このセクションでは、Windows コンテナーに固有のいくつかの方法について説明します。 

Dockerfile のベスト プラクティスについては、「[Best practices for writing Dockerfiles on Docker.com]( https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/)」(Docker.com で Dockerfile を作成するときのベスト プラクティス) を参照してください。

### 関連するアクションをグループ化する

各 `RUN` 命令によってコンテナー イメージに 1 つの新しいレイヤーが作成されるので、複数のアクションを 1 つの `RUN` 命令にグループ化するとレイヤーの数を減らすことができます。 レイヤーの数を最小限に抑えてもイメージのサイズに大きな影響はありませんが、後の例で示すように、関連するアクションをグループ化するとイメージのサイズに大きな影響があります。

次に示す 2 つの例の操作は、同じ機能を持つコンテナー イメージを作成しますが、2 つの Dockerfile の構成は異なります。 結果のイメージも比較します。  

1 番目の例は、Visual Studio の再頒布可能パッケージをダウンロードし、抽出して、クリーンアップします。 これらの各アクションは、それぞれの `RUN` 命令で実行されます。

```none
FROM windowsservercore

RUN powershell.exe -Command Invoke-WebRequest "https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe" -OutFile c:\python-3.5.1.exe
RUN powershell.exe -Command Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait
RUN powershell.exe -Command Remove-Item c:\python-3.5.1.exe -Force
```

結果として作成されるイメージは、`RUN` 命令ごとに 1 つずつ、3 つの追加レイヤーで構成されます。

```none
docker history doc-example-1

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
a395ca26777f        15 seconds ago      cmd /S /C powershell.exe -Command Remove-Item   24.56 MB
6c137f466d28        28 seconds ago      cmd /S /C powershell.exe -Command Start-Proce   178.6 MB
957147160e8d        3 minutes ago       cmd /S /C powershell.exe -Command Invoke-WebR   125.7 MB
```

2 番目の例は、同じ操作ですが、すべてのステップを同じ `RUN` 命令で実行します。 `RUN` 命令の各ステップは Dockerfile では別の行になっており、改行には '\' 文字が使用されています。 

```none
FROM windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

結果のイメージは、`RUN` 命令に対する 1 つの追加レイヤーで構成されます。

```none
docker history doc-example-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
69e44f37c748        54 seconds ago      cmd /S /C powershell.exe -Command   $ErrorAct   216.3 MB                
```

### 余分なファイルを削除する

インストーラーなどのファイルが使用後に必要ない場合は、ファイルを削除するとイメージのサイズが減ります。 この処理は、イメージ レイヤーへのファイルのコピーが行われるのと同じステップで行う必要があります。 そうすることで、ファイルが下位レベルのイメージ レイヤーに残るのを防ぐことができます。

この例では、Python パッケージをダウンロードし、実行して、実行可能ファイルを削除しています。 このすべてを 1 つの `RUN` 操作で行うので、作成されるイメージ レイヤーは 1 つです。

```none
FROM windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

## ビルドの速さを最適化する

### 複数の行

Docker のビルド速度を最適化するときは、操作を複数の個別の命令に分けると有効な場合があります。 複数の `RUN` 操作を使用すると、キャッシュの効率が向上します。 `RUN` 命令ごとに個別のレイヤーが作成されるため、同じステップが異なる Docker ビルド操作で既に実行されている場合は、このキャッシュされた操作 (イメージ レイヤー) が再利用されます。 結果として、Docker のビルドの実行時間が短縮されます。

次の例では、Apache と Visual Studio の再配布パッケージをダウンロードし、インストールした後、不要なファイルを削除しています。 このすべてを 1 つの `RUN` 命令で行います。 いずれかのアクションが更新された場合、すべてのアクションが再実行されます。

```none
FROM windowsservercore

RUN powershell -Command \
    
  # Download software ; \
    
  wget https://www.apachelounge.com/download/VC11/binaries/httpd-2.4.18-win32-VC11.zip -OutFile c:\apache.zip ; \
  wget "https://download.microsoft.com/download/1/6/B/16B06F60-3B20-4FF2-B699-5E9B7962F9AE/VSU_4/vcredist_x86.exe" -OutFile c:\vcredist.exe ; \
  wget -Uri http://windows.php.net/downloads/releases/php-5.5.33-Win32-VC11-x86.zip -OutFile c:\php.zip ; \
    
  # Install Software ; \
    
  Expand-Archive -Path c:\php.zip -DestinationPath c:\php ; \
  Expand-Archive -Path c:\apache.zip -DestinationPath c:\ ; \
  start-Process c:\vcredistexe -ArgumentList '/quiet' -Wait ; \
    
  # Remove unneeded files ; \
     
  Remove-Item c:\apache.zip -Force; \
  Remove-Item c:\vcredist.exe -Force
```

結果のイメージは 2 つのレイヤーで構成されます。1 つは基本 OS イメージのもので、もう 1 つは 1 つの `RUN` 命令のすべての操作を含みます。

```none
docker history doc-sample-1

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
9bdf3a21fd41        8 minutes ago       cmd /S /C powershell -Command     Invoke-WebR   205.8 MB
6801d964fda5        5 months ago                                                        0 B
```

比較するため、次の例では同じアクションを 3 つの `RUN` 命令に分けてあります。 この場合、各 `RUN` 命令はコンテナー イメージ レイヤーにキャッシュされ、以降の Dockerfile のビルドでは、変更されたアクションだけを再実行する必要があります。

```none
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

結果のイメージは、基本 OS イメージ用に 1 つと、`RUN` 命令ごとに 1 つずつ、全部で 4 つのレイヤーで構成されます。 各 `RUN` 命令は専用のレイヤーで実行されるため、後でまたこの Dockerfile を実行すると、または同じ命令セットを別の Dockerfile で実行すると、キャッシュされたイメージ レイヤーが使用されて、ビルド時間が短くなります。 イメージ キャッシュを使用するときは命令の順序が重要です。詳細については次のセクションで説明します。

```none
docker history doc-sample-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
ddf43b1f3751        6 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   127.2 MB
d43abb81204a        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   66.46 MB
7a21073861a1        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   115.8 MB
6801d964fda5        5 months ago
```

### 命令の手順

Dockerfile は上から下に処理され、各命令はキャッシュされているレイヤーと比較されます。 キャッシュされたレイヤーがない命令が見つかると、その命令およびそれ以降のすべての命令が新しいコンテナー イメージ レイヤーで処理されます。 このため、命令の配置の順序が重要になります。 変化しない命令は、Dockerfile の前の方に配置します。 変化する可能性のある命令は、Dockerfile の後の方に配置します。 そうすることで、既存のキャッシュが役に立たなくなる可能性が減ります。

この例の目的は、Dockerfile の命令の順序がキャッシュの有効性に及ぼす影響を示すことです。 この簡単な Dockerfile では、4 つの番号付きフォルダーが作成されます。  

```none
FROM windowsservercore

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-3
RUN mkdir test-4
```
結果のイメージには、基本 OS イメージ用に 1 つと、`RUN` 命令ごとに 1 つずつ、全部で 5 つのレイヤーがあります。

```none
docker history doc-sample-1

IMAGE               CREATED              CREATED BY               SIZE                COMMENT
afba1a3def0a        38 seconds ago       cmd /S /C mkdir test-4   42.46 MB
86f1fe772d5c        49 seconds ago       cmd /S /C mkdir test-3   42.35 MB
68fda53ce682        About a minute ago   cmd /S /C mkdir test-2   6.745 MB
5e5aa8ba1bc2        About a minute ago   cmd /S /C mkdir test-1   7.12 MB
6801d964fda5        5 months ago                                  0 B    
```

docker ファイルは若干変更されています。 3 番目 `RUN` 命令が変更されていることに注意してください。 この Dockerfile に対して Docker ビルドを実行すると、前の例と同じ最初の 3 つの命令は、キャッシュされたイメージ レイヤーを使用します。 ただし、変更された `RUN` 命令はキャッシュされていないので、その命令および以降のすべての命令のために新しいレイヤーが作成されます。

```none
FROM windowsservercore

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-5
RUN mkdir test-4
```

新しいイメージのイメージ ID を前の例と比較すると、最初の 3 つのレイヤー (下から上に向かって) は共有されていますが、4 番目と 5 番目は固有であることがわかります。

```none
docker history doc-sample-2

IMAGE               CREATED             CREATED BY               SIZE                COMMENT
c92cc95632fb        28 seconds ago      cmd /S /C mkdir test-4   5.644 MB
2f05e6f5c523        37 seconds ago      cmd /S /C mkdir test-5   5.01 MB
68fda53ce682        3 minutes ago       cmd /S /C mkdir test-2   6.745 MB
5e5aa8ba1bc2        4 minutes ago       cmd /S /C mkdir test-1   7.12 MB
6801d964fda5        5 months ago                                 0 B
```

## 表面的な最適化

### 命令の場合

Dockerfile の命令は大文字/小文字を区別されませんが、規則では大文字を使うようになっています。 これにより、命令の呼び出しと命令の操作が区別されて、読みやすさが向上します。 次の 2 つの例ではこの概念を示します。 

小文字の場合:
```none
# Sample Dockerfile

from windowsservercore
run dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
run echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
cmd [ "cmd" ]
```
大文字の場合: 
```none
# Sample Dockerfile

FROM windowsservercore
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

### 行の折り返し

長くて複雑な操作は、バックスラッシュ `\` 記号を使って複数の行に分けることができます。 次の Dockerfile は、Visual Studio の再頒布可能パッケージをインストールし、インストーラー ファイルを削除して、構成ファイルを作成します。 これら 3 つの操作すべてが 1 行で指定されています。

```none
FROM windowsservercore

RUN powershell -Command c:\vcredist_x86.exe /quiet ; Remove-Item c:\vcredist_x86.exe -Force ; New-Item c:\config.ini
```
1 つの `RUN` 命令の各操作を個別の行で指定するようにコマンドを書き直したものを次に示します。 

```none
FROM windowsservercore

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait ; \
    Remove-Item c:\vcredist_x86.exe -Force ; \
    New-Item c:\config.ini
```

## 参考資料

[Dockerfile on Windows](./manage_windows_dockerfile.md) (Windows での Dockerfile)

[Dockerfile 作成のベスト プラクティス (Docker.com)](https://docs.docker.com/engine/reference/builder/)



<!--HONumber=Jun16_HO4-->


