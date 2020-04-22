<!-- markdownlint-disable MD002 MD041 -->

この演習では、Microsoft Graph をアプリケーションに組み込みます。 このアプリケーションでは、microsoft graph[クライアント](https://github.com/microsoftgraph/msgraph-sdk-javascript)ライブラリを使用して microsoft graph への呼び出しを行います。

## <a name="get-calendar-events-from-outlook"></a>Outlook からカレンダー イベントを取得する

1. を`./src/GraphService.ts`開き、次の関数を追加します。

    :::code language="typescript" source="../demo/graph-tutorial/src/GraphService.ts" id="getEventsSnippet":::

    このコードの実行内容を考えましょう。

    - 呼び出される URL は `/me/events` です。
    - この`select`メソッドは、各イベントに対して返されるフィールドを、ビューが実際に使用するものだけに制限します。
    - メソッド`orderby`は、生成された日付と時刻で結果を並べ替えます。最新のアイテムが最初に表示されます。

1. 応答コンポーネントを作成して、呼び出しの結果を表示します。 という名前`./src` `Calendar.tsx`のディレクトリに新しいファイルを作成し、次のコードを追加します。

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

    ここでは、JSON のイベントの配列をページにレンダリングするだけです。

1. この新しいコンポーネントをアプリに追加します。 を`./src/App.tsx`開き、次`import`のステートメントをファイルの先頭に追加します。

    ```typescript
    import Calendar from './Calendar';
    ```

1. 次のコンポーネントを既存`<Route>`のの直後に追加します。

    ```typescript
    <Route exact path="/calendar"
      render={(props) =>
        this.props.isAuthenticated ?
          <Calendar {...props} /> :
          <Redirect to="/" />
      } />
    ```

1. 変更内容を保存し、アプリを再起動します。 サインインして、ナビゲーションバーの [**予定表**] リンクをクリックします。 すべてが正常に機能していれば、ユーザーのカレンダーにイベントの JSON ダンプが表示されます。

## <a name="display-the-results"></a>結果の表示

これで、 `Calendar`コンポーネントを更新して、よりわかりやすい方法でイベントを表示することができます。

1. 既存`render`の関数`./src/Calendar.js`を次の関数に置き換えます。

    :::code language="typescript" source="../demo/graph-tutorial/src/Calendar.tsx" id="renderSnippet":::

    これにより、イベントのコレクションがループ処理され、テーブル行が1つずつ追加されます。

1. 変更を保存し、アプリを再起動します。 [**予定表**] リンクをクリックすると、アプリがイベントの表を表示するようになります。

    ![イベント表のスクリーンショット](./images/add-msgraph-01.png)
