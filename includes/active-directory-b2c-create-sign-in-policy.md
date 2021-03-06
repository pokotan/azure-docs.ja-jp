アプリケーションにサインインできるようにするには、サインイン ポリシーを作成する必要があります。 このポリシーは、サインイン中のコンシューマーのエクスペリエンスと、サインインが成功したときにアプリケーションが受け取るトークンのコンテンツを記述します。

[!INCLUDE [active-directory-b2c-portal-navigate-b2c-service](active-directory-b2c-portal-navigate-b2c-service.md)] **[サインイン ポリシー]** をクリックします。

ブレードの上部にある **[+追加]** をクリックします。

**[名前]** によって、アプリケーションで使用されるサインイン ポリシー名が決定されます。 たとえば、「**SiIn**」と入力します。

**[ID プロバイダー]** をクリックし、**[ローカル アカウント サインイン]** を選択します。 既に構成されている場合は、ソーシャル ID プロバイダーを選択することもできます。 **[OK]**をクリックします。

**[アプリケーション クレーム]**をクリックします。 ここで、サインイン エクスペリエンスの成功後にアプリケーションに戻されるトークンで返される要求を選択します。 たとえば、"**表示名**"、"**ID プロバイダー**"、"**郵便番号**"、および "**ユーザーのオブジェクト ID**" などを選択します。 **[OK]**をクリックします。

**[作成]**をクリックします。 作成したポリシーが**サインイン ポリシー** ブレードに "**B2C_1_SiIn**" と表示されます (**B2C\_1\_** フラグメントが自動的に追加されます)。

**[B2C_1_SiIn]** をクリックすることによってポリシーを開きます。

**[アプリケーション]** ドロップダウンで **[Contoso B2C app] \(Contoso B2C アプリ)** を、また **[Reply URL / Redirect URI] \(応答 URL/リダイレクト URI)** ドロップダウンで `https://localhost:44321/` を選択します。

**[今すぐ実行]**をクリックします。 新しいブラウザー タブが開き、アプリケーションへのサインインのコンシューマー エクスペリエンスを確認できます。

> [!NOTE]
> ポリシーの作成と更新が有効になるまで、最大で 1 分間かかります。
>