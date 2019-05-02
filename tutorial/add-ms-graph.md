<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="37875-101">この演習では、Microsoft Graph をアプリケーションに組み込みます。</span><span class="sxs-lookup"><span data-stu-id="37875-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span> <span data-ttu-id="37875-102">このアプリケーションでは、microsoft graph[クライアント](https://github.com/microsoftgraph/msgraph-sdk-javascript)ライブラリを使用して microsoft Graph への呼び出しを行います。</span><span class="sxs-lookup"><span data-stu-id="37875-102">For this application, you will use the [microsoft-graph-client](https://github.com/microsoftgraph/msgraph-sdk-javascript) library to make calls to Microsoft Graph.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="37875-103">Outlook から予定表のイベントを取得する</span><span class="sxs-lookup"><span data-stu-id="37875-103">Get calendar events from Outlook</span></span>

<span data-ttu-id="37875-104">最初に、新しいメソッドを`./src/GraphService.js`ファイルに追加して、予定表からイベントを取得します。</span><span class="sxs-lookup"><span data-stu-id="37875-104">Start by adding a new method to the `./src/GraphService.js` file to get the events from the calendar.</span></span> <span data-ttu-id="37875-105">次の関数を追加します。</span><span class="sxs-lookup"><span data-stu-id="37875-105">Add the following function.</span></span>

```js
export async function getEvents(accessToken) {
  const client = getAuthenticatedClient(accessToken);

  const events = await client
    .api('/me/events')
    .select('subject,organizer,start,end')
    .orderby('createdDateTime DESC')
    .get();

  return events;
}
```

<span data-ttu-id="37875-106">このコードの内容を検討してください。</span><span class="sxs-lookup"><span data-stu-id="37875-106">Consider what this code is doing.</span></span>

- <span data-ttu-id="37875-107">呼び出し先の URL は`/me/events`になります。</span><span class="sxs-lookup"><span data-stu-id="37875-107">The URL that will be called is `/me/events`.</span></span>
- <span data-ttu-id="37875-108">この`select`メソッドは、各イベントに対して返されるフィールドを、ビューが実際に使用するものだけに制限します。</span><span class="sxs-lookup"><span data-stu-id="37875-108">The `select` method limits the fields returned for each events to just those the view will actually use.</span></span>
- <span data-ttu-id="37875-109">メソッド`orderby`は、生成された日付と時刻で結果を並べ替えます。最新のアイテムが最初に表示されます。</span><span class="sxs-lookup"><span data-stu-id="37875-109">The `orderby` method sorts the results by the date and time they were created, with the most recent item being first.</span></span>

<span data-ttu-id="37875-110">次に、応答コンポーネントを作成して、呼び出しの結果を表示します。</span><span class="sxs-lookup"><span data-stu-id="37875-110">Now create a React component to display the results of the call.</span></span> <span data-ttu-id="37875-111">という名前`./src` `Calendar.js`のディレクトリに新しいファイルを作成し、次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="37875-111">Create a new file in the `./src` directory named `Calendar.js` and add the following code.</span></span>

```JSX
import React from 'react';
import { Table } from 'reactstrap';
import moment from 'moment';
import config from './Config';
import { getEvents } from './GraphService';

// Helper function to format Graph date/time
function formatDateTime(dateTime) {
  return moment.utc(dateTime).local().format('M/D/YY h:mm A');
}

export default class Calendar extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      events: []
    };
  }

  async componentDidMount() {
    try {
      // Get the user's access token
      var accessToken = await window.msal.acquireTokenSilent(config.scopes);
      // Get the user's events
      var events = await getEvents(accessToken);
      // Update the array of events in state
      this.setState({events: events.value});
    }
    catch(err) {
      this.props.showError('ERROR', JSON.stringify(err));
    }
  }

  render() {
    return (
      <pre><code>{JSON.stringify(this.state.events, null, 2)}</code></pre>
    );
  }
}
```

<span data-ttu-id="37875-112">ここでは、JSON のイベントの配列をページにレンダリングするだけです。</span><span class="sxs-lookup"><span data-stu-id="37875-112">For now this just renders the array of events in JSON on the page.</span></span> <span data-ttu-id="37875-113">この新しいコンポーネントをアプリに追加します。</span><span class="sxs-lookup"><span data-stu-id="37875-113">Add this new component to the app.</span></span> <span data-ttu-id="37875-114">を`./src/App.js`開き、次`import`のステートメントをファイルの先頭に追加します。</span><span class="sxs-lookup"><span data-stu-id="37875-114">Open `./src/App.js` and add the following `import` statement to the top of the file.</span></span>

```js
import Calendar from './Calendar';
```

<span data-ttu-id="37875-115">次に、既存`<Route>`ののすぐ後に次のコンポーネントを追加します。</span><span class="sxs-lookup"><span data-stu-id="37875-115">Then add the following component just after the existing `<Route>`.</span></span>

```JSX
<Route exact path="/calendar"
  render={(props) =>
    <Calendar {...props}
      showError={this.setErrorMessage.bind(this)} />
  } />
```

<span data-ttu-id="37875-116">変更内容を保存し、アプリを再起動します。</span><span class="sxs-lookup"><span data-stu-id="37875-116">Save your changes and restart the app.</span></span> <span data-ttu-id="37875-117">サインインして、ナビゲーションバーの [**予定表**] リンクをクリックします。</span><span class="sxs-lookup"><span data-stu-id="37875-117">Sign in and click the **Calendar** link in the nav bar.</span></span> <span data-ttu-id="37875-118">すべてが動作する場合は、ユーザーの予定表にイベントの JSON ダンプが表示されます。</span><span class="sxs-lookup"><span data-stu-id="37875-118">If everything works, you should see a JSON dump of events on the user's calendar.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="37875-119">結果を表示する</span><span class="sxs-lookup"><span data-stu-id="37875-119">Display the results</span></span>

<span data-ttu-id="37875-120">これで、 `Calendar`コンポーネントを更新して、よりわかりやすい方法でイベントを表示することができます。</span><span class="sxs-lookup"><span data-stu-id="37875-120">Now you can update the `Calendar` component to display the events in a more user-friendly manner.</span></span> <span data-ttu-id="37875-121">既存`render`の関数`./src/Calendar.js`を次の関数に置き換えます。</span><span class="sxs-lookup"><span data-stu-id="37875-121">Replace the existing `render` function in `./src/Calendar.js` with the following function.</span></span>

```JSX
render() {
  return (
    <div>
      <h1>Calendar</h1>
      <Table>
        <thead>
          <tr>
            <th scope="col">Organizer</th>
            <th scope="col">Subject</th>
            <th scope="col">Start</th>
            <th scope="col">End</th>
          </tr>
        </thead>
        <tbody>
          {this.state.events.map(
            function(event, index){
              return(
                <tr>
                  <td>{event.organizer.emailAddress.name}</td>
                  <td>{event.subject}</td>
                  <td>{formatDateTime(event.start.dateTime)}</td>
                  <td>{formatDateTime(event.end.dateTime)}</td>
                </tr>
              );
            })}
        </tbody>
      </Table>
    </div>
  );
}
```

<span data-ttu-id="37875-122">これにより、イベントのコレクションがループ処理され、テーブル行が1つずつ追加されます。</span><span class="sxs-lookup"><span data-stu-id="37875-122">This loops through the collection of events and adds a table row for each one.</span></span> <span data-ttu-id="37875-123">変更を保存し、アプリを再起動します。</span><span class="sxs-lookup"><span data-stu-id="37875-123">Save the changes and restart the app.</span></span> <span data-ttu-id="37875-124">[**予定表**] リンクをクリックすると、アプリがイベントの表を表示するようになります。</span><span class="sxs-lookup"><span data-stu-id="37875-124">Click on the **Calendar** link and the app should now render a table of events.</span></span>

![イベントの表のスクリーンショット](./images/add-msgraph-01.png)