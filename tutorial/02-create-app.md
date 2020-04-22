<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="6fa80-101">このセクションでは、新しい反応アプリを作成します。</span><span class="sxs-lookup"><span data-stu-id="6fa80-101">In this section you'll create a new React app.</span></span>

1. <span data-ttu-id="6fa80-102">コマンドラインインターフェイス (CLI) を開き、ファイルを作成する権限があるディレクトリに移動し、次のコマンドを実行して新しい反応アプリを作成します。</span><span class="sxs-lookup"><span data-stu-id="6fa80-102">Open your command-line interface (CLI), navigate to a directory where you have rights to create files, and run the following commands to create a new React app.</span></span>

    ```Shell
    npx create-react-app@3.4.0 graph-tutorial --template typescript
    ```

1. <span data-ttu-id="6fa80-103">コマンドが完了したら、CLI の`graph-tutorial`ディレクトリに移動し、次のコマンドを実行してローカル web サーバーを開始します。</span><span class="sxs-lookup"><span data-stu-id="6fa80-103">Once the command finishes, change to the `graph-tutorial` directory in your CLI and run the following command to start a local web server.</span></span>

    ```Shell
    npm start
    ```

<span data-ttu-id="6fa80-104">既定のブラウザーが開き[https://localhost:3000/](https://localhost:3000) 、既定の反応ページが表示されます。</span><span class="sxs-lookup"><span data-stu-id="6fa80-104">Your default browser opens to [https://localhost:3000/](https://localhost:3000) with a default React page.</span></span> <span data-ttu-id="6fa80-105">ブラウザーが開かない場合は、それを開き、 [https://localhost:3000/](https://localhost:3000)を参照して、新しいアプリが動作することを確認します。</span><span class="sxs-lookup"><span data-stu-id="6fa80-105">If your browser doesn't open, open it and browse to [https://localhost:3000/](https://localhost:3000) to verify that the new app works.</span></span>

## <a name="add-node-packages"></a><span data-ttu-id="6fa80-106">ノードパッケージを追加する</span><span class="sxs-lookup"><span data-stu-id="6fa80-106">Add Node packages</span></span>

<span data-ttu-id="6fa80-107">に進む前に、後で使用する追加のパッケージをインストールします。</span><span class="sxs-lookup"><span data-stu-id="6fa80-107">Before moving on, install some additional packages that you will use later:</span></span>

- <span data-ttu-id="6fa80-108">応答アプリケーション内の宣言型ルーティングのための[ルーター-dom](https://github.com/ReactTraining/react-router)を処理します。</span><span class="sxs-lookup"><span data-stu-id="6fa80-108">[react-router-dom](https://github.com/ReactTraining/react-router) for declarative routing inside the React app.</span></span>
- <span data-ttu-id="6fa80-109">スタイル設定と共通コンポーネントの[ブートストラップ](https://github.com/twbs/bootstrap)。</span><span class="sxs-lookup"><span data-stu-id="6fa80-109">[bootstrap](https://github.com/twbs/bootstrap) for styling and common components.</span></span>
- <span data-ttu-id="6fa80-110">ブートストラップに基づくコンポーネントに対応するための[reactstrap](https://github.com/reactstrap/reactstrap) 。</span><span class="sxs-lookup"><span data-stu-id="6fa80-110">[reactstrap](https://github.com/reactstrap/reactstrap) for React components based on Bootstrap.</span></span>
- <span data-ttu-id="6fa80-111">アイコンの[fontawesome](https://github.com/FortAwesome/Font-Awesome) 。</span><span class="sxs-lookup"><span data-stu-id="6fa80-111">[fontawesome-free](https://github.com/FortAwesome/Font-Awesome) for icons.</span></span>
- <span data-ttu-id="6fa80-112">日付と時刻を書式設定するための[モーメント](https://github.com/moment/moment)。</span><span class="sxs-lookup"><span data-stu-id="6fa80-112">[moment](https://github.com/moment/moment) for formatting dates and times.</span></span>
- <span data-ttu-id="6fa80-113">Azure Active Directory に認証し、アクセストークンを取得するための[msal](https://github.com/AzureAD/microsoft-authentication-library-for-js) 。</span><span class="sxs-lookup"><span data-stu-id="6fa80-113">[msal](https://github.com/AzureAD/microsoft-authentication-library-for-js) for authenticating to Azure Active Directory and retrieving access tokens.</span></span>
- <span data-ttu-id="6fa80-114">[microsoft graph-](https://github.com/microsoftgraph/msgraph-sdk-javascript) microsoft graph に電話をかけるためのクライアントです。</span><span class="sxs-lookup"><span data-stu-id="6fa80-114">[microsoft-graph-client](https://github.com/microsoftgraph/msgraph-sdk-javascript) for making calls to Microsoft Graph.</span></span>

<span data-ttu-id="6fa80-115">CLI で次のコマンドを実行します。</span><span class="sxs-lookup"><span data-stu-id="6fa80-115">Run the following command in your CLI.</span></span>

```Shell
npm install react-router-dom@5.1.2 @types/react-router-dom@5.1.3 bootstrap@4.4.1 reactstrap@8.4.1 @types/reactstrap@8.4.2
npm install @fortawesome/fontawesome-free@5.12.1 moment@2.24.0 msal@1.2.1 @microsoft/microsoft-graph-client@2.0.0 @types/microsoft-graph@1.12.0
```

## <a name="design-the-app"></a><span data-ttu-id="6fa80-116">アプリを設計する</span><span class="sxs-lookup"><span data-stu-id="6fa80-116">Design the app</span></span>

<span data-ttu-id="6fa80-117">最初に、アプリのナビゲーションバーを作成します。</span><span class="sxs-lookup"><span data-stu-id="6fa80-117">Start by creating a navbar for the app.</span></span>

1. <span data-ttu-id="6fa80-118">という名前`./src` `NavBar.tsx`のディレクトリに新しいファイルを作成し、次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="6fa80-118">Create a new file in the `./src` directory named `NavBar.tsx` and add the following code.</span></span>

    :::code language="typescript" source="../demo/graph-tutorial/src/NavBar.tsx" id="NavBarSnippet":::

1. <span data-ttu-id="6fa80-119">アプリのホームページを作成します。</span><span class="sxs-lookup"><span data-stu-id="6fa80-119">Create a home page for the app.</span></span> <span data-ttu-id="6fa80-120">という名前`./src` `Welcome.tsx`のディレクトリに新しいファイルを作成し、次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="6fa80-120">Create a new file in the `./src` directory named `Welcome.tsx` and add the following code.</span></span>

    :::code language="typescript" source="../demo/graph-tutorial/src/Welcome.tsx" id="WelcomeSnippet":::

1. <span data-ttu-id="6fa80-121">エラーメッセージ表示を作成して、ユーザーにメッセージを表示します。</span><span class="sxs-lookup"><span data-stu-id="6fa80-121">Create an error message display to display messages to the user.</span></span> <span data-ttu-id="6fa80-122">という名前`./src` `ErrorMessage.tsx`のディレクトリに新しいファイルを作成し、次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="6fa80-122">Create a new file in the `./src` directory named `ErrorMessage.tsx` and add the following code.</span></span>

    :::code language="typescript" source="../demo/graph-tutorial/src/ErrorMessage.tsx" id="ErrorMessageSnippet":::

1. <span data-ttu-id="6fa80-123">`./src/index.css` ファイルを開き、そのコンテンツ全体を次のように置き換えます。</span><span class="sxs-lookup"><span data-stu-id="6fa80-123">Open the `./src/index.css` file and replace its entire contents with the following.</span></span>

    :::code language="css" source="../demo/graph-tutorial/src/index.css":::

1. <span data-ttu-id="6fa80-124">を`./src/App.tsx`開き、内容全体を次のように置き換えます。</span><span class="sxs-lookup"><span data-stu-id="6fa80-124">Open `./src/App.tsx` and replace its entire contents with the following.</span></span>

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
                user={this.props.user}/>
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

1. <span data-ttu-id="6fa80-125">すべての変更を保存し、ページを更新します。</span><span class="sxs-lookup"><span data-stu-id="6fa80-125">Save all of your changes and refresh the page.</span></span> <span data-ttu-id="6fa80-126">この時点で、アプリの外観は大きく異なります。</span><span class="sxs-lookup"><span data-stu-id="6fa80-126">Now, the app should look very different.</span></span>

    ![デザインが変更されたホーム ページのスクリーンショット](images/create-app-01.png)
