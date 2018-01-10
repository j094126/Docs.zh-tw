---
uid: web-forms/overview/ajax-control-toolkit/textboxwatermark/using-textboxwatermark-in-a-formview-cs
title: "在 FormView 中 (C#) 中使用 TextBoxWatermark |Microsoft 文件"
author: wenz
description: "AJAX Control Toolkit TextBoxWatermark 控制項擴充文字方塊中，使方塊內會顯示文字。 當使用者按一下 [到] 方塊，它我..."
ms.author: aspnetcontent
manager: wpickett
ms.date: 06/02/2008
ms.topic: article
ms.assetid: e6ee90bf-32a5-4987-a384-15cc7dd30c8a
ms.technology: dotnet-webforms
ms.prod: .net-framework
msc.legacyurl: /web-forms/overview/ajax-control-toolkit/textboxwatermark/using-textboxwatermark-in-a-formview-cs
msc.type: authoredcontent
ms.openlocfilehash: 805ba26720fa7faed82101076ff84e576f4cfcf2
ms.sourcegitcommit: 9a9483aceb34591c97451997036a9120c3fe2baf
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/10/2017
---
<a name="using-textboxwatermark-in-a-formview-c"></a><span data-ttu-id="f5e36-104">使用 TextBoxWatermark 在 FormView 中 (C#)</span><span class="sxs-lookup"><span data-stu-id="f5e36-104">Using TextBoxWatermark in a FormView (C#)</span></span>
====================
<span data-ttu-id="f5e36-105">由[Christian Wenz](https://github.com/wenz)</span><span class="sxs-lookup"><span data-stu-id="f5e36-105">by [Christian Wenz](https://github.com/wenz)</span></span>

<span data-ttu-id="f5e36-106">[下載程式碼](http://download.microsoft.com/download/9/3/f/93f8daea-bebd-4821-833b-95205389c7d0/TextBoxWatermark1.cs.zip)或[下載 PDF](http://download.microsoft.com/download/b/6/a/b6ae89ee-df69-4c87-9bfb-ad1eb2b23373/textboxwatermark1CS.pdf)</span><span class="sxs-lookup"><span data-stu-id="f5e36-106">[Download Code](http://download.microsoft.com/download/9/3/f/93f8daea-bebd-4821-833b-95205389c7d0/TextBoxWatermark1.cs.zip) or [Download PDF](http://download.microsoft.com/download/b/6/a/b6ae89ee-df69-4c87-9bfb-ad1eb2b23373/textboxwatermark1CS.pdf)</span></span>

> <span data-ttu-id="f5e36-107">AJAX Control Toolkit TextBoxWatermark 控制項擴充文字方塊中，使方塊內會顯示文字。</span><span class="sxs-lookup"><span data-stu-id="f5e36-107">The TextBoxWatermark control in the AJAX Control Toolkit extends a text box so that a text is displayed within the box.</span></span> <span data-ttu-id="f5e36-108">當使用者按一下此方塊時，它會清空。</span><span class="sxs-lookup"><span data-stu-id="f5e36-108">When a user clicks into the box, it is emptied.</span></span> <span data-ttu-id="f5e36-109">如果使用者離開 [] 方塊中，而不需輸入文字，其中已預先的文字會重新出現。</span><span class="sxs-lookup"><span data-stu-id="f5e36-109">If the user leaves the box without entering text, the prefilled text reappears.</span></span> <span data-ttu-id="f5e36-110">這也是可能在 FormView 控制項內。</span><span class="sxs-lookup"><span data-stu-id="f5e36-110">This is also possible within a FormView control.</span></span>


## <a name="overview"></a><span data-ttu-id="f5e36-111">概觀</span><span class="sxs-lookup"><span data-stu-id="f5e36-111">Overview</span></span>

<span data-ttu-id="f5e36-112">`TextBoxWatermark` AJAX Control Toolkit 中的控制會擴充文字方塊中，使方塊內會顯示文字。</span><span class="sxs-lookup"><span data-stu-id="f5e36-112">The `TextBoxWatermark` control in the AJAX Control Toolkit extends a text box so that a text is displayed within the box.</span></span> <span data-ttu-id="f5e36-113">當使用者按一下此方塊時，它會清空。</span><span class="sxs-lookup"><span data-stu-id="f5e36-113">When a user clicks into the box, it is emptied.</span></span> <span data-ttu-id="f5e36-114">如果使用者離開 [] 方塊中，而不需輸入文字，其中已預先的文字會重新出現。</span><span class="sxs-lookup"><span data-stu-id="f5e36-114">If the user leaves the box without entering text, the prefilled text reappears.</span></span> <span data-ttu-id="f5e36-115">這也可能在`FormView`控制項。</span><span class="sxs-lookup"><span data-stu-id="f5e36-115">This is also possible within a `FormView` control.</span></span>

## <a name="steps"></a><span data-ttu-id="f5e36-116">步驟</span><span class="sxs-lookup"><span data-stu-id="f5e36-116">Steps</span></span>

<span data-ttu-id="f5e36-117">首先，資料來源是必要的。</span><span class="sxs-lookup"><span data-stu-id="f5e36-117">First of all, a data source is required.</span></span> <span data-ttu-id="f5e36-118">這個範例會使用 AdventureWorks 資料庫和 Microsoft SQL Server 2005 Express Edition。</span><span class="sxs-lookup"><span data-stu-id="f5e36-118">This sample uses the AdventureWorks database and the Microsoft SQL Server 2005 Express Edition.</span></span> <span data-ttu-id="f5e36-119">資料庫是選擇性的一部分 （包括 express edition） 的 Visual Studio 安裝，也會在個別下載[https://go.microsoft.com/fwlink/?LinkId=64064](https://go.microsoft.com/fwlink/?LinkId=64064)。</span><span class="sxs-lookup"><span data-stu-id="f5e36-119">The database is an optional part of a Visual Studio installation (including express edition) and is also available as a separate download under [https://go.microsoft.com/fwlink/?LinkId=64064](https://go.microsoft.com/fwlink/?LinkId=64064).</span></span> <span data-ttu-id="f5e36-120">AdventureWorks 資料庫是 SQL Server 2005 範例和範例資料庫的一部分 (從下列網址下載[https://www.microsoft.com/downloads/details.aspx?FamilyID=e719ecf7-9f46-4312-af89-6ad8702e4e6e&amp;DisplayLang = en](https://www.microsoft.com/downloads/details.aspx?FamilyID=e719ecf7-9f46-4312-af89-6ad8702e4e6e&amp;DisplayLang=en))。</span><span class="sxs-lookup"><span data-stu-id="f5e36-120">The AdventureWorks database is part of the SQL Server 2005 Samples and Sample Databases (download at [https://www.microsoft.com/downloads/details.aspx?FamilyID=e719ecf7-9f46-4312-af89-6ad8702e4e6e&amp;DisplayLang=en](https://www.microsoft.com/downloads/details.aspx?FamilyID=e719ecf7-9f46-4312-af89-6ad8702e4e6e&amp;DisplayLang=en)).</span></span> <span data-ttu-id="f5e36-121">若要設定資料庫的最簡單方式是使用 Microsoft SQL Server Management Studio Express ([https://www.microsoft.com/downloads/details.aspx?FamilyID = c243a5ae-4bd1-4e3d-94b8-5a0f62bf7796&amp;DisplayLang = en](https://www.microsoft.com/downloads/details.aspx?FamilyID=c243a5ae-4bd1-4e3d-94b8-5a0f62bf7796&amp;DisplayLang=en)) 和附加`AdventureWorks.mdf`資料庫檔案。</span><span class="sxs-lookup"><span data-stu-id="f5e36-121">The easiest way to set the database up is to use the Microsoft SQL Server Management Studio Express ([https://www.microsoft.com/downloads/details.aspx?FamilyID=c243a5ae-4bd1-4e3d-94b8-5a0f62bf7796&amp;DisplayLang=en](https://www.microsoft.com/downloads/details.aspx?FamilyID=c243a5ae-4bd1-4e3d-94b8-5a0f62bf7796&amp;DisplayLang=en)) and attach the `AdventureWorks.mdf` database file.</span></span>

<span data-ttu-id="f5e36-122">此範例中，我們假設 SQL Server 2005 Express Edition 執行個體稱為`SQLEXPRESS`位在同一部電腦與網頁伺服器; 這也是預設的安裝程式。</span><span class="sxs-lookup"><span data-stu-id="f5e36-122">For this sample, we assume that the instance of the SQL Server 2005 Express Edition is called `SQLEXPRESS` and resides on the same machine as the web server; this is also the default setup.</span></span> <span data-ttu-id="f5e36-123">如果您的設定不同，您必須調整資料庫的連接資訊。</span><span class="sxs-lookup"><span data-stu-id="f5e36-123">If your setup differs, you have to adapt the connection information for the database.</span></span>

<span data-ttu-id="f5e36-124">為了啟用 ASP.NET AJAX 和控制工具組的功能`ScriptManager`控制項必須任意位置放置在頁面 (但內部`<form>`項目):</span><span class="sxs-lookup"><span data-stu-id="f5e36-124">In order to activate the functionality of ASP.NET AJAX and the Control Toolkit, the `ScriptManager` control must be put anywhere on the page (but within the `<form>` element):</span></span>

[!code-aspx[Main](using-textboxwatermark-in-a-formview-cs/samples/sample1.aspx)]

<span data-ttu-id="f5e36-125">然後，將資料來源加入至 [支援] 頁面`DELETE`，`INSERT`和`UPDATE`SQL 陳述式。</span><span class="sxs-lookup"><span data-stu-id="f5e36-125">Then, add a data source to the page which supports the `DELETE`, `INSERT` and `UPDATE` SQL statements.</span></span> <span data-ttu-id="f5e36-126">如果您使用 Visual Studio 小幫手來建立資料來源，請記住，請在目前版本中的 bug 不前置資料表名稱 (`Vendor`) 與`Purchasing`。</span><span class="sxs-lookup"><span data-stu-id="f5e36-126">If you are using the Visual Studio assistant to create the data source, mind that a bug in the current version does not prefix the table name (`Vendor`) with `Purchasing`.</span></span> <span data-ttu-id="f5e36-127">下列標記會顯示正確的語法：</span><span class="sxs-lookup"><span data-stu-id="f5e36-127">The following markup shows the correct syntax:</span></span>

[!code-aspx[Main](using-textboxwatermark-in-a-formview-cs/samples/sample2.aspx)]

<span data-ttu-id="f5e36-128">請記住的名稱 (`ID`) 的資料來源，因為它會用於`DataSourceID`屬性`FormView`控制項。</span><span class="sxs-lookup"><span data-stu-id="f5e36-128">Remember the name (`ID`) of the data source, since it will be used in the `DataSourceID` property of the `FormView` control.</span></span> <span data-ttu-id="f5e36-129">`<InsertItemTemplate>`區段`FormView`包含文字方塊中，後者由擴充`TextBoxWatermarkExtender`控制項。</span><span class="sxs-lookup"><span data-stu-id="f5e36-129">The `<InsertItemTemplate>` section of the `FormView` contains a textbox which is extended by the `TextBoxWatermarkExtender` control.</span></span> <span data-ttu-id="f5e36-130">請確定 extender 位於範本內且不該期間之外。</span><span class="sxs-lookup"><span data-stu-id="f5e36-130">Make sure that the extender resides within the template and not outside of it.</span></span>

[!code-aspx[Main](using-textboxwatermark-in-a-formview-cs/samples/sample3.aspx)]

<span data-ttu-id="f5e36-131">現在當使用者變更的插入模式為`FormView`控制項，在文字欄位感謝來使新的供應商`TextBoxWatermarkExtender`控制項。</span><span class="sxs-lookup"><span data-stu-id="f5e36-131">Now when the user changes into the insert mode of the `FormView` control, the text field for the new vendor is prefilled thanks to the `TextBoxWatermarkExtender` control.</span></span> <span data-ttu-id="f5e36-132">在文字方塊中按一下可讓您填入文字會消失。</span><span class="sxs-lookup"><span data-stu-id="f5e36-132">A click inside the textbox lets the filler text disappear.</span></span>


<span data-ttu-id="f5e36-133">[![在欄位中的浮水印來自擴充項](using-textboxwatermark-in-a-formview-cs/_static/image2.png)](using-textboxwatermark-in-a-formview-cs/_static/image1.png)</span><span class="sxs-lookup"><span data-stu-id="f5e36-133">[![The watermark in the field comes from the extender](using-textboxwatermark-in-a-formview-cs/_static/image2.png)](using-textboxwatermark-in-a-formview-cs/_static/image1.png)</span></span>

<span data-ttu-id="f5e36-134">在欄位中的浮水印來自擴充項 ([按一下以檢視完整大小的影像](using-textboxwatermark-in-a-formview-cs/_static/image3.png))</span><span class="sxs-lookup"><span data-stu-id="f5e36-134">The watermark in the field comes from the extender ([Click to view full-size image](using-textboxwatermark-in-a-formview-cs/_static/image3.png))</span></span>

>[!div class="step-by-step"]
[<span data-ttu-id="f5e36-135">下一步</span><span class="sxs-lookup"><span data-stu-id="f5e36-135">Next</span></span>](using-textboxwatermark-with-validation-controls-cs.md)