---
uid: web-forms/overview/data-access/editing-inserting-and-deleting-data/adding-client-side-confirmation-when-deleting-vb
title: "刪除 (VB) 時新增用戶端確認 |Microsoft 文件"
author: rick-anderson
description: "我們建立了到目前為止的介面，使用者不小心刪除資料他們在想要按一下 [編輯] 按鈕時，按一下 [刪除] 按鈕。 在此 t..."
ms.author: aspnetcontent
manager: wpickett
ms.date: 07/17/2006
ms.topic: article
ms.assetid: 6331e02e-c465-4cdf-bd3f-f07680c289d6
ms.technology: dotnet-webforms
ms.prod: .net-framework
msc.legacyurl: /web-forms/overview/data-access/editing-inserting-and-deleting-data/adding-client-side-confirmation-when-deleting-vb
msc.type: authoredcontent
ms.openlocfilehash: 3461f9ec4f139f1ea0e60a01b898e67e7ebd7f54
ms.sourcegitcommit: 9a9483aceb34591c97451997036a9120c3fe2baf
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/10/2017
---
<a name="adding-client-side-confirmation-when-deleting-vb"></a><span data-ttu-id="47abc-104">刪除 (VB) 時新增用戶端確認</span><span class="sxs-lookup"><span data-stu-id="47abc-104">Adding Client-Side Confirmation When Deleting (VB)</span></span>
====================
<span data-ttu-id="47abc-105">由[Scott Mitchell](https://twitter.com/ScottOnWriting)</span><span class="sxs-lookup"><span data-stu-id="47abc-105">by [Scott Mitchell](https://twitter.com/ScottOnWriting)</span></span>

<span data-ttu-id="47abc-106">[下載範例應用程式](http://download.microsoft.com/download/9/c/1/9c1d03ee-29ba-4d58-aa1a-f201dcc822ea/ASPNET_Data_Tutorial_22_VB.exe)或[下載 PDF](adding-client-side-confirmation-when-deleting-vb/_static/datatutorial22vb1.pdf)</span><span class="sxs-lookup"><span data-stu-id="47abc-106">[Download Sample App](http://download.microsoft.com/download/9/c/1/9c1d03ee-29ba-4d58-aa1a-f201dcc822ea/ASPNET_Data_Tutorial_22_VB.exe) or [Download PDF](adding-client-side-confirmation-when-deleting-vb/_static/datatutorial22vb1.pdf)</span></span>

> <span data-ttu-id="47abc-107">我們建立了到目前為止的介面，使用者不小心刪除資料他們在想要按一下 [編輯] 按鈕時，按一下 [刪除] 按鈕。</span><span class="sxs-lookup"><span data-stu-id="47abc-107">In the interfaces we've created so far, a user can accidentally delete data by clicking the Delete button when they meant to click the Edit button.</span></span> <span data-ttu-id="47abc-108">在本教學課程中，我們會將新增用戶端確認對話方塊出現時按一下 [刪除] 按鈕。</span><span class="sxs-lookup"><span data-stu-id="47abc-108">In this tutorial we'll add a client-side confirmation dialog box that appears when the Delete button is clicked.</span></span>


## <a name="introduction"></a><span data-ttu-id="47abc-109">簡介</span><span class="sxs-lookup"><span data-stu-id="47abc-109">Introduction</span></span>

<span data-ttu-id="47abc-110">在過去幾個教學課程我們已看到如何使用我們的應用程式架構、 ObjectDataSource 和 Web 控制項的資料即會一併提供插入、 編輯和刪除功能。</span><span class="sxs-lookup"><span data-stu-id="47abc-110">Over the past several tutorials we ve seen how to use our application architecture, ObjectDataSource, and the data Web controls in concert to provide inserting, editing, and deleting capabilities.</span></span> <span data-ttu-id="47abc-111">刪除介面我們 ve 檢查到目前為止已包含刪除的按鈕，當按下時，導致回傳而叫用 ObjectDataSource 的`Delete()`方法。</span><span class="sxs-lookup"><span data-stu-id="47abc-111">The deleting interfaces we ve examined thus far have been composed of a Delete button that, when clicked, causes a postback and invokes the ObjectDataSource s `Delete()` method.</span></span> <span data-ttu-id="47abc-112">`Delete()`方法再叫用已設定的方法與商務邏輯層，會傳播到資料存取層，發出的實際呼叫`DELETE`到資料庫的陳述式。</span><span class="sxs-lookup"><span data-stu-id="47abc-112">The `Delete()` method then invokes the configured method from the Business Logic Layer, which propagates the call down to the Data Access Layer, issuing the actual `DELETE` statement to the database.</span></span>

<span data-ttu-id="47abc-113">雖然此使用者介面可讓訪客刪除記錄的 GridView、 DetailsView 或在 FormView 的控制項，其欠缺任何種類的確認，當使用者按一下 [刪除] 按鈕。</span><span class="sxs-lookup"><span data-stu-id="47abc-113">While this user interface enables visitors to delete records through the GridView, DetailsView, or FormView controls, it lacks any sort of confirmation when the user clicks the Delete button.</span></span> <span data-ttu-id="47abc-114">如果使用者不小心按下 [刪除] 按鈕他們在想要按一下時編輯，他們想要更新的記錄將改為刪除。</span><span class="sxs-lookup"><span data-stu-id="47abc-114">If a user accidentally clicks the Delete button when they meant to click Edit, the record they meant to update will instead be deleted.</span></span> <span data-ttu-id="47abc-115">為了避免此情形，在本教學課程中，我們會將新增用戶端確認對話方塊出現時按一下 [刪除] 按鈕。</span><span class="sxs-lookup"><span data-stu-id="47abc-115">To help prevent this, in this tutorial we'll add a client-side confirmation dialog box that appears when the Delete button is clicked.</span></span>

<span data-ttu-id="47abc-116">JavaScript`confirm(string)`函式會顯示其字串輸入的參數為強制回應對話方塊，來自配備兩個按鈕的 [確定] 並取消 （請參閱圖 1） 內的文字。</span><span class="sxs-lookup"><span data-stu-id="47abc-116">The JavaScript `confirm(string)` function displays its string input parameter as the text inside a modal dialog box that comes equipped with two buttons - OK and Cancel (see Figure 1).</span></span> <span data-ttu-id="47abc-117">`confirm(string)`函式會傳回布林值，視何種按鈕 (`true`，如果使用者按一下 [確定]，和`false`如果按 [取消])。</span><span class="sxs-lookup"><span data-stu-id="47abc-117">The `confirm(string)` function returns a Boolean value depending on what button is clicked (`true`, if the user clicks OK, and `false` if they click Cancel).</span></span>


![JavaScript confirm(string) 方法顯示強制回應，用戶端 Messagebox](adding-client-side-confirmation-when-deleting-vb/_static/image1.png)

<span data-ttu-id="47abc-119">**圖 1**: JavaScript`confirm(string)`方法會顯示強制回應，用戶端的 Messagebox</span><span class="sxs-lookup"><span data-stu-id="47abc-119">**Figure 1**: The JavaScript `confirm(string)` Method Displays a Modal, Client-Side Messagebox</span></span>


<span data-ttu-id="47abc-120">如果值為表單提交期間`false`已取消送出表單，則會傳回從用戶端事件處理常式。</span><span class="sxs-lookup"><span data-stu-id="47abc-120">During a form submission, if a value of `false` is returned from a client-side event handler then the form submission is cancelled.</span></span> <span data-ttu-id="47abc-121">使用這項功能，我們可以有刪除按鈕的用戶端`onclick`事件處理常式傳回的值呼叫`confirm("Are you sure you want to delete this product?")`。</span><span class="sxs-lookup"><span data-stu-id="47abc-121">Using this feature, we can have the Delete button s client-side `onclick` event handler return the value of a call to `confirm("Are you sure you want to delete this product?")`.</span></span> <span data-ttu-id="47abc-122">如果使用者按一下 [取消] 5d;，`confirm(string)`會傳回 false，因而導致取消送出表單。</span><span class="sxs-lookup"><span data-stu-id="47abc-122">If the user clicks Cancel, `confirm(string)` will return false, thereby causing the form submission to cancel.</span></span> <span data-ttu-id="47abc-123">無回傳，刪除贏了 t 產品的 [刪除] 按鈕已按下。</span><span class="sxs-lookup"><span data-stu-id="47abc-123">With no postback, the product whose Delete button was clicked won t be deleted.</span></span> <span data-ttu-id="47abc-124">不過，使用者按一下 [確定] 確認對話方塊中的，如果回傳仍而每況愈下，而且將會刪除產品。</span><span class="sxs-lookup"><span data-stu-id="47abc-124">If, however, the user clicks OK in the confirmation dialog box, the postback will continue unabated and the product will be deleted.</span></span> <span data-ttu-id="47abc-125">請參閱[使用 JavaScript s`confirm()`方法，以控制表單送出](http://www.webreference.com/programming/javascript/confirm/)如需有關這項技術。</span><span class="sxs-lookup"><span data-stu-id="47abc-125">Consult [Using JavaScript s `confirm()` Method to Control Form Submission](http://www.webreference.com/programming/javascript/confirm/) for more information on this technique.</span></span>

<span data-ttu-id="47abc-126">加入必要的用戶端指令碼會有些許如果使用比使用 CommandField 的範本。</span><span class="sxs-lookup"><span data-stu-id="47abc-126">Adding the necessary client-side script differs slightly if using templates than when using a CommandField.</span></span> <span data-ttu-id="47abc-127">因此，在此教學課程中我們將探討 FormView 」 和 「 GridView 範例。</span><span class="sxs-lookup"><span data-stu-id="47abc-127">Therefore, in this tutorial we will look at both a FormView and GridView example.</span></span>

> [!NOTE]
> <span data-ttu-id="47abc-128">使用用戶端確認技術，像是討論在本教學課程中，假設與支援 JavaScript 的瀏覽器瀏覽您的使用者，而且您有已啟用 JavaScript。</span><span class="sxs-lookup"><span data-stu-id="47abc-128">Using client-side confirmation techniques, like the ones discussed in this tutorial, assumes that your users are visiting with browsers that support JavaScript and that they have JavaScript enabled.</span></span> <span data-ttu-id="47abc-129">如果其中一個這些假設沒有特定使用者，則為 true，按一下 [刪除] 按鈕將會立即回傳 （不顯示確認訊息方塊）。</span><span class="sxs-lookup"><span data-stu-id="47abc-129">If either of these assumptions are not true for a particular user, clicking the Delete button will immediately cause a postback (not displaying a confirm messagebox).</span></span>


## <a name="step-1-creating-a-formview-that-supports-deletion"></a><span data-ttu-id="47abc-130">步驟 1： 建立 FormView 支援刪除</span><span class="sxs-lookup"><span data-stu-id="47abc-130">Step 1: Creating a FormView That Supports Deletion</span></span>

<span data-ttu-id="47abc-131">啟動新增 FormView`ConfirmationOnDelete.aspx`頁面`EditInsertDelete`資料夾中，繫結至新的 ObjectDataSource 回提取產品資訊，透過`ProductsBLL`類別的`GetProducts()`方法。</span><span class="sxs-lookup"><span data-stu-id="47abc-131">Start by adding a FormView to the `ConfirmationOnDelete.aspx` page in the `EditInsertDelete` folder, binding it to a new ObjectDataSource that pulls back the product information via the `ProductsBLL` class s `GetProducts()` method.</span></span> <span data-ttu-id="47abc-132">也設定 ObjectDataSource 以便`ProductsBLL`類別 s`DeleteProduct(productID)`方法對應到 ObjectDataSource 的`Delete()`方法; 請確認下拉式清單會設定為 （無） 的 INSERT 和 UPDATE 索引標籤。</span><span class="sxs-lookup"><span data-stu-id="47abc-132">Also configure the ObjectDataSource so that the `ProductsBLL` class s `DeleteProduct(productID)` method is mapped to the ObjectDataSource s `Delete()` method; ensure that the INSERT and UPDATE tabs drop-down lists are set to (None).</span></span> <span data-ttu-id="47abc-133">最後，檢查 FormView s 智慧標籤的 啟用分頁核取方塊。</span><span class="sxs-lookup"><span data-stu-id="47abc-133">Finally, check the Enable Paging checkbox in the FormView s smart tag.</span></span>

<span data-ttu-id="47abc-134">完成這些步驟之後, 的新 ObjectDataSource s 宣告式標記看起來如下所示：</span><span class="sxs-lookup"><span data-stu-id="47abc-134">After these steps, the new ObjectDataSource s declarative markup will look like the following:</span></span>


[!code-aspx[Main](adding-client-side-confirmation-when-deleting-vb/samples/sample1.aspx)]

<span data-ttu-id="47abc-135">如同我們未使用開放式並行存取的過去範例，請花一點時間來清除 ObjectDataSource 的`OldValuesParameterFormatString`屬性。</span><span class="sxs-lookup"><span data-stu-id="47abc-135">As in our past examples that did not use optimistic concurrency, take a moment to clear out the ObjectDataSource s `OldValuesParameterFormatString` property.</span></span>

<span data-ttu-id="47abc-136">因為它已繫結至 ObjectDataSource 控制項僅支援 刪除 FormView 的`ItemTemplate`提供只有 刪除 按鈕，缺少的新功能和更新的按鈕。</span><span class="sxs-lookup"><span data-stu-id="47abc-136">Since it has been bound to an ObjectDataSource control that only supports deleting, the FormView s `ItemTemplate` offers only the Delete button, lacking the New and Update buttons.</span></span> <span data-ttu-id="47abc-137">在 FormView s 宣告式標記，不過，包含多餘`EditItemTemplate`和`InsertItemTemplate`，可以移除。</span><span class="sxs-lookup"><span data-stu-id="47abc-137">The FormView s declarative markup, however, includes a superfluous `EditItemTemplate` and `InsertItemTemplate`, which can be removed.</span></span> <span data-ttu-id="47abc-138">請花一點時間自訂`ItemTemplate`因此也就是資料欄位會顯示產品的子集。</span><span class="sxs-lookup"><span data-stu-id="47abc-138">Take a moment to customize the `ItemTemplate` so that is shows only a subset of the product data fields.</span></span> <span data-ttu-id="47abc-139">我已設定採擷顯示中的產品的名稱`<h3>`標題上方及其供應商和類別目錄的名稱 （以及 [刪除] 按鈕）。</span><span class="sxs-lookup"><span data-stu-id="47abc-139">I ve configured mine to show the product s name in an `<h3>` heading above its supplier and category names (along with the Delete button).</span></span>


[!code-aspx[Main](adding-client-side-confirmation-when-deleting-vb/samples/sample2.aspx)]

<span data-ttu-id="47abc-140">這些變更，我們有完全正常運作的網頁，可讓使用者透過產品一個切換一次，只要按一下 [刪除] 按鈕刪除產品的能力。</span><span class="sxs-lookup"><span data-stu-id="47abc-140">With these changes, we have a fully functional web page that allows a user to toggle through the products one at a time, with the ability to delete a product by simply clicking the Delete button.</span></span> <span data-ttu-id="47abc-141">圖 2 為止顯示進度的螢幕擷取畫面，透過瀏覽器檢視時。</span><span class="sxs-lookup"><span data-stu-id="47abc-141">Figure 2 shows a screen shot of our progress thus far when viewed through a browser.</span></span>


<span data-ttu-id="47abc-142">[![在 FormView 顯示單一產品的相關資訊](adding-client-side-confirmation-when-deleting-vb/_static/image3.png)](adding-client-side-confirmation-when-deleting-vb/_static/image2.png)</span><span class="sxs-lookup"><span data-stu-id="47abc-142">[![The FormView Shows Information About a Single Product](adding-client-side-confirmation-when-deleting-vb/_static/image3.png)](adding-client-side-confirmation-when-deleting-vb/_static/image2.png)</span></span>

<span data-ttu-id="47abc-143">**圖 2**: FormView 顯示資訊有關單一產品 ([按一下以檢視完整大小的影像](adding-client-side-confirmation-when-deleting-vb/_static/image4.png))</span><span class="sxs-lookup"><span data-stu-id="47abc-143">**Figure 2**: The FormView Shows Information About a Single Product ([Click to view full-size image](adding-client-side-confirmation-when-deleting-vb/_static/image4.png))</span></span>


## <a name="step-2-calling-the-confirmstring-function-from-the-delete-buttons-client-side-onclick-event"></a><span data-ttu-id="47abc-144">步驟 2： 從刪除按鈕用戶端 onclick 事件呼叫 confirm(string) 函式</span><span class="sxs-lookup"><span data-stu-id="47abc-144">Step 2: Calling the confirm(string) Function from the Delete Buttons Client-Side onclick Event</span></span>

<span data-ttu-id="47abc-145">在 FormView 建立時，使用最後一個步驟是設定 [刪除] 按鈕這類，當它 s 按一下訪客，JavaScript`confirm(string)`函式會叫用。</span><span class="sxs-lookup"><span data-stu-id="47abc-145">With the FormView created, the final step is to configure the Delete button such that when it s clicked by the visitor, the JavaScript `confirm(string)` function is invoked.</span></span> <span data-ttu-id="47abc-146">用戶端指令碼加入按鈕、 LinkButton 或 ImageButton 的用戶端`onclick`事件即可使用`OnClientClick property`，這是為 ASP.NET 2.0 的新功能。</span><span class="sxs-lookup"><span data-stu-id="47abc-146">Adding client-side script to a Button, LinkButton, or ImageButton s client-side `onclick` event can be accomplished through the use of the `OnClientClick property`, which is new to ASP.NET 2.0.</span></span> <span data-ttu-id="47abc-147">因為我們想要的數值`confirm(string)`函式傳回時，只需將此屬性設定：`return confirm('Are you certain that you want to delete this product?');`</span><span class="sxs-lookup"><span data-stu-id="47abc-147">Since we want to have the value of the `confirm(string)` function returned, simply set this property to: `return confirm('Are you certain that you want to delete this product?');`</span></span>

<span data-ttu-id="47abc-148">這項變更後刪除 LinkButton s 宣告式語法看起來應該像這樣：</span><span class="sxs-lookup"><span data-stu-id="47abc-148">After this change the Delete LinkButton s declarative syntax should look something like:</span></span>


[!code-aspx[Main](adding-client-side-confirmation-when-deleting-vb/samples/sample3.aspx)]

<span data-ttu-id="47abc-149">S 都是這麼簡單 ！</span><span class="sxs-lookup"><span data-stu-id="47abc-149">That s all there is to it!</span></span> <span data-ttu-id="47abc-150">圖 3 顯示此確認的螢幕擷取畫面中的動作。</span><span class="sxs-lookup"><span data-stu-id="47abc-150">Figure 3 shows a screen shot of this confirmation in action.</span></span> <span data-ttu-id="47abc-151">按一下 [刪除] 按鈕會顯示確認對話方塊。</span><span class="sxs-lookup"><span data-stu-id="47abc-151">Clicking the Delete button brings up the confirm dialog box.</span></span> <span data-ttu-id="47abc-152">如果使用者按一下 [取消] 5d;，回傳會取消，並不會刪除產品。</span><span class="sxs-lookup"><span data-stu-id="47abc-152">If the user clicks Cancel, the postback is cancelled and the product is not deleted.</span></span> <span data-ttu-id="47abc-153">如果，不過，使用者按一下 [確定]，繼續回傳和 ObjectDataSource 的`Delete()`叫用方法時，正在刪除的資料庫記錄中累積。</span><span class="sxs-lookup"><span data-stu-id="47abc-153">If, however, the user clicks OK, the postback continues and the ObjectDataSource s `Delete()` method is invoked, culminating in the database record being deleted.</span></span>

> [!NOTE]
> <span data-ttu-id="47abc-154">將字串傳遞至`confirm(string)`JavaScript 函式以單引號 （而非引號） 分隔。</span><span class="sxs-lookup"><span data-stu-id="47abc-154">The string passed into the `confirm(string)` JavaScript function is delimited with apostrophes (rather than quotation marks).</span></span> <span data-ttu-id="47abc-155">在 JavaScript 中，字串可以使用任一字元分隔。</span><span class="sxs-lookup"><span data-stu-id="47abc-155">In JavaScript, strings can be delimited using either character.</span></span> <span data-ttu-id="47abc-156">我們使用單引號這裡這樣的分隔符號字串傳遞至`confirm(string)`並不會引入模稜兩可使用的分隔符號用於`OnClientClick`屬性值。</span><span class="sxs-lookup"><span data-stu-id="47abc-156">We use apostrophes here so that the delimiters for the string passed into `confirm(string)` do not introduce an ambiguity with the delimiters used for the `OnClientClick` property value.</span></span>


<span data-ttu-id="47abc-157">[![確認訊息會立即顯示時按一下 [刪除] 按鈕](adding-client-side-confirmation-when-deleting-vb/_static/image6.png)](adding-client-side-confirmation-when-deleting-vb/_static/image5.png)</span><span class="sxs-lookup"><span data-stu-id="47abc-157">[![A Confirmation is Now Displayed When Clicking the Delete Button](adding-client-side-confirmation-when-deleting-vb/_static/image6.png)](adding-client-side-confirmation-when-deleting-vb/_static/image5.png)</span></span>

<span data-ttu-id="47abc-158">**圖 3**: 確認會立即顯示時按一下 [刪除] 按鈕 ([按一下以檢視完整大小的影像](adding-client-side-confirmation-when-deleting-vb/_static/image7.png))</span><span class="sxs-lookup"><span data-stu-id="47abc-158">**Figure 3**: A Confirmation is Now Displayed When Clicking the Delete Button ([Click to view full-size image](adding-client-side-confirmation-when-deleting-vb/_static/image7.png))</span></span>


## <a name="step-3-configuring-the-onclientclick-property-for-the-delete-button-in-a-commandfield"></a><span data-ttu-id="47abc-159">步驟 3： 在 CommandField 中設定 [刪除] 按鈕 OnClientClick 屬性</span><span class="sxs-lookup"><span data-stu-id="47abc-159">Step 3: Configuring the OnClientClick Property for the Delete Button in a CommandField</span></span>

<span data-ttu-id="47abc-160">使用時按鈕、 LinkButton 或 imagebutton 已直接在範本中，確認對話方塊中可以有與其相關聯只要設定其`OnClientClick`屬性，以傳回結果的 JavaScript`confirm(string)`函式。</span><span class="sxs-lookup"><span data-stu-id="47abc-160">When working with a Button, LinkButton, or ImageButton directly in a template, a confirmation dialog box can be associated with it by simply configuring its `OnClientClick` property to return the results of the JavaScript `confirm(string)` function.</span></span> <span data-ttu-id="47abc-161">不過，CommandField-這會將刪除按鈕的欄位加入 GridView 或 DetailsView-沒有`OnClientClick`可以宣告方式設定的屬性。</span><span class="sxs-lookup"><span data-stu-id="47abc-161">However, the CommandField - which adds a field of Delete buttons to a GridView or DetailsView - does not have an `OnClientClick` property that can be set declaratively.</span></span> <span data-ttu-id="47abc-162">相反地，我們必須以程式設計方式參考 [刪除] 按鈕，在適當的 GridView 或 DetailsView s`DataBound`事件處理常式，並將其設定其`OnClientClick`那里屬性。</span><span class="sxs-lookup"><span data-stu-id="47abc-162">Instead, we must programmatically reference the Delete button in the GridView or DetailsView s appropriate `DataBound` event handler, and then set its `OnClientClick` property there.</span></span>

> [!NOTE]
> <span data-ttu-id="47abc-163">設定 [刪除] 按鈕 s 時`OnClientClick`中適當屬性`DataBound`事件處理常式，都可存取的資料繫結至目前的記錄。</span><span class="sxs-lookup"><span data-stu-id="47abc-163">When setting the Delete button s `OnClientClick` property in the appropriate `DataBound` event handler, we have access to the data was bound to the current record.</span></span> <span data-ttu-id="47abc-164">這表示，就可以將確認訊息来包含有關特定的記錄詳細資料，例如，「 是您確定要刪除 Chai 產品嗎？ 」</span><span class="sxs-lookup"><span data-stu-id="47abc-164">This means we can extend the confirmation message to include details about the particular record, such as, "Are you sure you want to delete the Chai product?"</span></span> <span data-ttu-id="47abc-165">這類自訂，也可以使用資料繫結語法的範本中。</span><span class="sxs-lookup"><span data-stu-id="47abc-165">Such customization is also possible in templates using databinding syntax.</span></span>


<span data-ttu-id="47abc-166">做法是設定`OnClientClick`刪除 button(s) CommandField 時，可讓 s 中的屬性加入至頁面的 GridView。</span><span class="sxs-lookup"><span data-stu-id="47abc-166">To practice setting the `OnClientClick` property for the Delete button(s) in a CommandField, let s add a GridView to the page.</span></span> <span data-ttu-id="47abc-167">設定要使用相同的 ObjectDataSource 控制項 FormView 使用這個 GridView。</span><span class="sxs-lookup"><span data-stu-id="47abc-167">Configure this GridView to use the same ObjectDataSource control that the FormView uses.</span></span> <span data-ttu-id="47abc-168">也會限制為僅包含產品的名稱、 類別和供應商 BoundFields GridView s。</span><span class="sxs-lookup"><span data-stu-id="47abc-168">Also limit the GridView s BoundFields to only include the product s name, category, and supplier.</span></span> <span data-ttu-id="47abc-169">最後，請檢查 GridView s 智慧標籤啟用刪除核取方塊。</span><span class="sxs-lookup"><span data-stu-id="47abc-169">Lastly, check the Enable Deleting checkbox from the GridView s smart tag.</span></span> <span data-ttu-id="47abc-170">這會新增 CommandField GridView s`Columns`集合，其`ShowDeleteButton`屬性設定為`true`。</span><span class="sxs-lookup"><span data-stu-id="47abc-170">This will add a CommandField to the GridView s `Columns` collection with its `ShowDeleteButton` property set to `true`.</span></span>

<span data-ttu-id="47abc-171">進行這些變更之後，GridView s 的宣告式標記看起來應該如下所示：</span><span class="sxs-lookup"><span data-stu-id="47abc-171">After making these changes, your GridView s declarative markup should look like the following:</span></span>


[!code-aspx[Main](adding-client-side-confirmation-when-deleting-vb/samples/sample4.aspx)]

<span data-ttu-id="47abc-172">CommandField 包含單一的刪除 LinkButton 執行個體可以透過程式設計方式存取從 GridView 的`RowDataBound`事件處理常式。</span><span class="sxs-lookup"><span data-stu-id="47abc-172">The CommandField contains a single Delete LinkButton instance that can be accessed programmatically from the GridView s `RowDataBound` event handler.</span></span> <span data-ttu-id="47abc-173">一旦參考，我們可以設定其`OnClientClick`屬性據此。</span><span class="sxs-lookup"><span data-stu-id="47abc-173">Once referenced, we can set its `OnClientClick` property accordingly.</span></span> <span data-ttu-id="47abc-174">建立事件處理常式`RowDataBound`使用下列程式碼的事件：</span><span class="sxs-lookup"><span data-stu-id="47abc-174">Create an event handler for the `RowDataBound` event using the following code:</span></span>


[!code-vb[Main](adding-client-side-confirmation-when-deleting-vb/samples/sample5.vb)]

<span data-ttu-id="47abc-175">這個事件處理常式的資料列 （其將會有 [刪除] 按鈕） 搭配運作，且一開始會以程式設計方式參考 [刪除] 按鈕。</span><span class="sxs-lookup"><span data-stu-id="47abc-175">This event handler works with data rows (those that will have the Delete button) and begins by programmatically referencing the Delete button.</span></span> <span data-ttu-id="47abc-176">在一般使用以下模式：</span><span class="sxs-lookup"><span data-stu-id="47abc-176">In general use the following pattern:</span></span>


[!code-vb[Main](adding-client-side-confirmation-when-deleting-vb/samples/sample6.vb)]

<span data-ttu-id="47abc-177">*ButtonType*是正由 CommandField-按鈕、 LinkButton 或 ImageButton 按鈕的型別。</span><span class="sxs-lookup"><span data-stu-id="47abc-177">*ButtonType* is the type of button being used by the CommandField - Button, LinkButton, or ImageButton.</span></span> <span data-ttu-id="47abc-178">根據預設，CommandField 會使用 LinkButtons，但您可以自訂這個經由 CommandField 的`ButtonType property`。</span><span class="sxs-lookup"><span data-stu-id="47abc-178">By default, the CommandField uses LinkButtons, but this can be customized via the CommandField s `ButtonType property`.</span></span> <span data-ttu-id="47abc-179">*CommandFieldIndex*是序數索引內 GridView 的 CommandField`Columns`集合，而*controlIndex* CommandField s 內的 [刪除] 按鈕的索引`Controls`集合。</span><span class="sxs-lookup"><span data-stu-id="47abc-179">The *commandFieldIndex* is the ordinal index of the CommandField within the GridView s `Columns` collection, whereas the *controlIndex* is the index of the Delete button within the CommandField s `Controls` collection.</span></span> <span data-ttu-id="47abc-180">*ControlIndex*值取決於按鈕的位置相對於其他按鈕 CommandField 中。</span><span class="sxs-lookup"><span data-stu-id="47abc-180">The *controlIndex* value depends on the button s position relative to other buttons in the CommandField.</span></span> <span data-ttu-id="47abc-181">例如，如果只有 CommandField 中顯示的按鈕 [刪除] 按鈕，使用索引為 0。</span><span class="sxs-lookup"><span data-stu-id="47abc-181">For example, if the only button displayed in the CommandField is the Delete button, use an index of 0.</span></span> <span data-ttu-id="47abc-182">如果，不過，沒有位於 [刪除] 按鈕，[編輯] 按鈕使用索引為 2。</span><span class="sxs-lookup"><span data-stu-id="47abc-182">If, however, there s an Edit button that precedes the Delete button, use an index of 2.</span></span> <span data-ttu-id="47abc-183">使用索引為 2 的原因是因為兩個控制項所新增的 [刪除] 按鈕前 CommandField: [編輯] 按鈕和 LiteralControl s 用於某些之間加入空格編輯和刪除按鈕。</span><span class="sxs-lookup"><span data-stu-id="47abc-183">The reason an index of 2 is used is because two controls are added by the CommandField before the Delete button: the Edit button and a LiteralControl that s used to add some space between the Edit and Delete buttons.</span></span>

<span data-ttu-id="47abc-184">特定範例中，CommandField 使用 LinkButtons 並，正在最左邊 欄位中，具有*commandFieldIndex*為 0。</span><span class="sxs-lookup"><span data-stu-id="47abc-184">For our particular example, the CommandField uses LinkButtons and, being the left-most field, has a *commandFieldIndex* of 0.</span></span> <span data-ttu-id="47abc-185">因為沒有任何其他按鈕但 CommandField 中的 [刪除] 按鈕，所以我們使用*controlIndex*為 0。</span><span class="sxs-lookup"><span data-stu-id="47abc-185">Since there are no other buttons but the Delete button in the CommandField, we use a *controlIndex* of 0.</span></span>

<span data-ttu-id="47abc-186">在參考 CommandField 中的 [刪除] 按鈕之後, 我們接下來會抓取繫結至目前的 GridView 資料列的產品的相關資訊。</span><span class="sxs-lookup"><span data-stu-id="47abc-186">After referencing the Delete button in the CommandField, we next grab information about the product bound to the current GridView row.</span></span> <span data-ttu-id="47abc-187">最後，設定 [刪除] 按鈕的`OnClientClick`適當的 javascript 中，包括產品的名稱的屬性。</span><span class="sxs-lookup"><span data-stu-id="47abc-187">Finally, we set the Delete button s `OnClientClick` property to the appropriate JavaScript, which includes the product s name.</span></span> <span data-ttu-id="47abc-188">因為 JavaScript 字串傳遞至`confirm(string)`函式使用我們必須逸出任何出現在產品的名稱的所有格符號所有格符號分隔。</span><span class="sxs-lookup"><span data-stu-id="47abc-188">Since the JavaScript string passed into the `confirm(string)` function is delimited using apostrophes we must escape any apostrophes that appear within the product s name.</span></span> <span data-ttu-id="47abc-189">特別是，在產品的名稱中的任何單引號逸出與 「`\'`"。</span><span class="sxs-lookup"><span data-stu-id="47abc-189">In particular, any apostrophes in the product s name are escaped with "`\'`".</span></span>

<span data-ttu-id="47abc-190">完成這些變更，按一下自訂的確認對話方塊 （請參閱圖 4） 的 GridView 會顯示在 [刪除] 按鈕。</span><span class="sxs-lookup"><span data-stu-id="47abc-190">With these changes complete, clicking on a Delete button in the GridView displays a customized confirmation dialog box (see Figure 4).</span></span> <span data-ttu-id="47abc-191">做為與確認 messagebox 來自 FormView 中，如果使用者按一下 [取消] 5d; 回傳已取消，因此可以防止發生刪除。</span><span class="sxs-lookup"><span data-stu-id="47abc-191">As with the confirmation messagebox from the FormView, if the user clicks Cancel the postback is cancelled, thereby preventing the deletion from occurring.</span></span>

> [!NOTE]
> <span data-ttu-id="47abc-192">這項技術也可用來以程式設計方式存取 CommandField 在 DetailsView 中的 刪除 按鈕。</span><span class="sxs-lookup"><span data-stu-id="47abc-192">This technique can also be used to programmatically access the Delete button in the CommandField in a DetailsView.</span></span> <span data-ttu-id="47abc-193">為 DetailsView，不過，您 d 建立事件處理常式`DataBound`事件，因為在 DetailsView 沒有`RowDataBound`事件。</span><span class="sxs-lookup"><span data-stu-id="47abc-193">For the DetailsView, however, you d create an event handler for the `DataBound` event, since the DetailsView does not have a `RowDataBound` event.</span></span>


<span data-ttu-id="47abc-194">[![按一下 [GridView 的刪除] 按鈕會顯示自訂的確認對話方塊](adding-client-side-confirmation-when-deleting-vb/_static/image9.png)](adding-client-side-confirmation-when-deleting-vb/_static/image8.png)</span><span class="sxs-lookup"><span data-stu-id="47abc-194">[![Clicking the GridView s Delete Button Displays a Customized Confirmation Dialog Box](adding-client-side-confirmation-when-deleting-vb/_static/image9.png)](adding-client-side-confirmation-when-deleting-vb/_static/image8.png)</span></span>

<span data-ttu-id="47abc-195">**圖 4**： 按一下 GridView 的刪除 按鈕會顯示自訂確認對話方塊 ([按一下以檢視完整大小的影像](adding-client-side-confirmation-when-deleting-vb/_static/image10.png))</span><span class="sxs-lookup"><span data-stu-id="47abc-195">**Figure 4**: Clicking the GridView s Delete Button Displays a Customized Confirmation Dialog Box ([Click to view full-size image](adding-client-side-confirmation-when-deleting-vb/_static/image10.png))</span></span>


## <a name="using-templatefields"></a><span data-ttu-id="47abc-196">使用 TemplateFields</span><span class="sxs-lookup"><span data-stu-id="47abc-196">Using TemplateFields</span></span>

<span data-ttu-id="47abc-197">其中一個 CommandField 的缺點是它的按鈕都必須透過索引而產生的物件必須可以轉換成適當的按鈕類型 （按鈕、 LinkButton 或 ImageButton）。</span><span class="sxs-lookup"><span data-stu-id="47abc-197">One of the disadvantages of the CommandField is that its buttons must be accessed through indexing and that the resulting object must be cast to the appropriate button type (Button, LinkButton, or ImageButton).</span></span> <span data-ttu-id="47abc-198">使用 「 神奇號碼 」 和硬式編碼類型邀請不到執行階段才發現的問題。</span><span class="sxs-lookup"><span data-stu-id="47abc-198">Using "magic numbers" and hard-coded types invites problems that cannot be discovered until runtime.</span></span> <span data-ttu-id="47abc-199">例如，如果您或其他開發人員，將新按鈕加入至某個時間點以後 （例如 [編輯] 按鈕），或變更 CommandField`ButtonType`屬性，而沒有錯誤，仍將編譯現有的程式碼，但是瀏覽頁面可能會造成例外狀況或非預期的行為，取決於您的程式碼撰寫的方式和所做的變更。</span><span class="sxs-lookup"><span data-stu-id="47abc-199">For example, if you, or another developer, adds new buttons to the CommandField at some point in the future (such as an Edit button) or changes the `ButtonType` property, the existing code will still compile without error, but visiting the page may cause an exception or unexpected behavior, depending on how your code was written and what changes were made.</span></span>

<span data-ttu-id="47abc-200">另一個方法是將 GridView 和 DetailsView 的 CommandFields TemplateFields 轉換。</span><span class="sxs-lookup"><span data-stu-id="47abc-200">An alternative approach is to convert the GridView and DetailsView s CommandFields into TemplateFields.</span></span> <span data-ttu-id="47abc-201">這會產生具有為 TemplateField `ItemTemplate` CommandField 中每個按鈕具有 LinkButton （按鈕或 ImageButton）。</span><span class="sxs-lookup"><span data-stu-id="47abc-201">This will generate a TemplateField with an `ItemTemplate` that has a LinkButton (or Button or ImageButton) for each button in the CommandField.</span></span> <span data-ttu-id="47abc-202">這些按鈕`OnClientClick`屬性可以指派以宣告方式，如我們看到與 FormView，或以程式設計方式存取在適當`DataBound`事件處理常式使用下列模式：</span><span class="sxs-lookup"><span data-stu-id="47abc-202">These buttons `OnClientClick` properties can be assigned declaratively, as we saw with the FormView, or can be programmatically accessed in the appropriate `DataBound` event handler using the following pattern:</span></span>


[!code-vb[Main](adding-client-side-confirmation-when-deleting-vb/samples/sample7.vb)]

<span data-ttu-id="47abc-203">其中*controlID*按鈕 s 值`ID`屬性。</span><span class="sxs-lookup"><span data-stu-id="47abc-203">Where *controlID* is the value of the button s `ID` property.</span></span> <span data-ttu-id="47abc-204">雖然這種模式仍需要硬式編碼的類型轉換的它將不再需要編製索引，以便進行變更，而導致執行階段錯誤的配置。</span><span class="sxs-lookup"><span data-stu-id="47abc-204">While this pattern still requires a hard-coded type for the cast, it removes the need for indexing, allowing for the layout to change without resulting in a runtime error.</span></span>

## <a name="summary"></a><span data-ttu-id="47abc-205">總結</span><span class="sxs-lookup"><span data-stu-id="47abc-205">Summary</span></span>

<span data-ttu-id="47abc-206">JavaScript`confirm(string)`函式是控制表單送出工作流程的常用的技術。</span><span class="sxs-lookup"><span data-stu-id="47abc-206">The JavaScript `confirm(string)` function is a commonly used technique for controlling form submission workflow.</span></span> <span data-ttu-id="47abc-207">在執行時，函式會顯示強制回應，用戶端 對話方塊，其中包含兩個按鈕的 確定 和 取消。</span><span class="sxs-lookup"><span data-stu-id="47abc-207">When executed, the function displays a modal, client-side dialog box that includes two buttons, OK and Cancel.</span></span> <span data-ttu-id="47abc-208">如果使用者按一下 [確定]，`confirm(string)`函式會傳回`true`; 按一下 [取消] 5d; 傳回`false`。</span><span class="sxs-lookup"><span data-stu-id="47abc-208">If the user clicks OK, the `confirm(string)` function returns `true`; clicking Cancel returns `false`.</span></span> <span data-ttu-id="47abc-209">這項功能，結合其他瀏覽器 s 的行為，如果要取消送出表單送出程序期間的事件處理常式傳回`false`，可用來刪除記錄時顯示確認訊息方塊。</span><span class="sxs-lookup"><span data-stu-id="47abc-209">This functionality, coupled with a browser s behavior to cancel a form submission if an event handler during the submission process returns `false`, can be used to display a confirmation messagebox when deleting a record.</span></span>

<span data-ttu-id="47abc-210">`confirm(string)`函式可以是按鈕 Web 控制項的用戶端相關聯`onclick`事件處理常式透過控制項的`OnClientClick`屬性。</span><span class="sxs-lookup"><span data-stu-id="47abc-210">The `confirm(string)` function can be associated with a Button Web control s client-side `onclick` event handler through the control s `OnClientClick` property.</span></span> <span data-ttu-id="47abc-211">使用範本-其中一個 FormView 的範本中或在 DetailsView 或 GridView-TemplateField 中的 [刪除] 按鈕時這個屬性可以設定以宣告方式或以程式設計的方式，正如我們所見在本教學課程。</span><span class="sxs-lookup"><span data-stu-id="47abc-211">When working with a Delete button in a template - either in one of the FormView s templates or in a TemplateField in the DetailsView or GridView - this property can be set either declaratively or programmatically, as we saw in this tutorial.</span></span>

<span data-ttu-id="47abc-212">祝您程式設計 ！</span><span class="sxs-lookup"><span data-stu-id="47abc-212">Happy Programming!</span></span>

## <a name="about-the-author"></a><span data-ttu-id="47abc-213">關於作者</span><span class="sxs-lookup"><span data-stu-id="47abc-213">About the Author</span></span>

<span data-ttu-id="47abc-214">[Scott Mitchell](http://www.4guysfromrolla.com/ScottMitchell.shtml)，作者的七個 ASP/ASP.NET 書籍和的創辦[4GuysFromRolla.com](http://www.4guysfromrolla.com)，已從 1998 年使用 Microsoft Web 技術。</span><span class="sxs-lookup"><span data-stu-id="47abc-214">[Scott Mitchell](http://www.4guysfromrolla.com/ScottMitchell.shtml), author of seven ASP/ASP.NET books and founder of [4GuysFromRolla.com](http://www.4guysfromrolla.com), has been working with Microsoft Web technologies since 1998.</span></span> <span data-ttu-id="47abc-215">Scott 可做為獨立顧問、 訓練和寫入器。</span><span class="sxs-lookup"><span data-stu-id="47abc-215">Scott works as an independent consultant, trainer, and writer.</span></span> <span data-ttu-id="47abc-216">他最新的活頁簿[ *Sam 教導您自己 ASP.NET 2.0 24 小時內*](https://www.amazon.com/exec/obidos/ASIN/0672327384/4guysfromrollaco)。</span><span class="sxs-lookup"><span data-stu-id="47abc-216">His latest book is [*Sams Teach Yourself ASP.NET 2.0 in 24 Hours*](https://www.amazon.com/exec/obidos/ASIN/0672327384/4guysfromrollaco).</span></span> <span data-ttu-id="47abc-217">他可以在達到[ mitchell@4GuysFromRolla.com。](mailto:mitchell@4GuysFromRolla.com)或透過他的部落格，這可以在找到[http://ScottOnWriting.NET](http://ScottOnWriting.NET)。</span><span class="sxs-lookup"><span data-stu-id="47abc-217">He can be reached at [mitchell@4GuysFromRolla.com.](mailto:mitchell@4GuysFromRolla.com) or via his blog, which can be found at [http://ScottOnWriting.NET](http://ScottOnWriting.NET).</span></span>

>[!div class="step-by-step"]
<span data-ttu-id="47abc-218">[上一頁](implementing-optimistic-concurrency-vb.md)
[下一頁](limiting-data-modification-functionality-based-on-the-user-vb.md)</span><span class="sxs-lookup"><span data-stu-id="47abc-218">[Previous](implementing-optimistic-concurrency-vb.md)
[Next](limiting-data-modification-functionality-based-on-the-user-vb.md)</span></span>