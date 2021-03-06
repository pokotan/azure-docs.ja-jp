---

title: "Azure Active Directory のグループのライセンスに関する問題を解決する | Microsoft Docs"
description: "Azure Active Directory でグループベースのライセンスを使用している場合にライセンス割り当ての問題を特定して解決する方法"
services: active-directory
keywords: "Azure AD のライセンス"
documentationcenter: 
author: curtand
manager: femila
editor: 
ms.assetid: 
ms.service: active-directory
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 06/05/2017
ms.author: curtand
ms.custom: H1Hack27Feb2017
ms.translationtype: Human Translation
ms.sourcegitcommit: f537befafb079256fba0529ee554c034d73f36b0
ms.openlocfilehash: bfa951a897c9b383072c0d29c9a4266c163fe753
ms.contentlocale: ja-jp
ms.lasthandoff: 07/08/2017


---

# <a name="identify-and-resolve-license-assignment-problems-for-a-group-in-azure-active-directory"></a>Azure Active Directory のグループのライセンスに関する問題を特定して解決する

Azure Active Directory (Azure AD) のグループベースのライセンスでは、ユーザーのライセンス付与に関してエラー状態の概念が導入されています。 この記事では、ユーザーがこの状態になることがある理由を説明します。

グループベースのライセンスを使用せずにライセンスを個別のユーザーに直接割り当てると、割り当て処理が失敗することがあります。 たとえば、ユーザーに対して PowerShell コマンドレット `Set-MsolUserLicense`を実行した場合、このコマンドレットは、さまざまなビジネス ロジックに関連する理由によってエラーになる可能性があります。 たとえば、ライセンス数の不足や、同時に割り当てることができない 2 つのサービス プランの競合などが発生することがあります。 問題は即座に報告されます。

グループベースのライセンスを使用している場合も同じエラーが発生する可能性がありますが、この場合のエラーは Azure AD サービスがライセンスを割り当てるときにバック グラウンドで発生します。 このため、エラーを即座に伝達することはできません。 その代わりに、エラーはユーザー オブジェクトに記録され、管理ポータル経由で報告されます。 ユーザーへのライセンス付与という元の操作が無効になることはありませんが、後で調査して解決できるようにエラー状態にあることが記録されます。

## <a name="how-to-find-license-assignment-errors"></a>ライセンスの割り当てエラーを見つける方法

1. 特定のグループ内でエラー状態にあるユーザーを見つけるには、そのグループのブレードを開きます。 エラー状態にあるユーザーが存在する場合は、**[ライセンス]** の下に通知が表示されます。

![グループ、エラー通知](media/active-directory-licensing-group-problem-resolution-azure-portal/group-error-notification.png)

2. その通知をクリックし、影響を受けているすべてのユーザーの一覧を開きます。 個別に各ユーザーをクリックすると、詳細を表示できます。

![グループ、エラー状態のユーザーの一覧](media/active-directory-licensing-group-problem-resolution-azure-portal/list-of-users-with-errors.png)

3. 少なくとも 1 つのエラーを含むすべてのグループを見つけるには、**Azure Active Directory** ブレードで、**[ライセンス]**、**[概要]** の順に選択します。 注意が必要なグループが場合は、情報ボックスが表示されます。

![概要、エラー状態にあるグループに関する情報](media/active-directory-licensing-group-problem-resolution-azure-portal/group-errors-widget.png)

4. エラーのあるすべてのグループの一覧を表示するには、ボックスをクリックします。 各グループをクリックすると詳細を表示できます。

![概要、エラーのあるグループの一覧](media/active-directory-licensing-group-problem-resolution-azure-portal/list-of-groups-with-errors.png)


以下では、考えられる問題とその解決方法を個別に説明します。

## <a name="not-enough-licenses"></a>ライセンス数の不足

**問題:** グループに指定されたいずれかの製品について、利用できるライセンスの数が不足しています。 該当する製品のライセンスを追加で購入するか、他のユーザーかグループから未使用のライセンスを解放する必要があります。

利用できるライセンスの数を確認するには、**[Azure Active Directory]** > **[ライセンス]** > **[すべての製品]** を選択します。

ライセンスを使用中のユーザーとグループを確認するには、製品をクリックします。 **[ライセンスされているユーザー]** で、ライセンスが直接割り当てられているか 1 つ以上のグループ経由で割り当てられているすべてのユーザーがわかります。 **[ライセンスされているグループ]** で、その製品が割り当てられているすべてのグループがわかります。

**PowerShell:** PowerShell コマンドレットは、このエラーを _CountViolation_ として報告します。

## <a name="conflicting-service-plans"></a>サービス プランの競合

**問題:** グループに指定されたいずれかの製品に含まれているサービス プランが、他の製品でユーザーに既に割り当てられている別のサービス プランと競合しています。 一部のサービス プランは、関連する他のサービス プランと同じユーザーに割り当てられないように構成されています。

次の例を考えてみます。 すべてのプランを有効にした Office 365 Enterprise **E1** のライセンスがユーザーに直接割り当てられているとします。 このユーザーが、Office 365 Enterprise **E3** 製品を割り当てられたグループに追加された場合、 この製品には E1 に含まれているプランと重複できないサービス プランが含まれているため、グループのライセンス割り当ては "競合するサービス プラン" エラーで失敗します。 この例では、競合しているサービス プランは次のとおりです。

-   SharePoint Online (プラン 2) と SharePoint Online (プラン 1) の競合
-   Exchange Online (プラン 2) と Exchange Online (プラン 1) の競合

この競合を解決するには、ユーザーに直接割り当てられている E1 ライセンスの 2 つのプランを無効にする必要があります。 または、グループのライセンス割り当てを変更して、E3 ライセンスのプランを無効にする必要があります。 E3 ライセンスのコンテキストで冗長である場合は、E1 ライセンスをユーザーから削除することもできます。

製品ライセンスの競合を解決する方法の決定は、常に管理者に委ねられます。 ライセンスの競合が Azure AD によって自動的に解決されることはありません。

**PowerShell:** PowerShell コマンドレットは、このエラーを _MutuallyExclusiveViolation_ として報告します。

## <a name="other-products-depend-on-this-license"></a>特定のライセンスに依存する他の製品

**問題:** グループに指定されたいずれかの製品に、他の製品の他のサービス プランが機能するうえで有効である必要があるサービス プランが含まれています。 このエラーは、Azure AD がサービス プランを削除しようとしたときに発生します。 これは、たとえばユーザーがグループから削除された結果として起こる可能性があります。

この問題を解決するには、他の方法を使用して目的のプランがユーザーに引き続き割り当てられていること、または依存するサービスがこれらのユーザーに対して無効になっていることを確認する必要があります。 その後、ユーザーからグループ ライセンスを問題なく削除できます。

**PowerShell:** PowerShell コマンドレットは、このエラーを _DependencyViolation_ として報告します。

## <a name="usage-location-isnt-allowed"></a>利用場所が許可されていない

**問題:** 一部の Microsoft サービスは、地域の法律と規制が理由ですべての場所で利用することはできません。 ライセンスをユーザーに割り当てる前に、ユーザーの [利用場所] プロパティを指定する必要があります。 これは、Azure Portal の **[ユーザー]** > **[プロファイル]** > **[設定]** セクションで実行できます。

利用場所がサポートされていないユーザーへのグループ ライセンスの割り当てが Azure AD により試行されると、その操作は失敗し、エラーがユーザーに記録されます。

この問題を解決するには、利用場所がサポートされていないユーザーをライセンス グループから削除します。 または、現在の利用場所の値が実際のユーザーの場所を表していない場合は、ライセンスが次回正しく割り当てられるように値を変更できます (新しい場所がサポートされている場合)。

**PowerShell:** PowerShell コマンドレットは、このエラーを _ProhibitedInUsageLocationViolation_ として報告します。

> [!NOTE]
> Azure AD によってグループ ライセンスが割り当てられるとき、利用場所が指定されていないユーザーは、ディレクトリの場所を継承します。 管理者は、地域の法律と規制を遵守するために、グループベースのライセンスを使用する前に、ユーザーに対して正しい利用場所の値を設定することをお勧めします。

## <a name="what-happens-when-theres-more-than-one-product-license-on-a-group"></a>1 つのグループに複数の製品ライセンスがある場合

1 つのグループに複数の製品ライセンスを割り当てることができます。 たとえば、Office 365 Enterprise E3 と Enterprise Mobility + Security をグループに割り当てて、ユーザーによる製品内の全サービスの使用を簡単に有効にできます。

グループに指定したすべてのライセンスは、Azure AD によって各ユーザーに割り当てられます。 Azure AD が、ビジネス ロジックの問題 (ライセンス数の不足、ユーザーに対して有効な他のサービスとの競合など) が原因でいずれかの製品を割り当てることができない場合、グループの他のライセンスも割り当てられません。

割り当てが失敗したユーザーとその失敗の影響を受けている製品を確認できます。

## <a name="license-assignment-fails-silently-for-a-user-due-to-duplicate-proxy-addresses-in-exchange-online"></a>Exchange Online でプロキシ アドレスの重複があるために、ユーザーのライセンス割り当てがエラーを記録することなく失敗する

Exchange Online を使用している場合は、テナント内の一部のユーザーが、同じプロキシ アドレスの値で間違って構成されている可能性があります。 グループ ベースのライセンスがこのようなユーザーにライセンス割り当てを試みると、失敗し、エラーは記録されません (上で説明した他のエラーの場合と異なります)。これは、この機能のプレビュー バージョンの制限によるもので、*一般提供*が開始されるまでには対応する予定です。

> [!TIP]
> 一部のユーザーがライセンスを得られず、そのユーザーに関してエラーが記録されていない場合は、まず、ユーザーに重複するプロキシ アドレスがないかどうか確認してください。
> これは、次の PowerShell コマンドレットを Exchange Online に対して実行することで確認できます。
```
Run Get-Recipient | where {$_.EmailAddresses -match "user@contoso.onmicrosoft.com"} | fL Name, RecipientType,emailaddresses
```
> [リモート PowerShel を使用して Exchange Online に接続する方法](https://technet.microsoft.com/library/jj984289.aspx)など、この問題の詳細については、[こちらの記事](https://support.microsoft.com/help/3042584/-proxy-address-address-is-already-being-used-error-message-in-exchange-online)をご覧ください。

割り当てに失敗したユーザーのプロキシ アドレスの問題が解決した後には、グループのライセンス処理を強制して、ライセンスを再度適用できるようになったことを確認してください。

## <a name="how-do-you-force-license-processing-in-a-group-to-resolve-errors"></a>グループでライセンスの処理を強制してエラーを解決する方法

エラーを解決するために実行した手順によっては、グループの処理を手動でトリガーしてユーザーの状態を更新する必要があります。

たとえば、直接的に割り当てられたライセンスをユーザーから削除して一部のライセンスを解放した場合、前に失敗したグループの処理を手動でトリガーして、全ユーザー メンバーに完全にライセンスを付与する必要があります。 これを行うには、グループのブレードで **[ライセンス]** を開き、ツール バーの **[再処理]** ボタンを選択します。

## <a name="next-steps"></a>次のステップ

グループによるライセンス管理の他のシナリオについては、以下を参照してください。

* [Azure Active Directory でのグループへのライセンス割り当て](active-directory-licensing-group-assignment-azure-portal.md)
* [Azure Active Directory のグループベースのライセンスとは](active-directory-licensing-whatis-azure-portal.md)
* [Azure Active Directory で個別にライセンスを付与されたユーザーをグループベースのライセンスに移行する方法](active-directory-licensing-group-migration-azure-portal.md)
* [Azure Active Directory グループベース ライセンスのその他のシナリオ](active-directory-licensing-group-advanced.md)

