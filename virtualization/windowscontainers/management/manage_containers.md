---
author: neilpeterson
redirect_url: ../quick_start/manage_docker
---

# Windows Server コンテナー管理

<g id="1" ctype="x-strong">この記事は暫定的な内容であり、変更される可能性があります。</g>

コンテナーのライフ サイクルには、コンテナーの開始、停止、削除などの操作が含まれます。 これらの操作を実行する際に、コンテナー イメージの一覧を取得し、コンテナーのネットワークを管理して、コンテナー リソースを制限する必要がある場合もあります。 このドキュメントでは、Docker を使用した基本的なコンテナー管理タスクについて詳しく説明します。また、詳細な記事のリンクも紹介します。

## コンテナーを管理する

### コンテナーを作成する

Docker によってコンテナーを作成するには、<g id="2" ctype="x-code">docker run</g> を使用します。

```none
PS C:\> docker run -p 80:80 windowsservercoreiis
```

Docker <g id="2" ctype="x-code">run</g> コマンドの詳細については、<g id="4CapsExtId1" ctype="x-link"><g id="4CapsExtId2" ctype="x-linkText">Docker run リファレンス</g><g id="4CapsExtId3" ctype="x-title"></g></g>を参照してください。

### コンテナーを停止します。

Docker によってコンテナーを停止するには、<g id="2" ctype="x-code">docker stop</g> コマンドを使用します。

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

Docker によってコンテナーを削除するには、<g id="2" ctype="x-code">docker rm</g> コマンドを使用します。

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

Docker rm コマンドの詳細については、<g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">Docker rm リファレンス</g><g id="2CapsExtId3" ctype="x-title"></g></g>を参照してください。






<!--HONumber=Apr16_HO5-->


