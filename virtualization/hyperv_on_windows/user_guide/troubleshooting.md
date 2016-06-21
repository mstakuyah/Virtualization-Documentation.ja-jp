---
title: &30871497 Windows 10 の Hyper-V のトラブルシューティング
description: Windows 10 の Hyper-V のトラブルシューティング
keywords: windows 10, hyper-v
author: scooley
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: &1950684382 windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: f0ec8eb4-ffc4-4bf1-9a19-7a8c3975b359
---

# Windows 10 の Hyper-V のトラブルシューティング

## Windows 10 に更新しましたが、下位レベル (Windows 8.1 または Server 2012 R2) のホストに接続できません

Windows 10 では、Hyper-V マネージャーがリモート管理の WinRM に移動されました。 つまり、Hyper-V マネージャーを使用して管理するには、リモート管理をリモート ホスト上で有効にする必要があります。

詳細については、「<g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">リモート Hyper-V ホストの管理</g><g id="2CapsExtId3" ctype="x-title"></g></g>」を参照してください。

## チェックポイントの種類を変更しましたが、誤った種類のチェックポイントを取得し続けています

VMConnect からチェックポイントを取得していて、Hyper-V マネージャーのチェックポイントの種類を変更した場合、いずれの種類であっても取得されるチェックポイントは、VMConnect を開いたときに指定したチェックポイントの種類になります。

VMConnect を閉じて、もう一度開き、正しいチェックポイントの種類を取得するようにします。

## フラッシュ ドライブ上に仮想ハード ディスクを作成しようとしましたが、エラー メッセージが表示されます

これらのファイル システムはアクセス制御リスト (ACL) を指定しないため、また、4 GB より大きいファイルをサポートしないため、Hyper-V では FAT または FAT32 でフォーマットされたディスク ドライブをサポートしません。 ExFAT でフォーマットされたディスクは、ACL 機能を制限付きで提供するため、セキュリティ上の理由により、サポートもされません。
PowerShell では、"システムで '\[path to VHD\]' を作成できませんでした: ファイル システム制限のため、要求された操作を完了できませんでした (0x80070299)。" のエラー メッセージが表示されます。

代わりに、NTFS フォーマットのドライブを使用します。

## インストールしようとすると、このメッセージが表示されます。"Hyper-V をインストールできません: プロセッサは、第 2 レベルのアドレス変換 (SLAT) をサポートしていません。"

Hyper-V では、仮想マシンを実行するために SLAT が必要です。 コンピューターが SLAT をサポートしていない場合、仮想マシンのホストにすることはできません。

管理ツールのインストールのみを行う場合は、<g id="4" ctype="x-strong">[プログラムと機能]</g> の <g id="6" ctype="x-strong">[Windows の機能の有効化または無効化]</g> で、<g id="2" ctype="x-strong">[Hyper-V プラットフォーム]</g> の選択を解除します。






<!--HONumber=May16_HO1-->


