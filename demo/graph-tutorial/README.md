# <a name="getting-started-with-create-react-app"></a><span data-ttu-id="9b0b5-101">React アプリの作成の開始</span><span class="sxs-lookup"><span data-stu-id="9b0b5-101">Getting Started with Create React App</span></span>

<span data-ttu-id="9b0b5-102">このプロジェクトは、React アプリの作成 [によってブートストラップされました](https://github.com/facebook/create-react-app)。</span><span class="sxs-lookup"><span data-stu-id="9b0b5-102">This project was bootstrapped with [Create React App](https://github.com/facebook/create-react-app).</span></span>

## <a name="available-scripts"></a><span data-ttu-id="9b0b5-103">利用可能なスクリプト</span><span class="sxs-lookup"><span data-stu-id="9b0b5-103">Available Scripts</span></span>

<span data-ttu-id="9b0b5-104">プロジェクト ディレクトリで、次のコマンドを実行できます。</span><span class="sxs-lookup"><span data-stu-id="9b0b5-104">In the project directory, you can run:</span></span>

### `yarn start`

<span data-ttu-id="9b0b5-105">開発モードでアプリを実行します。</span><span class="sxs-lookup"><span data-stu-id="9b0b5-105">Runs the app in the development mode.</span></span>\
<span data-ttu-id="9b0b5-106">ブラウザー [http://localhost:3000](http://localhost:3000) で開き、表示します。</span><span class="sxs-lookup"><span data-stu-id="9b0b5-106">Open [http://localhost:3000](http://localhost:3000) to view it in the browser.</span></span>

<span data-ttu-id="9b0b5-107">編集を行った場合、ページは再読み込みされます。</span><span class="sxs-lookup"><span data-stu-id="9b0b5-107">The page will reload if you make edits.</span></span>\
<span data-ttu-id="9b0b5-108">また、コンソールに lint エラーが表示されます。</span><span class="sxs-lookup"><span data-stu-id="9b0b5-108">You will also see any lint errors in the console.</span></span>

### `yarn test`

<span data-ttu-id="9b0b5-109">対話型ウォッチ モードでテスト ランナーを起動します。</span><span class="sxs-lookup"><span data-stu-id="9b0b5-109">Launches the test runner in the interactive watch mode.</span></span>\
<span data-ttu-id="9b0b5-110">詳細については、テストの [実行に関するセクション](https://facebook.github.io/create-react-app/docs/running-tests) を参照してください。</span><span class="sxs-lookup"><span data-stu-id="9b0b5-110">See the section about [running tests](https://facebook.github.io/create-react-app/docs/running-tests) for more information.</span></span>

### `yarn build`

<span data-ttu-id="9b0b5-111">実稼働環境用のアプリをフォルダーに `build` ビルドします。</span><span class="sxs-lookup"><span data-stu-id="9b0b5-111">Builds the app for production to the `build` folder.</span></span>\
<span data-ttu-id="9b0b5-112">React を実稼働モードで正しくバンドルし、最適なパフォーマンスを得るビルドを最適化します。</span><span class="sxs-lookup"><span data-stu-id="9b0b5-112">It correctly bundles React in production mode and optimizes the build for the best performance.</span></span>

<span data-ttu-id="9b0b5-113">ビルドは最小化され、ファイル名にはハッシュが含まれます。</span><span class="sxs-lookup"><span data-stu-id="9b0b5-113">The build is minified and the filenames include the hashes.\</span></span>
<span data-ttu-id="9b0b5-114">アプリを展開する準備ができました。</span><span class="sxs-lookup"><span data-stu-id="9b0b5-114">Your app is ready to be deployed!</span></span>

<span data-ttu-id="9b0b5-115">詳細については、展開に関 [するセクション](https://facebook.github.io/create-react-app/docs/deployment) を参照してください。</span><span class="sxs-lookup"><span data-stu-id="9b0b5-115">See the section about [deployment](https://facebook.github.io/create-react-app/docs/deployment) for more information.</span></span>

### `yarn eject`

<span data-ttu-id="9b0b5-116">**注: これは一方的な操作です。一度 `eject` 実行すると、戻れなきます。**</span><span class="sxs-lookup"><span data-stu-id="9b0b5-116">**Note: this is a one-way operation. Once you `eject`, you can’t go back!**</span></span>

<span data-ttu-id="9b0b5-117">ビルド ツールと構成の選択肢に満足できない場合は、いつでも `eject` 実行できます。</span><span class="sxs-lookup"><span data-stu-id="9b0b5-117">If you aren’t satisfied with the build tool and configuration choices, you can `eject` at any time.</span></span> <span data-ttu-id="9b0b5-118">このコマンドを実行すると、プロジェクトから 1 つのビルドの依存関係が削除されます。</span><span class="sxs-lookup"><span data-stu-id="9b0b5-118">This command will remove the single build dependency from your project.</span></span>

<span data-ttu-id="9b0b5-119">代わりに、すべての構成ファイルと推移的な依存関係 (webpack、クリップボード、ESLint など) がプロジェクトにコピーされ、完全に制御できます。</span><span class="sxs-lookup"><span data-stu-id="9b0b5-119">Instead, it will copy all the configuration files and the transitive dependencies (webpack, Babel, ESLint, etc) right into your project so you have full control over them.</span></span> <span data-ttu-id="9b0b5-120">コマンド以外のすべてのコマンドは引き続き動作しますが、コピーされたスクリプトをポイントして、それらを `eject` 調整できます。</span><span class="sxs-lookup"><span data-stu-id="9b0b5-120">All of the commands except `eject` will still work, but they will point to the copied scripts so you can tweak them.</span></span> <span data-ttu-id="9b0b5-121">この時点で、ユーザーは自分で行います。</span><span class="sxs-lookup"><span data-stu-id="9b0b5-121">At this point you’re on your own.</span></span>

<span data-ttu-id="9b0b5-122">You don't have to use `eject` .</span><span class="sxs-lookup"><span data-stu-id="9b0b5-122">You don’t have to ever use `eject`.</span></span> <span data-ttu-id="9b0b5-123">選択された機能セットは、小規模および中小規模の展開に適しています。この機能を使用する義務を感じてはならない。</span><span class="sxs-lookup"><span data-stu-id="9b0b5-123">The curated feature set is suitable for small and middle deployments, and you shouldn’t feel obligated to use this feature.</span></span> <span data-ttu-id="9b0b5-124">ただし、準備が整っているときにカスタマイズできない場合、このツールは役に立たなかったと理解しています。</span><span class="sxs-lookup"><span data-stu-id="9b0b5-124">However we understand that this tool wouldn’t be useful if you couldn’t customize it when you are ready for it.</span></span>

## <a name="learn-more"></a><span data-ttu-id="9b0b5-125">詳細情報</span><span class="sxs-lookup"><span data-stu-id="9b0b5-125">Learn More</span></span>

<span data-ttu-id="9b0b5-126">詳細については、「React アプリの作成 [」のドキュメントを参照してください](https://facebook.github.io/create-react-app/docs/getting-started)。</span><span class="sxs-lookup"><span data-stu-id="9b0b5-126">You can learn more in the [Create React App documentation](https://facebook.github.io/create-react-app/docs/getting-started).</span></span>

<span data-ttu-id="9b0b5-127">React の詳細については、React のドキュメント [を参照してください](https://reactjs.org/)。</span><span class="sxs-lookup"><span data-stu-id="9b0b5-127">To learn React, check out the [React documentation](https://reactjs.org/).</span></span>
