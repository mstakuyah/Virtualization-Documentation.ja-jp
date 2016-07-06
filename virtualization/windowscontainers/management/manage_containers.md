---
author: neilpeterson
redirect_url: ../quick_start/manage_docker
translationtype: Human Translation
ms.sourcegitcommit: 2b85875eae1dcf1e50162e69c53dbf1ac7463450
ms.openlocfilehash: 8921cbd910bf657ddc4998e4214c1e9f9c3a01e9

---

# Windows Server コンテナー管理

**この記事は暫定的な内容であり、変更される可能性があります。** 

コンテナーのライフ サイクルには、コンテナーの開始、停止、削除などの操作が含まれます。 これらの操作を実行する際に、コンテナー イメージの一覧を取得し、コンテナーのネットワークを管理して、コンテナー リソースを制限する必要がある場合もあります。 このドキュメントでは、Docker を使用した基本的なコンテナー管理タスクについて詳しく説明します。また、詳細な記事のリンクも紹介します。 

## コンテナーを管理する

### コンテナーを作成する

Docker によってコンテナーを作成するには、`docker run` を使用します。

```none
PS C:\> docker run -p 80:80 windowsservercoreiis
```

Docker `run` コマンドの詳細については、「[Docker run reference]( https://docs.docker.com/engine/reference/run/)」をご覧ください。

### コンテナーを停止します。

Docker によってコンテナーを停止するには、`docker stop` コマンドを使用します。

```none
PS C:\> docker stop tender_panini

tender_panini
```

この例では、Docker によって実行中のすべてのコンテナーを停止します。

```none
PS C:\> docker stop $(docker ps -q)

fd9a978faac8
b51e4be8132e
```

### コンテナーを削除する

Docker によってコンテナーを削除するには、`docker rm` コマンドを使用します。

```none
PS C:\> docker rm prickly_pike

prickly_pike
``` 

Docker によってすべてのコンテナーを削除するには、次を実行します。

```none
PS C:\> docker rm $(docker ps -aq)

dc3e282c064d
2230b0433370
```

Docker rm コマンドの詳細については、[Docker rm リファレンス](https://docs.docker.com/engine/reference/commandline/rm/)をご覧ください。



<!--HONumber=Jun16_HO4-->


