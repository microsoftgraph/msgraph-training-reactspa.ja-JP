<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="29424-101">この演習では、前の演習のアプリケーションを拡張して、Azure AD での認証をサポートします。</span><span class="sxs-lookup"><span data-stu-id="29424-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="29424-102">これは、Microsoft Graph を呼び出すのに必要な OAuth アクセス トークンを取得するために必要です。</span><span class="sxs-lookup"><span data-stu-id="29424-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="29424-103">この手順では [、Microsoft](https://github.com/AzureAD/microsoft-authentication-library-for-js) 認証ライブラリ ライブラリをアプリケーションに統合します。</span><span class="sxs-lookup"><span data-stu-id="29424-103">In this step you will integrate the [Microsoft Authentication Library](https://github.com/AzureAD/microsoft-authentication-library-for-js) library into the application.</span></span>

1. <span data-ttu-id="29424-104">ディレクトリに新しいファイルを `./src` 作成し、 `Config.ts` 次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="29424-104">Create a new file in the `./src` directory named `Config.ts` and add the following code.</span></span>

    :::code language="typescript" source="../demo/graph-tutorial/src/Config.example.ts":::

    <span data-ttu-id="29424-105">アプリケーション `YOUR_APP_ID_HERE` 登録ポータルのアプリケーション ID に置き換える。</span><span class="sxs-lookup"><span data-stu-id="29424-105">Replace `YOUR_APP_ID_HERE` with the application ID from the Application Registration Portal.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="29424-106">git などのソース管理を使っている場合は、アプリ ID が誤って漏洩しないように、ファイルをソース管理から除外する良い時期です `Config.ts` 。</span><span class="sxs-lookup"><span data-stu-id="29424-106">If you're using source control such as git, now would be a good time to exclude the `Config.ts` file from source control to avoid inadvertently leaking your app ID.</span></span>

## <a name="implement-sign-in"></a><span data-ttu-id="29424-107">サインインの実装</span><span class="sxs-lookup"><span data-stu-id="29424-107">Implement sign-in</span></span>

<span data-ttu-id="29424-108">このセクションでは、認証プロバイダーを作成し、サインインとサインアウトを実装します。</span><span class="sxs-lookup"><span data-stu-id="29424-108">In this section you'll create an authentication provider and implement sign-in and sign-out.</span></span>

1. <span data-ttu-id="29424-109">ディレクトリに新しいファイルを `./src` 作成し、 `AuthProvider.tsx` 次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="29424-109">Create a new file in the `./src` directory named `AuthProvider.tsx` and add the following code.</span></span>

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
      (WrappedComponent: new (props: AuthComponentProps, context?: any) => T): React.ComponentClass {
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
            error={ this.state.error }
            isAuthenticated={ this.state.isAuthenticated }
            user={ this.state.user }
            login={ () => this.login() }
            logout={ () => this.logout() }
            getAccessToken={ (scopes: string[]) => this.getAccessToken(scopes) }
            setError={ (message: string, debug: string) => this.setErrorMessage(message, debug) }
            { ...this.props } />;
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
          catch (err) {
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
            error: { message: message, debug: debug }
          });
        }

        normalizeError(error: string | Error): any {
          var normalizedError = {};
          if (typeof (error) === 'string') {
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

1. <span data-ttu-id="29424-110">次 `./src/App.tsx` のステートメントを `import` 開き、ファイルの一番上に追加します。</span><span class="sxs-lookup"><span data-stu-id="29424-110">Open `./src/App.tsx` and add the following `import` statement to the top of the file.</span></span>

    ```typescript
    import withAuthProvider, { AuthComponentProps } from './AuthProvider';
    ```

1. <span data-ttu-id="29424-111">行を次の `class App extends Component<any> {` 行に置き換える。</span><span class="sxs-lookup"><span data-stu-id="29424-111">Replace the line `class App extends Component<any> {` with the following.</span></span>

    ```typescript
    class App extends Component<AuthComponentProps> {
    ```

1. <span data-ttu-id="29424-112">行を次の `export default App;` 行に置き換える。</span><span class="sxs-lookup"><span data-stu-id="29424-112">Replace the line `export default App;` with the following.</span></span>

    ```typescript
    export default withAuthProvider(App);
    ```

1. <span data-ttu-id="29424-113">変更内容を保存し、ブラウザーを更新します。</span><span class="sxs-lookup"><span data-stu-id="29424-113">Save your changes and refresh the browser.</span></span> <span data-ttu-id="29424-114">[サインイン] ボタンをクリックすると、読み込まれるポップアップ ウィンドウが表示されます `https://login.microsoftonline.com` 。</span><span class="sxs-lookup"><span data-stu-id="29424-114">Click the sign-in button and you should see a pop-up window that loads `https://login.microsoftonline.com`.</span></span> <span data-ttu-id="29424-115">Microsoft アカウントでログインし、要求されたアクセス許可に同意します。</span><span class="sxs-lookup"><span data-stu-id="29424-115">Login with your Microsoft account and consent to the requested permissions.</span></span> <span data-ttu-id="29424-116">アプリ ページが更新され、トークンが表示されます。</span><span class="sxs-lookup"><span data-stu-id="29424-116">The app page should refresh, showing the token.</span></span>

### <a name="get-user-details"></a><span data-ttu-id="29424-117">ユーザーの詳細情報を取得する</span><span class="sxs-lookup"><span data-stu-id="29424-117">Get user details</span></span>

<span data-ttu-id="29424-118">このセクションでは、Microsoft Graph からユーザーの詳細を取得します。</span><span class="sxs-lookup"><span data-stu-id="29424-118">In this section you will get the user's details from Microsoft Graph.</span></span>

1. <span data-ttu-id="29424-119">ディレクトリに新しいファイルを `./src` 作成し、次 `GraphService.ts` のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="29424-119">Create a new file in the `./src` directory called `GraphService.ts` and add the following code.</span></span>

    :::code language="typescript" source="../demo/graph-tutorial/src/GraphService.ts" id="graphServiceSnippet1":::

    <span data-ttu-id="29424-120">これにより、`getUserDetails` 関数が実装されます。これは、Microsoft Graph SDK を使用して `/me` エンドポイントを呼び出して結果を返します。</span><span class="sxs-lookup"><span data-stu-id="29424-120">This implements the `getUserDetails` function, which uses the Microsoft Graph SDK to call the `/me` endpoint and return the result.</span></span>

1. <span data-ttu-id="29424-121">次 `./src/AuthProvider.tsx` のステートメントを `import` 開き、ファイルの一番上に追加します。</span><span class="sxs-lookup"><span data-stu-id="29424-121">Open `./src/AuthProvider.tsx` and add the following `import` statement to the top of the file.</span></span>

    ```typescript
    import { getUserDetails } from './GraphService';
    ```

1. <span data-ttu-id="29424-122">既存の `getUserProfile` 関数を、以下のコードで置換します。</span><span class="sxs-lookup"><span data-stu-id="29424-122">Replace the existing `getUserProfile` function with the following code.</span></span>

    :::code language="typescript" source="../demo/graph-tutorial/src/AuthProvider.tsx" id="getUserProfileSnippet" highlight="6-18":::

1. <span data-ttu-id="29424-123">変更内容を保存し、アプリを起動します。サインイン後にホーム ページに戻る必要がありますが、サインイン中を示す UI が変更される必要があります。</span><span class="sxs-lookup"><span data-stu-id="29424-123">Save your changes and start the app, after sign-in you should end up back on the home page, but the UI should change to indicate that you are signed-in.</span></span>

    ![サインイン後のホーム ページのスクリーンショット](./images/add-aad-auth-01.png)

1. <span data-ttu-id="29424-125">右上隅にあるユーザー アバターをクリックして、[サインアウト **] リンクにアクセス** します。</span><span class="sxs-lookup"><span data-stu-id="29424-125">Click the user avatar in the top right corner to access the **Sign Out** link.</span></span> <span data-ttu-id="29424-126">**[サインアウト]** をクリックすると、セッションがリセットされ、ホーム ページに戻ります。</span><span class="sxs-lookup"><span data-stu-id="29424-126">Clicking **Sign Out** resets the session and returns you to the home page.</span></span>

    ![[サインアウト] リンクのドロップダウン メニューのスクリーンショット](./images/add-aad-auth-02.png)

## <a name="storing-and-refreshing-tokens"></a><span data-ttu-id="29424-128">トークンの保存と更新</span><span class="sxs-lookup"><span data-stu-id="29424-128">Storing and refreshing tokens</span></span>

<span data-ttu-id="29424-129">この時点で、アプリケーションはアクセス トークンを持ち、API 呼び出しのヘッダー `Authorization` で送信されます。</span><span class="sxs-lookup"><span data-stu-id="29424-129">At this point your application has an access token, which is sent in the `Authorization` header of API calls.</span></span> <span data-ttu-id="29424-130">これは、アプリがユーザーの代わりに Microsoft Graph にアクセスできるトークンです。</span><span class="sxs-lookup"><span data-stu-id="29424-130">This is the token that allows the app to access the Microsoft Graph on the user's behalf.</span></span>

<span data-ttu-id="29424-131">ただし、このトークンは一時的なものです。</span><span class="sxs-lookup"><span data-stu-id="29424-131">However, this token is short-lived.</span></span> <span data-ttu-id="29424-132">トークンは発行後 1 時間で期限切れになります。</span><span class="sxs-lookup"><span data-stu-id="29424-132">The token expires an hour after it is issued.</span></span> <span data-ttu-id="29424-133">ここで、更新トークンが役に立ちます。</span><span class="sxs-lookup"><span data-stu-id="29424-133">This is where the refresh token becomes useful.</span></span> <span data-ttu-id="29424-134">更新トークンを使用すると、ユーザーが再度サインインしなくても、アプリは新しいアクセス トークンを要求できます。</span><span class="sxs-lookup"><span data-stu-id="29424-134">The refresh token allows the app to request a new access token without requiring the user to sign in again.</span></span>

<span data-ttu-id="29424-135">アプリは MSAL ライブラリを使うので、トークンストレージや更新ロジックを実装する必要は一切必要ではありません。</span><span class="sxs-lookup"><span data-stu-id="29424-135">Because the app is using the MSAL library, you do not have to implement any token storage or refresh logic.</span></span> <span data-ttu-id="29424-136">トークン `PublicClientApplication` はブラウザー セッションにキャッシュされます。</span><span class="sxs-lookup"><span data-stu-id="29424-136">The `PublicClientApplication` caches the token in the browser session.</span></span> <span data-ttu-id="29424-137">この `acquireTokenSilent` メソッドは最初にキャッシュされたトークンをチェックし、有効期限が切れていない場合はトークンを返します。</span><span class="sxs-lookup"><span data-stu-id="29424-137">The `acquireTokenSilent` method first checks the cached token, and if it is not expired, it returns it.</span></span> <span data-ttu-id="29424-138">有効期限が切れている場合は、キャッシュされた更新トークンを使用して新しい更新トークンを取得します。</span><span class="sxs-lookup"><span data-stu-id="29424-138">If it is expired, it uses the cached refresh token to obtain a new one.</span></span> <span data-ttu-id="29424-139">このメソッドは、次のモジュールでさらに使用します。</span><span class="sxs-lookup"><span data-stu-id="29424-139">You'll use this method more in the following module.</span></span>
