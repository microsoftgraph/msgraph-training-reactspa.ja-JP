<!-- markdownlint-disable MD002 MD041 -->

この演習では、アプリケーションに Microsoft Graphを組み込む必要があります。 このアプリケーションでは[、microsoft-graph-client ライブラリ](https://github.com/microsoftgraph/msgraph-sdk-javascript)を使用して Microsoft Graph。

## <a name="get-calendar-events-from-outlook"></a>Outlook からカレンダー イベントを取得する

1. 次 `./src/GraphService.ts` の関数を開いて追加します。

    :::code language="typescript" source="../demo/graph-tutorial/src/GraphService.ts" id="getUserWeekCalendarSnippet":::

    このコードの実行内容を考えましょう。

    - 呼び出される URL は `/me/calendarview` です。
    - このメソッドは、要求にヘッダーを追加し、応答の時間がユーザーの優先タイム ゾーン `header` `Prefer: outlook.timezone=""` に入る原因です。
    - メソッド `query` は、カレンダー ビューの時間ウィンドウを定義するパラメーター `startDateTime` `endDateTime` とパラメーターを追加します。
    - メソッド `select` は、各イベントに返されるフィールドを、ビューが実際に使用するフィールドに制限します。
    - このメソッドは、作成された日付と時刻で結果を並べ替え、最新のアイテム `orderby` を最初に並べ替える。
    - この `top` メソッドは、1 ページの結果を 25 イベントに制限します。
    - 応答に値が含まれている場合は、使用可能な結果が多くあることを示すオブジェクトを使用して、コレクションをページ移動してすべての結果 `@odata.nextLink` `PageIterator` を取得します。 [](https://docs.microsoft.com/graph/sdks/paging?tabs=typeScript)

1. 呼び出React結果を表示するコンポーネントを作成します。 という名前のディレクトリに新しい `./src` ファイルを作成 `Calendar.tsx` し、次のコードを追加します。

    ```typescript
    import React from 'react';
    import { NavLink as RouterNavLink } from 'react-router-dom';
    import { Table } from 'reactstrap';
    import moment, { Moment } from 'moment-timezone';
    import { findIana } from "windows-iana";
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
            var ianaTimeZones = findIana(this.props.user.timeZone);

            // Get midnight on the start of the current week in the user's timezone,
            // but in UTC. For example, for Pacific Standard Time, the time value would be
            // 07:00:00Z
            var startOfWeek = moment.tz(ianaTimeZones![0].valueOf()).startOf('week').utc();

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

    今のところ、これはページ上の JSON でイベントの配列をレンダリングします。

1. この新しいコンポーネントをアプリに追加します。 次 `./src/App.tsx` のステートメントを開 `import` き、ファイルの上部に追加します。

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

1. 変更内容を保存し、アプリを再起動します。 サインインして、ナビゲーション バー **の [予定表** ] リンクをクリックします。 すべてが正常に機能していれば、ユーザーのカレンダーにイベントの JSON ダンプが表示されます。

## <a name="display-the-results"></a>結果の表示

これで、コンポーネントを更新 `Calendar` して、よりユーザーフレンドリーな方法でイベントを表示できます。

1. という名前のディレクトリに新しい `./src` ファイルを作成 `Calendar.css` し、次のコードを追加します。

    :::code language="css" source="../demo/graph-tutorial/src/Calendar.css":::

1. 1 日Reactテーブル行としてレンダリングするイベント コンポーネントを作成します。 という名前のディレクトリに新しい `./src` ファイルを作成 `CalendarDayRow.tsx` し、次のコードを追加します。

    :::code language="typescript" source="../demo/graph-tutorial/src/CalendarDayRow.tsx" id="CalendarDayRowSnippet":::

1. `import` **Calendar.tsx** の上部に次のステートメントを追加します。

    ```typescript
    import CalendarDayRow from './CalendarDayRow';
    import './Calendar.css';
    ```

1. 既存の関数を `render` 次の `./src/Calendar.tsx` 関数に置き換える。

    :::code language="typescript" source="../demo/graph-tutorial/src/Calendar.tsx" id="renderSnippet":::

    これにより、イベントがそれぞれの日に分割され、各日のテーブル セクションがレンダリングされます。

1. 変更を保存し、アプリを再起動します。 [予定表] **リンクをクリック** すると、アプリはイベントのテーブルを表示する必要があります。

    ![イベント表のスクリーンショット](./images/add-msgraph-01.png)
