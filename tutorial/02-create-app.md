<!-- markdownlint-disable MD002 MD041 -->

このセクションでは、新しい React アプリを作成します。

1. コマンドライン インターフェイス (CLI) を開き、ファイルを作成する権限を持つディレクトリに移動し、次のコマンドを実行して新しい React アプリを作成します。

    ```Shell
    npx create-react-app@4.0.1 graph-tutorial --template typescript
    ```

1. コマンドが終了したら、CLI 内のディレクトリに移動し、次のコマンドを実行してローカル `graph-tutorial` Web サーバーを起動します。

    ```Shell
    yarn start
    ```

    > [!NOTE]
    > [インストールされていない場合は、代](https://yarnpkg.com/)わりに使用 `npm start` できます。

既定のブラウザーが既定の [https://localhost:3000/](https://localhost:3000) React ページで開きます。 ブラウザーが開かない場合は、ブラウザーを開き、参照して新しいアプリ [https://localhost:3000/](https://localhost:3000) が動作を確認します。

## <a name="add-node-packages"></a>ノード パッケージの追加

次に進む前に、後で使用する追加のパッケージをインストールします。

- [React アプリ内での宣言型ルーティング用の react-router-dom。](https://github.com/ReactTraining/react-router)
- [スタイル設定](https://github.com/twbs/bootstrap) と一般的なコンポーネントのブートストラップ。
- Bootstrap に基づく React コンポーネント用の[reactstrap。](https://github.com/reactstrap/reactstrap)
- [アイコン用の fontawesome-free。](https://github.com/FortAwesome/Font-Awesome)
- [moment](https://github.com/moment/moment) for formatting dates and times.
- [Windows タイム ゾーンを](https://github.com/rubenillodo/windows-iana) IANA 形式に変換する windows-iana。
- [msal-browser](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-browser) for authenticating to Azure Active Directory and retrieving access tokens.
- [Microsoft Graph を呼び](https://github.com/microsoftgraph/msgraph-sdk-javascript) 出す microsoft-graph-client。

CLI で次のコマンドを実行します。

```Shell
yarn add react-router-dom@5.2.0 @types/react-router-dom@5.1.7 bootstrap@4.6.0 reactstrap@8.9.0 @types/reactstrap@8.7.2 @fortawesome/fontawesome-free@5.15.2
yarn add moment@2.29.1 moment-timezone@0.5.32 windows-iana@4.2.1 @azure/msal-browser@2.10.0 @microsoft/microsoft-graph-client@2.2.1 @types/microsoft-graph@1.28.0
```

## <a name="design-the-app"></a>アプリを設計する

まず、アプリのナビゲーション バーを作成します。

1. ディレクトリに新しいファイルを `./src` 作成し、 `NavBar.tsx` 次のコードを追加します。

    :::code language="typescript" source="../demo/graph-tutorial/src/NavBar.tsx" id="NavBarSnippet":::

1. アプリのホーム ページを作成します。 ディレクトリに新しいファイルを `./src` 作成し、 `Welcome.tsx` 次のコードを追加します。

    :::code language="typescript" source="../demo/graph-tutorial/src/Welcome.tsx" id="WelcomeSnippet":::

1. ユーザーにメッセージを表示するエラー メッセージ表示を作成します。 ディレクトリに新しいファイルを `./src` 作成し、 `ErrorMessage.tsx` 次のコードを追加します。

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

1. 変更内容をすべて保存し、アプリを再起動します。 これで、アプリは非常に異なって表示されます。

    ![デザインが変更されたホーム ページのスクリーンショット](images/create-app-01.png)
