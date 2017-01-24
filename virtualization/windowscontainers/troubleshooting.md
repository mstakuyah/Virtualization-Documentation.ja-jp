---
title: "Windows コンテナーのトラブルシューティング"
description: "Windows コンテナーと Docker に関するトラブルシューティングのヒント、自動スクリプト、およびログ情報"
keywords: "docker, コンテナー, トラブルシューティング, ログ"
author: PatrickLang
ms.date: 12/19/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: ebd79cd3-5fdd-458d-8dc8-fc96408958b5
translationtype: Human Translation
ms.sourcegitcommit: c65353d0b6dff233819dcc8f4f92eb186bf3b8fc
ms.openlocfilehash: 9f28c35c6eaddd8bcf3883863b63251378f845a7

---

# トラブルシューティング

コンピューターのセットアップやコンテナーの実行で問題が発生したときのために、 一般的な問題を検出する PowerShell スクリプトを用意しました。 最初にこのスクリプトを実行して、結果を調べてみてください。

```PowerShell
Invoke-WebRequest https://aka.ms/Debug-ContainerHost.ps1 -UseBasicParsing | Invoke-Expression
```
このスクリプトで実行されるすべてのテストの一覧と一般的な解決策が、スクリプトの [Readme ファイル](https://github.com/Microsoft/Virtualization-Documentation/blob/live/windows-server-container-tools/Debug-ContainerHost/README.md)に記載されています。

問題の原因を特定できない場合は、スクリプトの出力を[コンテナー フォーラム](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers)で投稿してください。 Windows Insider 参加者や開発者も集まるこのコミュニティは、手助けを求めるには最適の場所です。


## ログを見つける
Windows コンテナーの管理には、さまざまなサービスが使用されています。 次の各セクションで、ログの場所をサービス別に示します。

### Docker エンジン
Docker エンジンは、ファイルではなく Windows 'アプリケーション' イベント ログに記録します。 これらのログは、Windows PowerShell を使用することで、簡単に読み取り、並べ替え、およびフィルター処理することができます。

たとえば、過去 5 分間の Docker エンジン ログを古い順に表示できます。

```
Get-EventLog -LogName Application -Source Docker -After (Get-Date).AddMinutes(-5) | Sort-Object Time 
```

また、別のツールで読み取り可能な CSV ファイル、またはスプレッドシートに簡単にパイプすることができます。

```
Get-EventLog -LogName Application -Source Docker -After (Get-Date).AddMinutes(-30)  | Sort-Object Time | Export-CSV ~/last30minutes.CSV
```

#### デバッグ ログを有効にする
Docker エンジンのデバッグ レベルのログ出力を有効にすることもできます。 これは、トラブルシューティングに必要な詳細情報が通常のログでは得られない場合に役立つ可能性があります。

管理者特権でのコマンド プロンプトを開いてから、`sc.exe qc docker` を実行して Docker サービスの現在のコマンド ラインを取得します。
例:
```none
C:\> sc.exe qc docker
[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: docker
        TYPE               : 10  WIN32_OWN_PROCESS
        START_TYPE         : 2   AUTO_START
        ERROR_CONTROL      : 1   NORMAL
        BINARY_PATH_NAME   : "C:\Program Files\Docker\dockerd.exe" --run-service
        LOAD_ORDER_GROUP   :
        TAG                : 0
        DISPLAY_NAME       : Docker Engine
        DEPENDENCIES       :
        SERVICE_START_NAME : LocalSystem
```

現在の `BINARY_PATH_NAME` を次のとおりに変更します。
- 末尾に -D を追加する
- " をそれぞれ \ でエスケープする
- コマンド全体を " で囲む

変更したら、`sc.exe config docker binpath= ` の後に変更後の文字列を付けて実行します。 たとえば、 
```none
sc.exe config docker binpath= "\"C:\Program Files\Docker\dockerd.exe\" --run-service -D"
```


次に、Docker サービスを再起動します。
```none
sc.exe stop docker
sc.exe start docker
```

このようにすると、大量のログがアプリケーション イベント ログに出力されるため、トラブルシューティングが完了したら `-D` オプションを削除することをお勧めします。 上記の同じ手順を、`-D` を付けずに実行してからサービスを再起動すると、デバッグ ログは出力されなくなります。


### ホスト コンテナー サービス
Docker エンジンは、Windows 固有のホスト コンテナー サービスに依存します。 このサービスには、次のような独立したログがあります。 
- Microsoft-Windows-Hyper-V-Compute-Admin
- Microsoft-Windows-Hyper-V-Compute-Operational

これらはイベント ビューアーに表示され、PowerShell を使用して照会することもできます。

たとえば、
```PowerShell
Get-WinEvent -LogName Microsoft-Windows-Hyper-V-Compute-Admin
Get-WinEvent -LogName Microsoft-Windows-Hyper-V-Compute-Operational 
```




<!--HONumber=Jan17_HO3-->


