<!-- markdownlint-disable MD002 MD041 -->

この演習では、Microsoft Graph をアプリケーションに組み込む必要があります。 このアプリケーションでは [、microsoft-graph-client ライブラリ](https://github.com/microsoftgraph/msgraph-sdk-javascript) を使用して Microsoft Graph を呼び出します。

## <a name="get-calendar-events-from-outlook"></a>Outlook からカレンダー イベントを取得する

1. 次 `./src/GraphService.ts` の関数を開いて追加します。

    :::code language="typescript" source="../demo/graph-tutorial/src/GraphService.ts" id="getUserWeekCalendarSnippet":::

    このコードの実行内容を考えましょう。

    - 呼び出される URL は `/me/calendarview` です。
    - このメソッドは要求にヘッダーを追加し、応答の時間がユーザーの優先タイム ゾーン `header` `Prefer: outlook.timezone=""` に入っています。
    - この `query` メソッドは、 `startDateTime` カレンダー ビュー `endDateTime` の時間のウィンドウを定義するパラメーターとパラメーターを追加します。
    - この `select` メソッドは、各イベントで返されるフィールドを、ビューが実際に使用するフィールドに制限します。
    - このメソッドは、作成された日付と時刻で結果を並べ替え、最新のアイテム `orderby` が最初に表示されます。
    - この `top` メソッドは、1 ページの結果を 25 イベントに制限します。
    - 応答に使用可能な結果が多くあることを示す値が含まれている場合は、オブジェクトを使用してコレクションをページ移動し、すべての結果 `@odata.nextLink` `PageIterator` を取得します。 [](https://docs.microsoft.com/graph/sdks/paging?tabs=typeScript)

1. React コンポーネントを作成して、呼び出しの結果を表示します。 ディレクトリに新しいファイルを `./src` 作成し、 `Calendar.tsx` 次のコードを追加します。

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
        if (this.props.user && !this.state.eventsLoaded)
        {
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
          catch (err) {
            this.props.setError('ERROR', JSON.stringify(err));
          }
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

    ここでは、JSON でイベントの配列をページにレンダリングします。

1. この新しいコンポーネントをアプリに追加します。 次 `./src/App.tsx` のステートメントを `import` 開き、ファイルの一番上に追加します。

    ```typescript
    import Calendar from './Calendar';
    ```

1. 既存のコンポーネントの直後に次のコンポーネントを追加します `<Route>` 。

    ```typescript
    <Route exact path="/calendar"
      render={(props) =>
        this.props.isAuthenticated ?
          <Calendar {...props} /> :
          <Redirect to="/" />
      } />
    ```

1. 変更内容を保存し、アプリを再起動します。 サインインし、ナビゲーション バー **の [予定表** ] リンクをクリックします。 すべてが正常に機能していれば、ユーザーのカレンダーにイベントの JSON ダンプが表示されます。

## <a name="display-the-results"></a>結果の表示

これで、コンポーネントを `Calendar` 更新して、より使い分け的な方法でイベントを表示できます。

1. ディレクトリに新しいファイルを `./src` 作成し、 `Calendar.css` 次のコードを追加します。

    :::code language="css" source="../demo/graph-tutorial/src/Calendar.css":::

1. テーブルの行として 1 日でイベントをレンダリングする React コンポーネントを作成します。 ディレクトリに新しいファイルを `./src` 作成し、 `CalendarDayRow.tsx` 次のコードを追加します。

    :::code language="typescript" source="../demo/graph-tutorial/src/CalendarDayRow.tsx" id="CalendarDayRowSnippet":::

1. `import` **Calendar.tsx** の上部に次のステートメントを追加します。

    ```typescript
    import CalendarDayRow from './CalendarDayRow';
    import './Calendar.css';
    ```

1. 既存の関数を `render` 次の `./src/Calendar.tsx` 関数に置き換える。

    :::code language="typescript" source="../demo/graph-tutorial/src/Calendar.tsx" id="renderSnippet":::

    これにより、イベントがそれぞれの日に分割され、各日のテーブル セクションがレンダリングされます。

1. 変更を保存し、アプリを再起動します。 [カレンダー] **リンクを** クリックすると、アプリはイベントのテーブルをレンダリングする必要があります。

    ![イベント表のスクリーンショット](./images/add-msgraph-01.png)
