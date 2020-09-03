<!-- markdownlint-disable MD002 MD041 -->

この演習では、Microsoft Graph をアプリケーションに組み込みます。 このアプリケーションでは、microsoft graph [クライアント](https://github.com/microsoftgraph/msgraph-sdk-javascript) ライブラリを使用して microsoft graph への呼び出しを行います。

## <a name="get-calendar-events-from-outlook"></a>Outlook からカレンダー イベントを取得する

1. を開き、 `./src/GraphService.ts` 次の関数を追加します。

    :::code language="typescript" source="../demo/graph-tutorial/src/GraphService.ts" id="getUserWeekCalendarSnippet":::

    このコードの実行内容を考えましょう。

    - 呼び出される URL は `/me/calendarview` です。
    - この `header` メソッドは、 `Prefer: outlook.timezone=""` 要求にヘッダーを追加して、応答内の時間をユーザーの優先タイムゾーンに追加します。
    - メソッドは、 `query` とパラメーターを追加し `startDateTime` `endDateTime` 、予定表ビューの時間のウィンドウを定義します。
    - `select`このメソッドは、各イベントに対して返されるフィールドを、ビューが実際に使用するものだけに制限します。
    - メソッドは、 `orderby` 生成された日付と時刻で結果を並べ替えます。最新のアイテムが最初に表示されます。
    - この `top` メソッドは、結果を最初の50イベントに制限します。
    - 応答に値が含まれており `@odata.nextLink` 、さらに結果が使用可能であることを示している場合は、 `PageIterator` オブジェクトを使用し [てコレクションのページ](https://docs.microsoft.com/graph/sdks/paging?tabs=typeScript) を取得し、すべての結果を取得します。

1. 応答コンポーネントを作成して、呼び出しの結果を表示します。 という名前のディレクトリに新しいファイルを作成 `./src` `Calendar.tsx` し、次のコードを追加します。

    ```typescript
    import React from 'react';
    import { NavLink as RouterNavLink } from 'react-router-dom';
    import { Table } from 'reactstrap';
    import moment from 'moment-timezone';
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

    ここでは、JSON のイベントの配列をページにレンダリングするだけです。

1. この新しいコンポーネントをアプリに追加します。 を開き、 `./src/App.tsx` 次の `import` ステートメントをファイルの先頭に追加します。

    ```typescript
    import Calendar from './Calendar';
    ```

1. 次のコンポーネントを既存のの直後に追加し `<Route>` ます。

    ```typescript
    <Route exact path="/calendar"
      render={(props) =>
        this.props.isAuthenticated ?
          <Calendar {...props} /> :
          <Redirect to="/" />
      } />
    ```

1. 変更内容を保存し、アプリを再起動します。 サインインして、ナビゲーションバーの [ **予定表** ] リンクをクリックします。 すべてが正常に機能していれば、ユーザーのカレンダーにイベントの JSON ダンプが表示されます。

## <a name="display-the-results"></a>結果の表示

これで、コンポーネントを更新して、 `Calendar` よりわかりやすい方法でイベントを表示することができます。

1. という名前のディレクトリに新しいファイルを作成 `./src` `Calendar.css` し、次のコードを追加します。

    :::code language="css" source="../demo/graph-tutorial/src/Calendar.css":::

1. イベントをテーブル行として1日でレンダリングするように、反応するコンポーネントを作成します。 という名前のディレクトリに新しいファイルを作成 `./src` `CalendarDayRow.tsx` し、次のコードを追加します。

    :::code language="typescript" source="../demo/graph-tutorial/src/CalendarDayRow.tsx" id="CalendarDayRowSnippet":::

1. 次のステートメントを、 `import` **Calendar. tsx**の先頭に追加します。

    ```typescript
    import CalendarDayRow from './CalendarDayRow';
    import './Calendar.css';
    ```

1. 既存の関数を次の関数に置き換え `render` `./src/Calendar.tsx` ます。

    :::code language="typescript" source="../demo/graph-tutorial/src/Calendar.tsx" id="renderSnippet":::

    これにより、イベントがそれぞれの日に分割され、各日のテーブルセクションがレンダリングされます。

1. 変更を保存し、アプリを再起動します。 [ **予定表** ] リンクをクリックすると、アプリがイベントの表を表示するようになります。

    ![イベント表のスクリーンショット](./images/add-msgraph-01.png)
