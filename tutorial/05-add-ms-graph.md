<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="6d306-101">この演習では、Microsoft Graph をアプリケーションに組み込みます。</span><span class="sxs-lookup"><span data-stu-id="6d306-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span> <span data-ttu-id="6d306-102">このアプリケーションでは、microsoft graph [クライアント](https://github.com/microsoftgraph/msgraph-sdk-javascript) ライブラリを使用して microsoft graph への呼び出しを行います。</span><span class="sxs-lookup"><span data-stu-id="6d306-102">For this application, you will use the [microsoft-graph-client](https://github.com/microsoftgraph/msgraph-sdk-javascript) library to make calls to Microsoft Graph.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="6d306-103">Outlook からカレンダー イベントを取得する</span><span class="sxs-lookup"><span data-stu-id="6d306-103">Get calendar events from Outlook</span></span>

1. <span data-ttu-id="6d306-104">を開き、 `./src/GraphService.ts` 次の関数を追加します。</span><span class="sxs-lookup"><span data-stu-id="6d306-104">Open `./src/GraphService.ts` and add the following function.</span></span>

    :::code language="typescript" source="../demo/graph-tutorial/src/GraphService.ts" id="getUserWeekCalendarSnippet":::

    <span data-ttu-id="6d306-105">このコードの実行内容を考えましょう。</span><span class="sxs-lookup"><span data-stu-id="6d306-105">Consider what this code is doing.</span></span>

    - <span data-ttu-id="6d306-106">呼び出される URL は `/me/calendarview` です。</span><span class="sxs-lookup"><span data-stu-id="6d306-106">The URL that will be called is `/me/calendarview`.</span></span>
    - <span data-ttu-id="6d306-107">この `header` メソッドは、 `Prefer: outlook.timezone=""` 要求にヘッダーを追加して、応答内の時間をユーザーの優先タイムゾーンに追加します。</span><span class="sxs-lookup"><span data-stu-id="6d306-107">The `header` method adds the `Prefer: outlook.timezone=""` header to the request, causing the times in the response to be in the user's preferred time zone.</span></span>
    - <span data-ttu-id="6d306-108">メソッドは、 `query` とパラメーターを追加し `startDateTime` `endDateTime` 、予定表ビューの時間のウィンドウを定義します。</span><span class="sxs-lookup"><span data-stu-id="6d306-108">The `query` method adds the `startDateTime` and `endDateTime` parameters, defining the window of time for the calendar view.</span></span>
    - <span data-ttu-id="6d306-109">`select`このメソッドは、各イベントに対して返されるフィールドを、ビューが実際に使用するものだけに制限します。</span><span class="sxs-lookup"><span data-stu-id="6d306-109">The `select` method limits the fields returned for each events to just those the view will actually use.</span></span>
    - <span data-ttu-id="6d306-110">メソッドは、 `orderby` 生成された日付と時刻で結果を並べ替えます。最新のアイテムが最初に表示されます。</span><span class="sxs-lookup"><span data-stu-id="6d306-110">The `orderby` method sorts the results by the date and time they were created, with the most recent item being first.</span></span>
    - <span data-ttu-id="6d306-111">この `top` メソッドは、結果を最初の50イベントに制限します。</span><span class="sxs-lookup"><span data-stu-id="6d306-111">The `top` method limits the results to the first 50 events.</span></span>
    - <span data-ttu-id="6d306-112">応答に値が含まれており `@odata.nextLink` 、さらに結果が使用可能であることを示している場合は、 `PageIterator` オブジェクトを使用し [てコレクションのページ](https://docs.microsoft.com/graph/sdks/paging?tabs=typeScript) を取得し、すべての結果を取得します。</span><span class="sxs-lookup"><span data-stu-id="6d306-112">If the response contains an `@odata.nextLink` value, indicating there are more results available, a `PageIterator` object is used to [page through the collection](https://docs.microsoft.com/graph/sdks/paging?tabs=typeScript) to get all of the results.</span></span>

1. <span data-ttu-id="6d306-113">応答コンポーネントを作成して、呼び出しの結果を表示します。</span><span class="sxs-lookup"><span data-stu-id="6d306-113">Create a React component to display the results of the call.</span></span> <span data-ttu-id="6d306-114">という名前のディレクトリに新しいファイルを作成 `./src` `Calendar.tsx` し、次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="6d306-114">Create a new file in the `./src` directory named `Calendar.tsx` and add the following code.</span></span>

    ```typescript
    import React from 'react';
    import { NavLink as RouterNavLink } from 'react-router-dom';
    import { Table } from 'reactstrap';
    import moment, { Moment } from 'moment-timezone';
    import { findOneIana } from "windows-iana";
    import { Event } from 'microsoft-graph';
    import { config } from './Config';
    import { getUserWeekCalendar } from './GraphService';
    import withAuthProvider, { AuthComponentProps } from './AuthProvider';

    interface CalendarState {
      eventsLoaded: boolean;
      events: Event[];
      startOfWeek: Moment | undefined;
    }

    class Calendar extends React.Component<AuthComponentProps, CalendarState> {
      constructor(props: any) {
        super(props);

        this.state = {
          eventsLoaded: false,
          events: [],
          startOfWeek: undefined
        };
      }

      async componentDidUpdate() {
        try {
          // Get the user's access token
          var accessToken = await this.props.getAccessToken(config.scopes);
          // Convert user's Windows time zone ("Pacific Standard Time")
          // to IANA format ("America/Los_Angeles")
          // Moment needs IANA format
          var ianaTimeZone = findOneIana(this.props.user.timeZone);

          // Get midnight on the start of the current week in the user's timezone,
          // but in UTC. For example, for Pacific Standard Time, the time value would be
          // 07:00:00Z
          var startOfWeek = moment.tz(ianaTimeZone!.valueOf()).startOf('week').utc();

          // Get the user's events
          var events = await getUserWeekCalendar(accessToken, this.props.user.timeZone, startOfWeek);

          // Update the array of events in state
          this.setState({
            eventsLoaded: true,
            events: events,
            startOfWeek: startOfWeek
          });
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

    <span data-ttu-id="6d306-115">ここでは、JSON のイベントの配列をページにレンダリングするだけです。</span><span class="sxs-lookup"><span data-stu-id="6d306-115">For now this just renders the array of events in JSON on the page.</span></span>

1. <span data-ttu-id="6d306-116">この新しいコンポーネントをアプリに追加します。</span><span class="sxs-lookup"><span data-stu-id="6d306-116">Add this new component to the app.</span></span> <span data-ttu-id="6d306-117">を開き、 `./src/App.tsx` 次の `import` ステートメントをファイルの先頭に追加します。</span><span class="sxs-lookup"><span data-stu-id="6d306-117">Open `./src/App.tsx` and add the following `import` statement to the top of the file.</span></span>

    ```typescript
    import Calendar from './Calendar';
    ```

1. <span data-ttu-id="6d306-118">次のコンポーネントを既存のの直後に追加し `<Route>` ます。</span><span class="sxs-lookup"><span data-stu-id="6d306-118">Add the following component just after the existing `<Route>`.</span></span>

    ```typescript
    <Route exact path="/calendar"
      render={(props) =>
        this.props.isAuthenticated ?
          <Calendar {...props} /> :
          <Redirect to="/" />
      } />
    ```

1. <span data-ttu-id="6d306-119">変更内容を保存し、アプリを再起動します。</span><span class="sxs-lookup"><span data-stu-id="6d306-119">Save your changes and restart the app.</span></span> <span data-ttu-id="6d306-120">サインインして、ナビゲーションバーの [ **予定表** ] リンクをクリックします。</span><span class="sxs-lookup"><span data-stu-id="6d306-120">Sign in and click the **Calendar** link in the nav bar.</span></span> <span data-ttu-id="6d306-121">すべてが正常に機能していれば、ユーザーのカレンダーにイベントの JSON ダンプが表示されます。</span><span class="sxs-lookup"><span data-stu-id="6d306-121">If everything works, you should see a JSON dump of events on the user's calendar.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="6d306-122">結果の表示</span><span class="sxs-lookup"><span data-stu-id="6d306-122">Display the results</span></span>

<span data-ttu-id="6d306-123">これで、コンポーネントを更新して、 `Calendar` よりわかりやすい方法でイベントを表示することができます。</span><span class="sxs-lookup"><span data-stu-id="6d306-123">Now you can update the `Calendar` component to display the events in a more user-friendly manner.</span></span>

1. <span data-ttu-id="6d306-124">という名前のディレクトリに新しいファイルを作成 `./src` `Calendar.css` し、次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="6d306-124">Create a new file in the `./src` directory named `Calendar.css` and add the following code.</span></span>

    :::code language="css" source="../demo/graph-tutorial/src/Calendar.css":::

1. <span data-ttu-id="6d306-125">イベントをテーブル行として1日でレンダリングするように、反応するコンポーネントを作成します。</span><span class="sxs-lookup"><span data-stu-id="6d306-125">Create a React component to render events in a single day as table rows.</span></span> <span data-ttu-id="6d306-126">という名前のディレクトリに新しいファイルを作成 `./src` `CalendarDayRow.tsx` し、次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="6d306-126">Create a new file in the `./src` directory named `CalendarDayRow.tsx` and add the following code.</span></span>

    :::code language="typescript" source="../demo/graph-tutorial/src/CalendarDayRow.tsx" id="CalendarDayRowSnippet":::

1. <span data-ttu-id="6d306-127">次のステートメントを、 `import` **Calendar. tsx** の先頭に追加します。</span><span class="sxs-lookup"><span data-stu-id="6d306-127">Add the following `import` statements to the top of **Calendar.tsx** .</span></span>

    ```typescript
    import CalendarDayRow from './CalendarDayRow';
    import './Calendar.css';
    ```

1. <span data-ttu-id="6d306-128">既存の関数を次の関数に置き換え `render` `./src/Calendar.tsx` ます。</span><span class="sxs-lookup"><span data-stu-id="6d306-128">Replace the existing `render` function in `./src/Calendar.tsx` with the following function.</span></span>

    :::code language="typescript" source="../demo/graph-tutorial/src/Calendar.tsx" id="renderSnippet":::

    <span data-ttu-id="6d306-129">これにより、イベントがそれぞれの日に分割され、各日のテーブルセクションがレンダリングされます。</span><span class="sxs-lookup"><span data-stu-id="6d306-129">This splits the events into their respective days and renders a table section for each day.</span></span>

1. <span data-ttu-id="6d306-130">変更を保存し、アプリを再起動します。</span><span class="sxs-lookup"><span data-stu-id="6d306-130">Save the changes and restart the app.</span></span> <span data-ttu-id="6d306-131">[ **予定表** ] リンクをクリックすると、アプリがイベントの表を表示するようになります。</span><span class="sxs-lookup"><span data-stu-id="6d306-131">Click on the **Calendar** link and the app should now render a table of events.</span></span>

    ![イベント表のスクリーンショット](./images/add-msgraph-01.png)
