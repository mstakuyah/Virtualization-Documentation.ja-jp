---
title: Windows 上のコンテナー内のデバイス
description: Windows 上のコンテナーに対してどのようなデバイスサポートが存在するか
keywords: docker、コンテナー、デバイス、ハードウェア
author: cwilhit
ms.openlocfilehash: 1ad63c158a42f116882c949b242274dde8d893fc
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910602"
---
# <a name="devices-in-containers-on-windows"></a>Windows 上のコンテナー内のデバイス

既定では、Windows コンテナーには、Linux コンテナーと同様に、ホストデバイスへの最小限のアクセス権が付与されます。 ホストハードウェアデバイスにアクセスして通信するために、有用なワークロードや、必要に応じて、特定のワークロードがあります。 このガイドでは、コンテナーでサポートされているデバイスと開始方法について説明します。

## <a name="requirements"></a>要件

この機能を機能させるには、環境が次の要件を満たしている必要があります。
- コンテナーホストでは、Windows Server 2019 または Windows 10、バージョン1809以降が実行されている必要があります。
- コンテナーの基本イメージのバージョンは1809以降である必要があります。
- コンテナーは、プロセス分離モードで実行されている Windows コンテナーである必要があります。
- コンテナーホストは、Docker エンジン19.03 以降を実行している必要があります。

## <a name="run-a-container-with-a-device"></a>デバイスを使用したコンテナーの実行

デバイスでコンテナーを開始するには、次のコマンドを使用します。

```shell
docker run --isolation=process --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

`{interface class guid}` を適切な[デバイスインターフェイスクラス GUID](https://docs.microsoft.com/windows-hardware/drivers/install/overview-of-device-interface-classes)に置き換える必要があります。これについては、以下のセクションを参照してください。

複数のデバイスでコンテナーを起動するには、次のコマンドと文字列を複数の `--device` 引数と共に使用します。

```shell
docker run --isolation=process --device="class/{interface class GUID}" --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

Windows では、すべてのデバイスが実装するインターフェイスクラスのリストを宣言します。 このコマンドを Docker に渡すことで、要求されたクラスの実装として識別されるすべてのデバイスがコンテナーに組み込まれるようになります。

これは、デバイスがホストから割り当てられ**ていない**ことを意味します。 代わりに、ホストはコンテナーと共有します。 同様に、クラス GUID を指定するため、その GUID を実装する_すべて_のデバイスがコンテナーと共有されます。

## <a name="what-devices-are-supported"></a>サポートされているデバイス

現在、次のデバイス (およびそのデバイスインターフェイスクラス Guid) がサポートされています。
  
<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:75%" cellpadding="5" cellspacing="5">
<thead>
<tr valign="top">
<th><center>デバイスの種類</center></th>
<th><center>インターフェイスクラス GUID</center></th>
</tr>
</thead>
<tbody>
<tr valign="top">
<td><center>GPIO</center></td>
<td><center>916EF1CB-8426-468D-A6F7-9AE8076881B3</center></td>
</tr>
<tr valign="top">
<td><center>I2C バス</center></td>
<td><center>A11EE3C6-8421-4202-A3E7-B91FF90188E4</center></td>
</tr>
<tr valign="top">
<td><center>COM ポート</center></td>
<td><center>86ECE1E0-8089-11D0-9CE4-08003E301F73</center></td>
</tr>
<tr valign="top">
<td><center>SPI バス</center></td>
<td><center>DCDE6AF9-6610-4285-828F-CAAF78C424CC</center></td>
</tr>
<tr valign="top">
<td><center>DirectX GPU アクセラレーション</center></td>
<td><center><a href="https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/gpu-acceleration">GPU アクセラレータ</a>に関するドキュメントを参照してください。</center></td>
</tr>
</tbody>
</table>

> [!IMPORTANT]
> デバイスのサポートはドライバーに依存します。 上の表で定義されていないクラス Guid を渡そうとすると、未定義の動作が発生する可能性があります。

## <a name="hyper-v-isolated-windows-container-support"></a>Hyper-v 分離 Windows コンテナーのサポート

Hyper-v 分離 Windows コンテナーにおけるワークロードのデバイス割り当てとデバイス共有は、現在サポートされていません。

## <a name="hyper-v-isolated-linux-container-support"></a>Hyper-v 分離された Linux コンテナーのサポート

Hyper-v 分離 Linux コンテナーにおけるワークロードのデバイス割り当てとデバイス共有は、現在サポートされていません。
