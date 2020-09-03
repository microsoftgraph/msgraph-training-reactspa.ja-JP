<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="c292c-101">このセクションでは、ユーザーの予定表にイベントを作成する機能を追加します。</span><span class="sxs-lookup"><span data-stu-id="c292c-101">In this section you will add the ability to create events on the user's calendar.</span></span>

## <a name="add-method-to-graphservice"></a><span data-ttu-id="c292c-102">GraphService にメソッドを追加する</span><span class="sxs-lookup"><span data-stu-id="c292c-102">Add method to GraphService</span></span>

1. <span data-ttu-id="c292c-103">**/Src/GraphService.ts**を開き、次の関数を追加して、新しいイベントを作成します。</span><span class="sxs-lookup"><span data-stu-id="c292c-103">Open **./src/GraphService.ts** and add the following function to create a new event.</span></span>

    :::code language="typescript" source="../demo/graph-tutorial/src/GraphService.ts" id="createEventSnippet":::

## <a name="create-new-event-form"></a><span data-ttu-id="c292c-104">新しいイベントフォームを作成する</span><span class="sxs-lookup"><span data-stu-id="c292c-104">Create new event form</span></span>

1. <span data-ttu-id="c292c-105">**Newevent**という名前の新しいファイルを作成し、次のコードを追加し**ます。**</span><span class="sxs-lookup"><span data-stu-id="c292c-105">Create a new file in the **./src** directory named **NewEvent.tsx** and add the following code.</span></span>

    :::code language="typescript" source="../demo/graph-tutorial/src/NewEvent.tsx" id="NewEventSnippet":::

1. <span data-ttu-id="c292c-106">**/Src/App.tsx**を開き、次の `import` ステートメントをファイルの先頭に追加します。</span><span class="sxs-lookup"><span data-stu-id="c292c-106">Open **./src/App.tsx** and add the following `import` statement to the top of the file.</span></span>

    ```typescript
    import NewEvent from './NewEvent';
    ```

1. <span data-ttu-id="c292c-107">新しいイベントフォームに新しいルートを追加します。</span><span class="sxs-lookup"><span data-stu-id="c292c-107">Add a new route to the new event form.</span></span> <span data-ttu-id="c292c-108">他の要素の直後に、次のコードを追加し `Route` ます。</span><span class="sxs-lookup"><span data-stu-id="c292c-108">Add the following code just after the other `Route` elements.</span></span>

    ```typescript
    <Route exact path="/newevent"
      render={(props) =>
        this.props.isAuthenticated ?
          <NewEvent {...props} /> :
          <Redirect to="/" />
      } />
    ```

    <span data-ttu-id="c292c-109">完全なステートメントは、次のようになり `return` ます。</span><span class="sxs-lookup"><span data-stu-id="c292c-109">The full `return` statement should now look like this.</span></span>

    :::code language="typescript" source="../demo/graph-tutorial/src/App.tsx" id="renderSnippet" highlight="23-28":::

1. <span data-ttu-id="c292c-110">アプリを更新し、[カレンダー] ビューに移動します。</span><span class="sxs-lookup"><span data-stu-id="c292c-110">Refresh the app and browse to the calendar view.</span></span> <span data-ttu-id="c292c-111">[ **New event** ] ボタンをクリックします。</span><span class="sxs-lookup"><span data-stu-id="c292c-111">Click the **New event** button.</span></span> <span data-ttu-id="c292c-112">フィールドに入力し、[ **作成**] をクリックします。</span><span class="sxs-lookup"><span data-stu-id="c292c-112">Fill in the fields and click **Create**.</span></span>

    ![新しいイベントフォームのスクリーンショット](./images/create-event-01.png)
