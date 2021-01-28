<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="e4eed-101">このチュートリアルでは、Microsoft Graph API を使用してユーザーの予定表情報を取得する React シングルページ アプリを作成する方法について説明します。</span><span class="sxs-lookup"><span data-stu-id="e4eed-101">This tutorial teaches you how to build a React single-page app that uses the Microsoft Graph API to retrieve calendar information for a user.</span></span>

> [!TIP]
> <span data-ttu-id="e4eed-102">完成したチュートリアルをダウンロードするだけの場合は [、GitHub](https://github.com/microsoftgraph/msgraph-training-reactspa)リポジトリをダウンロードまたは複製できます。</span><span class="sxs-lookup"><span data-stu-id="e4eed-102">If you prefer to just download the completed tutorial, you can download or clone the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-reactspa).</span></span>

## <a name="prerequisites"></a><span data-ttu-id="e4eed-103">前提条件</span><span class="sxs-lookup"><span data-stu-id="e4eed-103">Prerequisites</span></span>

<span data-ttu-id="e4eed-104">このチュートリアルを開始する前に、開発用 [Node.js](https://nodejs.org) にインストールする必要があります。</span><span class="sxs-lookup"><span data-stu-id="e4eed-104">Before you start this tutorial, you should have [Node.js](https://nodejs.org) installed on your development machine.</span></span> <span data-ttu-id="e4eed-105">ダウンロード オプションがない場合Node.jsダウンロード オプションについては、前のリンクを参照してください。</span><span class="sxs-lookup"><span data-stu-id="e4eed-105">If you do not have Node.js, visit the previous link for download options.</span></span>

<span data-ttu-id="e4eed-106">また、メールボックスを持つ個人用の Microsoft アカウントが Outlook.com Microsoft の仕事用アカウントまたは学校アカウントである必要があります。</span><span class="sxs-lookup"><span data-stu-id="e4eed-106">You should also have either a personal Microsoft account with a mailbox on Outlook.com, or a Microsoft work or school account.</span></span> <span data-ttu-id="e4eed-107">Microsoft アカウントをお持ちない場合は、無料アカウントを取得するためのオプションが 2 つ提供されています。</span><span class="sxs-lookup"><span data-stu-id="e4eed-107">If you don't have a Microsoft account, there are a couple of options to get a free account:</span></span>

- <span data-ttu-id="e4eed-108">新しい [個人用 Microsoft アカウントにサインアップできます](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1)。</span><span class="sxs-lookup"><span data-stu-id="e4eed-108">You can [sign up for a new personal Microsoft account](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1).</span></span>
- <span data-ttu-id="e4eed-109">Office [365 開発者プログラムにサインアップして、365](https://developer.microsoft.com/office/dev-program) サブスクリプションを無料Office取得できます。</span><span class="sxs-lookup"><span data-stu-id="e4eed-109">You can [sign up for the Office 365 Developer Program](https://developer.microsoft.com/office/dev-program) to get a free Office 365 subscription.</span></span>

> [!NOTE]
> <span data-ttu-id="e4eed-110">このチュートリアルは、ノード バージョン 14.15.0 で記述されています。</span><span class="sxs-lookup"><span data-stu-id="e4eed-110">This tutorial was written with Node version 14.15.0.</span></span> <span data-ttu-id="e4eed-111">このガイドの手順は他のバージョンでも動作する可能性がありますが、テストは行ってはいではありません。</span><span class="sxs-lookup"><span data-stu-id="e4eed-111">The steps in this guide may work with other versions, but that has not been tested.</span></span>

## <a name="feedback"></a><span data-ttu-id="e4eed-112">フィードバック</span><span class="sxs-lookup"><span data-stu-id="e4eed-112">Feedback</span></span>

<span data-ttu-id="e4eed-113">このチュートリアルに関するフィードバックは [、GitHub リポジトリで提供してください](https://github.com/microsoftgraph/msgraph-training-reactspa)。</span><span class="sxs-lookup"><span data-stu-id="e4eed-113">Please provide any feedback on this tutorial in the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-reactspa).</span></span>
