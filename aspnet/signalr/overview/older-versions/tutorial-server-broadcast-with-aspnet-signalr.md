---
uid: signalr/overview/older-versions/tutorial-server-broadcast-with-aspnet-signalr
title: "教程： 使用 ASP.NET SignalR 广播的服务器 1.x |Microsoft 文档"
author: pfletcher
description: "本教程演示如何创建的 web 应用程序使用 ASP.NET SignalR 来提供服务器广播的功能。 服务器广播的意味着这 communic..."
ms.author: aspnetcontent
manager: wpickett
ms.date: 04/10/2013
ms.topic: article
ms.assetid: ab7b2554-956a-4f6d-b2a0-4ae0c62e8580
ms.technology: dotnet-signalr
ms.prod: .net-framework
msc.legacyurl: /signalr/overview/older-versions/tutorial-server-broadcast-with-aspnet-signalr
msc.type: authoredcontent
ms.openlocfilehash: afb2fa9b3dfd80a2aa49fffae71965fc2098442f
ms.sourcegitcommit: 9a9483aceb34591c97451997036a9120c3fe2baf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/10/2017
---
<a name="tutorial-server-broadcast-with-aspnet-signalr-1x"></a>教程： 使用 ASP.NET SignalR 广播的服务器 1.x
====================
通过[Patrick Fletcher](https://github.com/pfletcher)， [Tom Dykstra](https://github.com/tdykstra)

> 本教程演示如何创建的 web 应用程序使用 ASP.NET SignalR 来提供服务器广播的功能。 服务器广播意味着发送到客户端的通信由服务器启动。 这种情况下需要不同的编程方法与例如聊天应用程序，在其中通信发送到客户端启动的一个或多个客户端的对等方案。
> 
> 将在本教程中创建的应用程序模拟股票行情，服务器广播功能的典型情况。
> 
> 在本教程的注释是欢迎。 如果你有与本教程不直接相关的问题，你可以发布到[ASP.NET SignalR 论坛](https://forums.asp.net/1254.aspx/1?ASP+NET+SignalR)或[StackOverflow.com](http://stackoverflow.com)。


## <a name="overview"></a>概述

[Microsoft.AspNet.SignalR.Sample](http://nuget.org/packages/microsoft.aspnet.signalr.sample) NuGet 包安装 Visual Studio 项目中的一个示例模拟股票行情自动收录器应用程序。 在本教程的第一部分，你将从头开始创建该应用程序的简化的版本。 在本教程的其余部分中，你将安装 NuGet 包，并检查的其他功能和它所创建的代码。

股票行情自动收录器应用程序是一种实时应用程序要在其中定期"push"或广播，将通知从所有连接的客户端到服务器的代表。

你将在本教程的第一部分中生成应用程序显示股票数据网格。

![StockTicker 初始版本](tutorial-server-broadcast-with-aspnet-signalr/_static/image1.png)

定期服务器随机更新股票价格并将更新推送到所有连接的客户端。 在浏览器数字和符号中的**更改**和 **%** 列动态更改以响应来自服务器的通知。 如果你打开到相同的 URL，其他浏览器，它们都同时显示相同的数据和对数据的相同更改。

本教程包含以下各节：

- [先决条件](#prerequisites)
- [创建项目](#createproject)
- [添加 SignalR NuGet 包](#nugetpackages)
- [设置服务器代码](#server)
- [设置客户端代码](#client)
- [测试应用程序](#test)
- [启用日志记录](#enablelogging)
- [安装和查看完整的 StockTicker 示例](#fullsample)
- [后续步骤](#nextsteps)

> [!NOTE]
> 如果你不想要完成的生成应用程序的步骤，你可以在新安装 SignalR.Sample 包**空的 ASP.NET Web 应用程序**项目，并通读这些步骤以使代码的说明。 本教程的第一部分介绍 SignalR.Sample 代码的子集并第二部分说明 SignalR.Sample 包中的其他功能的主要功能。


<a id="prerequisites"></a>

## <a name="prerequisites"></a>先决条件

在开始之前，请确保你有 Visual Studio 2012 或 2010 SP1 安装在计算机上。 如果你没有 Visual Studio，请参阅[ASP.NET 下载](https://www.asp.net/downloads)即可免费 Visual Studio 2012 Express for Web。

如果你有 Visual Studio 2010，请确保[NuGet](https://visualstudiogallery.msdn.microsoft.com/27077b70-9dad-4c64-adcf-c7cf6bc9970c)安装。

<a id="createproject"></a>

## <a name="create-the-project"></a>创建项目

1. 从**文件**菜单上，单击**新项目**。
2. 在**新项目**对话框框中，展开**C#**下**模板**和选择**Web**。
3. 选择**ASP.NET 空 Web 应用程序**模板，将项目*SignalR.StockTicker*，然后单击**确定**。

    ![“新建项目”对话框](tutorial-server-broadcast-with-aspnet-signalr/_static/image2.png)

<a id="nugetpackages"></a>

## <a name="add-the-signalr-nuget-packages"></a>添加 SignalR NuGet 包

### <a name="add-the-signalr-and-jquery-nuget-packages"></a>添加 SignalR 和 JQuery NuGet 包

你可以通过安装 NuGet 包添加到项目的 SignalR 功能。

1. 单击**工具 |库包管理器 |程序包管理器控制台**。
2. 在程序包管理器中输入以下命令。

    [!code-powershell[Main](tutorial-server-broadcast-with-aspnet-signalr/samples/sample1.ps1)]

    SignalR 包安装大量其他 NuGet 包依赖关系。 完成安装后你将具有所有 ASP.NET 应用程序中使用 SignalR 所需的服务器和客户端组件。

<a id="server"></a>

## <a name="set-up-the-server-code"></a>设置服务器代码

在本节中你设置的服务器运行的代码。

### <a name="create-the-stock-class"></a>创建 Stock 类

你首先创建 Stock 模型类将用于存储和传输有关常用的信息。

1. 在项目文件夹中创建新的类文件，将其*Stock.cs*，然后将模板代码替换为以下代码：

    [!code-csharp[Main](tutorial-server-broadcast-with-aspnet-signalr/samples/sample2.cs)]

    在创建 stocks 时将设置的两个属性是符号 (例如，microsoft 的 MSFT) 和价格。 其他属性取决于你何时以及如何设置价格。 设置价格，第一次的值获取传播到 DayOpen。 当设置价格，更改和 PercentChange 属性值进行计算的后续时间基于价格和 DayOpen 之间的差异。

### <a name="create-the-stockticker-and-stocktickerhub-classes"></a>创建的 StockTicker 和 StockTickerHub 类

你将使用 SignalR 中心 API 以处理服务器到客户端交互。 从 SignalR Hub 类派生的 StockTickerHub 类将处理从客户端接收连接和方法调用。 你还需要维护股票数据并运行要定期触发价格更新，独立于客户端连接的计时器对象。 因为中心实例是暂时的不能置于中心类，这些函数。 为每个操作的中心，例如连接和从客户端到服务器的调用创建的中心类实例。 因此，它以保持常用的数据，更新价格，并广播价格更新的机制必须在单独的类，该类将命名 StockTicker 中运行。

![从 StockTicker 广播](tutorial-server-broadcast-with-aspnet-signalr/_static/image4.png)

你只想要在服务器上，运行，因此你需要指向单一 StockTicker 实例中设置从每个 StockTickerHub 实例引用的 StockTicker 类的一个实例。 StockTicker 类具有能够进行广播到客户端，因为它具有股票数据并触发更新，但 StockTicker 不是中心类。 因此，StockTicker 类必须获取对 SignalR Hub 连接上下文对象的引用。 然后，它可以使用 SignalR 连接上下文对象将广播到客户端。

1. 在**解决方案资源管理器**，右键单击项目，然后单击**添加新项**。
2. 如果你有 Visual Studio 2012 [ASP.NET 和 Web Tools 2012.2 更新](https://go.microsoft.com/fwlink/?LinkId=279941)，单击**Web**下**Visual C#**和选择**SignalR Hub Class**项模板。 否则，请选择**类**模板。
3. 将新类*StockTickerHub.cs*，然后单击**添加**。

    ![添加 StockTickerHub.cs](tutorial-server-broadcast-with-aspnet-signalr/_static/image5.png)
4. 模板代码替换为以下代码：

    [!code-csharp[Main](tutorial-server-broadcast-with-aspnet-signalr/samples/sample3.cs)]

    [中心](https://msdn.microsoft.com/en-us/library/microsoft.aspnet.signalr.hub(v=vs.111).aspx)类用于定义客户端可以调用在服务器的方法。 要定义一种方法： `GetAllStocks()`。 当客户端最初连接到服务器时，它将调用此方法以获取所有其当前价格 stocks 的列表。 该方法可以同步执行，并返回`IEnumerable<Stock>`因为它从内存返回数据。 如果该方法必须获取数据，通过执行某项，需要等待，如数据库查找或 web 服务调用，则会指定`Task<IEnumerable<Stock>>`作为要启用异步处理的返回值。 有关详细信息，请参阅[ASP.NET SignalR 中心 API 指南-服务器-时以异步方式执行](index.md)。

    HubName 属性指定将如何在客户端上的 JavaScript 代码中引用的中心。 如果不使用此属性在客户端上的默认名称是类名，在这种情况下将是 stockTickerHub 混合使用大小写版本。

    正如您将看到更高版本创建 StockTicker 类时，将在其静态实例属性创建该类的单一实例。 StockTicker 单一实例仍保留在内存无论有多少客户端连接或断开连接，并确保实例正在 GetAllStocks 方法用于返回当前股票信息。
5. 在项目文件夹中创建新的类文件，将其*StockTicker.cs*，然后将模板代码替换为以下代码：

    [!code-csharp[Main](tutorial-server-broadcast-with-aspnet-signalr/samples/sample4.cs)]

    因为多个线程将运行 StockTicker 代码的同一个实例，StockTicker 类必须是 threadsafe。

    ### <a name="storing-the-singleton-instance-in-a-static-field"></a>将单一实例存储在静态字段

    此代码初始化静态\_支持具有的类，并且此实例的实例属性的实例字段是唯一的类可以创建的实例，因为构造函数标记为私有。 [延迟初始化](https://msdn.microsoft.com/en-us/library/dd997286.aspx)用于\_实例字段，不适用于性能原因，但若要确保创建的实例是 threadsafe。

    [!code-csharp[Main](tutorial-server-broadcast-with-aspnet-signalr/samples/sample5.cs)]

    客户端连接到服务器，每次在一个单独的线程中运行的 StockTickerHub 类的新实例获取 StockTicker 单一实例的 StockTicker.Instance 静态属性，如你所看到的前面的 StockTickerHub 类。

    ### <a name="storing-stock-data-in-a-concurrentdictionary"></a>在 ConcurrentDictionary 中存储常用数据

    构造函数初始化\_使用一些示例股票数据和 GetAllStocks stocks 集合返回 stocks。 如您前面看到的这是在客户端可以调用的中心类中的服务器方法 StockTickerHub.GetAllStocks 反过来返回 stocks 此集合。

    [!code-csharp[Main](tutorial-server-broadcast-with-aspnet-signalr/samples/sample6.cs)]

    [!code-csharp[Main](tutorial-server-broadcast-with-aspnet-signalr/samples/sample7.cs)]

    Stocks 集合指[ConcurrentDictionary](https://msdn.microsoft.com/en-us/library/dd287191.aspx)对线程安全的类型。 作为替代方法，您可以使用[字典](https://msdn.microsoft.com/en-us/library/xfhwa508.aspx)对象，并显式锁定字典时对其进行更改。

    对于此示例应用程序，它是确定在内存中存储应用程序数据并释放 StockTicker 实例时丢失数据。 在实际应用中，您将使用如数据库后端数据存储区。

    ### <a name="periodically-updating-stock-prices"></a>定期更新股票价格

    构造函数会启动定期调用更新上的随机基础的股票价格的方法的计时器对象。

    [!code-csharp[Main](tutorial-server-broadcast-with-aspnet-signalr/samples/sample8.cs)]

    UpdateStockPrices 称为由计时器，传入参数中的 null。 更新价格，锁上执行前\_updateStockPricesLock 对象。 如果另一个线程已更新价格，，然后它调用 TryUpdateStockPrice 列表中每个股票代码将检查。 TryUpdateStockPrice 方法来决定是否进行更改的股票价格和数量，将其更改。 如果更改股票价格，BroadcastStockPrice 称为广播到所有连接的客户端的股票价格更改。

    \_UpdatingStockPrices 标志标记为[易失性](https://msdn.microsoft.com/en-us/library/x13ttww7.aspx)以确保对其的访问是 threadsafe。

    [!code-csharp[Main](tutorial-server-broadcast-with-aspnet-signalr/samples/sample9.cs)]

    在实际应用中，TryUpdateStockPrice 方法将调用 web 服务以查找价格;它在此代码中使用的随机数生成器随机进行更改。

    ### <a name="getting-the-signalr-context-so-that-the-stockticker-class-can-broadcast-to-clients"></a>获取 SignalR 上下文，以便可以向客户端进行广播 StockTicker 类

    因为价格更改此处来自 StockTicker 对象，所以这是需要在所有连接的客户端上调用 updateStockPrice 方法的对象。 在中心类必须 API 调用客户端方法，但 StockTicker 不是派生自的中心类，而未对任何中心对象的引用。 因此，要将广播到连接的客户端，StockTicker 类必须 SignalR 上下文实例获取 StockTickerHub 类并使用它来在客户端上调用方法。

    代码获取对 SignalR 上下文的引用，当它创建的单一类实例，将引用传递给构造函数中，并构造函数将其放置在客户端属性。

    有两个原因你想要只需一次获取上下文的原因： 获取上下文是代价高昂的操作，并一次获取确保发送到客户端的消息的预期的顺序均保留。

    [!code-csharp[Main](tutorial-server-broadcast-with-aspnet-signalr/samples/sample10.cs)]

    获取上下文的客户端属性，并将其放置在 StockTickerClient 属性允许你编写代码来调用方法的客户端就像在中心类中的显示相同。 例如，若要将广播到所有客户端可以编写 Clients.All.updateStockPrice(stock)。

    正在调用在 BroadcastStockPrice updateStockPrice 方法尚不存在;当你编写在客户端运行的代码，你将从更高版本添加它。 因为 Clients.All 是动态的这意味着将在运行时计算该表达式可以引用 updateStockPrice 此处。 当此方法调用执行时，SignalR 将将发送方法名称和参数值到客户端，并在客户端有一个名为 updateStockPrice 方法，将调用该方法和参数值将传递给它。

    Clients.All 意味着将发送到所有客户端。 SignalR 可其他选项来指定哪些客户端或客户端将发送到的组。 有关详细信息，请参阅[HubConnectionContext](https://msdn.microsoft.com/en-us/library/microsoft.aspnet.signalr.hubs.hubconnectioncontext(v=vs.111).aspx)。

### <a name="register-the-signalr-route"></a>注册 SignalR 路由

服务器需要知道要截获地将应用到 SignalR 的 URL。 为此，你将添加到某些代码*Global.asax*文件。

1. 在**解决方案资源管理器**，右键单击该项目，然后单击**添加新项**。
2. 选择**全局应用程序类**项模板，并依次**添加**。

    ![添加 global.asax](tutorial-server-broadcast-with-aspnet-signalr/_static/image6.png)
3. 将 SignalR 路由注册代码添加到应用程序\_Start 方法：

    [!code-csharp[Main](tutorial-server-broadcast-with-aspnet-signalr/samples/sample11.cs)]

    默认情况下，所有 SignalR 流量的基 URL 是"/ signalr"，并且"/ signalr/中心"用于检索动态生成的 JavaScript 文件，用于定义必须在你的应用程序中的所有中心的代理。 MapHubs 方法包括让你可以指定不同的基 URL 和某些 SignalR 选项实例中的重载[HubConfiguration](https://msdn.microsoft.com/en-us/library/microsoft.aspnet.signalr.hubconfiguration(v=vs.111).aspx)类。
4. 添加 using 语句文件的顶部：

    [!code-csharp[Main](tutorial-server-broadcast-with-aspnet-signalr/samples/sample12.cs)]
5. 保存并关闭*Global.asax*文件中，并生成项目。

你现在已经完成设置服务器代码。 在下一节将设置客户端。

<a id="client"></a>

## <a name="set-up-the-client-code"></a>设置客户端代码

1. 在项目文件夹中，创建一个新的 HTML 文件并将其命名*StockTicker.html*。
2. 模板代码替换为以下代码：

    [!code-html[Main](tutorial-server-broadcast-with-aspnet-signalr/samples/sample13.html)]

    HTML 5 列、 一个标题行，与具有跨所有 5 列的单个单元格的数据行创建一个表。 此数据行显示"正在加载..."和应用程序启动时暂时后，才会显示。 JavaScript 代码将删除该行，并在其位置行中添加与服务器中检索到的常用数据。

    脚本标记指定 jQuery 脚本文件、 SignalR 核心脚本文件、 SignalR 代理脚本文件和稍后要创建的 StockTicker 脚本文件。 SignalR 代理脚本文件的说明进行操作，它指定"/ signalr/中心"URL，动态生成，并在这种情况下定义 StockTickerHub.GetAllStocks 的中心类、 方法的代理方法。 如果你愿意，你可以通过使用手动生成此 JavaScript 文件[SignalR 实用工具](http://nuget.org/packages/Microsoft.AspNet.SignalR.Utils/)和禁用 MapHubs 方法调用中的动态文件创建。
3. > [!IMPORTANT]
 > 请确保 JavaScript 文件中引用*StockTicker.html*正确无误。 也就是说，确保你的脚本标记 (1.8.2 在示例中) 中的 jQuery 版本是否与你的项目中的 jQuery 版本相同*脚本*文件夹，并确保你的脚本标记中的 SignalR 版本是 SignalR 相同在项目的版本*脚本*文件夹。 如有必要，请更改脚本标记中的文件名称。
4. 在**解决方案资源管理器**，右键单击*StockTicker.html*，然后单击**设为起始页**。
5. 在项目文件夹中创建一个新的 JavaScript 文件并将其命名*StockTicker.js*...
6. 模板代码替换为以下代码：

    [!code-javascript[Main](tutorial-server-broadcast-with-aspnet-signalr/samples/sample14.js)]

    $.connection 指 SignalR 代理。 该代码 StockTickerHub 类获取代理的引用，并将其放入行情自动收录器变量。 代理名称是由 [HubName] 属性设置的名称：

    [!code-javascript[Main](tutorial-server-broadcast-with-aspnet-signalr/samples/sample15.js)]

    [!code-csharp[Main](tutorial-server-broadcast-with-aspnet-signalr/samples/sample16.cs)]

    定义所有变量和函数后，代码文件中的最后一行通过调用 SignalR 启动函数初始化 SignalR 连接。 启动函数以异步方式执行，并返回[jQuery 延迟对象](http://api.jquery.com/category/deferred-object/)，这意味着你可以调用要指定要在异步操作完成时调用的函数的 done 的函数...

    [!code-javascript[Main](tutorial-server-broadcast-with-aspnet-signalr/samples/sample17.js)]

    Init 函数调用在服务器上的 getAllStocks 函数，并使用服务器返回用于更新常用的表的信息。 请注意，默认情况下，您都需要使用 camel 大小写客户端上虽然方法名称为 pascal 大小写形式在服务器上。 Camel 大小写规则仅适用于方法，而非对象。 例如，您引用库存。符号和 stock。价格、 不 stock.symbol 或 stock.price。

    [!code-javascript[Main](tutorial-server-broadcast-with-aspnet-signalr/samples/sample18.js)]

    [!code-csharp[Main](tutorial-server-broadcast-with-aspnet-signalr/samples/sample19.cs)]

    如果你想要在客户端，使用 pascal 大小写或如果你想要使用完全不同的方法名称，无法修饰 HubMethodName 属性具有的中心方法相同的方式你修饰 HubName 属性具有的中心类本身。

    Init 方法，在为每个常用的对象，从服务器收到通过调用 formatStock 对常用的对象，格式属性创建的表行的 HTML，然后通过调用取代 (定义在顶部*StockTicker.js*) rowTemplate 变量中的占位符替换为常用对象属性值。 生成的 HTML 随后会追加到常用的表。

    通过将其作为执行异步启动函数完成后的回调函数中传递调用 init。 如果您调用 init 作为单独的 JavaScript 语句调用开始后，该函数将失败，因为它将不等待启动函数完成建立连接的情况下立即执行。 在这种情况下，init 函数会尝试调用 getAllStocks 函数之前建立服务器连接。

    当服务器更改股票的价格时，它将在连接的客户端上调用 updateStockPrice。 若要使其可供调用从服务器情况下，该函数添加到 stockTicker 代理的客户端属性。

    [!code-javascript[Main](tutorial-server-broadcast-with-aspnet-signalr/samples/sample20.js)]

    UpdateStockPrice 函数设置从服务器接收到的表行与 init 函数相同的方式的常用对象的格式。 但是，而不是将行追加到表，它将找到股票的当前行表中并替换为新替换该行。

<a id="test"></a>

## <a name="test-the-application"></a>测试应用程序

1. 按 F5 在调试模式下运行该应用程序。

    常用表最初显示"正在加载..."行，然后显示初始的常用数据时，出现短暂延迟后，然后股票价格启动以更改。

    ![“加载”](tutorial-server-broadcast-with-aspnet-signalr/_static/image7.png)

    ![初始股票表](tutorial-server-broadcast-with-aspnet-signalr/_static/image8.png)

    ![从服务器接收更改的常用表](tutorial-server-broadcast-with-aspnet-signalr/_static/image9.png)
2. 从浏览器地址栏中复制的 URL 并将其粘贴到一个或多个新的浏览器窗口。

    初始股票显示第一个浏览器相同，因此同时发生了更改。
3. 关闭所有浏览器并打开新浏览器，然后转到相同的 URL。

    StockTicker 单一实例对象方面持续运行在服务器中，因此常用表屏幕将显示 stocks 已继续更改。 （看不到具有零更改图形的初始表）。
4. 关闭浏览器。

<a id="enablelogging"></a>

## <a name="enable-logging"></a>启用日志记录

SignalR 具有内置的日志记录功能，你可以启用客户端上以帮助解决疑难问题。 在本部分启用日志记录，并说明如何日志告诉你哪些以下传输方法使用 SignalR 的示例，请参阅：

- [Websocket](http://en.wikipedia.org/wiki/WebSocket)、 IIS 8 和当前浏览器支持。
- [服务器发送事件](http://en.wikipedia.org/wiki/Server-sent_events)、 Internet Explorer 以外的浏览器支持。
- [不限次数帧](http://en.wikipedia.org/wiki/Comet_(programming)#Hidden_iframe)，支持由 Internet Explorer。
- [Ajax 长轮询](http://en.wikipedia.org/wiki/Comet_(programming)#Ajax_with_long_polling)，所有浏览器支持。

对于任何给定的连接，SignalR 选择服务器和客户端支持的最佳传输方法。

1. 打开*StockTicker.js*并添加要启用日志记录之前初始化的连接在文件末尾的代码的代码行：

    [!code-javascript[Main](tutorial-server-broadcast-with-aspnet-signalr/samples/sample21.js)]
2. 按 F5 运行项目。
3. 打开浏览器的开发人员工具窗口，并选择控制台以查看日志。 你可能必须刷新页面以查看 Signalr 协商新连接的传输方法的日志。

    如果在 Windows 8 (IIS 8) 上正在运行 Internet Explorer 10，则传输方法将是 Websocket。

    ![IE 10 IIS 8 控制台](tutorial-server-broadcast-with-aspnet-signalr/_static/image10.png)

    如果在 Windows 7 (IIS 7.5) 上正在运行 Internet Explorer 10，则传输方法将是 iframe。

    ![IE 10 控制台中，IIS 7.5](tutorial-server-broadcast-with-aspnet-signalr/_static/image11.png)

    在 Firefox，安装的 Firebug 外接程序以获取控制台窗口。 如果你在 Windows 8 (IIS 8) 上运行 Firefox 19，传输方法将是 Websocket。

    ![Firefox 19 IIS 8 Websocket](tutorial-server-broadcast-with-aspnet-signalr/_static/image12.png)

    如果你在 Windows 7 (IIS 7.5) 上运行 Firefox 19，传输方法将是服务器发送事件。

    ![Firefox 19 IIS 7.5 控制台](tutorial-server-broadcast-with-aspnet-signalr/_static/image13.png)

<a id="fullsample"></a>

## <a name="install-and-review-the-full-stockticker-sample"></a>安装和查看完整的 StockTicker 示例

StockTicker 安装的应用程序是通过[Microsoft.AspNet.SignalR.Sample](http://nuget.org/packages/microsoft.aspnet.signalr.sample) NuGet 包包含多个功能比你刚刚创建从零开始的简化版本。 在本教程的此部分中，你将安装 NuGet 包，并检查新功能和实现它们的代码。

### <a name="install-the-signalrsample-nuget-package"></a>安装 SignalR.Sample NuGet 包

1. 在**解决方案资源管理器**，右键单击项目，然后单击**管理 NuGet 包**。
2. 在**管理 NuGet 包**对话框中，单击**联机**，输入*SignalR.Sample*中**联机搜索**框中，并依次**安装**中**SignalR.Sample**包。

    ![安装 SignalR.Sample 包](tutorial-server-broadcast-with-aspnet-signalr/_static/image14.png)
3. 在*Global.asax*文件，注释掉 RouteTable.Routes.MapHubs(); 行你前面在应用程序中添加\_Start 方法。

    中的代码*Global.asax*不再需要因为 SignalR.Sample 包注册中的 SignalR 路由*应用\_Start/RegisterHubs.cs*文件：

    [!code-csharp[Main](tutorial-server-broadcast-with-aspnet-signalr/samples/sample22.cs)]

    引用的程序集特性 WebActivator 类包含在 WebActivatorEx NuGet 包，它将安装为 SignalR.Sample 包的依赖项。
4. 在**解决方案资源管理器**，展开*SignalR.Sample*由安装 SignalR.Sample 包的文件夹。
5. 在*SignalR.Sample*文件夹中，右键单击*StockTicker.html*，然后单击**设为起始页**。

    > [!NOTE]
    > 安装 SignalR.Sample NuGet 包可能会更改的版本中，你拥有的 jQuery 你*脚本*文件夹。 新*StockTicker.html*该包将安装的文件*SignalR.Sample*文件夹将为与包安装，但如果你想要运行你的原始的jQuery版本同步*StockTicker.html*再次文件中，你可能需要首先更新中的脚本标记的 jQuery 引用。

### <a name="run-the-application"></a>运行此应用程序

1. 按 F5 运行该应用程序。

    除了前面所看到的网格中，完整股票行情自动收录器应用程序显示水平滚动窗口显示相同的常用数据。 当你运行的应用程序第一次时，"市场""已关闭"，你看到静态网格中，然后不滚动行情自动收录器窗口。

    ![StockTicker 屏幕开始](tutorial-server-broadcast-with-aspnet-signalr/_static/image15.png)

    当你单击**公开市场**、**实时股票行情自动收录器**框开始水平，向下滚动并在服务器启动定期广播上的随机基础的股票价格更改。 每次股票价格更改，同时**实时股票表格**网格和**实时股票行情自动收录器**框会更新。 当常用的价格更改为正，使用绿色背景; 显示股票并且时更改为负，包含一个红色背景显示股票。

    ![StockTicker 应用市场打开](tutorial-server-broadcast-with-aspnet-signalr/_static/image16.png)

    **关闭市场**按钮停止所做的更改并停止行情自动收录器滚动和**重置**按钮可重置所有股票数据到初始状态之前启动价格更改。 如果您打开多个浏览器窗口并转到同一个 URL，你将看到相同的数据在同一时间在每个浏览器中动态更新。 当你单击的按钮之一时，所有浏览器都具有相同的方式在同一时间。

### <a name="live-stock-ticker-display"></a>实时股票行情自动收录器显示

**实时股票行情自动收录器**显示为无序的列表中的 CSS 样式格式化为单个行的 div 元素。 行情自动收录器会被初始化并更新与表相同的方式： 通过替换中的占位符&lt;li&gt;模板字符串和动态添加&lt;li&gt;元素与&lt;ul&gt;元素。 使用 jQuery 动画函数改变左边距的分区中的无序列表中滚动执行

股票行情 HTML:

[!code-html[Main](tutorial-server-broadcast-with-aspnet-signalr/samples/sample23.html)]

股票行情 CSS:

[!code-html[Main](tutorial-server-broadcast-with-aspnet-signalr/samples/sample24.html)]

JQuery 代码使之滚动：

[!code-javascript[Main](tutorial-server-broadcast-with-aspnet-signalr/samples/sample25.js)]

### <a name="additional-methods-on-the-server-that-the-client-can-call"></a>客户端可以调用的服务器上的其他方法

StockTickerHub 类定义客户端可以调用的四个附加方法：

[!code-csharp[Main](tutorial-server-broadcast-with-aspnet-signalr/samples/sample26.cs)]

在页面顶部的按钮响应称为 OpenMarket、 CloseMarket 和重置。 它们演示了触发立即传播到所有客户端的状态的更改的一台客户端的模式。 上述每种方法中调用一个方法 StockTicker 类该效果的市场状态更改，然后广播的新状态。

在 StockTicker 类中，通过返回 MarketState 枚举值的 MarketState 属性维护市场的状态：

[!code-csharp[Main](tutorial-server-broadcast-with-aspnet-signalr/samples/sample27.cs)]

每个更改市场状态方法这样做，在一个锁块内因为 StockTicker 类必须是 threadsafe:

[!code-csharp[Main](tutorial-server-broadcast-with-aspnet-signalr/samples/sample28.cs)]

若要确保此代码是 threadsafe，\_支持 MarketState 属性的 marketState 字段标记为易失的

[!code-csharp[Main](tutorial-server-broadcast-with-aspnet-signalr/samples/sample29.cs)]

除它们调用不同的方法在客户端上定义外的 BroadcastMarketStateChange 和 BroadcastMarketReset 方法是类似于你已看到的 BroadcastStockPrice 方法：

[!code-csharp[Main](tutorial-server-broadcast-with-aspnet-signalr/samples/sample30.cs)]

### <a name="additional-functions-on-the-client-that-the-server-can-call"></a>该服务器可以调用的客户端上的其他函数

UpdateStockPrice 函数现在处理网格和行情自动收录器显示，并使用 jQuery.Color 闪烁红色和绿色的颜色。

中的新功能*SignalR.StockTicker.js*启用和禁用按钮基于市场状态，并且它们停止或开始行情自动收录器窗口水平滚动。 由于多个函数将被添加到 ticker.client， [jQuery 扩展函数](http://api.jquery.com/jQuery.extend/)用于添加它们。

[!code-javascript[Main](tutorial-server-broadcast-with-aspnet-signalr/samples/sample31.js)]

### <a name="additional-client-setup-after-establishing-the-connection"></a>在建立连接后的其他客户端安装程序

客户端建立连接后，它具有一些附加工作以执行操作： 找出市场是否是打开还是关闭以便调用的相应 marketOpened 或 marketClosed 函数，并附加到按钮的服务器方法调用。

[!code-javascript[Main](tutorial-server-broadcast-with-aspnet-signalr/samples/sample32.js)]

服务器方法不是有线到之前的按钮后建立的连接，从而使代码不能尝试之前可用，调用服务器方法。

<a id="nextsteps"></a>

## <a name="next-steps"></a>后续步骤

在本教程中，你已了解如何从服务器将消息广播到所有连接的客户端，在定期和响应来自任何客户端通知 SignalR 应用程序进行编程。 使用多线程的单一实例维护服务器状态的模式也还可在多玩家 online 游戏方案。 有关示例，请参阅[基于 SignalR ShootR 游戏](https://github.com/NTaylorMullen/ShootR)。

显示对等通信方案的教程，请参阅[Getting Started with SignalR](index.md)和[实时更新使用 SignalR](index.md)。

若要了解更多高级的 SignalR 开发概念，请访问以下站点 SignalR 源代码和资源：

- [ASP.NET SignalR](https://asp.net/signalr/)
- [SignalR 项目](http://signalr.net/)
- [SignalR Github 和示例](https://github.com/SignalR/SignalR)
- [SignalR Wiki](https://github.com/SignalR/SignalR/wiki)
