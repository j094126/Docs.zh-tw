---
title: "在 Windows 服務的主機"
author: tdykstra
description: "了解如何裝載在 Windows 服務的 ASP.NET Core 應用程式。"
manager: wpickett
ms.author: tdykstra
ms.custom: mvc
ms.date: 01/30/2018
ms.prod: aspnet-core
ms.technology: aspnet
ms.topic: article
uid: host-and-deploy/windows-service
ms.openlocfilehash: c14a1f62bce4d06be3b8e6356f45cd5e330a0751
ms.sourcegitcommit: f2a11a89037471a77ad68a67533754b7bb8303e2
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/01/2018
---
# <a name="host-an-aspnet-core-app-in-a-windows-service"></a><span data-ttu-id="9e42f-103">裝載在 Windows 服務的 ASP.NET Core 應用程式</span><span class="sxs-lookup"><span data-stu-id="9e42f-103">Host an ASP.NET Core app in a Windows Service</span></span>

<span data-ttu-id="9e42f-104">由[Tom Dykstra](https://github.com/tdykstra)</span><span class="sxs-lookup"><span data-stu-id="9e42f-104">By [Tom Dykstra](https://github.com/tdykstra)</span></span>

<span data-ttu-id="9e42f-105">裝載在 Windows 上的 ASP.NET Core 應用程式，而不使用 IIS 執行它的建議的方式[Windows 服務](/dotnet/framework/windows-services/introduction-to-windows-service-applications)。</span><span class="sxs-lookup"><span data-stu-id="9e42f-105">The recommended way to host an ASP.NET Core app on Windows without using IIS is to run it in a [Windows Service](/dotnet/framework/windows-services/introduction-to-windows-service-applications).</span></span> <span data-ttu-id="9e42f-106">當裝載為 Windows 服務，應用程式可以自動啟動之後重新開機，而不需要人為介入損毀。</span><span class="sxs-lookup"><span data-stu-id="9e42f-106">When hosted as a Windows Service, the app can automatically start after reboots and crashes without requiring human intervention.</span></span>

<span data-ttu-id="9e42f-107">[檢視或下載範例程式碼](https://github.com/aspnet/Docs/tree/master/aspnetcore/host-and-deploy/windows-service/sample)([如何下載](xref:tutorials/index#how-to-download-a-sample))。</span><span class="sxs-lookup"><span data-stu-id="9e42f-107">[View or download sample code](https://github.com/aspnet/Docs/tree/master/aspnetcore/host-and-deploy/windows-service/sample) ([how to download](xref:tutorials/index#how-to-download-a-sample)).</span></span> <span data-ttu-id="9e42f-108">如需如何執行範例應用程式的指示，請參閱 「 範例的*README.md*檔案。</span><span class="sxs-lookup"><span data-stu-id="9e42f-108">For instructions on how to run the sample app, see the sample's *README.md* file.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="9e42f-109">必要條件</span><span class="sxs-lookup"><span data-stu-id="9e42f-109">Prerequisites</span></span>

* <span data-ttu-id="9e42f-110">應用程式必須執行.NET Framework 執行階段。</span><span class="sxs-lookup"><span data-stu-id="9e42f-110">The app must run on the .NET Framework runtime.</span></span> <span data-ttu-id="9e42f-111">在*.csproj*檔案中，指定適當的值[TargetFramework](/nuget/schema/target-frameworks)和[RuntimeIdentifier](/dotnet/articles/core/rid-catalog)。</span><span class="sxs-lookup"><span data-stu-id="9e42f-111">In the *.csproj* file, specify appropriate values for [TargetFramework](/nuget/schema/target-frameworks) and [RuntimeIdentifier](/dotnet/articles/core/rid-catalog).</span></span> <span data-ttu-id="9e42f-112">以下為範例：</span><span class="sxs-lookup"><span data-stu-id="9e42f-112">Here's an example:</span></span>

  [!code-xml[](windows-service/sample/AspNetCoreService.csproj?range=3-6)]

  <span data-ttu-id="9e42f-113">使用 Visual Studio 中建立專案時**ASP.NET Core 應用程式 (.NET Framework)**範本。</span><span class="sxs-lookup"><span data-stu-id="9e42f-113">When creating a project in Visual Studio, use the **ASP.NET Core Application (.NET Framework)** template.</span></span>

* <span data-ttu-id="9e42f-114">如果應用程式接收要求，從網際網路 （不只是從內部網路），則必須使用[HTTP.sys](xref:fundamentals/servers/httpsys)網頁伺服器 (以前稱為[WebListener](xref:fundamentals/servers/weblistener) ASP.NET Core 1.x 應用程式) 而不是[Kestrel](xref:fundamentals/servers/kestrel)。</span><span class="sxs-lookup"><span data-stu-id="9e42f-114">If the app receives requests from the Internet (not just from an internal network), it must use the [HTTP.sys](xref:fundamentals/servers/httpsys) web server (formerly known as [WebListener](xref:fundamentals/servers/weblistener) for ASP.NET Core 1.x apps) rather than [Kestrel](xref:fundamentals/servers/kestrel).</span></span> <span data-ttu-id="9e42f-115">IIS 建議是作為反向 proxy 伺服器使用 Kestrel 邊緣部署。</span><span class="sxs-lookup"><span data-stu-id="9e42f-115">IIS is recommended for use as a reverse proxy server with Kestrel for edge deployments.</span></span> <span data-ttu-id="9e42f-116">如需詳細資訊，請參閱[何時搭配使用 Kestrel 與反向 Proxy](xref:fundamentals/servers/kestrel#when-to-use-kestrel-with-a-reverse-proxy)。</span><span class="sxs-lookup"><span data-stu-id="9e42f-116">For more information, see [When to use Kestrel with a reverse proxy](xref:fundamentals/servers/kestrel#when-to-use-kestrel-with-a-reverse-proxy).</span></span>

## <a name="getting-started"></a><span data-ttu-id="9e42f-117">使用者入門</span><span class="sxs-lookup"><span data-stu-id="9e42f-117">Getting started</span></span>

<span data-ttu-id="9e42f-118">本節說明將現有的 ASP.NET Core 專案設定為在服務執行所需的最小變更。</span><span class="sxs-lookup"><span data-stu-id="9e42f-118">This section explains the minimum changes required to set up an existing ASP.NET Core project to run in a service.</span></span>

1. <span data-ttu-id="9e42f-119">安裝 NuGet 套件[Microsoft.AspNetCore.Hosting.WindowsServices](https://www.nuget.org/packages/Microsoft.AspNetCore.Hosting.WindowsServices/)。</span><span class="sxs-lookup"><span data-stu-id="9e42f-119">Install the NuGet package [Microsoft.AspNetCore.Hosting.WindowsServices](https://www.nuget.org/packages/Microsoft.AspNetCore.Hosting.WindowsServices/).</span></span>

1. <span data-ttu-id="9e42f-120">在變更下列`Program.Main`:</span><span class="sxs-lookup"><span data-stu-id="9e42f-120">Make the following changes in `Program.Main`:</span></span>
  
   * <span data-ttu-id="9e42f-121">呼叫`host.RunAsService`而不是`host.Run`。</span><span class="sxs-lookup"><span data-stu-id="9e42f-121">Call `host.RunAsService` instead of `host.Run`.</span></span>
  
   * <span data-ttu-id="9e42f-122">如果程式碼會呼叫`UseContentRoot`，發行位置，而不是使用路徑`Directory.GetCurrentDirectory()`。</span><span class="sxs-lookup"><span data-stu-id="9e42f-122">If the code calls `UseContentRoot`, use a path to the publish location instead of `Directory.GetCurrentDirectory()`.</span></span>

   # <a name="aspnet-core-2xtabaspnetcore2x"></a>[<span data-ttu-id="9e42f-123">ASP.NET Core 2.x</span><span class="sxs-lookup"><span data-stu-id="9e42f-123">ASP.NET Core 2.x</span></span>](#tab/aspnetcore2x)

   [!code-csharp[](windows-service/sample/Program.cs?name=ServiceOnly&highlight=3-4,7,12)]

   # <a name="aspnet-core-1xtabaspnetcore1x"></a>[<span data-ttu-id="9e42f-124">ASP.NET Core 1.x</span><span class="sxs-lookup"><span data-stu-id="9e42f-124">ASP.NET Core 1.x</span></span>](#tab/aspnetcore1x)

   [!code-csharp[](windows-service/sample_snapshot/Program.cs?name=ServiceOnly&highlight=3-4,8,14)]

   ---

1. <span data-ttu-id="9e42f-125">發行應用程式的資料夾。</span><span class="sxs-lookup"><span data-stu-id="9e42f-125">Publish the app to a folder.</span></span> <span data-ttu-id="9e42f-126">使用[dotnet 發行](/dotnet/articles/core/tools/dotnet-publish)或[Visual Studio 發行設定檔](xref:host-and-deploy/visual-studio-publish-profiles)所發行的資料夾。</span><span class="sxs-lookup"><span data-stu-id="9e42f-126">Use [dotnet publish](/dotnet/articles/core/tools/dotnet-publish) or a [Visual Studio publish profile](xref:host-and-deploy/visual-studio-publish-profiles) that publishes to a folder.</span></span>

1. <span data-ttu-id="9e42f-127">測試建立和啟動服務。</span><span class="sxs-lookup"><span data-stu-id="9e42f-127">Test by creating and starting the service.</span></span>

   <span data-ttu-id="9e42f-128">若要使用的系統管理權限開啟命令殼層[sc.exe](https://technet.microsoft.com/library/bb490995)命令列工具來建立及啟動服務。</span><span class="sxs-lookup"><span data-stu-id="9e42f-128">Open a command shell with administrative privileges to use the [sc.exe](https://technet.microsoft.com/library/bb490995) command-line tool to create and start a service.</span></span> <span data-ttu-id="9e42f-129">如果服務為 MyService，發行至`c:\svc`，及名為 AspNetCoreService，命令：</span><span class="sxs-lookup"><span data-stu-id="9e42f-129">If the service is named MyService, published to `c:\svc`, and named AspNetCoreService, the commands are:</span></span>

   ```console
   sc create MyService binPath="c:\svc\aspnetcoreservice.exe"
   sc start MyService
   ```

   <span data-ttu-id="9e42f-130">`binPath`值是應用程式的可執行檔，其中包含可執行檔名稱的路徑。</span><span class="sxs-lookup"><span data-stu-id="9e42f-130">The `binPath` value is the path to the app's executable, which includes the executable file name.</span></span>

   ![主控台視窗建立及啟動範例](windows-service/_static/create-start.png)

   <span data-ttu-id="9e42f-132">當命令完成時，瀏覽至與相同的路徑做為主控台應用程式執行時 (根據預設， `http://localhost:5000`):</span><span class="sxs-lookup"><span data-stu-id="9e42f-132">When these commands finish, browse to the same path as when running as a console app (by default, `http://localhost:5000`):</span></span>

   ![在服務中執行](windows-service/_static/running-in-service.png)

## <a name="provide-a-way-to-run-outside-of-a-service"></a><span data-ttu-id="9e42f-134">提供服務外部執行的方式</span><span class="sxs-lookup"><span data-stu-id="9e42f-134">Provide a way to run outside of a service</span></span>

<span data-ttu-id="9e42f-135">更輕鬆地測試和偵錯時執行外部服務，所以可將呼叫的程式碼加入`RunAsService`只在某些情況下。</span><span class="sxs-lookup"><span data-stu-id="9e42f-135">It's easier to test and debug when running outside of a service, so it's customary to add code that calls `RunAsService` only under certain conditions.</span></span> <span data-ttu-id="9e42f-136">例如，應用程式可以執行為主控台應用程式與`--console`命令列引數，或是如果附加偵錯工具：</span><span class="sxs-lookup"><span data-stu-id="9e42f-136">For example, the app can run as a console app with a `--console` command-line argument or if the debugger is attached:</span></span>

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[<span data-ttu-id="9e42f-137">ASP.NET Core 2.x</span><span class="sxs-lookup"><span data-stu-id="9e42f-137">ASP.NET Core 2.x</span></span>](#tab/aspnetcore2x)

[!code-csharp[](windows-service/sample/Program.cs?name=ServiceOrConsole)]

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[<span data-ttu-id="9e42f-138">ASP.NET Core 1.x</span><span class="sxs-lookup"><span data-stu-id="9e42f-138">ASP.NET Core 1.x</span></span>](#tab/aspnetcore1x)

[!code-csharp[](windows-service/sample_snapshot/Program.cs?name=ServiceOrConsole)]

---

## <a name="handle-stopping-and-starting-events"></a><span data-ttu-id="9e42f-139">停止和啟動的事件處理</span><span class="sxs-lookup"><span data-stu-id="9e42f-139">Handle stopping and starting events</span></span>

<span data-ttu-id="9e42f-140">若要處理`OnStarting`， `OnStarted`，和`OnStopping`事件，進行下列的其他變更：</span><span class="sxs-lookup"><span data-stu-id="9e42f-140">To handle `OnStarting`, `OnStarted`, and `OnStopping` events, make the following additional changes:</span></span>

1. <span data-ttu-id="9e42f-141">建立衍生自類別`WebHostService`:</span><span class="sxs-lookup"><span data-stu-id="9e42f-141">Create a class that derives from `WebHostService`:</span></span>

   [!code-csharp[](windows-service/sample/CustomWebHostService.cs?name=NoLogging)]

1. <span data-ttu-id="9e42f-142">建立擴充方法給`IWebHost`傳遞之任何自訂`WebHostService`至`ServiceBase.Run`:</span><span class="sxs-lookup"><span data-stu-id="9e42f-142">Create an extension method for `IWebHost` that passes the custom `WebHostService` to `ServiceBase.Run`:</span></span>

   [!code-csharp[](windows-service/sample/WebHostServiceExtensions.cs?name=ExtensionsClass)]

1. <span data-ttu-id="9e42f-143">在`Program.Main`，呼叫新的擴充方法， `RunAsCustomService`，而不是`RunAsService`:</span><span class="sxs-lookup"><span data-stu-id="9e42f-143">In `Program.Main`, call the new extension method, `RunAsCustomService`, instead of `RunAsService`:</span></span>

   # <a name="aspnet-core-2xtabaspnetcore2x"></a>[<span data-ttu-id="9e42f-144">ASP.NET Core 2.x</span><span class="sxs-lookup"><span data-stu-id="9e42f-144">ASP.NET Core 2.x</span></span>](#tab/aspnetcore2x)

   [!code-csharp[](windows-service/sample/Program.cs?name=HandleStopStart&highlight=24)]

   # <a name="aspnet-core-1xtabaspnetcore1x"></a>[<span data-ttu-id="9e42f-145">ASP.NET Core 1.x</span><span class="sxs-lookup"><span data-stu-id="9e42f-145">ASP.NET Core 1.x</span></span>](#tab/aspnetcore1x)

   [!code-csharp[](windows-service/sample_snapshot/Program.cs?name=HandleStopStart&highlight=26)]

   ---

<span data-ttu-id="9e42f-146">如果自訂`WebHostService`程式碼需要從相依性插入 （例如記錄器） 服務，取得從`Services`屬性`IWebHost`:</span><span class="sxs-lookup"><span data-stu-id="9e42f-146">If the custom `WebHostService` code requires a service from dependency injection (such as a logger), obtain it from the `Services` property of `IWebHost`:</span></span>

[!code-csharp[](windows-service/sample/CustomWebHostService.cs?name=Logging&highlight=7)]

## <a name="acknowledgments"></a><span data-ttu-id="9e42f-147">通知</span><span class="sxs-lookup"><span data-stu-id="9e42f-147">Acknowledgments</span></span>

<span data-ttu-id="9e42f-148">撰寫本文時的協助的已發行的來源：</span><span class="sxs-lookup"><span data-stu-id="9e42f-148">This article was written with the help of published sources:</span></span>

* [<span data-ttu-id="9e42f-149">為 Windows 服務裝載的 ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="9e42f-149">Hosting ASP.NET Core as Windows service</span></span>](https://stackoverflow.com/questions/37346383/hosting-asp-net-core-as-windows-service/37464074)
* [<span data-ttu-id="9e42f-150">如何在裝載您的 ASP.NET 核心 Windows 服務中</span><span class="sxs-lookup"><span data-stu-id="9e42f-150">How to host your ASP.NET Core in a Windows Service</span></span>](https://dotnetthoughts.net/how-to-host-your-aspnet-core-in-a-windows-service/)