<!-- markdownlint-disable MD002 MD041 -->

この演習では、Azure AD での認証をサポートするために、前の手順で作成したアプリケーションを拡張します。 これは、Microsoft Graph を呼び出すために必要な OAuth アクセストークンを取得するために必要です。 この手順では、アプリケーションに[Microsoft 認証ライブラリ](https://github.com/AzureAD/microsoft-authentication-library-for-js)ライブラリを統合します。

という名前`./src` `Config.js`のディレクトリに新しいファイルを作成し、次のコードを追加します。

```js
module.exports = {
  appId: 'YOUR_APP_ID_HERE',
  scopes: [
    "user.read",
    "calendars.read"
  ]
};
```

を`YOUR_APP_ID_HERE`アプリケーション登録ポータルからのアプリケーション ID に置き換えます。

> [!IMPORTANT]
> Git などのソース管理を使用している場合は、この時点で、ソース管理`Config.js`からファイルを除外して、アプリ ID が誤ってリークしないようにすることをお勧めします。

を`./src/App.js`開き、次`import`のステートメントをファイルの先頭に追加します。

```js
import config from './Config';
import { UserAgentApplication } from 'msal';
```

## <a name="implement-sign-in"></a>サインインの実装

最初`constructor`に、 `App`クラスのを更新して`UserAgentApplication`クラスのインスタンスを作成し、 `getUser`を呼び出して、ログインしているユーザーが既に存在するかどうかを確認します。 既存`constructor`のを次のものに置き換えます。

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

このコードでは`UserAgentApplication` 、アプリケーション ID を使用してクラスを初期化し、ユーザーが存在するかどうかを確認します。 ユーザーがいる場合は、true に`isAuthenticated`設定されます。 この`getUserProfile`メソッドはまだ実装されていません。 これは、後で実装します。

次に、 `App`クラスに関数を追加してログインを行います。 次の関数を `App` クラスに追加します。

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

このメソッドは、 `loginPopup`関数を呼び出してログインを行い、 `getUserProfile`関数を呼び出します。

ここで、ログアウトに関数を追加します。 次の関数を `App` クラスに追加します。

```js
logout() {
  this.userAgentApplication.logout();
}
```

メソッド`login` `logout`とメソッドが実装されたので`NavBar` 、 `Welcome` `render` `App`クラスのメソッドの要素と要素を更新します。 既存`NavBar`の要素を次のように置き換えます。

```JSX
<NavBar
  isAuthenticated={this.state.isAuthenticated}
  authButtonMethod={this.state.isAuthenticated ? this.logout.bind(this) : this.login.bind(this)}
  user={this.state.user}/>
```

既存`Welcome`の要素を次のように置き換えます。

```JSX
<Welcome {...props}
  isAuthenticated={this.state.isAuthenticated}
  user={this.state.user}
  authButtonMethod={this.login.bind(this)} />
```

最後に、 `getUserProfile`関数を実装します。 次の関数を `App` クラスに追加します。

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

このコードは`acquireTokenSilent` 、アクセストークンを取得するための呼び出しを行い、トークンをエラーとして出力するだけです。

変更を保存し、ブラウザーを更新します。 [サインイン] ボタンをクリックすると、に`https://login.microsoftonline.com`リダイレクトされます。 Microsoft アカウントを使用してログインし、要求されたアクセス許可に同意します。 アプリページが更新され、トークンが表示されます。

### <a name="get-user-details"></a>ユーザーの詳細を取得する

最初に、Microsoft Graph のすべての呼び出しを保持する新しいファイルを作成します。 という名前`./src` `GraphService.js`のディレクトリに新しいファイルを作成し、次のコードを追加します。

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

これにより`getUserDetails` 、関数が実装されます。これは、 `/me` Microsoft Graph SDK を使用してエンドポイントを呼び出し、結果を返します。

この関数`getUserProfile`を呼び出す`./src/App.js`には、のメソッドを更新します。 最初に、次`import`のステートメントをファイルの先頭に追加します。

```js
import { getUserDetails } from './GraphService';
```

既存の `getUserProfile` 関数を、以下のコードで置換します。

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

変更を保存してアプリを開始した場合は、サインイン後にホームページに戻る必要がありますが、UI は、サインインしていることを示すように変更する必要があります。

![サインイン後のホームページのスクリーンショット](./images/add-aad-auth-01.png)

右上隅にあるユーザーアバターをクリックして、[**サインアウト**] リンクにアクセスします。 [**サインアウト**] をクリックすると、セッションがリセットされ、ホームページに戻ります。

![[サインアウト] リンクがあるドロップダウンメニューのスクリーンショット](./images/add-aad-auth-02.png)

## <a name="storing-and-refreshing-tokens"></a>トークンの格納と更新

この時点で、アプリケーションには、API 呼び出しの`Authorization`ヘッダーで送信されるアクセストークンがあります。 これは、アプリがユーザーに代わって Microsoft Graph にアクセスできるようにするトークンです。

ただし、このトークンは存続期間が短くなります。 トークンが発行された後、有効期限が切れる時間になります。 ここでは、更新トークンが有効になります。 更新トークンを使用すると、ユーザーが再度サインインする必要なく、新しいアクセストークンをアプリで要求できます。

アプリは MSAL ライブラリを使用しているため、トークンストレージを実装したり、ロジックを更新したりする必要はありません。 この`UserAgentApplication`メソッドは、ブラウザーセッションでトークンをキャッシュします。 この`acquireTokenSilent`メソッドは、最初にキャッシュされたトークンをチェックし、期限が切れていない場合はそれを返します。 有効期限が切れている場合は、キャッシュされた更新トークンを使用して新しいものを取得します。 このメソッドは、次のモジュールで使用します。
