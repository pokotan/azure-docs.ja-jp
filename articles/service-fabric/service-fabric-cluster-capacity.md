---
title: "Service Fabric クラスターの容量計画 | Microsoft Docs"
description: "Service Fabric クラスターの容量計画に関する考慮事項。 Nodetypes、持続性と信頼性レベル"
services: service-fabric
documentationcenter: .net
author: ChackDan
manager: timlt
editor: 
ms.assetid: 4c584f4a-cb1f-400c-b61f-1f797f11c982
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 07/24/2017
ms.author: chackdan
ms.translationtype: HT
ms.sourcegitcommit: 99523f27fe43f07081bd43f5d563e554bda4426f
ms.openlocfilehash: 270d79944465176d3df467f7145ff82594302c3d
ms.contentlocale: ja-jp
ms.lasthandoff: 08/05/2017

---
# <a name="service-fabric-cluster-capacity-planning-considerations"></a>Service Fabric クラスターの容量計画に関する考慮事項
容量計画は、運用環境へのデプロイにおいて重要なステップとなります。 ここでは、そのプロセスの一環として考慮すべき事柄をいくつか取り上げます。

* クラスターで最初に必要となるノード タイプの数
* ノード タイプごとの特性 (サイズ、プライマリ/非プライマリ、インターネット接続、VM 数など)
* クラスターの信頼性と耐久性の特徴

それぞれの項目について簡単に見ていきましょう。

## <a name="the-number-of-node-types-your-cluster-needs-to-start-out-with"></a>クラスターで最初に必要となるノード タイプの数
まず、これから作成するクラスターを何の目的に使用するのか、また、そのクラスターにどのようなアプリケーションをデプロイするのかを把握する必要があります。 クラスターの目的が明確でないとすれば、容量計画プロセスに着手するのはおそらく時期尚早です。

クラスターで最初に必要となるノード タイプの数をはっきりさせましょう。  ノード タイプはそれぞれ 1 つの仮想マシン スケール セットに対応付けられます。 各ノードの種類は、個別にスケール アップまたはスケール ダウンすることができ、さまざまなセットのポートを開き、異なる容量のメトリックスを持つことができます。 そのためノード タイプの数は、基本的に次の要因を考慮して決めることになります。

* アプリケーションに複数のサービスが存在するか、またその中に、外部に公開 (インターネットに接続) しなければならないサービスはあるか。 標準的なアプリケーションには、クライアントからの入力を受け取るフロントエンド ゲートウェイ サービスと、そのフロントエンド サービスと通信するバックエンド サービスが存在します。 したがってこのケースでは、最終的に少なくとも 2 つのノード タイプが必要となります。
* アプリケーションの構成要素として、インフラストラクチャ ニーズの異なる複数のサービスが存在するか (大容量の RAM が必要、高い CPU 処理能力が必要など)。 たとえばデプロイするアプリケーションに、フロントエンド サービスとバックエンド サービスが含まれていると仮定します。 フロントエンド サービスはバックエンド サービスよりも小さい VM (D2 など) で実行できますが、インターネットに対してポートを開放する必要があります。  一方バックエンド サービスは計算負荷が高く、より大きな VM (D4、D6、D15 など) で実行する必要はありますが、インターネット接続は不要です。
  
  この場合、1 つのノード タイプにすべてのサービスをデプロイすることもできますが、Microsoft としては、2 つのノード タイプから成るクラスターにデプロイすることをお勧めします。  そうすることでノード タイプごとに、インターネットの接続性や VM サイズなど異なる特性を持たせることができます。 VM 数を個別に増減させることもできます。  
* 将来を予測することはできないので、まずは把握している事実を踏まえ、アプリケーションの立ち上げ時に必要となるノード タイプの数を決めましょう。 ノード タイプは後からいつでも追加または削除できます。 Service Fabric クラスターには少なくとも 1 つのノード タイプが必要です。

## <a name="the-properties-of-each-node-type"></a>各ノード タイプの特性
**ノード タイプ** は、Cloud Services のロールと同等のものと見なすことができます。 ノードのタイプには、VM のサイズ、VM の数、プロパティが定義されています。 Service Fabric クラスターで定義されているすべてのノード タイプは、個別の仮想マシン スケール セットとしてセットアップされます。 仮想マシン スケール セットは、セットとして仮想マシンのコレクションをデプロイおよび管理するために使用できる Azure 計算リソースです。 ノード タイプはそれぞれ別々の仮想マシン スケール セットとして定義されているため、個別にスケールアップまたはスケールダウンすることができ、また異なるポートの組み合わせを開放し、異なる容量メトリックを割り当てることができます。

ノード タイプと仮想マシン スケール セットとの関係、いずれかのインスタンスに RDP で接続する方法、新しいポートを開く方法などの詳細については[こちらのドキュメント](service-fabric-cluster-nodetypes.md)を参照してください。

クラスターには複数のノード タイプを指定できますが、運用環境のワークロードに使用するクラスターの場合、プライマリ ノード タイプ、つまりポータルで最初に定義したノードには、少なくとも 5 つの VM が必要です (テスト環境のクラスターの場合は 3 つ以上)。 Resource Manager テンプレートを使用してクラスターを作成する場合は、ノード タイプの定義で、**is Primary** 属性を探します。 Service Fabric のシステム サービスは、プライマリ ノード タイプに配置されます。  

### <a name="primary-node-type"></a>プライマリ ノード タイプ
1 つのクラスターに複数のノード タイプが存在する場合、そのいずれかをプライマリにする必要があります。 プライマリ ノード タイプには、次の特徴があります。

* プライマリ ノード タイプの**最小 VM サイズ**は、選択した**耐久性レベル**によって決まります。 既定の耐久性レベルは Bronze です。 耐久性レベルとは何か、どのようなプランがあるかについては、このページの下の方で説明しています。  
* プライマリ ノード タイプの**最低 VM 数**は、選択した**信頼性レベル**によって決まります。 既定の信頼性レベルは Silver です。 信頼性レベルとは何か、どのようなプランがあるかについては、このページの下の方で説明しています。 

 

* Service Fabric システム サービス (クラスター マネージャー サービス、イメージ ストア サービスなど) はプライマリ ノード タイプに置かれるので、クラスターの信頼性と耐久性は、プライマリ ノード タイプに選択した信頼性レベルのプランと耐久性レベルのプランによって決まります。

![ノードが 2 種類あるクラスターのスクリーン ショット ][SystemServices]

### <a name="non-primary-node-type"></a>非プライマリ ノード タイプ
クラスターに複数のノード タイプが存在するとき、プライマリ ノード タイプは 1 つだけで、それ以外はすべて非プライマリ ノード タイプです。 非プライマリ ノード タイプには、次の特徴があります。

* このノード タイプの最小 VM サイズは、選択した耐久性レベルによって決まります。 既定の耐久性レベルは Bronze です。 耐久性レベルとは何か、どのようなプランがあるかについては、このページの下の方で説明しています。  
* このノード タイプの最低 VM 数は 1 です。 ただしこの数は、このノード タイプで実行するアプリケーション/サービスのレプリカ数に基づいて選ぶ必要があります。 ノード タイプの VM 数は、クラスターのデプロイ後に増やすことができます。

## <a name="the-durability-characteristics-of-the-cluster"></a>クラスターの耐久性の特徴
耐久性レベルは、ご利用の VM が、基になる Azure インフラストラクチャに対して持つ特権をシステムに表明する目的で使用します。 プライマリ ノード タイプでは、Service Fabric がこの特権を使って、システム サービスやステートフル サービスのクォーラム要件に影響を及ぼす、VM レベルのインフラストラクチャ要求 (VM の再起動、VM の再イメージ化、VM の移行など) を一時停止させることができます。 非プライマリ ノード タイプの場合も、Service Fabric がこの特権の下で、ノード内で実行されるステートフル サービスのクォーラム要件に影響を及ぼす、VM レベルのインフラストラクチャ要求 (VM の再起動、VM の再イメージ化、VM の移行など) を一時停止させることができます。

この特権は次のプランで表されます。

* Gold - 1 つの更新ドメインにつき 2 時間、インフラストラクチャ ジョブを一時停止させることができます。 Gold の持続性は、D15_V2 や G5 などのフル ノードの VM SKU でのみ有効にできます。
* Silver - 1 つの更新ドメインにつき 10 分、インフラストラクチャ ジョブを一時停止させることができ、単一コア以上のすべての Standard VM で使用できます。
* Bronze - 特権はありません。 これは既定値で、クラスターでステートレス ワークロードのみを実行する場合に推奨されます。

各ノード タイプの耐久性レベルを選択する必要があります。同じクラスター内で、耐久性レベルが Gold または Silver の 1 つのノード タイプを選択し、別のノード タイプの耐久性レベルを Bronze にすることもできます。**耐久性が Gold または silver であるすべてのノード タイプについては、少なくとも 5 つのノードを維持する必要があります**。 

**Silver または Gold 耐久性レベル使用の長所**
 
1. スケールイン操作に必要な手順数を減らすことができます (つまり、ノードの非アクティブ化と Remove-ServiceFabricNodeState が自動的に呼び出されます)。
2. お客様が開始したインプレース VM SKU 変更操作または Azure インフラストラクチャ操作によるデータ消失のリスクを減らすことができます。
     
**Silver または Gold 耐久性レベル使用の短所**
 
1. 仮想マシン スケール セットやその他の関連する Azure リソースへのデプロイが遅れたり、タイムアウトしたりする可能性があり、クラスターまたはインフラストラクチャ レベルの問題によって完全にブロックされる可能性があります。 
2. Azure インフラストラクチャの操作中に自動化されたノードの非アクティブ化により[レプリカ ライフサイクル イベント](service-fabric-reliable-services-advanced-usage.md#stateful-service-replica-lifecycle ) (プライマリ スワップなど) の数が増えます。



### <a name="recommendations-on-when-to-use-silver-or-gold-durability-levels"></a>どのような場合に Silver または Gold 耐久性レベルを使用するか

Silver または Gold 耐久性は、頻繁なスケールイン (VM インスタンス数の削減) が予測されるステートフル サービスをホストするすべてのノード タイプに適しており、またこれらのスケールイン操作を簡略化するためにデプロイ操作が遅れることを厭わない場合に適しています。 スケールアウト シナリオ (VM インスタンスの追加) は、耐久性レベルの選択では問題になりません。問題になるのは、スケールインのみです。

### <a name="operational-recommendations-for-the-node-type-that-you-have-set-to-silver-or-gold-durability-level"></a>Silver または Gold 耐久性レベルに設定したノード タイプの操作上の推奨事項

1. クラスターとアプリケーションを常に正常な状態に維持し、アプリケーションが適切なタイミングですべての[サービス レプリカのライフサイクル イベント](service-fabric-reliable-services-advanced-usage.md#stateful-service-replica-lifecycle) (ビルドのレプリカが停止するなど) に応答することを確認します。
2. VM の SKU を変更する場合はより安全な方法を採用します (スケールアップ/ダウン)。仮想マシン スケール セットの VM SKU の変更は本質的に安全でない操作であるため、可能な限り避けるようにしてください。 一般的な問題を避けるために実行できる手順は次のとおりです。
    - **非プライマリ nodetypes の場合:** 新しい仮想マシン スケール セットを作成し、サービス配置制約を新しい仮想マシン スケール セット/ノード タイプを含めるように変更したら、古い仮想マシン スケール セット インスタンスの数を、一度に 1 ノードずつ 0 に減らします (このようにするのは、ノードの削除がクラスターの信頼性に影響しないようにするためです)。
    - **プライマリ nodetype の場合**: プライマリ ノード タイプの VM SKU は変更しないことをお勧めします。 新しい SKU の理由が容量である場合は、インスタンスを追加するか、可能であれば、新しいクラスターを作成することをお勧めします。 選択肢がない場合は、新しい SKU が反映されるように仮想マシン スケール セットのモデル定義を変更します。 クラスターに nodetype が 1 つしかない場合は、すべてのステートフルなアプリケーションが、適切なタイミングですべての[サービス レプリカのライフ サイクル イベント](service-fabric-reliable-services-advanced-usage.md#stateful-service-replica-lifecycle) (ビルドでのレプリカの停止など) に応答すること、サービス レプリカのリビルド時間が 5 分未満であることを確認します (Silver 耐久性レベルの場合)。 
3. MR が有効な任意の仮想マシン スケール セットに最小数 5 つのノードを用意します。
4. ランダムな VM インスタンスは削除せず、常に仮想マシン スケール セットのスケールダウン機能を使用するようにします。 ランダムな VM インスタンスを削除すると、UD と FD に分散された VM インスタンスで不均衡が生じる可能性があります。 この不均衡は、サービス インスタンス/サービス レプリカ間で適切に負荷分散を行うシステムの機能に悪影響を及ぼす場合があります。
6. 自動スケールを使用した場合、スケールイン (VM インスタンスの削除) が一度に 1 ノードに実行されるようにルールを設定します。 一度に複数のインスタンスをスケールダウンすることは安全ではありません。
7. プライマリ ノード タイプをスケールダウンする場合は、信頼性レベルで許されるより多くスケールダウンしないでください。


## <a name="the-reliability-characteristics-of-the-cluster"></a>クラスターの信頼性の特徴
信頼性レベルは、クラスターのプライマリ ノード タイプで実行するシステム サービスのレプリカ数を設定する目的で使用します。 レプリカ数が増えるほど、クラスター内のシステム サービスの信頼性が上がります。  

信頼性レベルは、以下のプランから選ぶことができます。

* Platinum - ターゲット レプリカ セット数を 9 としてシステム サービスを実行します。
* Gold - ターゲット レプリカ セット数を 7 としてシステム サービスを実行します。
* Silver - ターゲット レプリカ セット数を 5 としてシステム サービスを実行します。 
* Bronze - ターゲット レプリカ セット数を 3 としてシステム サービスを実行します。

> [!NOTE]
> 選択した信頼性レベルによって、プライマリ ノード タイプに必要な最小ノード数が決まります。 
> 
> 


### <a name="recommendations-for-the-reliability-tier"></a>信頼性レベルに関する推奨事項。

 クラスターのサイズ (すべてのノード タイプの VM インスタンスの総数) を増減するときには、クラスターの信頼性を別のレベルに更新する必要があります。 クラスターの信頼性レベルを変更すると、システム サービスのレプリカ セット数を変更するために必要なクラスターのアップグレードが開始されます。 ノードの追加など、クラスターにさらに変更を行う場合は、このアップグレードが完了してからにしてください。  アップグレードの進行状況を監視するには、Service Fabric Explorer を使用するか、[Get-ServiceFabricClusterUpgrade](/powershell/module/servicefabric/get-servicefabricclusterupgrade?view=azureservicefabricps) を実行します。

以下に、信頼性レベルを選択するときの推奨事項を示します。

| **クラスター サイズ** | **信頼性レベル** |
| --- | --- |
| 1 |信頼性レベルのパラメーターを指定しないでください。システムが計算します。 |
| 3 |ブロンズ |
| 5 または 6|シルバー |
| 7 または 8 |ゴールド |
| 9 以上 |Platinum |




## <a name="primary-node-type---capacity-guidance"></a>プライマリ ノード タイプ - 容量に関するガイダンス

ここでは、プライマリ ノード タイプの容量計画のガイダンスを示します。

1. **Azure で運用ワークロードを実行する VM インスタンスの数:** 最小プライマリ ノード タイプ サイズとして 5 を指定する必要があります。 
2. **Azure でテスト ワークロードを実行する VM インスタンスの数:** 最小プライマリ ノード タイプ サイズとして 1 または 3 を指定できます。 1 つのノード クラスターは、特別な構成で実行されるため、そのクラスターからのスケールアウトがサポートされません。 1 つのノード クラスターは、Resource Manager テンプレートで信頼性がないため、その構成 (構成値の設定が不十分) を削除するか、指定しないようにする必要があります。 1 つのノード クラスターの設定をポータルを通じて行った場合、構成は自動的に行われます。 1 および 3 ノード クラスターは、運用ワークロードの実行ではサポートされません。 
3. **VM SKU:** プライマリ ノード タイプはシステム サービスの実行場所となるため、VM SKU を選択する場合、クラスターに配置する予定の全ピーク負荷を考慮に入れる必要があります。 この要点を例え話で説明するために、プライマリ ノード タイプを "肺" と考えてください。そこから脳に酸素が送られるため、脳に十分な酸素が届かなければ身体に悪影響が出るということです。 

クラスターに必要な容量は、クラスターで実行する予定のワークロードによって決まるため、特定のワークロードについて定性的なガイダンスを示すことはできませんが、以下に、検討を始めるのに役立つ包括的なガイダンスを示します。

運用環境のワークロードの場合は次のとおりです。 


- 推奨される VM SKU は、Standard D3 か Standard D3_V2、またはローカル SSD が 14 GB 以上のこれらと同等の SKU です
- サポートされる最小の VM SKU は、Standard D1 か Standard D1_V2、またはローカル SSD が 14 GB 以上のこれらと同等の SKU です 
- コア数が少ない Standard A0 のような VM SKU は、運用ワークロードではサポートされません。
- Standard A1 SKU は、パフォーマンス上の理由から運用ワークロードではサポートされません。


## <a name="non-primary-node-type---capacity-guidance-for-stateful-workloads"></a>非プライマリ ノード タイプ - ステートフル ワークロードの容量ガイダンス

このガイダンスは、非プライマリ ノード タイプで実行している Service Fabric の[信頼性の高いコレクションまたは Reliable Actors](service-fabric-choose-framework.md) を使用するステートフル ワークロードを対象としています。


**VM インスタンスの数:** ステートフルな運用ワークロードの場合、最小レプリカ数およびターゲット レプリカ数を 5 にして実行することをお勧めします。 こうすることで、安定状態では各障害ドメインとアップグレード ドメインに (1 つのレプリカ セットから) 1 つのレプリカが配置されることになります。 プライマリ ノード タイプの信頼性レベルという概念は、システム サービスに対してこの設定を指定する方法です。 そのため、ステートフル サービスにも同じ考慮事項が適用されます。

このため、運用ワークロードで推奨される非プライマリ ノード タイプの最小サイズは、ステートフル ワークロードを実行する場合は 5 になります。


**VM SKU:** これは、アプリケーション サービスの実行場所となるノード タイプになるため、VM SKU を選択する場合は、各ノードに配置する予定のピーク負荷を考慮に入れる必要があります。 ノード タイプに必要な容量を決めるのは、クラスターで実行する予定のワークロードです。このため、特定のワークロードについて定性的なガイダンスを示すことはできませんが、以下に、検討を始めるのに役立つ包括的なガイダンスを示します。

運用環境のワークロードの場合は次のとおりです。 

- 推奨される VM SKU は、Standard D3 か Standard D3_V2、またはローカル SSD が 14 GB 以上のこれらと同等の SKU です
- サポートされる最小の VM SKU は、Standard D1 か Standard D1_V2、またはローカル SSD が 14 GB 以上のこれらと同等の SKU です 
- コア数が少ない Standard A0 のような VM SKU は、運用ワークロードではサポートされません。
- 特に Standard A1 SKU は、パフォーマンス上の理由から運用ワークロードではサポートされません。


## <a name="non-primary-node-type---capacity-guidance-for-stateless-workloads"></a>非プライマリ ノード タイプ - ステートレス ワークロードの容量ガイダンス

非プライマリ ノード タイプで実行しているステートフル ワークロードのガイダンス。

**VM インスタンスの数:** ステートレスな運用ワークロードの場合は、サポートされる最小の非プライマリ ノード タイプ サイズは 2 です。 このサイズではアプリケーションのステートレス インスタンスを 2 つ実行できるため、1 つの VM インスタンスが損失してもサービスを存続させることができます。 

> [!NOTE]
> クラスターがバージョン 5.6 以前の Service Fabric で実行されている場合、ランタイムの欠陥 (この問題は 5.6 で修正されます) のため、非プライマリ ノード タイプを 5 未満にスケールダウンするとクラスターの正常性が異常になります。この状態は [Remove-ServiceFabricNodeState cmd](https://docs.microsoft.com/powershell/servicefabric/vlatest/Remove-ServiceFabricNodeState) を適切なノード名を指定して呼び出すまで続きます。 詳細については、[Service Fabric クラスターのスケールイン/アウト](service-fabric-cluster-scale-up-down.md)に関する記事を参照してください。
> 
>

**VM SKU:** これは、アプリケーション サービスの実行場所となるノード タイプになるため、VM SKU を選択する場合は、各ノードに配置する予定のピーク負荷を考慮に入れる必要があります。 ノード タイプに必要な容量を決めるのは、クラスターで実行する予定のワークロードです。このため、特定のワークロードについて定性的なガイダンスを示すことはできませんが、以下に、検討を始めるのに役立つ包括的なガイダンスを示します。

運用環境のワークロードの場合は次のとおりです。 


- 推奨される VM SKU は、Standard D3 か Standard D3_V2、またはこれらと同等の SKU です 
- サポートされる最小の VM SKU は、Standard D1 か Standard D1_V2、またはこれらと同等の SKU です 
- コア数が少ない Standard A0 のような VM SKU は、運用ワークロードではサポートされません。
- Standard A1 SKU は、パフォーマンス上の理由から運用ワークロードではサポートされません。

<!--Every topic should have next steps and links to the next logical set of content to keep the customer engaged-->

## <a name="next-steps"></a>次のステップ
容量計画が完了し、クラスターをセットアップしたら、以下のドキュメントをお読みください。

* [Service Fabric クラスターのセキュリティ](service-fabric-cluster-security.md)
* [Service Fabric の正常性モデルの概要](service-fabric-health-introduction.md)
* [ノード タイプと仮想マシン スケール セットの関係](service-fabric-cluster-nodetypes.md)

<!--Image references-->
[SystemServices]: ./media/service-fabric-cluster-capacity/SystemServices.png

