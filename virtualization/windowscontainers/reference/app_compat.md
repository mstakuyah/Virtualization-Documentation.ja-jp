---
title: Windows のコンテナーでのアプリケーションの互換性
description: Windows のコンテナーでのアプリケーションの互換性。
keywords: docker, containers
author: scooley
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 3e524458-bd03-400e-913f-210335add8dc
---

# Windows のコンテナーでのアプリケーションの互換性

**この記事は暫定的な内容であり、変更される可能性があります。** 

これはプレビューです。  Windows で実行されるアプリケーションは、最終的にはコンテナーでも実行できるようになるはずですが、これは現時点でのアプリケーションの互換性の状態を確認するためのものです。

このドキュメントは、私たちの経験を共有することのみを目的としています。

何かこの一覧にでしょうか。  ご使用の環境での失敗や成功があれば、[フォーラム](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers)を通じてお知らせください。

## Windows Server のコンテナー

Windows Server コンテナー内では、次のアプリケーションの実行を試みました。  これらの結果は、アプリケーションが正しく機能することを保証するものではありません。

| **名前** | **バージョン** | **Windows Server Core ベースのイメージ** | **Nano Server ベースのイメージ** | **備考** |
|:-----|:-----|:-----|:-----|:-----|
| .NET | 3.5 | はい | Unknown |  | 
| .NET | 4.6 | はい | Unknown |  | 
| .NET CLR | 5 beta 6 | はい | はい| X64 および x86 の両方、 | 
| アクティブな Python | 3.4.1 | はい | はい | |
| Apache Cassandra || はい | Unkown | 
| Apache CouchDB | 1.6.1 | いいえ | いいえ | |
| Apache Hadoop | | はい | いいえ | |
| Apache HTTPD | 2.4 | はい | はい | dedup フィルターがロードされている場合。VC++ ランタイムはインストールされません。 次を使用して dedup をアンロードします。 `fltmc unload dedup` |
| Apache Tomcat | 8.0.24 x64 | はい | Unknown | |
| ASP.NET | 4.6 | はい | Unkown | |
| ASP.NET | 5 beta 6 | はい | はい | X64 と x86 の両方 |
| Django | |はい|はい| |
| Go | 1.4.2 | はい | はい | |
| インターネット インフォメーション サービス | 10.0 | はい | はい | HTTPS/TLS が機能しません。  dedup フィルターがロードされている場合。VC++ ランタイムはインストールされません。 次を使用して dedup をアンロードします。 `fltmc unload dedup` |
| Java | 1.8.0_51 | はい | はい | サーバーのバージョンを使用します。 クライアントのバージョンが正しくインストールされません。 |
| MongoDB | 3.0.4 | はい | Unkown | |
| MySQL | 5.6.26 | はい | はい | |
| NGinx | 1.9.3 | はい | はい | |
| Node.js | 0.12.6 | 部分的 | 部分的 | NPM は、パッケージのダウンロードに失敗します。 |
| Perl | | はい | Unkown | |
| PHP | 5.6.11 | はい | はい | dedup フィルターがロードされている場合。VC++ ランタイムはインストールされません。 次を使用して dedup をアンロードします。 `fltmc unload dedup` |
| PostgreSQL | 9.4.4 | はい | Unknown | dedup フィルターがロードされている場合。VC++ ランタイムはインストールされません。 次を使用して dedup をアンロードします。 `fltmc unload dedup` |
| Python | 3.4.3 | はい | はい | |
| R | 3.2.1 | いいえ | いいえ | |
| RabbitMQ | 3.5.x | はい | Unknown | |
| Redis | 2.8.2101 | はい | はい | |
| Ruby | 2.2.2 | はい | はい | X64 および x86 の両方、 | 
| Ruby on Rails | 4.2.3 | はい | はい | |
| SQLite | 3.8.11.1 | はい | いいえ | |
| SQL Server Express | 2014 | はい | 不明 | SQL Express 2014 をインストールする、[コミュニティから提供された Dockerfile](https://github.com/brogersyh/Dockerfiles-for-windows/tree/master/sqlexpress) を作成することによって、すばやく開始できます。 |
| Sysinternals ツール | * | ○ | はい | のみ、GUI を必要としないものをしようとしました。 PsExec は、現在の設計では機能しません | 

## Hyper-V コンテナー

Hyper-V コンテナー内では、次のアプリケーションの実行を試みました。  これらの結果は、アプリケーションが正しく動作することを保証するものではありません。

| **名前** | **バージョン** | **Nano Server ベースのイメージ** | **備考** |
|:-----|:-----|:-----|:-----|
| Apache Hadoop | | いいえ | |
| Apache HTTPD | 2.4 | はい | dedup フィルターがロードされている場合。VC++ ランタイムはインストールされません。 次を使用して dedup をアンロードします。 `fltmc unload dedup` |
| ASP.NET | 5 beta 6 | はい | X64 と x86 の両方 |
| Django |  | はい | イメージが DockerFile を使用して作成され、python バイナリがその一部としてコピーされている場合、Python は動作しません。 コンテナーを起動し、python バイナリをコピーします。 |
| Go | 1.4.2 | はい | |
| インターネット インフォメーション サービス | 10.0 | はい | HTTPS/TLS が機能しません。  IIS は、dism を使用して直接インストールされません。  dism コマンドを使用して、IIS の無人インストールを実行します。 |
| Java | 1.8.0_51 | はい | サーバーのバージョンを使用します。 クライアントのバージョンが正しくインストールされません。 |
| MySQL | 5.6.26 | はい | |
| NGinx | 1.9.3 | はい | |
| Node.js | 0.12.6 | 部分的 | NPM は、パッケージのダウンロードに失敗します。 |
| Python | 3.4.3 | はい |  イメージが DockerFile を使用して作成され、python バイナリがその一部としてコピーされている場合、Python は動作しません。 コンテナーを起動し、python バイナリをコピーします。 |
| Redis | 2.8.2101 | はい | |
| Ruby | 2.2.2 | はい | X64 および x86 の両方、 | 
| Ruby on Rails | 4.2.3 | はい | |
| Sysinternals ツール | | はい | GUI を必要としないもののみ試しました。 PsExec は、現在の設計では機能しません。 |

## 経験談をお知らせください
何かこの一覧にでしょうか。  ご使用の環境での失敗や成功があれば、[フォーラム](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers)を通じてお知らせください。


<!--HONumber=May16_HO3-->


