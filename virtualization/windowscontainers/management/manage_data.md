---
title: "コンテナー データ ボリューム"
description: "Windows コンテナーでデータ ボリュームを作成して管理します。"
keywords: "Docker, コンテナー"
author: neilpeterson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: f5998534-917b-453c-b873-2953e58535b1
redirect_url: https://docs.docker.com/engine/tutorials/dockervolumes/
translationtype: Human Translation
ms.sourcegitcommit: 59626096d428072dec098c7817e2d6b39c10e9cf
ms.openlocfilehash: 60dc25d879b5c8de6515899d9de9e920a08b7e62

---

# コンテナー データ ボリューム

**この記事は暫定的な内容であり、変更される可能性があります。** 

コンテナーを作成するときに、場合によっては、新しいデータ ディレクトリを作成したり、既存のディレクトリをコンテナーに追加したりする必要があります。 このような場合は、データ ボリュームを追加して対応できます。 データ ボリュームは、コンテナーとコンテナー ホストの両方から参照できるので、双方でデータを共有できます。 また、データ ボリュームは、同じコンテナー ホスト上の複数のコンテナーで共有できます。 このドキュメントでは、データ ボリュームの作成、検査、および削除について詳しく説明します。

## データ ボリューム

### 新しいデータ ボリュームを作成する

`docker run` コマンドの `-v` パラメーターを使用して、新しいデータ ボリュームを作成します。 既定では、新しいデータ ボリュームはホストの 'c:\ProgramData\Docker\volumes' に保存されます。

この例では、'new-data-volume' というデータ ボリュームを作成します。 このデータ ボリュームには、実行されているコンテナーの 'c:\new-data-volume' でアクセスできます。

```none
docker run -it -v c:\new-data-volume windowsservercore cmd
```

ボリュームの作成の詳細については、[docker.com のコンテナーのデータ管理に関するページ](https://docs.docker.com/engine/userguide/containers/dockervolumes/#data-volumes)を参照してください。

### 既存のディレクトリをマウントする

新しいデータ ボリュームを作成するだけでなく、ホストの既存のディレクトリをコンテナーに渡したい場合があります。 この処理は、`docker run` コマンドの `-v` パラメーターで実行することもできます。 ホスト ディレクトリ内のすべてのファイルは、コンテナーでも使用できます。 マウントされたボリュームにコンテナーが作成したファイルは、ホストで使用できます。 1 つのディレクトリを多数のコンテナーにマウントすることができます。 この構成では、コンテナー間でデータを共有できます。

この例では、'c:\source' という同じディレクトリが、'c:\destination' としてコンテナーにマウントされています。

```none
docker run -it -v c:\source:c:\destination windowsservercore cmd
```

ホスト ディレクトリのマウントの詳細については、[docker.com のコンテナーのデータ管理に関するページ](https://docs.docker.com/engine/userguide/containers/dockervolumes/#mount-a-host-directory-as-a-data-volume)を参照してください。

### 単一ファイルをマウントする

単一ファイルを Windows コンテナーにマウントすることはできません。 次のコマンドの実行は失敗しませんが、結果のコンテナーにファイルが含まれていません。 

```none
docker run -it -v c:\config\config.ini microsoft/windowsservercore cmd
```

回避策として、コンテナーにマウントするすべてのファイルをディレクトリからマウントする必要があります。

```none
docker run -it -v c:\config:c:\config microsoft/windowsservercore cmd
```

### 完全なドライブをマウントする

次のようなコマンドを使用して、完全なドライブをマウントできます。

```none
docker run -it -v d:\:d: windowsservercore cmd
```

この時点で、ドライブ 2 を部分的にマウントすることはできません。 たとえば、次のことはできません。

```none
docker run -it -v d:\source:d:\destination windowsservercore cmd
```

### データ ボリューム コンテナー

データ ボリュームを他の実行中のコンテナーから継承するには、`docker run` コマンドの `--volumes-from` パラメーターを使用します。 この継承を使用して、コンテナー化されたアプリケーション用にデータ ボリュームをホストするという明確な目的でコンテナーを作成できます。 

この例では、’cocky_bell’ というコンテナーからデータ ボリュームを新しいコンテナーにマウントします。 新しいコンテナーを起動すると、このボリュームのデータは、コンテナー内で実行されるアプリケーションに使用できるようになります。  

```none
docker run -it --volumes-from cocky_bell windowsservercore cmd
```

データベース コンテナーの詳細については、[docker.com のコンテナーのデータ管理に関するページ](https://docs.docker.com/engine/userguide/containers/dockervolumes/#mount-a-host-file-as-a-data-volume)を参照してください。

### 共有データ ボリュームを検査する

マウントされているボリュームは、`docker inspect` コマンドを使用して表示できます。

```none
docker inspect backstabbing_kowalevski
```

このコマンドを実行すると、’Mounts’ というセクションなど、コンテナーに関する情報が返されます。たとえば、ソース ディレクトリ、対象ディレクトリなど、マウントされたボリュームに関するデータなどです。

```none
"Mounts": [
    {
        "Source": "c:\\container-share",
        "Destination": "c:\\data",
        "Mode": "",
        "RW": true,
        "Propagation": ""
}
```

ボリュームの検査の詳細については、[docker.com のコンテナーのデータ管理に関するページ](https://docs.docker.com/engine/userguide/containers/dockervolumes/#locating-a-volume)を参照してください。




<!--HONumber=Sep16_HO2-->


