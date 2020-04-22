<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="6b164-101">この演習では、Azure AD での認証をサポートするために、前の手順で作成したアプリケーションを拡張します。</span><span class="sxs-lookup"><span data-stu-id="6b164-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="6b164-102">これは、Microsoft Graph を呼び出すために必要な OAuth アクセストークンを取得するために必要です。</span><span class="sxs-lookup"><span data-stu-id="6b164-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="6b164-103">この手順では、アプリケーションに[Microsoft 認証ライブラリ](https://github.com/AzureAD/microsoft-authentication-library-for-js)ライブラリを統合します。</span><span class="sxs-lookup"><span data-stu-id="6b164-103">In this step you will integrate the [Microsoft Authentication Library](https://github.com/AzureAD/microsoft-authentication-library-for-js) library into the application.</span></span>

1. <span data-ttu-id="6b164-104">という名前`./src` `Config.ts`のディレクトリに新しいファイルを作成し、次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="6b164-104">Create a new file in the `./src` directory named `Config.ts` and add the following code.</span></span>

    :::code language="typescript" source="../demo/graph-tutorial/src/Config.ts.example":::

    <span data-ttu-id="6b164-105">を`YOUR_APP_ID_HERE`アプリケーション登録ポータルからのアプリケーション ID に置き換えます。</span><span class="sxs-lookup"><span data-stu-id="6b164-105">Replace `YOUR_APP_ID_HERE` with the application ID from the Application Registration Portal.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="6b164-106">Git などのソース管理を使用している場合は、この時点で、ソース管理`Config.ts`からファイルを除外して、アプリ ID が誤ってリークしないようにすることをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="6b164-106">If you're using source control such as git, now would be a good time to exclude the `Config.ts` file from source control to avoid inadvertently leaking your app ID.</span></span>

## <a name="implement-sign-in"></a><span data-ttu-id="6b164-107">サインインの実装</span><span class="sxs-lookup"><span data-stu-id="6b164-107">Implement sign-in</span></span>

<span data-ttu-id="6b164-108">このセクションでは、認証プロバイダーを作成し、サインインとサインアウトを実装します。</span><span class="sxs-lookup"><span data-stu-id="6b164-108">In this section you'll create an authentication provider and implement sign-in and sign-out.</span></span>

1. <span data-ttu-id="6b164-109">という名前`./src` `AuthProvider.tsx`のディレクトリに新しいファイルを作成し、次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="6b164-109">Create a new file in the `./src` directory named `AuthProvider.tsx` and add the following code.</span></span>

    ```typescript
    import React from 'react';
    import { UserAgentApplication } from 'msal';

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
        private userAgentApplication: UserAgentApplication;

        constructor(props: any) {
          super(props);
          this.state = {
            error: null,
            isAuthenticated: false,
            user: {}
          };

          // Initialize the MSAL application object
          this.userAgentApplication = new UserAgentApplication({
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
          var account = this.userAgentApplication.getAccount();

          if (account) {
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
            await this.userAgentApplication.loginPopup(
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
          this.userAgentApplication.logout();
        }

        async getAccessToken(scopes: string[]): Promise<string> {
          try {
            // Get the access token silently
            // If the cache contains a non-expired token, this function
            // will just return the cached token. Otherwise, it will
            // make a request to the Azure OAuth endpoint to get a token
            var silentResult = await this.userAgentApplication.acquireTokenSilent({
              scopes: scopes
            });

            return silentResult.accessToken;
          } catch (err) {
            // If a silent request fails, it may be because the user needs
            // to login or grant consent to one or more of the requested scopes
            if (this.isInteractionRequired(err)) {
              var interactiveResult = await this.userAgentApplication.acquireTokenPopup({
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
            error.message.indexOf('login_required') > -1
          );
        }
      }
    }
    ```

1. <span data-ttu-id="6b164-110">を`./src/App.tsx`開き、次`import`のステートメントをファイルの先頭に追加します。</span><span class="sxs-lookup"><span data-stu-id="6b164-110">Open `./src/App.tsx` and add the following `import` statement to the top of the file.</span></span>

    ```typescript
    import withAuthProvider, { AuthComponentProps } from './AuthProvider';
    ```

1. <span data-ttu-id="6b164-111">行`class App extends Component<any> {`を次のように置き換えます。</span><span class="sxs-lookup"><span data-stu-id="6b164-111">Replace the line `class App extends Component<any> {` with the following.</span></span>

    ```typescript
    class App extends Component<AuthComponentProps> {
    ```

1. <span data-ttu-id="6b164-112">行`export default App;`を次のように置き換えます。</span><span class="sxs-lookup"><span data-stu-id="6b164-112">Replace the line `export default App;` with the following.</span></span>

    ```typescript
    export default withAuthProvider(App);
    ```

1. <span data-ttu-id="6b164-113">変更を保存し、ブラウザーを更新します。</span><span class="sxs-lookup"><span data-stu-id="6b164-113">Save your changes and refresh the browser.</span></span> <span data-ttu-id="6b164-114">[サインイン] ボタンをクリックすると、`https://login.microsoftonline.com` にリダイレクトされます。</span><span class="sxs-lookup"><span data-stu-id="6b164-114">Click the sign-in button and you should be redirected to `https://login.microsoftonline.com`.</span></span> <span data-ttu-id="6b164-115">Microsoft アカウントを使用してログインし、要求されたアクセス許可に同意します。</span><span class="sxs-lookup"><span data-stu-id="6b164-115">Login with your Microsoft account and consent to the requested permissions.</span></span> <span data-ttu-id="6b164-116">アプリページが更新され、トークンが表示されます。</span><span class="sxs-lookup"><span data-stu-id="6b164-116">The app page should refresh, showing the token.</span></span>

### <a name="get-user-details"></a><span data-ttu-id="6b164-117">ユーザーの詳細情報を取得する</span><span class="sxs-lookup"><span data-stu-id="6b164-117">Get user details</span></span>

<span data-ttu-id="6b164-118">このセクションでは、Microsoft Graph からユーザーの詳細を取得します。</span><span class="sxs-lookup"><span data-stu-id="6b164-118">In this section you will get the user's details from Microsoft Graph.</span></span>

1. <span data-ttu-id="6b164-119">という名前`./src` `GraphService.ts`のディレクトリに新しいファイルを作成し、次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="6b164-119">Create a new file in the `./src` directory called `GraphService.ts` and add the following code.</span></span>

    :::code language="typescript" source="../demo/graph-tutorial/src/GraphService.ts" id="graphServiceSnippet1":::

    <span data-ttu-id="6b164-120">これにより、`getUserDetails` 関数が実装されます。これは、Microsoft Graph SDK を使用して `/me` エンドポイントを呼び出して結果を返します。</span><span class="sxs-lookup"><span data-stu-id="6b164-120">This implements the `getUserDetails` function, which uses the Microsoft Graph SDK to call the `/me` endpoint and return the result.</span></span>

1. <span data-ttu-id="6b164-121">を`./src/AuthProvider.tsx`開き、次`import`のステートメントをファイルの先頭に追加します。</span><span class="sxs-lookup"><span data-stu-id="6b164-121">Open `./src/AuthProvider.tsx` and add the following `import` statement to the top of the file.</span></span>

    ```typescript
    import { getUserDetails } from './GraphService';
    ```

1. <span data-ttu-id="6b164-122">既存の `getUserProfile` 関数を、以下のコードで置換します。</span><span class="sxs-lookup"><span data-stu-id="6b164-122">Replace the existing `getUserProfile` function with the following code.</span></span>

    :::code language="typescript" source="../demo/graph-tutorial/src/AuthProvider.tsx" id="getUserProfileSnippet" highlight="6-15":::

1. <span data-ttu-id="6b164-123">変更を保存し、アプリを開始します。サインインした後は、ホームページに戻る必要がありますが、UI はサインインしていることを示すように変更されます。</span><span class="sxs-lookup"><span data-stu-id="6b164-123">Save your changes and start the app, after sign-in you should end up back on the home page, but the UI should change to indicate that you are signed-in.</span></span>

    ![サインイン後のホーム ページのスクリーンショット](./images/add-aad-auth-01.png)

1. <span data-ttu-id="6b164-125">右上隅にあるユーザーアバターをクリックして、[**サインアウト**] リンクにアクセスします。</span><span class="sxs-lookup"><span data-stu-id="6b164-125">Click the user avatar in the top right corner to access the **Sign Out** link.</span></span> <span data-ttu-id="6b164-126">**[サインアウト]** をクリックすると、セッションがリセットされ、ホーム ページに戻ります。</span><span class="sxs-lookup"><span data-stu-id="6b164-126">Clicking **Sign Out** resets the session and returns you to the home page.</span></span>

    ![[サインアウト] リンクのドロップダウン メニューのスクリーンショット](./images/add-aad-auth-02.png)

## <a name="storing-and-refreshing-tokens"></a><span data-ttu-id="6b164-128">トークンの格納と更新</span><span class="sxs-lookup"><span data-stu-id="6b164-128">Storing and refreshing tokens</span></span>

<span data-ttu-id="6b164-129">この時点で、アプリケーションには、API 呼び出しの`Authorization`ヘッダーで送信されるアクセストークンがあります。</span><span class="sxs-lookup"><span data-stu-id="6b164-129">At this point your application has an access token, which is sent in the `Authorization` header of API calls.</span></span> <span data-ttu-id="6b164-130">これは、アプリがユーザーに代わって Microsoft Graph にアクセスできるようにするトークンです。</span><span class="sxs-lookup"><span data-stu-id="6b164-130">This is the token that allows the app to access the Microsoft Graph on the user's behalf.</span></span>

<span data-ttu-id="6b164-131">ただし、このトークンは一時的なものです。</span><span class="sxs-lookup"><span data-stu-id="6b164-131">However, this token is short-lived.</span></span> <span data-ttu-id="6b164-132">トークンが発行された後、有効期限が切れる時間になります。</span><span class="sxs-lookup"><span data-stu-id="6b164-132">The token expires an hour after it is issued.</span></span> <span data-ttu-id="6b164-133">ここで、更新トークンが役に立ちます。</span><span class="sxs-lookup"><span data-stu-id="6b164-133">This is where the refresh token becomes useful.</span></span> <span data-ttu-id="6b164-134">更新トークンを使用すると、ユーザーが再度サインインしなくても、アプリは新しいアクセス トークンを要求できます。</span><span class="sxs-lookup"><span data-stu-id="6b164-134">The refresh token allows the app to request a new access token without requiring the user to sign in again.</span></span>

<span data-ttu-id="6b164-135">アプリは MSAL ライブラリを使用しているため、トークンストレージを実装したり、ロジックを更新したりする必要はありません。</span><span class="sxs-lookup"><span data-stu-id="6b164-135">Because the app is using the MSAL library, you do not have to implement any token storage or refresh logic.</span></span> <span data-ttu-id="6b164-136">は`UserAgentApplication` 、ブラウザーセッションでトークンをキャッシュします。</span><span class="sxs-lookup"><span data-stu-id="6b164-136">The `UserAgentApplication` caches the token in the browser session.</span></span> <span data-ttu-id="6b164-137">この`acquireTokenSilent`メソッドは、最初にキャッシュされたトークンをチェックし、期限が切れていない場合はそれを返します。</span><span class="sxs-lookup"><span data-stu-id="6b164-137">The `acquireTokenSilent` method first checks the cached token, and if it is not expired, it returns it.</span></span> <span data-ttu-id="6b164-138">有効期限が切れている場合は、キャッシュされた更新トークンを使用して新しいものを取得します。</span><span class="sxs-lookup"><span data-stu-id="6b164-138">If it is expired, it uses the cached refresh token to obtain a new one.</span></span> <span data-ttu-id="6b164-139">このメソッドは、次のモジュールで使用します。</span><span class="sxs-lookup"><span data-stu-id="6b164-139">You'll use this method more in the following module.</span></span>
