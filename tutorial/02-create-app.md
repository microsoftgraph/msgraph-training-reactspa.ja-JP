<!-- markdownlint-disable MD002 MD041 -->

このセクションでは、新しい反応アプリを作成します。

1. コマンドラインインターフェイス (CLI) を開き、ファイルを作成する権限があるディレクトリに移動し、次のコマンドを実行して新しい反応アプリを作成します。

    ```Shell
    npx create-react-app@3.4.1 graph-tutorial --template typescript
    ```

1. コマンドが完了したら、 `graph-tutorial` CLI のディレクトリに移動し、次のコマンドを実行してローカル web サーバーを開始します。

    ```Shell
    npm start
    ```

既定のブラウザーが開き、既定の反応ページが表示さ [https://localhost:3000/](https://localhost:3000) れます。 ブラウザーが開かない場合は、それを開き、を参照して [https://localhost:3000/](https://localhost:3000) 、新しいアプリが動作することを確認します。

## <a name="add-node-packages"></a>ノードパッケージを追加する

に進む前に、後で使用する追加のパッケージをインストールします。

- 応答アプリケーション内の宣言型ルーティングのための[ルーター-dom](https://github.com/ReactTraining/react-router)を処理します。
- スタイル設定と共通コンポーネントの[ブートストラップ](https://github.com/twbs/bootstrap)。
- ブートストラップに基づくコンポーネントに対応するための[reactstrap](https://github.com/reactstrap/reactstrap) 。
- アイコンの[fontawesome](https://github.com/FortAwesome/Font-Awesome) 。
- 日付と時刻を書式設定するための[モーメント](https://github.com/moment/moment)。
- windows のタイムゾーンを IANA 形式に変換するための[windows の iana](https://github.com/rubenillodo/windows-iana) 。
- [msal-](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-browser) Azure Active Directory を認証し、アクセストークンを取得するためのブラウザー。
- [microsoft graph-](https://github.com/microsoftgraph/msgraph-sdk-javascript) microsoft graph に電話をかけるためのクライアントです。

CLI で次のコマンドを実行します。

```Shell
npm install react-router-dom@5.2.0 @types/react-router-dom@5.1.5 bootstrap@4.5.2 reactstrap@8.5.1 @types/reactstrap@8.5.1 @fortawesome/fontawesome-free@5.14.0
npm install moment@2.27.0 moment-timezone@0.5.31 windows-iana@4.2.1 @azure/msal-browser@2.1.0 @microsoft/microsoft-graph-client@2.0.0 @types/microsoft-graph@1.18.0
```

## <a name="design-the-app"></a>アプリを設計する

最初に、アプリのナビゲーションバーを作成します。

1. という名前のディレクトリに新しいファイルを作成 `./src` `NavBar.tsx` し、次のコードを追加します。

    :::code language="typescript" source="../demo/graph-tutorial/src/NavBar.tsx" id="NavBarSnippet":::

1. アプリのホームページを作成します。 という名前のディレクトリに新しいファイルを作成 `./src` `Welcome.tsx` し、次のコードを追加します。

    :::code language="typescript" source="../demo/graph-tutorial/src/Welcome.tsx" id="WelcomeSnippet":::

1. エラーメッセージ表示を作成して、ユーザーにメッセージを表示します。 という名前のディレクトリに新しいファイルを作成 `./src` `ErrorMessage.tsx` し、次のコードを追加します。

    :::code language="typescript" source="../demo/graph-tutorial/src/ErrorMessage.tsx" id="ErrorMessageSnippet":::

1. `./src/index.css` ファイルを開き、そのコンテンツ全体を次のように置き換えます。

    :::code language="css" source="../demo/graph-tutorial/src/index.css":::

1. `./src/App.tsx`を開き、内容全体を次のように置き換えます。

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

1. すべての変更を保存し、ページを更新します。 この時点で、アプリの外観は大きく異なります。

    ![デザインが変更されたホーム ページのスクリーンショット](images/create-app-01.png)
