<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="c316f-101">この演習では、Azure Active Directory 管理センターを使用して、新しい Azure AD web アプリケーション登録を作成します。</span><span class="sxs-lookup"><span data-stu-id="c316f-101">In this exercise, you will create a new Azure AD web application registration using the Azure Active Directory admin center.</span></span>

1. <span data-ttu-id="c316f-102">ブラウザーを開き、[Azure Active Directory 管理センター](https://aad.portal.azure.com)へ移動します。</span><span class="sxs-lookup"><span data-stu-id="c316f-102">Open a browser and navigate to the [Azure Active Directory admin center](https://aad.portal.azure.com).</span></span> <span data-ttu-id="c316f-103">**個人用アカウント** (別名: Microsoft アカウント)、または**職場/学校アカウント**を使用してログインします。</span><span class="sxs-lookup"><span data-stu-id="c316f-103">Login using a **personal account** (aka: Microsoft Account) or **Work or School Account**.</span></span>

1. <span data-ttu-id="c316f-104">左側のナビゲーションで **[Azure Active Directory]** を選択し、それから **[管理]** で **[アプリの登録]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="c316f-104">Select **Azure Active Directory** in the left-hand navigation, then select **App registrations** under **Manage**.</span></span>

    ![<span data-ttu-id="c316f-105">アプリの登録のスクリーンショット</span><span class="sxs-lookup"><span data-stu-id="c316f-105">A screenshot of the App registrations</span></span> ](./images/aad-portal-app-registrations.png)

    > [!NOTE]
    > <span data-ttu-id="c316f-106">Azure AD B2C ユーザーには、 **アプリの登録 (レガシー)** だけが表示されることがあります。</span><span class="sxs-lookup"><span data-stu-id="c316f-106">Azure AD B2C users may only see **App registrations (legacy)**.</span></span> <span data-ttu-id="c316f-107">この場合は、に直接移動してください [https://aka.ms/appregistrations](https://aka.ms/appregistrations) 。</span><span class="sxs-lookup"><span data-stu-id="c316f-107">In this case, please go directly to [https://aka.ms/appregistrations](https://aka.ms/appregistrations).</span></span>

1. <span data-ttu-id="c316f-108">**[新規登録]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="c316f-108">Select **New registration**.</span></span> <span data-ttu-id="c316f-109">**[アプリケーションを登録]** ページで、次のように値を設定します。</span><span class="sxs-lookup"><span data-stu-id="c316f-109">On the **Register an application** page, set the values as follows.</span></span>

    - <span data-ttu-id="c316f-110">`React Graph Tutorial` に **[名前]** を設定します。</span><span class="sxs-lookup"><span data-stu-id="c316f-110">Set **Name** to `React Graph Tutorial`.</span></span>
    - <span data-ttu-id="c316f-111">**[サポートされているアカウントの種類]** を **[任意の組織のディレクトリ内のアカウントと個人用の Microsoft アカウント]** に設定します。</span><span class="sxs-lookup"><span data-stu-id="c316f-111">Set **Supported account types** to **Accounts in any organizational directory and personal Microsoft accounts**.</span></span>
    - <span data-ttu-id="c316f-112">**[リダイレクト URI]** で、最初のドロップダウン リストを `Single-page application (SPA)` に設定し、それから `http://localhost:3000` に値を設定します。</span><span class="sxs-lookup"><span data-stu-id="c316f-112">Under **Redirect URI**, set the first drop-down to `Single-page application (SPA)` and set the value to `http://localhost:3000`.</span></span>

    ![[アプリケーションを登録する] ページのスクリーンショット](./images/aad-register-an-app.png)

1. <span data-ttu-id="c316f-114">**[登録]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="c316f-114">Choose **Register**.</span></span> <span data-ttu-id="c316f-115">[グラフの対応] **チュートリアル** ページで、 **アプリケーション (クライアント) ID** の値をコピーして保存します。次の手順で必要になります。</span><span class="sxs-lookup"><span data-stu-id="c316f-115">On the **React Graph Tutorial** page, copy the value of the **Application (client) ID** and save it, you will need it in the next step.</span></span>

    ![新しいアプリ登録のアプリケーション ID のスクリーンショット](./images/aad-application-id.png)
