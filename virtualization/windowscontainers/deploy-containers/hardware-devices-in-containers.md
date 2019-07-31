---
title: Windows のコンテナー内のデバイス
description: Windows のコンテナーにはどのようなデバイスがサポートされるか
keywords: docker、コンテナー、デバイス、ハードウェア
author: cwilhit
ms.openlocfilehash: ee9c5da5ef87dceb3374977670da2ea50ea87382
ms.sourcegitcommit: c4a3f88d1663dd19336bfd4ede0368cb18550ac7
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/31/2019
ms.locfileid: "9883165"
---
# <a name="devices-in-containers-on-windows"></a>Windows のコンテナー内のデバイス

既定では、Windows コンテナーには、Linux コンテナーのように、ホストデバイスへの最小限のアクセスが与えられます。 一部のワークロードは、ホストハードウェアデバイスにアクセスして通信するのに役立ちます。 このガイドでは、コンテナーでサポートされているデバイスと開始方法について説明します。

## <a name="requirements"></a>要件

この機能が機能するためには、環境が次の要件を満たしている必要があります。
- コンテナーホストでは、Windows Server 2019 または Windows 10 バージョン1809以降を実行している必要があります。
- コンテナーベースのイメージバージョンは、1809以降である必要があります。
- コンテナーは、プロセス分離モードで実行されている Windows コンテナーである必要があります。
- コンテナーホストは、Docker Engine 19.03 以降を実行している必要があります。

## <a name="run-a-container-with-a-device"></a>デバイスでコンテナーを実行する

デバイスでコンテナーを開始するには、次のコマンドを使用します。

```shell
docker run --isolation=process --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

は、 `{interface class guid}`次のセクションに記載されている適切な[デバイスインターフェイスクラスの GUID](https://docs.microsoft.com/windows-hardware/drivers/install/overview-of-device-interface-classes)に置き換える必要があります。

複数のデバイスでコンテナーを開始するには、次のコマンドを使用`--device`し、複数の引数を組み合わせて指定します。

```shell
docker run --isolation=process --device="class/{interface class GUID}" --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

Windows では、すべてのデバイスが実装するインターフェイスクラスの一覧を宣言します。 このコマンドを Docker に渡すことによって、要求されたクラスを実装することを示すすべてのデバイスがコンテナーに plumb されるようになります。

これは、ホストからデバイスに割り当てられ**ていない**ことを意味します。 代わりに、ホストがコンテナーと共有します。 同様に、クラスの GUID を指定しているため、その GUID を実装する_すべて_のデバイスがコンテナーと共有されます。

## <a name="what-devices-are-supported"></a>サポートされるデバイス

現在、次のデバイス (およびそのデバイスインターフェイスクラスの Guid) がサポートされています。
  
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
<td><center>DirectX GPU アクセラレータ</center></td>
<td><center>「 <a href="https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/gpu-acceleration">GPU アクセラレータ</a>に関するドキュメント」をご覧ください。</center></td>
</tr>
</tbody>
</table>

> [!TIP]
> ここに記載されているデバイスは、現在 Windows コンテナーでサポートされている_唯一_のデバイスです。 他のクラス Guid に合格しようとすると、コンテナーが開始できなくなります。

## <a name="hyper-v-isolated-windows-container-support"></a>Hyper-v-分離された Windows コンテナーのサポート

Hyper-v-分離された Windows コンテナーのワークロードに対するデバイスの割り当てとデバイスの共有は、現在サポートされていません。

## <a name="hyper-v-isolated-linux-container-support"></a>Hyper-v-分離された Linux コンテナーのサポート

Hyper-v でのワークロードのためのデバイスの割り当てとデバイス共有は、現在サポートされていません。