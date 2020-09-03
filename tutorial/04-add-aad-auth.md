<!-- markdownlint-disable MD002 MD041 -->

この演習では、Azure AD での認証をサポートするために、前の手順で作成したアプリケーションを拡張します。 これは、Microsoft Graph を呼び出すために必要な OAuth アクセストークンを取得するために必要です。 この手順では、アプリケーションに [Microsoft 認証ライブラリ](https://github.com/AzureAD/microsoft-authentication-library-for-js) ライブラリを統合します。

1. という名前のディレクトリに新しいファイルを作成 `./src` `Config.ts` し、次のコードを追加します。

    :::code language="typescript" source="../demo/graph-tutorial/src/Config.example.ts":::

    を `YOUR_APP_ID_HERE` アプリケーション登録ポータルからのアプリケーション ID に置き換えます。

    > [!IMPORTANT]
    > Git などのソース管理を使用している場合は、この時点で、ソース管理からファイルを除外して、 `Config.ts` アプリ ID が誤ってリークしないようにすることをお勧めします。

## <a name="implement-sign-in"></a>サインインの実装

このセクションでは、認証プロバイダーを作成し、サインインとサインアウトを実装します。

1. という名前のディレクトリに新しいファイルを作成 `./src` `AuthProvider.tsx` し、次のコードを追加します。

    ```typescript
    import React from 'react';
    import { PublicClientApplication } from '@azure/msal-browser';

    import { config } from './Config';

    export interface AuthComponentProps {
      error: any;
      isAuthenticated: boolean;
      user: any;
      login: Function;
      logout: Function;
      getAccessToken: Function;
      setError: Function;
    }

    interface AuthProviderState {
      error: any;
      isAuthenticated: boolean;
      user: any;
    }

    export default function withAuthProvider<T extends React.Component<AuthComponentProps>>
      (WrappedComponent: new(props: AuthComponentProps, context?: any) => T): React.ComponentClass {
      return class extends React.Component<any, AuthProviderState> {
        private publicClientApplication: PublicClientApplication;

        constructor(props: any) {
          super(props);
          this.state = {
            error: null,
            isAuthenticated: false,
            user: {}
          };

          // Initialize the MSAL application object
          this.publicClientApplication = new PublicClientApplication({
            auth: {
                clientId: config.appId,
                redirectUri: config.redirectUri
            },
            cache: {
                cacheLocation: "sessionStorage",
                storeAuthStateInCookie: true
            }
          });
        }

        componentDidMount() {
          // If MSAL already has an account, the user
          // is already logged in
          const accounts = this.publicClientApplication.getAllAccounts();

          if (accounts && accounts.length > 0) {
            // Enhance user object with data from Graph
            this.getUserProfile();
          }
        }

        render() {
          return <WrappedComponent
            error = { this.state.error }
            isAuthenticated = { this.state.isAuthenticated }
            user = { this.state.user }
            login = { () => this.login() }
            logout = { () => this.logout() }
            getAccessToken = { (scopes: string[]) => this.getAccessToken(scopes)}
            setError = { (message: string, debug: string) => this.setErrorMessage(message, debug)}
            {...this.props} {...this.state} />;
        }

        async login() {
          try {
            // Login via popup
            await this.publicClientApplication.loginPopup(
                {
                  scopes: config.scopes,
                  prompt: "select_account"
              });

            // After login, get the user's profile
            await this.getUserProfile();
          }
          catch(err) {
            this.setState({
              isAuthenticated: false,
              user: {},
              error: this.normalizeError(err)
            });
          }
        }

        logout() {
          this.publicClientApplication.logout();
        }

        async getAccessToken(scopes: string[]): Promise<string> {
          try {
            const accounts = this.publicClientApplication
              .getAllAccounts();

            if (accounts.length <= 0) throw new Error('login_required');
            // Get the access token silently
            // If the cache contains a non-expired token, this function
            // will just return the cached token. Otherwise, it will
            // make a request to the Azure OAuth endpoint to get a token
            var silentResult = await this.publicClientApplication
                .acquireTokenSilent({
                  scopes: scopes,
                  account: accounts[0]
                });

            return silentResult.accessToken;
          } catch (err) {
            // If a silent request fails, it may be because the user needs
            // to login or grant consent to one or more of the requested scopes
            if (this.isInteractionRequired(err)) {
              var interactiveResult = await this.publicClientApplication
                  .acquireTokenPopup({
                    scopes: scopes
                  });

              return interactiveResult.accessToken;
            } else {
              throw err;
            }
          }
        }

        async getUserProfile() {
          try {
            var accessToken = await this.getAccessToken(config.scopes);

            if (accessToken) {
              // TEMPORARY: Display the token in the error flash
              this.setState({
                isAuthenticated: true,
                error: { message: "Access token:", debug: accessToken }
              });
            }
          }
          catch(err) {
            this.setState({
              isAuthenticated: false,
              user: {},
              error: this.normalizeError(err)
            });
          }
        }

        setErrorMessage(message: string, debug: string) {
          this.setState({
            error: {message: message, debug: debug}
          });
        }

        normalizeError(error: string | Error): any {
          var normalizedError = {};
          if (typeof(error) === 'string') {
            var errParts = error.split('|');
            normalizedError = errParts.length > 1 ?
              { message: errParts[1], debug: errParts[0] } :
              { message: error };
          } else {
            normalizedError = {
              message: error.message,
              debug: JSON.stringify(error)
            };
          }
          return normalizedError;
        }

        isInteractionRequired(error: Error): boolean {
          if (!error.message || error.message.length <= 0) {
            return false;
          }

          return (
            error.message.indexOf('consent_required') > -1 ||
            error.message.indexOf('interaction_required') > -1 ||
            error.message.indexOf('login_required') > -1 ||
            error.message.indexOf('no_account_in_silent_request') > -1
          );
        }
      }
    }
    ```

1. を開き、 `./src/App.tsx` 次の `import` ステートメントをファイルの先頭に追加します。

    ```typescript
    import withAuthProvider, { AuthComponentProps } from './AuthProvider';
    ```

1. 行を `class App extends Component<any> {` 次のように置き換えます。

    ```typescript
    class App extends Component<AuthComponentProps> {
    ```

1. 行を `export default App;` 次のように置き換えます。

    ```typescript
    export default withAuthProvider(App);
    ```

1. 変更を保存し、ブラウザーを更新します。 [サインイン] ボタンをクリックすると、読み込まれるポップアップウィンドウが表示さ `https://login.microsoftonline.com` れます。 Microsoft アカウントを使用してログインし、要求されたアクセス許可に同意します。 アプリページが更新され、トークンが表示されます。

### <a name="get-user-details"></a>ユーザーの詳細情報を取得する

このセクションでは、Microsoft Graph からユーザーの詳細を取得します。

1. という名前のディレクトリに新しいファイルを作成 `./src` `GraphService.ts` し、次のコードを追加します。

    :::code language="typescript" source="../demo/graph-tutorial/src/GraphService.ts" id="graphServiceSnippet1":::

    これにより、`getUserDetails` 関数が実装されます。これは、Microsoft Graph SDK を使用して `/me` エンドポイントを呼び出して結果を返します。

1. を開き、 `./src/AuthProvider.tsx` 次の `import` ステートメントをファイルの先頭に追加します。

    ```typescript
    import { getUserDetails } from './GraphService';
    ```

1. 既存の `getUserProfile` 関数を、以下のコードで置換します。

    :::code language="typescript" source="../demo/graph-tutorial/src/AuthProvider.tsx" id="getUserProfileSnippet" highlight="6-18":::

1. 変更を保存し、アプリを開始します。サインインした後は、ホームページに戻る必要がありますが、UI はサインインしていることを示すように変更されます。

    ![サインイン後のホーム ページのスクリーンショット](./images/add-aad-auth-01.png)

1. 右上隅にあるユーザーアバターをクリックして、[ **サインアウト** ] リンクにアクセスします。 **[サインアウト]** をクリックすると、セッションがリセットされ、ホーム ページに戻ります。

    ![[サインアウト] リンクのドロップダウン メニューのスクリーンショット](./images/add-aad-auth-02.png)

## <a name="storing-and-refreshing-tokens"></a>トークンの格納と更新

この時点で、アプリケーションには、API 呼び出しのヘッダーで送信されるアクセストークンがあり `Authorization` ます。 これは、アプリがユーザーに代わって Microsoft Graph にアクセスできるようにするトークンです。

ただし、このトークンは一時的なものです。 トークンが発行された後、有効期限が切れる時間になります。 ここで、更新トークンが役に立ちます。 更新トークンを使用すると、ユーザーが再度サインインしなくても、アプリは新しいアクセス トークンを要求できます。

アプリは MSAL ライブラリを使用しているため、トークンストレージを実装したり、ロジックを更新したりする必要はありません。 は、 `PublicClientApplication` ブラウザーセッションでトークンをキャッシュします。 この `acquireTokenSilent` メソッドは、最初にキャッシュされたトークンをチェックし、期限が切れていない場合はそれを返します。 有効期限が切れている場合は、キャッシュされた更新トークンを使用して新しいものを取得します。 このメソッドは、次のモジュールで使用します。
