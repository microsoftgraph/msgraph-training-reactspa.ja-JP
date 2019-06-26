<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="4f88b-101">この演習では、Azure AD での認証をサポートするために、前の手順で作成したアプリケーションを拡張します。</span><span class="sxs-lookup"><span data-stu-id="4f88b-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="4f88b-102">これは、Microsoft Graph を呼び出すために必要な OAuth アクセストークンを取得するために必要です。</span><span class="sxs-lookup"><span data-stu-id="4f88b-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="4f88b-103">この手順では、アプリケーションに[Microsoft 認証ライブラリ](https://github.com/AzureAD/microsoft-authentication-library-for-js)ライブラリを統合します。</span><span class="sxs-lookup"><span data-stu-id="4f88b-103">In this step you will integrate the [Microsoft Authentication Library](https://github.com/AzureAD/microsoft-authentication-library-for-js) library into the application.</span></span>

<span data-ttu-id="4f88b-104">という名前`./src` `Config.js`のディレクトリに新しいファイルを作成し、次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="4f88b-104">Create a new file in the `./src` directory named `Config.js` and add the following code.</span></span>

```js
module.exports = {
  appId: 'YOUR_APP_ID_HERE',
  scopes: [
    "user.read",
    "calendars.read"
  ]
};
```

<span data-ttu-id="4f88b-105">を`YOUR_APP_ID_HERE`アプリケーション登録ポータルからのアプリケーション ID に置き換えます。</span><span class="sxs-lookup"><span data-stu-id="4f88b-105">Replace `YOUR_APP_ID_HERE` with the application ID from the Application Registration Portal.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="4f88b-106">Git などのソース管理を使用している場合は、この時点で、ソース管理`Config.js`からファイルを除外して、アプリ ID が誤ってリークしないようにすることをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="4f88b-106">If you're using source control such as git, now would be a good time to exclude the `Config.js` file from source control to avoid inadvertently leaking your app ID.</span></span>

<span data-ttu-id="4f88b-107">を`./src/App.js`開き、次`import`のステートメントをファイルの先頭に追加します。</span><span class="sxs-lookup"><span data-stu-id="4f88b-107">Open `./src/App.js` and add the following `import` statements to the top of the file.</span></span>

```js
import config from './Config';
import { UserAgentApplication } from 'msal';
```

## <a name="implement-sign-in"></a><span data-ttu-id="4f88b-108">サインインの実装</span><span class="sxs-lookup"><span data-stu-id="4f88b-108">Implement sign-in</span></span>

<span data-ttu-id="4f88b-109">最初`constructor`に、 `App`クラスのを更新して`UserAgentApplication`クラスのインスタンスを作成し、 `getUser`を呼び出して、ログインしているユーザーが既に存在するかどうかを確認します。</span><span class="sxs-lookup"><span data-stu-id="4f88b-109">Start by updating the `constructor` for the `App` class to create an instance of the `UserAgentApplication` class and call `getUser` to see if there's already a logged-in user.</span></span> <span data-ttu-id="4f88b-110">既存`constructor`のを次のものに置き換えます。</span><span class="sxs-lookup"><span data-stu-id="4f88b-110">Replace the existing `constructor` with the following.</span></span>

```js
constructor(props) {
  super(props);

  this.userAgentApplication = new UserAgentApplication({
        auth: {
            clientId: config.appId
        },
        cache: {
            cacheLocation: "localStorage",
            storeAuthStateInCookie: true
        }
    });

  var user = this.userAgentApplication.getAccount();

  this.state = {
    isAuthenticated: (user !== null),
    user: {},
    error: null
  };

  if (user) {
    // Enhance user object with data from Graph
    this.getUserProfile();
  }
}
```

<span data-ttu-id="4f88b-111">このコードでは`UserAgentApplication` 、アプリケーション ID を使用してクラスを初期化し、ユーザーが存在するかどうかを確認します。</span><span class="sxs-lookup"><span data-stu-id="4f88b-111">This code initializes the `UserAgentApplication` class with your application ID and checks for the presence of a user.</span></span> <span data-ttu-id="4f88b-112">ユーザーがいる場合は、true に`isAuthenticated`設定されます。</span><span class="sxs-lookup"><span data-stu-id="4f88b-112">If there is a user, it sets `isAuthenticated` to true.</span></span> <span data-ttu-id="4f88b-113">この`getUserProfile`メソッドはまだ実装されていません。</span><span class="sxs-lookup"><span data-stu-id="4f88b-113">The `getUserProfile` method isn't implemented yet.</span></span> <span data-ttu-id="4f88b-114">これは、後で実装します。</span><span class="sxs-lookup"><span data-stu-id="4f88b-114">You will implement this a bit later.</span></span>

<span data-ttu-id="4f88b-115">次に、 `App`クラスに関数を追加してログインを行います。</span><span class="sxs-lookup"><span data-stu-id="4f88b-115">Next, add a function to the `App` class to do the login.</span></span> <span data-ttu-id="4f88b-116">次の関数を `App` クラスに追加します。</span><span class="sxs-lookup"><span data-stu-id="4f88b-116">Add the following function to the `App` class.</span></span>

```js
async login() {
  try {
    await this.userAgentApplication.loginPopup(
        {
          scopes: config.scopes,
          prompt: "select_account"
      });
    await this.getUserProfile();
  }
  catch(err) {
    var errParts = err.split('|');
    this.setState({
      isAuthenticated: false,
      user: {},
      error: { message: errParts[1], debug: errParts[0] }
    });
  }
}
```

<span data-ttu-id="4f88b-117">このメソッドは、 `loginPopup`関数を呼び出してログインを行い、 `getUserProfile`関数を呼び出します。</span><span class="sxs-lookup"><span data-stu-id="4f88b-117">This method calls the `loginPopup` function to do the login, then calls the `getUserProfile` function.</span></span>

<span data-ttu-id="4f88b-118">ここで、ログアウトに関数を追加します。</span><span class="sxs-lookup"><span data-stu-id="4f88b-118">Now add a function to logout.</span></span> <span data-ttu-id="4f88b-119">次の関数を `App` クラスに追加します。</span><span class="sxs-lookup"><span data-stu-id="4f88b-119">Add the following function to the `App` class.</span></span>

```js
logout() {
  this.userAgentApplication.logout();
}
```

<span data-ttu-id="4f88b-120">メソッド`login` `logout`とメソッドが実装されたので`NavBar` 、 `Welcome` `render` `App`クラスのメソッドの要素と要素を更新します。</span><span class="sxs-lookup"><span data-stu-id="4f88b-120">Now that the `login` and `logout` methods are implemented, update the `NavBar` and `Welcome` elements in the `render` method of the `App` class.</span></span> <span data-ttu-id="4f88b-121">既存`NavBar`の要素を次のように置き換えます。</span><span class="sxs-lookup"><span data-stu-id="4f88b-121">Replace the existing `NavBar` element with the following.</span></span>

```JSX
<NavBar
  isAuthenticated={this.state.isAuthenticated}
  authButtonMethod={this.state.isAuthenticated ? this.logout.bind(this) : this.login.bind(this)}
  user={this.state.user}/>
```

<span data-ttu-id="4f88b-122">既存`Welcome`の要素を次のように置き換えます。</span><span class="sxs-lookup"><span data-stu-id="4f88b-122">Replace the existing `Welcome` element with the following.</span></span>

```JSX
<Welcome {...props}
  isAuthenticated={this.state.isAuthenticated}
  user={this.state.user}
  authButtonMethod={this.login.bind(this)} />
```

<span data-ttu-id="4f88b-123">最後に、 `getUserProfile`関数を実装します。</span><span class="sxs-lookup"><span data-stu-id="4f88b-123">Finally, implement the `getUserProfile` function.</span></span> <span data-ttu-id="4f88b-124">次の関数を `App` クラスに追加します。</span><span class="sxs-lookup"><span data-stu-id="4f88b-124">Add the following function to the `App` class.</span></span>

```js
async getUserProfile() {
  try {
    // Get the access token silently
    // If the cache contains a non-expired token, this function
    // will just return the cached token. Otherwise, it will
    // make a request to the Azure OAuth endpoint to get a token

    var accessToken = await this.userAgentApplication.acquireTokenSilent({
        scopes: config.scopes
      });

    if (accessToken) {
      // TEMPORARY: Display the token in the error flash
      this.setState({
        isAuthenticated: true,
        error: { message: "Access token:", debug: accessToken.accessToken }
      });
    }
  }
  catch(err) {
    var errParts = err.split('|');
    this.setState({
      isAuthenticated: false,
      user: {},
      error: { message: errParts[1], debug: errParts[0] }
    });
  }
}
```

<span data-ttu-id="4f88b-125">このコードは`acquireTokenSilent` 、アクセストークンを取得するための呼び出しを行い、トークンをエラーとして出力するだけです。</span><span class="sxs-lookup"><span data-stu-id="4f88b-125">This code calls `acquireTokenSilent` to get an access token, then just outputs the token as an error.</span></span>

<span data-ttu-id="4f88b-126">変更を保存し、ブラウザーを更新します。</span><span class="sxs-lookup"><span data-stu-id="4f88b-126">Save your changes and refresh the browser.</span></span> <span data-ttu-id="4f88b-127">[サインイン] ボタンをクリックすると、に`https://login.microsoftonline.com`リダイレクトされます。</span><span class="sxs-lookup"><span data-stu-id="4f88b-127">Click the sign-in button and you should be redirected to `https://login.microsoftonline.com`.</span></span> <span data-ttu-id="4f88b-128">Microsoft アカウントを使用してログインし、要求されたアクセス許可に同意します。</span><span class="sxs-lookup"><span data-stu-id="4f88b-128">Login with your Microsoft account and consent to the requested permissions.</span></span> <span data-ttu-id="4f88b-129">アプリページが更新され、トークンが表示されます。</span><span class="sxs-lookup"><span data-stu-id="4f88b-129">The app page should refresh, showing the token.</span></span>

### <a name="get-user-details"></a><span data-ttu-id="4f88b-130">ユーザーの詳細を取得する</span><span class="sxs-lookup"><span data-stu-id="4f88b-130">Get user details</span></span>

<span data-ttu-id="4f88b-131">最初に、Microsoft Graph のすべての呼び出しを保持する新しいファイルを作成します。</span><span class="sxs-lookup"><span data-stu-id="4f88b-131">Start by creating a new file to hold all of your Microsoft Graph calls.</span></span> <span data-ttu-id="4f88b-132">という名前`./src` `GraphService.js`のディレクトリに新しいファイルを作成し、次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="4f88b-132">Create a new file in the `./src` directory called `GraphService.js` and add the following code.</span></span>

```js
var graph = require('@microsoft/microsoft-graph-client');

function getAuthenticatedClient(accessToken) {
  // Initialize Graph client
  const client = graph.Client.init({
    // Use the provided access token to authenticate
    // requests
    authProvider: (done) => {
      done(null, accessToken.accessToken);
    }
  });

  return client;
}

export async function getUserDetails(accessToken) {
  const client = getAuthenticatedClient(accessToken);

  const user = await client.api('/me').get();
  return user;
}
```

<span data-ttu-id="4f88b-133">これにより`getUserDetails` 、関数が実装されます。これは、 `/me` Microsoft Graph SDK を使用してエンドポイントを呼び出し、結果を返します。</span><span class="sxs-lookup"><span data-stu-id="4f88b-133">This implements the `getUserDetails` function, which uses the Microsoft Graph SDK to call the `/me` endpoint and return the result.</span></span>

<span data-ttu-id="4f88b-134">この関数`getUserProfile`を呼び出す`./src/App.js`には、のメソッドを更新します。</span><span class="sxs-lookup"><span data-stu-id="4f88b-134">Update the `getUserProfile` method in `./src/App.js` to call this function.</span></span> <span data-ttu-id="4f88b-135">最初に、次`import`のステートメントをファイルの先頭に追加します。</span><span class="sxs-lookup"><span data-stu-id="4f88b-135">First, add the following `import` statement to the top of the file.</span></span>

```js
import { getUserDetails } from './GraphService';
```

<span data-ttu-id="4f88b-136">既存の `getUserProfile` 関数を、以下のコードで置換します。</span><span class="sxs-lookup"><span data-stu-id="4f88b-136">Replace the existing `getUserProfile` function with the following code.</span></span>

```js
async getUserProfile() {
  try {
    // Get the access token silently
    // If the cache contains a non-expired token, this function
    // will just return the cached token. Otherwise, it will
    // make a request to the Azure OAuth endpoint to get a token

    var accessToken = await this.userAgentApplication.acquireTokenSilent({
      scopes: config.scopes
    });

    if (accessToken) {
      // Get the user's profile from Graph
      var user = await getUserDetails(accessToken);
      this.setState({
        isAuthenticated: true,
        user: {
          displayName: user.displayName,
          email: user.mail || user.userPrincipalName
        },
        error: null
      });
    }
  }
  catch(err) {
    var error = {};
    if (typeof(err) === 'string') {
      var errParts = err.split('|');
      error = errParts.length > 1 ?
        { message: errParts[1], debug: errParts[0] } :
        { message: err };
    } else {
      error = {
        message: err.message,
        debug: JSON.stringify(err)
      };
    }

    this.setState({
      isAuthenticated: false,
      user: {},
      error: error
    });
  }
}
```

<span data-ttu-id="4f88b-137">変更を保存してアプリを開始した場合は、サインイン後にホームページに戻る必要がありますが、UI は、サインインしていることを示すように変更する必要があります。</span><span class="sxs-lookup"><span data-stu-id="4f88b-137">Now if you save your changes and start the app, after sign-in you should end up back on the home page, but the UI should change to indicate that you are signed-in.</span></span>

![サインイン後のホームページのスクリーンショット](./images/add-aad-auth-01.png)

<span data-ttu-id="4f88b-139">右上隅にあるユーザーアバターをクリックして、[**サインアウト**] リンクにアクセスします。</span><span class="sxs-lookup"><span data-stu-id="4f88b-139">Click the user avatar in the top right corner to access the **Sign Out** link.</span></span> <span data-ttu-id="4f88b-140">[**サインアウト**] をクリックすると、セッションがリセットされ、ホームページに戻ります。</span><span class="sxs-lookup"><span data-stu-id="4f88b-140">Clicking **Sign Out** resets the session and returns you to the home page.</span></span>

![[サインアウト] リンクがあるドロップダウンメニューのスクリーンショット](./images/add-aad-auth-02.png)

## <a name="storing-and-refreshing-tokens"></a><span data-ttu-id="4f88b-142">トークンの格納と更新</span><span class="sxs-lookup"><span data-stu-id="4f88b-142">Storing and refreshing tokens</span></span>

<span data-ttu-id="4f88b-143">この時点で、アプリケーションには、API 呼び出しの`Authorization`ヘッダーで送信されるアクセストークンがあります。</span><span class="sxs-lookup"><span data-stu-id="4f88b-143">At this point your application has an access token, which is sent in the `Authorization` header of API calls.</span></span> <span data-ttu-id="4f88b-144">これは、アプリがユーザーに代わって Microsoft Graph にアクセスできるようにするトークンです。</span><span class="sxs-lookup"><span data-stu-id="4f88b-144">This is the token that allows the app to access the Microsoft Graph on the user's behalf.</span></span>

<span data-ttu-id="4f88b-145">ただし、このトークンは存続期間が短くなります。</span><span class="sxs-lookup"><span data-stu-id="4f88b-145">However, this token is short-lived.</span></span> <span data-ttu-id="4f88b-146">トークンが発行された後、有効期限が切れる時間になります。</span><span class="sxs-lookup"><span data-stu-id="4f88b-146">The token expires an hour after it is issued.</span></span> <span data-ttu-id="4f88b-147">ここでは、更新トークンが有効になります。</span><span class="sxs-lookup"><span data-stu-id="4f88b-147">This is where the refresh token becomes useful.</span></span> <span data-ttu-id="4f88b-148">更新トークンを使用すると、ユーザーが再度サインインする必要なく、新しいアクセストークンをアプリで要求できます。</span><span class="sxs-lookup"><span data-stu-id="4f88b-148">The refresh token allows the app to request a new access token without requiring the user to sign in again.</span></span>

<span data-ttu-id="4f88b-149">アプリは MSAL ライブラリを使用しているため、トークンストレージを実装したり、ロジックを更新したりする必要はありません。</span><span class="sxs-lookup"><span data-stu-id="4f88b-149">Because the app is using the MSAL library, you do not have to implement any token storage or refresh logic.</span></span> <span data-ttu-id="4f88b-150">この`UserAgentApplication`メソッドは、ブラウザーセッションでトークンをキャッシュします。</span><span class="sxs-lookup"><span data-stu-id="4f88b-150">The `UserAgentApplication` method caches the token in the browser session.</span></span> <span data-ttu-id="4f88b-151">この`acquireTokenSilent`メソッドは、最初にキャッシュされたトークンをチェックし、期限が切れていない場合はそれを返します。</span><span class="sxs-lookup"><span data-stu-id="4f88b-151">The `acquireTokenSilent` method first checks the cached token, and if it is not expired, it returns it.</span></span> <span data-ttu-id="4f88b-152">有効期限が切れている場合は、キャッシュされた更新トークンを使用して新しいものを取得します。</span><span class="sxs-lookup"><span data-stu-id="4f88b-152">If it is expired, it uses the cached refresh token to obtain a new one.</span></span> <span data-ttu-id="4f88b-153">このメソッドは、次のモジュールで使用します。</span><span class="sxs-lookup"><span data-stu-id="4f88b-153">You'll use this method more in the following module.</span></span>
