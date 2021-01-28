<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="812fa-101">このセクションでは、新しい React アプリを作成します。</span><span class="sxs-lookup"><span data-stu-id="812fa-101">In this section you'll create a new React app.</span></span>

1. <span data-ttu-id="812fa-102">コマンドライン インターフェイス (CLI) を開き、ファイルを作成する権限を持つディレクトリに移動し、次のコマンドを実行して新しい React アプリを作成します。</span><span class="sxs-lookup"><span data-stu-id="812fa-102">Open your command-line interface (CLI), navigate to a directory where you have rights to create files, and run the following commands to create a new React app.</span></span>

    ```Shell
    npx create-react-app@4.0.1 graph-tutorial --template typescript
    ```

1. <span data-ttu-id="812fa-103">コマンドが終了したら、CLI 内のディレクトリに移動し、次のコマンドを実行してローカル `graph-tutorial` Web サーバーを起動します。</span><span class="sxs-lookup"><span data-stu-id="812fa-103">Once the command finishes, change to the `graph-tutorial` directory in your CLI and run the following command to start a local web server.</span></span>

    ```Shell
    yarn start
    ```

    > [!NOTE]
    > <span data-ttu-id="812fa-104">[インストールされていない場合は、代](https://yarnpkg.com/)わりに使用 `npm start` できます。</span><span class="sxs-lookup"><span data-stu-id="812fa-104">If you do not have [Yarn](https://yarnpkg.com/) installed, you can use `npm start` instead.</span></span>

<span data-ttu-id="812fa-105">既定のブラウザーが既定の [https://localhost:3000/](https://localhost:3000) React ページで開きます。</span><span class="sxs-lookup"><span data-stu-id="812fa-105">Your default browser opens to [https://localhost:3000/](https://localhost:3000) with a default React page.</span></span> <span data-ttu-id="812fa-106">ブラウザーが開かない場合は、ブラウザーを開き、参照して新しいアプリ [https://localhost:3000/](https://localhost:3000) が動作を確認します。</span><span class="sxs-lookup"><span data-stu-id="812fa-106">If your browser doesn't open, open it and browse to [https://localhost:3000/](https://localhost:3000) to verify that the new app works.</span></span>

## <a name="add-node-packages"></a><span data-ttu-id="812fa-107">ノード パッケージの追加</span><span class="sxs-lookup"><span data-stu-id="812fa-107">Add Node packages</span></span>

<span data-ttu-id="812fa-108">次に進む前に、後で使用する追加のパッケージをインストールします。</span><span class="sxs-lookup"><span data-stu-id="812fa-108">Before moving on, install some additional packages that you will use later:</span></span>

- <span data-ttu-id="812fa-109">[React アプリ内での宣言型ルーティング用の react-router-dom。](https://github.com/ReactTraining/react-router)</span><span class="sxs-lookup"><span data-stu-id="812fa-109">[react-router-dom](https://github.com/ReactTraining/react-router) for declarative routing inside the React app.</span></span>
- <span data-ttu-id="812fa-110">[スタイル設定](https://github.com/twbs/bootstrap) と一般的なコンポーネントのブートストラップ。</span><span class="sxs-lookup"><span data-stu-id="812fa-110">[bootstrap](https://github.com/twbs/bootstrap) for styling and common components.</span></span>
- <span data-ttu-id="812fa-111">Bootstrap に基づく React コンポーネント用の[reactstrap。](https://github.com/reactstrap/reactstrap)</span><span class="sxs-lookup"><span data-stu-id="812fa-111">[reactstrap](https://github.com/reactstrap/reactstrap) for React components based on Bootstrap.</span></span>
- <span data-ttu-id="812fa-112">[アイコン用の fontawesome-free。](https://github.com/FortAwesome/Font-Awesome)</span><span class="sxs-lookup"><span data-stu-id="812fa-112">[fontawesome-free](https://github.com/FortAwesome/Font-Awesome) for icons.</span></span>
- <span data-ttu-id="812fa-113">[moment](https://github.com/moment/moment) for formatting dates and times.</span><span class="sxs-lookup"><span data-stu-id="812fa-113">[moment](https://github.com/moment/moment) for formatting dates and times.</span></span>
- <span data-ttu-id="812fa-114">[Windows タイム ゾーンを](https://github.com/rubenillodo/windows-iana) IANA 形式に変換する windows-iana。</span><span class="sxs-lookup"><span data-stu-id="812fa-114">[windows-iana](https://github.com/rubenillodo/windows-iana) for translating Windows time zones to IANA format.</span></span>
- <span data-ttu-id="812fa-115">[msal-browser](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-browser) for authenticating to Azure Active Directory and retrieving access tokens.</span><span class="sxs-lookup"><span data-stu-id="812fa-115">[msal-browser](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-browser) for authenticating to Azure Active Directory and retrieving access tokens.</span></span>
- <span data-ttu-id="812fa-116">[Microsoft Graph を呼び](https://github.com/microsoftgraph/msgraph-sdk-javascript) 出す microsoft-graph-client。</span><span class="sxs-lookup"><span data-stu-id="812fa-116">[microsoft-graph-client](https://github.com/microsoftgraph/msgraph-sdk-javascript) for making calls to Microsoft Graph.</span></span>

<span data-ttu-id="812fa-117">CLI で次のコマンドを実行します。</span><span class="sxs-lookup"><span data-stu-id="812fa-117">Run the following command in your CLI.</span></span>

```Shell
yarn add react-router-dom@5.2.0 @types/react-router-dom@5.1.7 bootstrap@4.6.0 reactstrap@8.9.0 @types/reactstrap@8.7.2 @fortawesome/fontawesome-free@5.15.2
yarn add moment@2.29.1 moment-timezone@0.5.32 windows-iana@4.2.1 @azure/msal-browser@2.10.0 @microsoft/microsoft-graph-client@2.2.1 @types/microsoft-graph@1.28.0
```

## <a name="design-the-app"></a><span data-ttu-id="812fa-118">アプリを設計する</span><span class="sxs-lookup"><span data-stu-id="812fa-118">Design the app</span></span>

<span data-ttu-id="812fa-119">まず、アプリのナビゲーション バーを作成します。</span><span class="sxs-lookup"><span data-stu-id="812fa-119">Start by creating a navbar for the app.</span></span>

1. <span data-ttu-id="812fa-120">ディレクトリに新しいファイルを `./src` 作成し、 `NavBar.tsx` 次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="812fa-120">Create a new file in the `./src` directory named `NavBar.tsx` and add the following code.</span></span>

    :::code language="typescript" source="../demo/graph-tutorial/src/NavBar.tsx" id="NavBarSnippet":::

1. <span data-ttu-id="812fa-121">アプリのホーム ページを作成します。</span><span class="sxs-lookup"><span data-stu-id="812fa-121">Create a home page for the app.</span></span> <span data-ttu-id="812fa-122">ディレクトリに新しいファイルを `./src` 作成し、 `Welcome.tsx` 次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="812fa-122">Create a new file in the `./src` directory named `Welcome.tsx` and add the following code.</span></span>

    :::code language="typescript" source="../demo/graph-tutorial/src/Welcome.tsx" id="WelcomeSnippet":::

1. <span data-ttu-id="812fa-123">ユーザーにメッセージを表示するエラー メッセージ表示を作成します。</span><span class="sxs-lookup"><span data-stu-id="812fa-123">Create an error message display to display messages to the user.</span></span> <span data-ttu-id="812fa-124">ディレクトリに新しいファイルを `./src` 作成し、 `ErrorMessage.tsx` 次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="812fa-124">Create a new file in the `./src` directory named `ErrorMessage.tsx` and add the following code.</span></span>

    :::code language="typescript" source="../demo/graph-tutorial/src/ErrorMessage.tsx" id="ErrorMessageSnippet":::

1. <span data-ttu-id="812fa-125">`./src/index.css` ファイルを開き、そのコンテンツ全体を次のように置き換えます。</span><span class="sxs-lookup"><span data-stu-id="812fa-125">Open the `./src/index.css` file and replace its entire contents with the following.</span></span>

    :::code language="css" source="../demo/graph-tutorial/src/index.css":::

1. <span data-ttu-id="812fa-126">コンテンツ `./src/App.tsx` 全体を開き、次の内容に置き換えてください。</span><span class="sxs-lookup"><span data-stu-id="812fa-126">Open `./src/App.tsx` and replace its entire contents with the following.</span></span>

    ```typescript
    import React, { Component } from 'react';
    import { BrowserRouter as Router, Route, Redirect } from 'react-router-dom';
    import { Container } from 'reactstrap';
    import NavBar from './NavBar';
    import ErrorMessage from './ErrorMessage';
    import Welcome from './Welcome';
    import 'bootstrap/dist/css/bootstrap.css';

    class App extends Component<any> {
      render() {
        let error = null;
        if (this.props.error) {
          error = <ErrorMessage
            message={this.props.error.message}
            debug={this.props.error.debug} />;
        }

        return (
          <Router>
            <div>
              <NavBar
                isAuthenticated={this.props.isAuthenticated}
                authButtonMethod={this.props.isAuthenticated ? this.props.logout : this.props.login}
                user={this.props.user} />
              <Container>
                {error}
                <Route exact path="/"
                  render={(props) =>
                    <Welcome {...props}
                      isAuthenticated={this.props.isAuthenticated}
                      user={this.props.user}
                      authButtonMethod={this.props.login} />
                  } />
              </Container>
            </div>
          </Router>
        );
      }
    }

    export default App;
    ```

1. <span data-ttu-id="812fa-127">変更内容をすべて保存し、アプリを再起動します。</span><span class="sxs-lookup"><span data-stu-id="812fa-127">Save all of your changes and restart the app.</span></span> <span data-ttu-id="812fa-128">これで、アプリは非常に異なって表示されます。</span><span class="sxs-lookup"><span data-stu-id="812fa-128">Now, the app should look very different.</span></span>

    ![デザインが変更されたホーム ページのスクリーンショット](images/create-app-01.png)
