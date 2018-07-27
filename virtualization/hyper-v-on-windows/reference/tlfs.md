---
title: ハイパーバイザーの仕様
description: ハイパーバイザーの仕様
keywords: windows 10、hyper-v
author: allenma
ms.date: 06/26/2018
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: aee64ad0-752f-4075-a115-2d6b983b4f49
ms.openlocfilehash: 26eaca5a583f8b2e11ca1e05e2fa9a4366fd8837
ms.sourcegitcommit: 594cc1728347646609ae1952ecc6c97fc659d0a9
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/07/2018
ms.locfileid: "2226208"
---
# <a name="hypervisor-specifications"></a>ハイパーバイザーの仕様

## <a name="hypervisor-top-level-functional-specification"></a>ハイパーバイザーのトップレベル機能仕様

Hyper-V ハイパーバイザー トップレベル機能仕様 (TLFS) では、他のオペレーティング システム コンポーネントに対する、外部から参照可能なハイパーバイザーの動作を説明します。 この仕様は、ゲスト オペレーティング システムの開発者にとって役立ちます。
  
> この仕様は、Microsoft Open Specification Promise の下で提供されます。  [Microsoft Open Specification Promise](https://msdn.microsoft.com/en-us/openspecifications) の詳細については、以下をご覧ください。  

#### <a name="download"></a>ダウンロード
リリース | ドキュメント
--- | ---
Windows Server 2016 (リビジョン C) | [Hypervisor Top Level Functional Specification v5.0c.pdf](https://github.com/MicrosoftDocs/Virtualization-Documentation/raw/live/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v5.0C.pdf)
Windows Server 2012 R2 (リビジョン B) | [Hypervisor Top Level Functional Specification v4.0b.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v4.0b.pdf)
Windows Server 2012 | [Hypervisor Top Level Functional Specification v3.0.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v3.0.pdf)
Windows Server 2008 R2 | [Hypervisor Top Level Functional Specification v2.0.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v2.0.pdf)

## <a name="requirements-for-implementing-the-microsoft-hypervisor-interface"></a>Microsoft ハイパーバイザー インターフェイスを実装するための要件

TLFS には、Microsoft 固有のハイパーバイザー アーキテクチャのあらゆる側面を網羅した説明が記載されています。このアーキテクチャはゲスト仮想マシンに対して "HV#1" として宣言されます。  ただし、サード パーティ製ハイパーバイザーでは、Microsoft HV#1 ハイパーバイザー仕様への準拠を宣言するために、TLFS に記載されているインターフェイスをすべて実装する必要はありません。 「Requirements for Implementing the Microsoft Hypervisor Interface」 (Microsoft ハイパーバイザー インターフェイスを実装するための要件) には、Microsoft HV#1 との互換性が求められるすべてのハイパーバイザーに最小限実装する必要があるハイパーバイザー インターフェイスが記載されています。

#### <a name="download"></a>ダウンロード

[Requirements for Implementing the Microsoft Hypervisor Interface.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Requirements%20for%20Implementing%20the%20Microsoft%20Hypervisor%20Interface.pdf)
