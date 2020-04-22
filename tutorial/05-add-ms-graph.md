<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="6b276-101">この演習では、Microsoft Graph をアプリケーションに組み込みます。</span><span class="sxs-lookup"><span data-stu-id="6b276-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span> <span data-ttu-id="6b276-102">このアプリケーションでは、microsoft graph[クライアント](https://github.com/microsoftgraph/msgraph-sdk-javascript)ライブラリを使用して microsoft graph への呼び出しを行います。</span><span class="sxs-lookup"><span data-stu-id="6b276-102">For this application, you will use the [microsoft-graph-client](https://github.com/microsoftgraph/msgraph-sdk-javascript) library to make calls to Microsoft Graph.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="6b276-103">Outlook からカレンダー イベントを取得する</span><span class="sxs-lookup"><span data-stu-id="6b276-103">Get calendar events from Outlook</span></span>

1. <span data-ttu-id="6b276-104">を`./src/GraphService.ts`開き、次の関数を追加します。</span><span class="sxs-lookup"><span data-stu-id="6b276-104">Open `./src/GraphService.ts` and add the following function.</span></span>

    :::code language="typescript" source="../demo/graph-tutorial/src/GraphService.ts" id="getEventsSnippet":::

    <span data-ttu-id="6b276-105">このコードの実行内容を考えましょう。</span><span class="sxs-lookup"><span data-stu-id="6b276-105">Consider what this code is doing.</span></span>

    - <span data-ttu-id="6b276-106">呼び出される URL は `/me/events` です。</span><span class="sxs-lookup"><span data-stu-id="6b276-106">The URL that will be called is `/me/events`.</span></span>
    - <span data-ttu-id="6b276-107">この`select`メソッドは、各イベントに対して返されるフィールドを、ビューが実際に使用するものだけに制限します。</span><span class="sxs-lookup"><span data-stu-id="6b276-107">The `select` method limits the fields returned for each events to just those the view will actually use.</span></span>
    - <span data-ttu-id="6b276-108">メソッド`orderby`は、生成された日付と時刻で結果を並べ替えます。最新のアイテムが最初に表示されます。</span><span class="sxs-lookup"><span data-stu-id="6b276-108">The `orderby` method sorts the results by the date and time they were created, with the most recent item being first.</span></span>

1. <span data-ttu-id="6b276-109">応答コンポーネントを作成して、呼び出しの結果を表示します。</span><span class="sxs-lookup"><span data-stu-id="6b276-109">Create a React component to display the results of the call.</span></span> <span data-ttu-id="6b276-110">という名前`./src` `Calendar.tsx`のディレクトリに新しいファイルを作成し、次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="6b276-110">Create a new file in the `./src` directory named `Calendar.tsx` and add the following code.</span></span>

    ```typescript
    import React from 'react';
    import { Table } from 'reactstrap';
    import moment from 'moment';
    import { Event } from 'microsoft-graph';
    import { config } from './Config';
    import { getEvents } from './GraphService';
    import withAuthProvider, { AuthComponentProps } from './AuthProvider';

    interface CalendarState {
      events: Event[];
    }

    // Helper function to format Graph date/time
    function formatDateTime(dateTime: string | undefined) {
      if (dateTime !== undefined) {
        return moment.utc(dateTime).local().format('M/D/YY h:mm A');
      }
    }

    class Calendar extends React.Component<AuthComponentProps, CalendarState> {
      constructor(props: any) {
        super(props);

        this.state = {
          events: []
        };
      }

      async componentDidMount() {
        try {
          // Get the user's access token
          var accessToken = await this.props.getAccessToken(config.scopes);
          // Get the user's events
          var events = await getEvents(accessToken);
          // Update the array of events in state
          this.setState({events: events.value});
        }
        catch(err) {
          this.props.setError('ERROR', JSON.stringify(err));
        }
      }

      render() {
        return (
          <pre><code>{JSON.stringify(this.state.events, null, 2)}</code></pre>
        );
      }
    }

    export default withAuthProvider(Calendar);
    ```

    <span data-ttu-id="6b276-111">ここでは、JSON のイベントの配列をページにレンダリングするだけです。</span><span class="sxs-lookup"><span data-stu-id="6b276-111">For now this just renders the array of events in JSON on the page.</span></span>

1. <span data-ttu-id="6b276-112">この新しいコンポーネントをアプリに追加します。</span><span class="sxs-lookup"><span data-stu-id="6b276-112">Add this new component to the app.</span></span> <span data-ttu-id="6b276-113">を`./src/App.tsx`開き、次`import`のステートメントをファイルの先頭に追加します。</span><span class="sxs-lookup"><span data-stu-id="6b276-113">Open `./src/App.tsx` and add the following `import` statement to the top of the file.</span></span>

    ```typescript
    import Calendar from './Calendar';
    ```

1. <span data-ttu-id="6b276-114">次のコンポーネントを既存`<Route>`のの直後に追加します。</span><span class="sxs-lookup"><span data-stu-id="6b276-114">Add the following component just after the existing `<Route>`.</span></span>

    ```typescript
    <Route exact path="/calendar"
      render={(props) =>
        this.props.isAuthenticated ?
          <Calendar {...props} /> :
          <Redirect to="/" />
      } />
    ```

1. <span data-ttu-id="6b276-115">変更内容を保存し、アプリを再起動します。</span><span class="sxs-lookup"><span data-stu-id="6b276-115">Save your changes and restart the app.</span></span> <span data-ttu-id="6b276-116">サインインして、ナビゲーションバーの [**予定表**] リンクをクリックします。</span><span class="sxs-lookup"><span data-stu-id="6b276-116">Sign in and click the **Calendar** link in the nav bar.</span></span> <span data-ttu-id="6b276-117">すべてが正常に機能していれば、ユーザーのカレンダーにイベントの JSON ダンプが表示されます。</span><span class="sxs-lookup"><span data-stu-id="6b276-117">If everything works, you should see a JSON dump of events on the user's calendar.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="6b276-118">結果の表示</span><span class="sxs-lookup"><span data-stu-id="6b276-118">Display the results</span></span>

<span data-ttu-id="6b276-119">これで、 `Calendar`コンポーネントを更新して、よりわかりやすい方法でイベントを表示することができます。</span><span class="sxs-lookup"><span data-stu-id="6b276-119">Now you can update the `Calendar` component to display the events in a more user-friendly manner.</span></span>

1. <span data-ttu-id="6b276-120">既存`render`の関数`./src/Calendar.js`を次の関数に置き換えます。</span><span class="sxs-lookup"><span data-stu-id="6b276-120">Replace the existing `render` function in `./src/Calendar.js` with the following function.</span></span>

    :::code language="typescript" source="../demo/graph-tutorial/src/Calendar.tsx" id="renderSnippet":::

    <span data-ttu-id="6b276-121">これにより、イベントのコレクションがループ処理され、テーブル行が1つずつ追加されます。</span><span class="sxs-lookup"><span data-stu-id="6b276-121">This loops through the collection of events and adds a table row for each one.</span></span>

1. <span data-ttu-id="6b276-122">変更を保存し、アプリを再起動します。</span><span class="sxs-lookup"><span data-stu-id="6b276-122">Save the changes and restart the app.</span></span> <span data-ttu-id="6b276-123">[**予定表**] リンクをクリックすると、アプリがイベントの表を表示するようになります。</span><span class="sxs-lookup"><span data-stu-id="6b276-123">Click on the **Calendar** link and the app should now render a table of events.</span></span>

    ![イベント表のスクリーンショット](./images/add-msgraph-01.png)
