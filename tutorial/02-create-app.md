<!-- markdownlint-disable MD002 MD041 -->

このセクションでは、新しいアプリを作成Reactします。

1. コマンド ライン インターフェイス (CLI) を開き、ファイルを作成する権限を持つディレクトリに移動し、次のコマンドを実行して新しいアプリReactします。

    ```Shell
    yarn create react-app graph-tutorial --template typescript
    ```

1. コマンドが完了したら、CLI のディレクトリに移動し、次のコマンドを実行してローカル `graph-tutorial` Web サーバーを起動します。

    ```Shell
    yarn start
    ```

    > [!NOTE]
    > Yarn が [インストールされていない場合](https://yarnpkg.com/) は、代わりに `npm start` 使用できます。

既定のブラウザーが開 [https://localhost:3000/](https://localhost:3000) き、既定のページReactされます。 ブラウザーが開かない場合は、ブラウザーを開いて参照して、新しいアプリ [https://localhost:3000/](https://localhost:3000) が動作するを確認します。

## <a name="add-node-packages"></a>ノード パッケージの追加

次に進む前に、後で使用する追加のパッケージをインストールします。

- [react-router-dom](https://github.com/ReactTraining/react-router)を使用して、アプリ内での宣言型ルーティングReactします。
- [スタイル設定](https://github.com/twbs/bootstrap) と一般的なコンポーネントのブートストラップ。
- ブートストラップに基づくReactコンポーネントの[reactstrap。](https://github.com/reactstrap/reactstrap)
- [fontawesome-free for](https://github.com/FortAwesome/Font-Awesome) icons.
- [日付と](https://github.com/moment/moment) 時刻を書式設定する時間。
- [windows-iana](https://github.com/rubenillodo/windows-iana)を使用して、Windowsを IANA 形式に変換します。
- [msal-browser](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-browser)を使用して、アクセス トークンAzure Active Directory取得します。
- [microsoft-graph-client for making](https://github.com/microsoftgraph/msgraph-sdk-javascript) calls to Microsoft Graph.

CLI で次のコマンドを実行します。

```Shell
yarn add react-router-dom@5.2.0 bootstrap@4.6.0 reactstrap@8.9.0 @fortawesome/fontawesome-free@5.15.3
yarn add moment-timezone@0.5.33 windows-iana@5.0.2 @azure/msal-browser@2.14.1 @microsoft/microsoft-graph-client@2.2.1
yarn add -D @types/react-router-dom@5.1.7 @types/microsoft-graph@1.36.0
```

## <a name="design-the-app"></a>アプリを設計する

まず、アプリのナビゲーション バーを作成します。

1. という名前のディレクトリに新しい `./src` ファイルを作成 `NavBar.tsx` し、次のコードを追加します。

    :::code language="typescript" source="../demo/graph-tutorial/src/NavBar.tsx" id="NavBarSnippet":::

1. アプリのホーム ページを作成します。 という名前のディレクトリに新しい `./src` ファイルを作成 `Welcome.tsx` し、次のコードを追加します。

    :::code language="typescript" source="../demo/graph-tutorial/src/Welcome.tsx" id="WelcomeSnippet":::

1. ユーザーにメッセージを表示するエラー メッセージ表示を作成します。 という名前のディレクトリに新しい `./src` ファイルを作成 `ErrorMessage.tsx` し、次のコードを追加します。

    :::code language="typescript" source="../demo/graph-tutorial/src/ErrorMessage.tsx" id="ErrorMessageSnippet":::

1. `./src/index.css` ファイルを開き、そのコンテンツ全体を次のように置き換えます。

    :::code language="css" source="../demo/graph-tutorial/src/index.css":::

1. コンテンツ `./src/App.tsx` 全体を開き、次の内容に置き換えてください。

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

1. 変更内容をすべて保存し、アプリを再起動します。 これで、アプリは非常に異なって見える必要があります。

    ![デザインが変更されたホーム ページのスクリーンショット](images/create-app-01.png)
