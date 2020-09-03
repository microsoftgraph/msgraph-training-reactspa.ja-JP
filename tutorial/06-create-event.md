<!-- markdownlint-disable MD002 MD041 -->

このセクションでは、ユーザーの予定表にイベントを作成する機能を追加します。

## <a name="add-method-to-graphservice"></a>GraphService にメソッドを追加する

1. **/Src/GraphService.ts**を開き、次の関数を追加して、新しいイベントを作成します。

    :::code language="typescript" source="../demo/graph-tutorial/src/GraphService.ts" id="createEventSnippet":::

## <a name="create-new-event-form"></a>新しいイベントフォームを作成する

1. **Newevent**という名前の新しいファイルを作成し、次のコードを追加し**ます。**

    :::code language="typescript" source="../demo/graph-tutorial/src/NewEvent.tsx" id="NewEventSnippet":::

1. **/Src/App.tsx**を開き、次の `import` ステートメントをファイルの先頭に追加します。

    ```typescript
    import NewEvent from './NewEvent';
    ```

1. 新しいイベントフォームに新しいルートを追加します。 他の要素の直後に、次のコードを追加し `Route` ます。

    ```typescript
    <Route exact path="/newevent"
      render={(props) =>
        this.props.isAuthenticated ?
          <NewEvent {...props} /> :
          <Redirect to="/" />
      } />
    ```

    完全なステートメントは、次のようになり `return` ます。

    :::code language="typescript" source="../demo/graph-tutorial/src/App.tsx" id="renderSnippet" highlight="23-28":::

1. アプリを更新し、[カレンダー] ビューに移動します。 [ **New event** ] ボタンをクリックします。 フィールドに入力し、[ **作成**] をクリックします。

    ![新しいイベントフォームのスクリーンショット](./images/create-event-01.png)
