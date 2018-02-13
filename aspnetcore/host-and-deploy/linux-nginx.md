---
title: "在 Linux 上使用 Nginx 裝載 ASP.NET Core"
author: rick-anderson
description: "描述如何設定 Nginx 做 Ubuntu 16.04 轉送 Kestrel 上執行 ASP.NET Core web 應用程式的 HTTP 流量上的反向 proxy。"
manager: wpickett
ms.author: riande
ms.custom: mvc
ms.date: 08/21/2017
ms.prod: asp.net-core
ms.technology: aspnet
ms.topic: article
uid: host-and-deploy/linux-nginx
ms.openlocfilehash: 1044a87a4dcc7636413078b0fc09ade206c97d0a
ms.sourcegitcommit: b83a5f731a9c02bdb1cc1e3f9a8bf273eb5b33e0
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/11/2018
---
# <a name="host-aspnet-core-on-linux-with-nginx"></a><span data-ttu-id="3df1e-103">在 Linux 上使用 Nginx 裝載 ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="3df1e-103">Host ASP.NET Core on Linux with Nginx</span></span>

<span data-ttu-id="3df1e-104">作者：[Sourabh Shirhatti](https://twitter.com/sshirhatti)</span><span class="sxs-lookup"><span data-stu-id="3df1e-104">By [Sourabh Shirhatti](https://twitter.com/sshirhatti)</span></span>

<span data-ttu-id="3df1e-105">本指南說明在 Ubuntu 16.04 伺服器上設定生產環境就緒的 ASP.NET Core 環境。</span><span class="sxs-lookup"><span data-stu-id="3df1e-105">This guide explains setting up a production-ready ASP.NET Core environment on an Ubuntu 16.04 Server.</span></span>

<span data-ttu-id="3df1e-106">**注意：** For Ubuntu 14.04 *supervisord*建議做為監視 Kestrel 程序的解決方案。</span><span class="sxs-lookup"><span data-stu-id="3df1e-106">**Note:** For Ubuntu 14.04, *supervisord* is recommended as a solution for monitoring the Kestrel process.</span></span> <span data-ttu-id="3df1e-107">*systemd* Ubuntu 14.04 上無法使用。</span><span class="sxs-lookup"><span data-stu-id="3df1e-107">*systemd* isn't available on Ubuntu 14.04.</span></span> [<span data-ttu-id="3df1e-108">請參閱本文件的舊版本</span><span class="sxs-lookup"><span data-stu-id="3df1e-108">See previous version of this document</span></span>](https://github.com/aspnet/Docs/blob/e9c1419175c4dd7e152df3746ba1df5935aaafd5/aspnetcore/publishing/linuxproduction.md)

<span data-ttu-id="3df1e-109">本指南：</span><span class="sxs-lookup"><span data-stu-id="3df1e-109">This guide:</span></span>

* <span data-ttu-id="3df1e-110">將現有的 ASP.NET Core 應用程式的反向 proxy 伺服器後方。</span><span class="sxs-lookup"><span data-stu-id="3df1e-110">Places an existing ASP.NET Core app behind a reverse proxy server.</span></span>
* <span data-ttu-id="3df1e-111">設定反向 proxy 伺服器，將要求轉送到 Kestrel web 伺服器。</span><span class="sxs-lookup"><span data-stu-id="3df1e-111">Sets up the reverse proxy server to forward requests to the Kestrel web server.</span></span>
* <span data-ttu-id="3df1e-112">可確保 web 應用程式啟動時執行的服務精靈為。</span><span class="sxs-lookup"><span data-stu-id="3df1e-112">Ensures the web app runs on startup as a daemon.</span></span>
* <span data-ttu-id="3df1e-113">設定可協助您重新啟動 web 應用程式的程序管理工具。</span><span class="sxs-lookup"><span data-stu-id="3df1e-113">Configures a process management tool to help restart the web app.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="3df1e-114">必要條件</span><span class="sxs-lookup"><span data-stu-id="3df1e-114">Prerequisites</span></span>

1. <span data-ttu-id="3df1e-115">以 sudo 權限使用標準使用者帳戶存取 Ubuntu 16.04 伺服器</span><span class="sxs-lookup"><span data-stu-id="3df1e-115">Access to an Ubuntu 16.04 Server with a standard user account with sudo privilege</span></span>
1. <span data-ttu-id="3df1e-116">現有的 ASP.NET Core 應用程式</span><span class="sxs-lookup"><span data-stu-id="3df1e-116">An existing ASP.NET Core app</span></span>

## <a name="copy-over-the-app"></a><span data-ttu-id="3df1e-117">複製應用程式</span><span class="sxs-lookup"><span data-stu-id="3df1e-117">Copy over the app</span></span>

<span data-ttu-id="3df1e-118">從開發環境執行 `dotnet publish`，將應用程式封裝到可在伺服器執行的獨立目錄中。</span><span class="sxs-lookup"><span data-stu-id="3df1e-118">Run `dotnet publish` from the dev environment to package an app into a self-contained directory that can run on the server.</span></span>

<span data-ttu-id="3df1e-119">將 ASP.NET Core 應用程式複製到使用任何工具整合到組織的工作流程 （例如，SCP，FTP） 伺服器。</span><span class="sxs-lookup"><span data-stu-id="3df1e-119">Copy the ASP.NET Core app to the server using whatever tool integrates into the organization's workflow (for example, SCP, FTP).</span></span> <span data-ttu-id="3df1e-120">測試應用程式，例如：</span><span class="sxs-lookup"><span data-stu-id="3df1e-120">Test the app, for example:</span></span>

* <span data-ttu-id="3df1e-121">從命令列執行`dotnet <app_assembly>.dll`。</span><span class="sxs-lookup"><span data-stu-id="3df1e-121">From the command line, run `dotnet <app_assembly>.dll`.</span></span>
* <span data-ttu-id="3df1e-122">在瀏覽器中，巡覽至 `http://<serveraddress>:<port>` 以確認應用程式可在 Linux 上正常運作。</span><span class="sxs-lookup"><span data-stu-id="3df1e-122">In a browser, navigate to `http://<serveraddress>:<port>` to verify the app works on Linux.</span></span> 
 
## <a name="configure-a-reverse-proxy-server"></a><span data-ttu-id="3df1e-123">設定反向 Proxy 伺服器</span><span class="sxs-lookup"><span data-stu-id="3df1e-123">Configure a reverse proxy server</span></span>

<span data-ttu-id="3df1e-124">反向 proxy 是常見的安裝程式來服務動態 web 應用程式。</span><span class="sxs-lookup"><span data-stu-id="3df1e-124">A reverse proxy is a common setup for serving dynamic web apps.</span></span> <span data-ttu-id="3df1e-125">反向 proxy 終止 HTTP 要求，並將它轉送至 ASP.NET Core 應用程式。</span><span class="sxs-lookup"><span data-stu-id="3df1e-125">A reverse proxy terminates the HTTP request and forwards it to the ASP.NET Core app.</span></span>

### <a name="why-use-a-reverse-proxy-server"></a><span data-ttu-id="3df1e-126">為何使用反向 Proxy 伺服器？</span><span class="sxs-lookup"><span data-stu-id="3df1e-126">Why use a reverse proxy server?</span></span>

<span data-ttu-id="3df1e-127">Kestrel 適合用來從 ASP.NET Core 提供動態內容。</span><span class="sxs-lookup"><span data-stu-id="3df1e-127">Kestrel is great for serving dynamic content from ASP.NET Core.</span></span> <span data-ttu-id="3df1e-128">不過，web 服務功能不是做為伺服器，如 IIS、 Apache 或 Nginx 豐富的功能。</span><span class="sxs-lookup"><span data-stu-id="3df1e-128">However, the web serving capabilities aren't as feature rich as servers such as IIS, Apache, or Nginx.</span></span> <span data-ttu-id="3df1e-129">反向 proxy 伺服器可以卸載提供靜態內容、 快取要求、 壓縮要求，以及從 HTTP 伺服器的 SSL 終止的工作。</span><span class="sxs-lookup"><span data-stu-id="3df1e-129">A reverse proxy server can offload work such as serving static content, caching requests, compressing requests, and SSL termination from the HTTP server.</span></span> <span data-ttu-id="3df1e-130">反向 Proxy 伺服器可能位在專用電腦上，或可能與 HTTP 伺服器一起部署。</span><span class="sxs-lookup"><span data-stu-id="3df1e-130">A reverse proxy server may reside on a dedicated machine or may be deployed alongside an HTTP server.</span></span>

<span data-ttu-id="3df1e-131">為達到本指南的目的，使用 Nginx 的單一執行個體。</span><span class="sxs-lookup"><span data-stu-id="3df1e-131">For the purposes of this guide, a single instance of Nginx is used.</span></span> <span data-ttu-id="3df1e-132">它會在相同的伺服器上和 HTTP 伺服器一起執行。</span><span class="sxs-lookup"><span data-stu-id="3df1e-132">It runs on the same server, alongside the HTTP server.</span></span> <span data-ttu-id="3df1e-133">根據需求，不同的安裝程式可能會選擇。</span><span class="sxs-lookup"><span data-stu-id="3df1e-133">Based on requirements, a different setup may be choosen.</span></span>

<span data-ttu-id="3df1e-134">要求會透過反向 proxy 來轉送的因為使用轉送標頭中介軟體從[Microsoft.AspNetCore.HttpOverrides](https://www.nuget.org/packages/Microsoft.AspNetCore.HttpOverrides/)封裝。</span><span class="sxs-lookup"><span data-stu-id="3df1e-134">Because requests are forwarded by reverse proxy, use the Forwarded Headers Middleware from the [Microsoft.AspNetCore.HttpOverrides](https://www.nuget.org/packages/Microsoft.AspNetCore.HttpOverrides/) package.</span></span> <span data-ttu-id="3df1e-135">中介軟體更新`Request.Scheme`，並使用`X-Forwarded-Proto`標頭，以便讓該重新導向 Uri 和其他的安全性原則運作正確。</span><span class="sxs-lookup"><span data-stu-id="3df1e-135">The middleware updates the `Request.Scheme`, using the `X-Forwarded-Proto` header, so that redirect URIs and other security policies work correctly.</span></span>

<span data-ttu-id="3df1e-136">使用任何類型的驗證中介軟體時，必須先執行轉送標頭中介軟體。</span><span class="sxs-lookup"><span data-stu-id="3df1e-136">When using any type of authentication middleware, the Forwarded Headers Middleware must run first.</span></span> <span data-ttu-id="3df1e-137">這種排序可確保驗證中介軟體可以取用的標頭值，並產生正確的重新導向 Uri。</span><span class="sxs-lookup"><span data-stu-id="3df1e-137">This ordering ensures that the authentication middleware can consume the header values and generate correct redirect URIs.</span></span>

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[<span data-ttu-id="3df1e-138">ASP.NET Core 2.x</span><span class="sxs-lookup"><span data-stu-id="3df1e-138">ASP.NET Core 2.x</span></span>](#tab/aspnetcore2x)

<span data-ttu-id="3df1e-139">叫用[UseForwardedHeaders](/dotnet/api/microsoft.aspnetcore.builder.forwardedheadersextensions.useforwardedheaders)方法中的`Startup.Configure`之前先呼叫[UseAuthentication](/dotnet/api/microsoft.aspnetcore.builder.authappbuilderextensions.useauthentication)或類似的驗證配置中介軟體：</span><span class="sxs-lookup"><span data-stu-id="3df1e-139">Invoke the [UseForwardedHeaders](/dotnet/api/microsoft.aspnetcore.builder.forwardedheadersextensions.useforwardedheaders) method in `Startup.Configure` before calling [UseAuthentication](/dotnet/api/microsoft.aspnetcore.builder.authappbuilderextensions.useauthentication) or similar authentication scheme middleware:</span></span>

```csharp
app.UseForwardedHeaders(new ForwardedHeadersOptions
{
    ForwardedHeaders = ForwardedHeaders.XForwardedFor | ForwardedHeaders.XForwardedProto
});

app.UseAuthentication();
```

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[<span data-ttu-id="3df1e-140">ASP.NET Core 1.x</span><span class="sxs-lookup"><span data-stu-id="3df1e-140">ASP.NET Core 1.x</span></span>](#tab/aspnetcore1x)

<span data-ttu-id="3df1e-141">叫用[UseForwardedHeaders](/dotnet/api/microsoft.aspnetcore.builder.forwardedheadersextensions.useforwardedheaders)方法中的`Startup.Configure`之前先呼叫[UseIdentity](/dotnet/api/microsoft.aspnetcore.builder.builderextensions.useidentity)和[UseFacebookAuthentication](/dotnet/api/microsoft.aspnetcore.builder.facebookappbuilderextensions.usefacebookauthentication)或類似的驗證配置中介軟體：</span><span class="sxs-lookup"><span data-stu-id="3df1e-141">Invoke the [UseForwardedHeaders](/dotnet/api/microsoft.aspnetcore.builder.forwardedheadersextensions.useforwardedheaders) method in `Startup.Configure` before calling [UseIdentity](/dotnet/api/microsoft.aspnetcore.builder.builderextensions.useidentity) and [UseFacebookAuthentication](/dotnet/api/microsoft.aspnetcore.builder.facebookappbuilderextensions.usefacebookauthentication) or similar authentication scheme middleware:</span></span>

```csharp
app.UseForwardedHeaders(new ForwardedHeadersOptions
{
    ForwardedHeaders = ForwardedHeaders.XForwardedFor | ForwardedHeaders.XForwardedProto
});

app.UseIdentity();
app.UseFacebookAuthentication(new FacebookOptions()
{
    AppId = Configuration["Authentication:Facebook:AppId"],
    AppSecret = Configuration["Authentication:Facebook:AppSecret"]
});
```

---

<span data-ttu-id="3df1e-142">如果沒有[ForwardedHeadersOptions](/dotnet/api/microsoft.aspnetcore.builder.forwardedheadersoptions)指定給中介軟體，轉送的預設標頭`None`。</span><span class="sxs-lookup"><span data-stu-id="3df1e-142">If no [ForwardedHeadersOptions](/dotnet/api/microsoft.aspnetcore.builder.forwardedheadersoptions) are specified to the middleware, the default headers to forward are `None`.</span></span>

### <a name="install-nginx"></a><span data-ttu-id="3df1e-143">安裝 Nginx</span><span class="sxs-lookup"><span data-stu-id="3df1e-143">Install Nginx</span></span>

```bash
sudo apt-get install nginx
```

> [!NOTE]
> <span data-ttu-id="3df1e-144">如果要安裝選擇性 Nginx 模組，建立從來源 Nginx 可能需要。</span><span class="sxs-lookup"><span data-stu-id="3df1e-144">If optional Nginx modules will be installed, building Nginx from source might be required.</span></span>

<span data-ttu-id="3df1e-145">使用 `apt-get` 來安裝 Nginx。</span><span class="sxs-lookup"><span data-stu-id="3df1e-145">Use `apt-get` to install Nginx.</span></span> <span data-ttu-id="3df1e-146">安裝程式建立的系統 V init 初始化指令碼，會在系統啟動時將 Nginx 執行為精靈。</span><span class="sxs-lookup"><span data-stu-id="3df1e-146">The installer creates a System V init script that runs Nginx as daemon on system startup.</span></span> <span data-ttu-id="3df1e-147">因為 Nginx 是第一次安裝，請透過執行明確啟動它：</span><span class="sxs-lookup"><span data-stu-id="3df1e-147">Since Nginx was installed for the first time, explicitly start it by running:</span></span>

```bash
sudo service nginx start
```

<span data-ttu-id="3df1e-148">確認瀏覽器會顯示 Nginx 的預設登陸頁面。</span><span class="sxs-lookup"><span data-stu-id="3df1e-148">Verify a browser displays the default landing page for Nginx.</span></span>

### <a name="configure-nginx"></a><span data-ttu-id="3df1e-149">設定 Nginx</span><span class="sxs-lookup"><span data-stu-id="3df1e-149">Configure Nginx</span></span>

<span data-ttu-id="3df1e-150">若要設定 Nginx 做為轉寄要求至我們的 ASP.NET Core 應用程式的反向 proxy，修改`/etc/nginx/sites-available/default`。</span><span class="sxs-lookup"><span data-stu-id="3df1e-150">To configure Nginx as a reverse proxy to forward requests to our ASP.NET Core app, modify `/etc/nginx/sites-available/default`.</span></span> <span data-ttu-id="3df1e-151">以文字編輯器開啟它，並以下列項目取代內容：</span><span class="sxs-lookup"><span data-stu-id="3df1e-151">Open it in a text editor, and replace the contents with the following:</span></span>

```
server {
    listen 80;
    location / {
        proxy_pass http://localhost:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection keep-alive;
        proxy_set_header Host $http_host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

<span data-ttu-id="3df1e-152">此 Nginx 組態檔會將連入的公用流量從連接埠 `80` 轉送到連接埠 `5000`。</span><span class="sxs-lookup"><span data-stu-id="3df1e-152">This Nginx configuration file forwards incoming public traffic from port `80` to port `5000`.</span></span>

<span data-ttu-id="3df1e-153">一旦建立 Nginx 組態之後，執行`sudo nginx -t`驗證組態檔的語法。</span><span class="sxs-lookup"><span data-stu-id="3df1e-153">Once the Nginx configuration is established, run `sudo nginx -t` to verify the syntax of the configuration files.</span></span> <span data-ttu-id="3df1e-154">如果組態檔測試成功時，強制以收取這些變更藉由執行 Nginx `sudo nginx -s reload`。</span><span class="sxs-lookup"><span data-stu-id="3df1e-154">If the configuration file test is successful, force Nginx to pick up the changes by running `sudo nginx -s reload`.</span></span>

## <a name="monitoring-the-app"></a><span data-ttu-id="3df1e-155">監視應用程式</span><span class="sxs-lookup"><span data-stu-id="3df1e-155">Monitoring the app</span></span>

<span data-ttu-id="3df1e-156">伺服器是安裝所做的要求轉送給`http://<serveraddress>:80`入 Kestrel 在上執行 ASP.NET Core 應用程式`http://127.0.0.1:5000`。</span><span class="sxs-lookup"><span data-stu-id="3df1e-156">The server is setup to forward requests made to `http://<serveraddress>:80` on to the ASP.NET Core app running on Kestrel at `http://127.0.0.1:5000`.</span></span> <span data-ttu-id="3df1e-157">不過，Nginx 並未設定來管理 Kestrel 程序。</span><span class="sxs-lookup"><span data-stu-id="3df1e-157">However, Nginx isn't set up to manage the Kestrel process.</span></span> <span data-ttu-id="3df1e-158">*systemd*可用來建立服務檔案，以啟動及監視基礎的 web 應用程式。</span><span class="sxs-lookup"><span data-stu-id="3df1e-158">*systemd* can be used to create a service file to start and monitor the underlying web app.</span></span> <span data-ttu-id="3df1e-159">*systemd* 是 init 系統，提供許多強大的啟動、停止和管理處理程序功能。</span><span class="sxs-lookup"><span data-stu-id="3df1e-159">*systemd* is an init system that provides many powerful features for starting, stopping, and managing processes.</span></span> 

### <a name="create-the-service-file"></a><span data-ttu-id="3df1e-160">建立服務檔</span><span class="sxs-lookup"><span data-stu-id="3df1e-160">Create the service file</span></span>

<span data-ttu-id="3df1e-161">建立服務定義檔：</span><span class="sxs-lookup"><span data-stu-id="3df1e-161">Create the service definition file:</span></span>

```bash
sudo nano /etc/systemd/system/kestrel-hellomvc.service
```

<span data-ttu-id="3df1e-162">以下是範例服務檔案的應用程式：</span><span class="sxs-lookup"><span data-stu-id="3df1e-162">The following is an example service file for the app:</span></span>

```ini
[Unit]
Description=Example .NET Web API App running on Ubuntu

[Service]
WorkingDirectory=/var/aspnetcore/hellomvc
ExecStart=/usr/bin/dotnet /var/aspnetcore/hellomvc/hellomvc.dll
Restart=always
RestartSec=10  # Restart service after 10 seconds if dotnet service crashes
SyslogIdentifier=dotnet-example
User=www-data
Environment=ASPNETCORE_ENVIRONMENT=Production
Environment=DOTNET_PRINT_TELEMETRY_MESSAGE=false

[Install]
WantedBy=multi-user.target
```

<span data-ttu-id="3df1e-163">**注意：**如果使用者*www 資料*不會使用設定，此處定義的使用者必須先建立並提供適當的擁有權的檔案。</span><span class="sxs-lookup"><span data-stu-id="3df1e-163">**Note:** If the user *www-data* isn't used by the configuration, the user defined here must be created first and given proper ownership for files.</span></span>
<span data-ttu-id="3df1e-164">**注意：** Linux 具有區分大小寫的檔案系統。</span><span class="sxs-lookup"><span data-stu-id="3df1e-164">**Note:** Linux has a case-sensitive file system.</span></span> <span data-ttu-id="3df1e-165">設定為 「 生產 」 會產生組態檔的搜尋 ASPNETCORE_ENVIRONMENT *appsettings。Production.json*，而非*appsettings.production.json*。</span><span class="sxs-lookup"><span data-stu-id="3df1e-165">Setting ASPNETCORE_ENVIRONMENT to "Production" results in searching for the configuration file *appsettings.Production.json*, not *appsettings.production.json*.</span></span>

<span data-ttu-id="3df1e-166">儲存檔案並啟用服務。</span><span class="sxs-lookup"><span data-stu-id="3df1e-166">Save the file and enable the service.</span></span>

```bash
systemctl enable kestrel-hellomvc.service
```

<span data-ttu-id="3df1e-167">啟動服務並確認它正在執行。</span><span class="sxs-lookup"><span data-stu-id="3df1e-167">Start the service and verify that it's running.</span></span>

```
systemctl start kestrel-hellomvc.service
systemctl status kestrel-hellomvc.service

● kestrel-hellomvc.service - Example .NET Web API App running on Ubuntu
    Loaded: loaded (/etc/systemd/system/kestrel-hellomvc.service; enabled)
    Active: active (running) since Thu 2016-10-18 04:09:35 NZDT; 35s ago
Main PID: 9021 (dotnet)
    CGroup: /system.slice/kestrel-hellomvc.service
            └─9021 /usr/local/bin/dotnet /var/aspnetcore/hellomvc/hellomvc.dll
```

<span data-ttu-id="3df1e-168">Web 應用程式完全設定和可以從在本機電腦上的瀏覽器存取的反向 proxy 設定和管理透過 systemd Kestrel， `http://localhost`。</span><span class="sxs-lookup"><span data-stu-id="3df1e-168">With the reverse proxy configured and Kestrel managed through systemd, the web app is fully configured and can be accessed from a browser on the local machine at `http://localhost`.</span></span> <span data-ttu-id="3df1e-169">它也是可從遠端電腦，限制可能會封鎖任何防火牆存取。</span><span class="sxs-lookup"><span data-stu-id="3df1e-169">It's also accessible from a remote machine, barring any firewall that might be blocking.</span></span> <span data-ttu-id="3df1e-170">檢查回應標頭`Server`標頭會顯示由 Kestrel ASP.NET Core 應用程式。</span><span class="sxs-lookup"><span data-stu-id="3df1e-170">Inspecting the response headers, the `Server` header shows the ASP.NET Core app being served by Kestrel.</span></span>

```text
HTTP/1.1 200 OK
Date: Tue, 11 Oct 2016 16:22:23 GMT
Server: Kestrel
Keep-Alive: timeout=5, max=98
Connection: Keep-Alive
Transfer-Encoding: chunked
```

### <a name="viewing-logs"></a><span data-ttu-id="3df1e-171">檢視記錄</span><span class="sxs-lookup"><span data-stu-id="3df1e-171">Viewing logs</span></span>

<span data-ttu-id="3df1e-172">因為 web 應用程式使用 Kestrel 管理使用`systemd`，所有事件和處理程序會都記錄到集中式的日誌。</span><span class="sxs-lookup"><span data-stu-id="3df1e-172">Since the web app using Kestrel is managed using `systemd`, all events and processes are logged to a centralized journal.</span></span> <span data-ttu-id="3df1e-173">不過，此日誌包含 `systemd` 管理的所有服務和處理程序的所有項目。</span><span class="sxs-lookup"><span data-stu-id="3df1e-173">However, this journal includes all entries for all services and processes managed by `systemd`.</span></span> <span data-ttu-id="3df1e-174">若要檢視 `kestrel-hellomvc.service` 的特定項目，請使用下列命令：</span><span class="sxs-lookup"><span data-stu-id="3df1e-174">To view the `kestrel-hellomvc.service`-specific items, use the following command:</span></span>

```bash
sudo journalctl -fu kestrel-hellomvc.service
```

<span data-ttu-id="3df1e-175">如需進一步篩選，例如 `--since today`、`--until 1 hour ago` 或這些項目的組合等時間選項，可以減少傳回的項目數量。</span><span class="sxs-lookup"><span data-stu-id="3df1e-175">For further filtering, time options such as `--since today`, `--until 1 hour ago` or a combination of these can reduce the amount of entries returned.</span></span>

```bash
sudo journalctl -fu kestrel-hellomvc.service --since "2016-10-18" --until "2016-10-18 04:00"
```

## <a name="securing-the-app"></a><span data-ttu-id="3df1e-176">保護應用程式</span><span class="sxs-lookup"><span data-stu-id="3df1e-176">Securing the app</span></span>

### <a name="enable-apparmor"></a><span data-ttu-id="3df1e-177">啟用 AppArmor</span><span class="sxs-lookup"><span data-stu-id="3df1e-177">Enable AppArmor</span></span>

<span data-ttu-id="3df1e-178">Linux 安全性模組 (LSM) 是一種架構，屬於後 Linux 2.6 Linux 核心。</span><span class="sxs-lookup"><span data-stu-id="3df1e-178">Linux Security Modules (LSM) is a framework that's part of the Linux kernel since Linux 2.6.</span></span> <span data-ttu-id="3df1e-179">LSM 支援不同的安全性模組實作。</span><span class="sxs-lookup"><span data-stu-id="3df1e-179">LSM supports different implementations of security modules.</span></span> <span data-ttu-id="3df1e-180">[AppArmor](https://wiki.ubuntu.com/AppArmor) 是 LSM，它實作的必要存取控制系統，可將程式限定在有限的資源集中。</span><span class="sxs-lookup"><span data-stu-id="3df1e-180">[AppArmor](https://wiki.ubuntu.com/AppArmor) is a LSM that implements a Mandatory Access Control system which allows confining the program to a limited set of resources.</span></span> <span data-ttu-id="3df1e-181">確定已啟用並正確設定 AppArmor。</span><span class="sxs-lookup"><span data-stu-id="3df1e-181">Ensure AppArmor is enabled and properly configured.</span></span>

### <a name="configuring-the-firewall"></a><span data-ttu-id="3df1e-182">設定防火牆</span><span class="sxs-lookup"><span data-stu-id="3df1e-182">Configuring the firewall</span></span>

<span data-ttu-id="3df1e-183">關閉所有不在使用中的外部連接埠。</span><span class="sxs-lookup"><span data-stu-id="3df1e-183">Close off all external ports that are not in use.</span></span> <span data-ttu-id="3df1e-184">簡單的防火牆 (ufw) 提供命令列介面供設定防火牆，為 `iptables` 提供前端。</span><span class="sxs-lookup"><span data-stu-id="3df1e-184">Uncomplicated firewall (ufw) provides a front end for `iptables` by providing a command line interface for configuring the firewall.</span></span> <span data-ttu-id="3df1e-185">確認`ufw`設定為允許任何所需的連接埠上的流量。</span><span class="sxs-lookup"><span data-stu-id="3df1e-185">Verify that `ufw` is configured to allow traffic on any ports needed.</span></span>

```bash
sudo apt-get install ufw
sudo ufw enable

sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```

### <a name="securing-nginx"></a><span data-ttu-id="3df1e-186">保護 Nginx</span><span class="sxs-lookup"><span data-stu-id="3df1e-186">Securing Nginx</span></span>

<span data-ttu-id="3df1e-187">Nginx 的預設分佈不會啟用 SSL。</span><span class="sxs-lookup"><span data-stu-id="3df1e-187">The default distribution of Nginx doesn't enable SSL.</span></span> <span data-ttu-id="3df1e-188">為啟用其他的安全性功能，請從來源建置。</span><span class="sxs-lookup"><span data-stu-id="3df1e-188">To enable additional security features, build from source.</span></span>

#### <a name="download-the-source-and-install-the-build-dependencies"></a><span data-ttu-id="3df1e-189">下載來源並安裝組建相依性</span><span class="sxs-lookup"><span data-stu-id="3df1e-189">Download the source and install the build dependencies</span></span>

```bash
# Install the build dependencies
sudo apt-get update
sudo apt-get install build-essential zlib1g-dev libpcre3-dev libssl-dev libxslt1-dev libxml2-dev libgd2-xpm-dev libgeoip-dev libgoogle-perftools-dev libperl-dev

# Download Nginx 1.10.0 or latest
wget http://www.nginx.org/download/nginx-1.10.0.tar.gz
tar zxf nginx-1.10.0.tar.gz
```

#### <a name="change-the-nginx-response-name"></a><span data-ttu-id="3df1e-190">變更 Nginx 回應名稱</span><span class="sxs-lookup"><span data-stu-id="3df1e-190">Change the Nginx response name</span></span>

<span data-ttu-id="3df1e-191">編輯 *src/http/ngx_http_header_filter_module.c*：</span><span class="sxs-lookup"><span data-stu-id="3df1e-191">Edit *src/http/ngx_http_header_filter_module.c*:</span></span>

```c
static char ngx_http_server_string[] = "Server: Web Server" CRLF;
static char ngx_http_server_full_string[] = "Server: Web Server" CRLF;
```

#### <a name="configure-the-options-and-build"></a><span data-ttu-id="3df1e-192">設定選項和組建</span><span class="sxs-lookup"><span data-stu-id="3df1e-192">Configure the options and build</span></span>

<span data-ttu-id="3df1e-193">規則運算式需要 PCRE 程式庫。</span><span class="sxs-lookup"><span data-stu-id="3df1e-193">The PCRE library is required for regular expressions.</span></span> <span data-ttu-id="3df1e-194">規則運算式用於 ngx_http_rewrite_module 的位置指示詞。</span><span class="sxs-lookup"><span data-stu-id="3df1e-194">Regular expressions are used in the location directive for the ngx_http_rewrite_module.</span></span> <span data-ttu-id="3df1e-195">http_ssl_module 新增 HTTPS 通訊協定的支援。</span><span class="sxs-lookup"><span data-stu-id="3df1e-195">The http_ssl_module adds HTTPS protocol support.</span></span>

<span data-ttu-id="3df1e-196">請考慮使用 web 應用程式防火牆像*ModSecurity*強化應用程式。</span><span class="sxs-lookup"><span data-stu-id="3df1e-196">Consider using a web app firewall like *ModSecurity* to harden the app.</span></span>

```bash
./configure
--with-pcre=../pcre-8.38
--with-zlib=../zlib-1.2.8
--with-http_ssl_module
--with-stream
--with-mail=dynamic
```

#### <a name="configure-ssl"></a><span data-ttu-id="3df1e-197">設定 SSL</span><span class="sxs-lookup"><span data-stu-id="3df1e-197">Configure SSL</span></span>

* <span data-ttu-id="3df1e-198">將伺服器設定為接聽連接埠上的 HTTPS 流量`443`藉由指定核發由受信任憑證授權單位 (CA) 的有效憑證。</span><span class="sxs-lookup"><span data-stu-id="3df1e-198">Configure the server to listen to HTTPS traffic on port `443` by specifying a valid certificate issued by a trusted Certificate Authority (CA).</span></span>

* <span data-ttu-id="3df1e-199">利用一些下列所述的作法強化安全性*/etc/nginx/nginx.conf*檔案。</span><span class="sxs-lookup"><span data-stu-id="3df1e-199">Harden the security by employing some of the practices depicted in the following */etc/nginx/nginx.conf* file.</span></span> <span data-ttu-id="3df1e-200">範例包括選擇更強的加密，重新導向 HTTPS 到 HTTP 的所有流量。</span><span class="sxs-lookup"><span data-stu-id="3df1e-200">Examples include choosing a stronger cipher and redirecting all traffic over HTTP to HTTPS.</span></span>

* <span data-ttu-id="3df1e-201">新增 `HTTP Strict-Transport-Security` (HSTS) 標頭可確保用戶端提出的所有後續要求都只會透過 HTTPS。</span><span class="sxs-lookup"><span data-stu-id="3df1e-201">Adding an `HTTP Strict-Transport-Security` (HSTS) header ensures all subsequent requests made by the client are over HTTPS only.</span></span>

* <span data-ttu-id="3df1e-202">請勿將加入 Strict 傳輸安全性標頭，或選擇適當`max-age`如果未來將會停用 SSL。</span><span class="sxs-lookup"><span data-stu-id="3df1e-202">Don't add the Strict-Transport-Security header or chose an appropriate `max-age` if SSL will be disabled in the future.</span></span>

<span data-ttu-id="3df1e-203">新增 */etc/nginx/Proxy.conf* 組態檔：</span><span class="sxs-lookup"><span data-stu-id="3df1e-203">Add the */etc/nginx/proxy.conf* configuration file:</span></span>

[!code-nginx[Main](linux-nginx/proxy.conf)]

<span data-ttu-id="3df1e-204">編輯 */etc/nginx/Proxy.conf* 組態檔。</span><span class="sxs-lookup"><span data-stu-id="3df1e-204">Edit the */etc/nginx/nginx.conf* configuration file.</span></span> <span data-ttu-id="3df1e-205">此範例在一個組態檔中同時包含 `http` 和 `server` 區段。</span><span class="sxs-lookup"><span data-stu-id="3df1e-205">The example contains both `http` and `server` sections in one configuration file.</span></span>

[!code-nginx[Main](linux-nginx/nginx.conf?highlight=2)]

#### <a name="secure-nginx-from-clickjacking"></a><span data-ttu-id="3df1e-206">保護 Nginx 免於點閱綁架</span><span class="sxs-lookup"><span data-stu-id="3df1e-206">Secure Nginx from clickjacking</span></span>
<span data-ttu-id="3df1e-207">點閱綁架是收集受感染使用者按一下動作的惡意技術。</span><span class="sxs-lookup"><span data-stu-id="3df1e-207">Clickjacking is a malicious technique to collect an infected user's clicks.</span></span> <span data-ttu-id="3df1e-208">點閱綁架誘騙受害者 (訪客) 到受感染的網站上執行按一下的動作。</span><span class="sxs-lookup"><span data-stu-id="3df1e-208">Clickjacking tricks the victim (visitor) into clicking on an infected site.</span></span> <span data-ttu-id="3df1e-209">使用 X-框架-選項來保護安全的站台。</span><span class="sxs-lookup"><span data-stu-id="3df1e-209">Use X-FRAME-OPTIONS to secure the site.</span></span>

<span data-ttu-id="3df1e-210">編輯 *nginx.conf* 檔案：</span><span class="sxs-lookup"><span data-stu-id="3df1e-210">Edit the *nginx.conf* file:</span></span>

```bash
sudo nano /etc/nginx/nginx.conf
```

<span data-ttu-id="3df1e-211">新增行 `add_header X-Frame-Options "SAMEORIGIN";` 並儲存檔案，然後重新啟動 Nginx。</span><span class="sxs-lookup"><span data-stu-id="3df1e-211">Add the line `add_header X-Frame-Options "SAMEORIGIN";` and save the file, then restart Nginx.</span></span>

#### <a name="mime-type-sniffing"></a><span data-ttu-id="3df1e-212">MIME 類型探查</span><span class="sxs-lookup"><span data-stu-id="3df1e-212">MIME-type sniffing</span></span>

<span data-ttu-id="3df1e-213">此標頭會防止大部分的瀏覽器對遠離宣告內容類型的回應進行 MIME 探查，因為標頭會指示瀏覽器不要覆寫回應內容類型。</span><span class="sxs-lookup"><span data-stu-id="3df1e-213">This header prevents most browsers from MIME-sniffing a response away from the declared content type, as the header instructs the browser not to override the response content type.</span></span> <span data-ttu-id="3df1e-214">使用 `nosniff` 選項，如果伺服器顯示的內容是 "text/html"，則瀏覽器就會將它轉譯為 "text/html"。</span><span class="sxs-lookup"><span data-stu-id="3df1e-214">With the `nosniff` option, if the server says the content is "text/html", the browser renders it as "text/html".</span></span>

<span data-ttu-id="3df1e-215">編輯 *nginx.conf* 檔案：</span><span class="sxs-lookup"><span data-stu-id="3df1e-215">Edit the *nginx.conf* file:</span></span>

```bash
sudo nano /etc/nginx/nginx.conf
```

<span data-ttu-id="3df1e-216">新增 `add_header X-Content-Type-Options "nosniff";` 行並儲存檔案，然後重新啟動 Nginx。</span><span class="sxs-lookup"><span data-stu-id="3df1e-216">Add the line `add_header X-Content-Type-Options "nosniff";` and save the file, then restart Nginx.</span></span>