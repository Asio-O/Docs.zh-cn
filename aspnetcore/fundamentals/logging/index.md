---
title: ASP.NET Core 中的日志记录
author: ardalis
description: 了解 ASP.NET Core 中的记录框架。 发现内置日志记录提供程序，并详细了解常见第三方提供程序。
ms.author: tdykstra
ms.date: 07/24/2018
uid: fundamentals/logging/index
ms.openlocfilehash: 35bb7fa51db541f825a79151fb7fbe85d48e1998
ms.sourcegitcommit: 028ad28c546de706ace98066c76774de33e4ad20
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/08/2018
ms.locfileid: "39655354"
---
# <a name="logging-in-aspnet-core"></a>ASP.NET Core 中的日志记录

作者：[Steve Smith](https://ardalis.com/) 和 [Tom Dykstra](https://github.com/tdykstra)

ASP.NET Core 支持适用于各种日志记录提供程序的日志记录 API。 通过内置提供程序，可向一个或多个目标发送日志，还可插入第三方记录框架。 本文介绍如何在代码中使用内置日志记录 API 和提供程序。

::: moniker range=">= aspnetcore-2.0"

[查看或下载示例代码](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/logging/index/sample2)（[如何下载](xref:tutorials/index#how-to-download-a-sample)）

::: moniker-end

::: moniker range="< aspnetcore-2.0"

[查看或下载示例代码](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/logging/index/sample)（[如何下载](xref:tutorials/index#how-to-download-a-sample)）

::: moniker-end

有关使用 IIS 进行托管时的 stdout 日志记录的信息，请参阅 <xref:host-and-deploy/iis/troubleshoot#aspnet-core-module-stdout-log>。 有关使用 Azure 应用服务进行 stdout 日志记录的信息，请参阅 <xref:host-and-deploy/azure-apps/troubleshoot#aspnet-core-module-stdout-log>。

## <a name="how-to-create-logs"></a>如何创建日志

要创建日志，请从[依赖关系注入](xref:fundamentals/dependency-injection)容器实现 [ILogger&lt;TCategoryName&gt;](/dotnet/api/microsoft.extensions.logging.ilogger-1) 对象：

[!code-csharp[](index/sample/Controllers/TodoController.cs?name=snippet_LoggerDI&highlight=7)]

然后在该记录器对象上调用日志记录方法：

[!code-csharp[](index/sample/Controllers/TodoController.cs?name=snippet_CallLogMethods&highlight=3,7)]

此示例使用 `TodoController` 类创建日志作为类别。 [本文的稍后部分](#log-category)对这些类别进行了介绍。

ASP.NET Core 不提供异步记录器方法，因为日志记录的速度应非常快，使用异步的代价是不值得的。 如果发现自己的实际情况与上述不同，请考虑更改记录方式。 如果数据存储速度较慢，请先将日志消息写入快速存储，稍后再将其转移至低速存储。 例如，记录到由另一进程读取和暂留以减缓存储的消息队列。

## <a name="how-to-add-providers"></a>如何添加提供程序

::: moniker range=">= aspnetcore-2.0"

日志记录提供程序通过 `ILogger` 对象获取所创建的消息，并显示或存储它们。 例如，控制台提供程序在控制台上显示消息，Azure App Service 提供程序可将消息存储在 Azure blob 存储中。

要使用提供程序，请在 Program.cs 中调用提供程序的 `Add<ProviderName>` 扩展方法：

[!code-csharp[](index/sample2/Program.cs?name=snippet_ExpandDefault&highlight=16,17)]

借助默认项目模板，可以在 Program.cs 中调用 [CreateDefaultBuilder](/dotnet/api/microsoft.aspnetcore.webhost.createdefaultbuilder) 扩展方法，从而启用控制台和调试日志记录提供程序：

[!code-csharp[](index/sample2/Program.cs?name=snippet_TemplateCode&highlight=7)]

::: moniker-end

::: moniker range="< aspnetcore-2.0"

日志记录提供程序通过 `ILogger` 对象获取所创建的消息，并显示或存储它们。 例如，控制台提供程序在控制台上显示消息，Azure App Service 提供程序可将消息存储在 Azure blob 存储中。

要使用提供程序，请安装其 NuGet 包，并在 [ILoggerFactory](/dotnet/api/microsoft.extensions.logging.iloggerfactory) 的实例上调用提供程序的扩展方法，如下方示例所示：

[!code-csharp[](index/sample//Startup.cs?name=snippet_AddConsoleAndDebug&highlight=3,5-7)]

ASP.NET Core [依赖关系注入](xref:fundamentals/dependency-injection) (DI) 将提供 `ILoggerFactory` 实例。 在 [Microsoft.Extensions.Logging.Console](https://www.nuget.org/packages/Microsoft.Extensions.Logging.Console/) 和 [Microsoft.Extensions.Logging.Debug](https://www.nuget.org/packages/Microsoft.Extensions.Logging.Debug/) 包中定义了 `AddConsole` 和 `AddDebug` 扩展方法。 每个扩展方法都调用 `ILoggerFactory.AddProvider` 方法，传入提供程序的一个实例。

> [!NOTE]
> [示例应用](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/logging/index/sample)在 `Startup.Configure` 方法中添加了日志提供程序。 要从先前执行的代码获取日志输出，请在 `Startup` 类构造函数中添加日志提供程序。

::: moniker-end

详细了解[内置日志记录提供程序](#built-in-logging-providers)，并查找本文稍后部分介绍的[第三方日志记录提供程序](#third-party-logging-providers)的链接。

## <a name="configuration"></a>配置

日志记录提供程序配置由一个或多个配置提供程序提供：

* 文件格式（INI、JSON 和 XML）。
* 命令行参数。
* 环境变量。
* 内存中的 .NET 对象。
* 未加密的[机密管理器](xref:security/app-secrets)存储。
* 加密的用户存储，如 [Azure Key Vault](xref:security/key-vault-configuration)。
* （已安装或已创建的）自定义提供程序。

例如，日志记录配置通常由应用设置文件的 `Logging` 部分提供。 以下示例显示了典型 *appsettings.Development.json* 文件的内容：

::: moniker range=">= aspnetcore-2.1"

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Debug",
      "System": "Information",
      "Microsoft": "Information"
    },
    "Console":
    {
      "IncludeScopes": "true"
    }
  }
}
```

`LogLevel` 项表示日志名称。 `Default` 项适用于未显式列出的日志。 其值表示应用于给定日志的[日志级别](#log-level)。 设置 `IncludeScopes`（示例中的 `Console`）的日志项指定是否为相关日志启用了[日志作用域](#log-scopes)。

::: moniker-end

::: moniker range="< aspnetcore-2.1"

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Debug",
      "System": "Information",
      "Microsoft": "Information"
    }
  }
}
```

`LogLevel` 项表示日志名称。 `Default` 项适用于未显式列出的日志。 其值表示应用于给定日志的[日志级别](#log-level)。

::: moniker-end

若要了解如何实现配置提供程序，请参阅 <xref:fundamentals/configuration/index>。

## <a name="sample-logging-output"></a>日志记录输出示例

从命令行运行上一部分所示的示例代码时，将在控制台中看到日志。 以下是控制台输出示例：

```console
info: Microsoft.AspNetCore.Hosting.Internal.WebHost[1]
      Request starting HTTP/1.1 GET http://localhost:5000/api/todo/0
info: Microsoft.AspNetCore.Mvc.Internal.ControllerActionInvoker[1]
      Executing action method TodoApi.Controllers.TodoController.GetById (TodoApi) with arguments (0) - ModelState is Valid
info: TodoApi.Controllers.TodoController[1002]
      Getting item 0
warn: TodoApi.Controllers.TodoController[4000]
      GetById(0) NOT FOUND
info: Microsoft.AspNetCore.Mvc.StatusCodeResult[1]
      Executing HttpStatusCodeResult, setting HTTP status code 404
info: Microsoft.AspNetCore.Mvc.Internal.ControllerActionInvoker[2]
      Executed action TodoApi.Controllers.TodoController.GetById (TodoApi) in 42.9286ms
info: Microsoft.AspNetCore.Hosting.Internal.WebHost[2]
      Request finished in 148.889ms 404
```

转到 `http://localhost:5000/api/todo/0`，触发上一部分所示的两个 `ILogger` 调用的执行，创建了以上日志。

在 Visual Studio 中运行示例应用程序时，“调试”窗口中将显示如下日志：

```console
Microsoft.AspNetCore.Hosting.Internal.WebHost:Information: Request starting HTTP/1.1 GET http://localhost:53104/api/todo/0  
Microsoft.AspNetCore.Mvc.Internal.ControllerActionInvoker:Information: Executing action method TodoApi.Controllers.TodoController.GetById (TodoApi) with arguments (0) - ModelState is Valid
TodoApi.Controllers.TodoController:Information: Getting item 0
TodoApi.Controllers.TodoController:Warning: GetById(0) NOT FOUND
Microsoft.AspNetCore.Mvc.StatusCodeResult:Information: Executing HttpStatusCodeResult, setting HTTP status code 404
Microsoft.AspNetCore.Mvc.Internal.ControllerActionInvoker:Information: Executed action TodoApi.Controllers.TodoController.GetById (TodoApi) in 152.5657ms
Microsoft.AspNetCore.Hosting.Internal.WebHost:Information: Request finished in 316.3195ms 404
```

由上一部分所示的 `ILogger` 调用创建的日志以“TodoApi.Controllers.TodoController”开头的。 以“Microsoft”类别开头的日志来自 ASP.NET Core。 ASP.NET Core 本身和应用程序代码使用相同的日志记录 API 和日志记录提供程序。

本文余下部分将介绍有关日志记录的某些详细信息及选项。

## <a name="nuget-packages"></a>NuGet 包

`ILogger` 和 `ILoggerFactory` 接口位于 [Microsoft.Extensions.Logging.Abstractions](https://www.nuget.org/packages/Microsoft.Extensions.Logging.Abstractions/) 中，其默认实现位于 [Microsoft.Extensions.Logging](https://www.nuget.org/packages/microsoft.extensions.logging/) 中。

## <a name="log-category"></a>日志类别

所创建的每个日志都包含一个类别。 在创建 `ILogger` 对象时指定类别。 类别可以是任意字符串，但约定使用写入日志的类的完全限定名称。 例如“TodoApi.Controllers.TodoController”。

可以将类别指定为字符串，或使用从类型派生类别的扩展方法。 要将类别指定为字符串，请在 `ILoggerFactory` 实例上调用 `CreateLogger`，如下所示。

[!code-csharp[](index/sample//Controllers/TodoController.cs?name=snippet_CreateLogger&highlight=7,10)]

如下方示例所示，在大多数情况下使用 `ILogger<T>` 更简单。

[!code-csharp[](index/sample//Controllers/TodoController.cs?name=snippet_LoggerDI&highlight=7)]

这相当于使用 `T` 的完全限定类型名称来调用 `CreateLogger`。

## <a name="log-level"></a>日志级别

每次写入日志时都需指定其 [LogLevel](/dotnet/api/microsoft.extensions.logging.logLevel)。 日志级别指示严重性或重要程度。 例如，如果方法正常结束则写入 `Information` 日志，如果方法返回 404 返回代码则写入 `Warning` 日志，如果捕获未知异常则写入 `Error` 日志。

在下列代码示例中，由方法的名称（如 `LogWarning`）指定日志级别。 第一个参数是[日志事件 ID](#log-event-id)。 第二个形参是[消息模板](#log-message-template)，包含由剩余方法形参提供的实参值的占位符。 本文稍后部分更详细地介绍了方法参数。

[!code-csharp[](index/sample//Controllers/TodoController.cs?name=snippet_CallLogMethods&highlight=3,7)]

在方法名称中包含级别的日志方法是 [ILogger 的扩展方法](/dotnet/api/microsoft.extensions.logging.loggerextensions)。 在后台，这些方法调用可接受 `LogLevel` 参数的 `Log` 方法。 可直接调用 `Log` 方法而不调用其中某个扩展方法，但语法相对复杂。 有关详细信息，请参阅 [ILogger 接口](/dotnet/api/microsoft.extensions.logging.ilogger)和[记录器扩展源代码](https://github.com/aspnet/Logging/blob/master/src/Microsoft.Extensions.Logging.Abstractions/LoggerExtensions.cs)。

ASP.NET Core 定义了以下[日志级别](/dotnet/api/microsoft.extensions.logging.loglevel)（按严重性从低到高排列）。

* 跟踪 = 0

  表示仅对于开发人员调试问题有价值的信息。 这些消息可能包含敏感应用程序数据，因此不得在生产环境中启用它们。 默认情况下禁用。 示例：`Credentials: {"User":"someuser", "Password":"P@ssword"}`

* 调试 = 1

  表示在开发和调试过程中短期有用的信息。 示例：`Entering method Configure with flag set to true.`。除非要排查问题，否则通常不会在生产中启用 `Debug` 级别日志，因为日志数量过多。

* 信息 = 2

  用于跟踪应用程序的常规流。 这些日志通常有长期价值。 示例：`Request received for path /api/todo`

* 警告 = 3

  表示应用程序流中的异常或意外事件。 可能包括不会中断应用程序运行但仍需调查的错误或其他条件。 `Warning` 日志级别常用于已处理的异常。 示例：`FileNotFoundException for file quotes.txt.`

* 错误 = 4

  表示无法处理的错误和异常。 这些消息指示的是当前活动或操作（如当前 HTTP 请求）中的失败，而不是应用程序范围的失败。 日志消息示例：`Cannot insert record due to duplicate key violation.`

* 严重 = 5

  需要立即关注的失败。 例如数据丢失、磁盘空间不足。

可以使用日志级别控制写入到特定存储介质或显示窗口的日志输出量。 例如在生产中，可将所有 `Information` 及以下级别的日志放在卷数据存储，将所有 `Warning` 及以上级别的日志放在值数据存储。 在开发期间，通常要将严重性为 `Warning` 或以上级别的日志发送到控制台。 需要进行故障排除时，可添加 `Debug` 级别。 本文稍后的[日志筛选](#log-filtering)部分介绍如何控制提供程序处理的日志级别。

ASP.NET Core 框架为框架事件写入 `Debug` 级别日志。 本文前面部分提供的日志示例排除了低于 `Information` 级别的日志，因此未显示 `Debug` 级别日志。 如果运行配置为显示控制台提供程序 `Debug` 及以上级别的日志的示例应用程序，则控制台日志示例如下所示。

```console
info: Microsoft.AspNetCore.Hosting.Internal.WebHost[1]
      Request starting HTTP/1.1 GET http://localhost:62555/api/todo/0
dbug: Microsoft.AspNetCore.Routing.Tree.TreeRouter[1]
      Request successfully matched the route with name 'GetTodo' and template 'api/Todo/{id}'.
dbug: Microsoft.AspNetCore.Mvc.Internal.ActionSelector[2]
      Action 'TodoApi.Controllers.TodoController.Update (TodoApi)' with id '089d59b6-92ec-472d-b552-cc613dfd625d' did not match the constraint 'Microsoft.AspNetCore.Mvc.Internal.HttpMethodActionConstraint'
dbug: Microsoft.AspNetCore.Mvc.Internal.ActionSelector[2]
      Action 'TodoApi.Controllers.TodoController.Delete (TodoApi)' with id 'f3476abe-4bd9-4ad3-9261-3ead09607366' did not match the constraint 'Microsoft.AspNetCore.Mvc.Internal.HttpMethodActionConstraint'
dbug: Microsoft.AspNetCore.Mvc.Internal.ControllerActionInvoker[1]
      Executing action TodoApi.Controllers.TodoController.GetById (TodoApi)
info: Microsoft.AspNetCore.Mvc.Internal.ControllerActionInvoker[1]
      Executing action method TodoApi.Controllers.TodoController.GetById (TodoApi) with arguments (0) - ModelState is Valid
info: TodoApi.Controllers.TodoController[1002]
      Getting item 0
warn: TodoApi.Controllers.TodoController[4000]
      GetById(0) NOT FOUND
dbug: Microsoft.AspNetCore.Mvc.Internal.ControllerActionInvoker[2]
      Executed action method TodoApi.Controllers.TodoController.GetById (TodoApi), returned result Microsoft.AspNetCore.Mvc.NotFoundResult.
info: Microsoft.AspNetCore.Mvc.StatusCodeResult[1]
      Executing HttpStatusCodeResult, setting HTTP status code 404
info: Microsoft.AspNetCore.Mvc.Internal.ControllerActionInvoker[2]
      Executed action TodoApi.Controllers.TodoController.GetById (TodoApi) in 0.8788ms
dbug: Microsoft.AspNetCore.Server.Kestrel[9]
      Connection id "0HL6L7NEFF2QD" completed keep alive response.
info: Microsoft.AspNetCore.Hosting.Internal.WebHost[2]
      Request finished in 2.7286ms 404
```

## <a name="log-event-id"></a>日志事件 ID

每次写入日志时都可指定一个事件 ID。 该示例应用通过使用本地定义的 `LoggingEvents` 类来执行此操作：

[!code-csharp[](index/sample//Controllers/TodoController.cs?name=snippet_CallLogMethods&highlight=3,7)]

[!code-csharp[](index/sample//Core/LoggingEvents.cs?name=snippet_LoggingEvents)]

事件 ID 是一个整数值，可用于关联多组已记录事件。 例如，向购物车添加项的日志可以是事件 ID 1000，完成购买的日志可以是事件 ID 1001。

在日志记录输出中，事件 ID 既可存储在字段里，也可包含在文本消息内，这由提供程序来决定。 调试提供程序不显示事件 ID，但控制台提供程序会将其显示在类别后的括号里：

```console
info: TodoApi.Controllers.TodoController[1002]
      Getting item invalidid
warn: TodoApi.Controllers.TodoController[4000]
      GetById(invalidid) NOT FOUND
```

## <a name="log-message-template"></a>日志消息模板

每次写入日志消息都会提供一个消息模板。 消息模板可以是字符串，也可以包含放置参数值的命名占位符。 该模板不是格式字符串，应对占位符进行命名而不是编号。

[!code-csharp[](index/sample//Controllers/TodoController.cs?name=snippet_CallLogMethods&highlight=3,7)]

占位符的顺序（而非其名称）决定了为其提供值的参数。 如果你有以下代码：

```csharp
string p1 = "parm1";
string p2 = "parm2";
_logger.LogInformation("Parameter values: {p2}, {p1}", p1, p2);
```

生成的日志消息如下所示：

```
Parameter values: parm1, parm2
```

记录框架以这种方式对消息进行格式化，以便日志记录提供程序能实现[语义日志记录（又称结构化日志记录）](https://softwareengineering.stackexchange.com/questions/312197/benefits-of-structured-logging-vs-basic-logging)。 由于传递到日志记录系统的是参数本身（而不仅仅是格式化的消息模板），因此除消息模板外，日志记录提供程序还可将参数值存储为字段。 如果将日志输出定向到 Azure 表存储，则记录器方法调用将如下所示：

```csharp
_logger.LogInformation("Getting item {ID} at {RequestTime}", id, DateTime.Now);
```

每个 Azure 表实体都具有 `ID` 和 `RequestTime` 属性，简化了对日志数据的查询。 无需分析文本消息的超时，即可找到特定 `RequestTime` 范围内的全部日志。

## <a name="logging-exceptions"></a>日志记录异常

记录器方法有可传入异常的重载，如下方示例所示：

[!code-csharp[](index/sample//Controllers/TodoController.cs?name=snippet_LogException&highlight=3)]

不同的提供程序处理异常信息的方式不同。 以下是上示代码的调试提供程序输出示例。

```
TodoApi.Controllers.TodoController:Warning: GetById(036dd898-fb01-47e8-9a65-f92eb73cf924) NOT FOUND

System.Exception: Item not found exception.
 at TodoApi.Controllers.TodoController.GetById(String id) in C:\logging\sample\src\TodoApi\Controllers\TodoController.cs:line 226
```

## <a name="log-filtering"></a>日志筛选

::: moniker range=">= aspnetcore-2.0"

可为特定或所有提供程序和类别指定最低日志级别。 最低级别以下的日志不会传递给该提供程序，因此不会显示或存储它们。

要禁止显示所有日志，可将 `LogLevel.None` 指定为最低日志级别。 `LogLevel.None` 的整数值为 6，它大于 `LogLevel.Critical` (5)。

### <a name="create-filter-rules-in-configuration"></a>在配置中创建筛选规则

项目模板创建调用 `CreateDefaultBuilder` 的代码，以便为控制台和调试提供程序设置日志记录。 `CreateDefaultBuilder` 方法还使用如下所示的代码，设置日志记录以查找 `Logging` 部分的配置：

[!code-csharp[](index/sample2/Program.cs?name=snippet_ExpandDefault&highlight=15)]

配置数据按提供程序和类别指定最低日志级别，如下方示例所示：

[!code-json[](index/sample2/appsettings.json)]

此 JSON 将创建六条筛选规则，一条用于调试提供程序，四条用于控制台提供程序，一条用于所有提供程序。 稍后你将了解在创建 `ILogger` 对象后，如何为每个提供程序从上述规则中选择其一。

### <a name="filter-rules-in-code"></a>代码中的筛选规则

可在代码中注册筛选规则，如下方示例所示：

[!code-csharp[](index/sample2/Program.cs?name=snippet_FilterInCode&highlight=4-5)]

第二个 `AddFilter` 使用类型名称来指定调试提供程序。 第一个 `AddFilter` 应用于全部提供程序，因为它未指定提供程序类型。

### <a name="how-filtering-rules-are-applied"></a>如何应用筛选规则

先前示例中显示的配置数据和 `AddFilter` 代码会创建下表所示的规则。 前六条由配置示例创建，后两条由代码示例创建。

| 数字 | 提供程序      | 类别的开头为...          | 最低日志级别 |
| :----: | ------------- | --------------------------------------- | ----------------- |
| 1      | 调试         | 全部类别                          | 信息       |
| 2      | 控制台       | Microsoft.AspNetCore.Mvc.Razor.Internal | 警告           |
| 3      | 控制台       | Microsoft.AspNetCore.Mvc.Razor.Razor    | 调试             |
| 4      | 控制台       | Microsoft.AspNetCore.Mvc.Razor          | Error             |
| 5      | 控制台       | 全部类别                          | 信息       |
| 6      | 全部提供程序 | 全部类别                          | 调试             |
| 7      | 全部提供程序 | 系统                                  | 调试             |
| 8      | 调试         | Microsoft                               | 跟踪             |

创建 `ILogger` 对象来写入日志时，`ILoggerFactory` 对象将从根据提供程序选择一条规则，将其应用于该记录器。 将按所选规则筛选该 `ILogger` 对象写入的所有消息。 从可用规则中为每个提供程序和类别对选择最具体的规则。

在为给定的类别创建 `ILogger` 时，以下算法将用于每个提供程序：

* 选择匹配提供程序或其别名的所有规则。 如果找不到，则选择具有空提供程序的所有规则。
* 根据上一步的结果，选择具有最长匹配类别前缀的规则。 如果找不到，则选择未指定类别的所有规则。
* 如果选择了多条规则，则采用最后一条。
* 如果未选择任何规则，则使用 `MinimumLevel`。

例如，假设你拥有上述规则列表，并为类别“Microsoft.AspNetCore.Mvc.Razor.RazorViewEngine”创建了 `ILogger` 对象：

* 对于调试提供程序，规则 1、6 和 8 适用。 规则 8 最为具体，因此选择了它。
* 对于控制台提供程序，符合的有规则 3、规则 4、规则 5 和规则 6。 规则 3 最为具体。

在使用 `ILogger` 为类别“Microsoft.AspNetCore.Mvc.Razor.RazorViewEngine”创建日志时，`Trace` 及以上级别的日志将发送到调试提供程序，`Debug` 及以上级别的日志将发送到控制台提供程序。

### <a name="provider-aliases"></a>提供程序别名

虽然可以使用类型名称在配置中指定提供程序，但每个提供程序都定义了更短且更易于使用的别名。 对于内置提供程序，请使用以下别名：

* 控制台
* 调试
* EventLog
* AzureAppServices
* TraceSource
* EventSource

### <a name="default-minimum-level"></a>默认最低级别

仅当配置或代码中的规则对给定提供程序和类别都不适用时，最低级别设置才会生效。 下面的示例演示如何设置最低级别：

[!code-csharp[](index/sample2/Program.cs?name=snippet_MinLevel&highlight=3)]

如果没有明确设置最低级别，则默认值为 `Information`，它表示 `Trace` 和 `Debug` 日志将被忽略。

### <a name="filter-functions"></a>筛选器函数

可向筛选器函数写入代码以应用筛选规则。 对配置或代码没有向其分配规则的所有提供程序和类别调用筛选器函数。 函数中的代码有权访问提供程序类型、类别和日志级别，以决定是否记录某条消息。 例如:

[!code-csharp[](index/sample2/Program.cs?name=snippet_FilterFunction&highlight=5-13)]

::: moniker-end

::: moniker range="< aspnetcore-2.0"

通过某些日志记录提供程序，可根据日志级别和类别指定何时向存储介质写入日志、何时忽略日志。

`AddConsole` 和 `AddDebug` 扩展方法提供允许传入筛选条件的重载。 下列示例代码将使控制台提供程序忽略低于 `Warning` 级别的日志，而使调试提供程序忽略由框架创建的日志。

[!code-csharp[](index/sample/Startup.cs?name=snippet_AddConsoleAndDebugWithFilter&highlight=6-7)]

`AddEventLog` 方法拥有接受 `EventLogSettings` 实例的重载，该实例的 `Filter` 属性中可能包含筛选函数。 TraceSource 提供程序不提供以上任何重载，因为其日志记录级别和其他参数都以它使用的 `SourceSwitch` 和 `TraceListener` 为依据。

可以使用 `WithFilter` 扩展方法为所有通过 `ILoggerFactory` 实例注册的提供程序设置筛选规则。 以下示例将（类别以“Microsoft”或“System”开头的）框架日志限制为警告，并在调试级别记录应用日志。

[!code-csharp[](index/sample/Startup.cs?name=snippet_FactoryFilter&highlight=6-11)]

要通过筛选来防止写入某个特定类别的所有日志，可将 `LogLevel.None` 指定为该类别的最低日志级别。 `LogLevel.None` 的整数值为 6，它大于 `LogLevel.Critical` (5)。

`WithFilter` 扩展方法由 [Microsoft.Extensions.Logging.Filter](https://www.nuget.org/packages/Microsoft.Extensions.Logging.Filter) NuGet 包提供。 该方法返回一个新的 `ILoggerFactory` 实例，该实例将筛选传递给注册的所有记录器提供程序的日志消息。 它不会影响其他任何 `ILoggerFactory` 实例，包括原始 `ILoggerFactory` 实例。

::: moniker-end

## <a name="log-scopes"></a>日志作用域

可以将逻辑操作集划入范围，从而将相同的数据附加到在此集中创建的每个日志。 例如，可让处理事务时创建的每个日志都包含事务 ID。

范围是由 [ILogger.BeginScope&lt;TState&gt;](/dotnet/api/microsoft.extensions.logging.ilogger.beginscope) 方法返回的 `IDisposable` 类型，持续至释放为止。 要使用作用域，请在 `using` 块中包装记录器调用，如下所示：

[!code-csharp[](index/sample//Controllers/TodoController.cs?name=snippet_Scopes&highlight=4-5,13)]

下列代码为控制台提供程序启用作用域：

::: moniker range="> aspnetcore-2.0"

Program.cs:

[!code-csharp[](index/sample2/Program.cs?name=snippet_Scopes&highlight=4)]

> [!NOTE]
> 要启用基于作用域的日志记录，必须先配置 `IncludeScopes` 控制台记录器选项。
>
> 若要了解关配置，请参阅[配置](#configuration)部分。

::: moniker-end

::: moniker range="= aspnetcore-2.0"

Program.cs:

[!code-csharp[](index/sample2/Program.cs?name=snippet_Scopes&highlight=4)]

> [!NOTE]
> 要启用基于作用域的日志记录，必须先配置 `IncludeScopes` 控制台记录器选项。

::: moniker-end

::: moniker range="< aspnetcore-2.0"

*Startup.cs*：

[!code-csharp[](index/sample/Startup.cs?name=snippet_Scopes&highlight=6)]

::: moniker-end

每条日志消息都包含作用域内的信息：

```
info: TodoApi.Controllers.TodoController[1002]
      => RequestId:0HKV9C49II9CK RequestPath:/api/todo/0 => TodoApi.Controllers.TodoController.GetById (TodoApi) => Message attached to logs created in the using block
      Getting item 0
warn: TodoApi.Controllers.TodoController[4000]
      => RequestId:0HKV9C49II9CK RequestPath:/api/todo/0 => TodoApi.Controllers.TodoController.GetById (TodoApi) => Message attached to logs created in the using block
      GetById(0) NOT FOUND
```

## <a name="built-in-logging-providers"></a>内置日志记录提供程序

ASP.NET Core 提供以下提供程序：

* [控制台](#console-provider)
* [调试](#debug-provider)
* [EventSource](#eventsource-provider)
* [EventLog](#windows-eventlog-provider)
* [TraceSource](#tracesource-provider)
* [Azure 应用服务](#azure-app-service-provider)

### <a name="console-provider"></a>控制台提供程序

[Microsoft.Extensions.Logging.Console](https://www.nuget.org/packages/Microsoft.Extensions.Logging.Console) 提供程序包向控制台发送日志输出。

::: moniker range=">= aspnetcore-2.0"

```csharp
logging.AddConsole();
```

::: moniker-end

::: moniker range="< aspnetcore-2.0"

```csharp
loggerFactory.AddConsole();
```

[AddConsole 重载](/dotnet/api/microsoft.extensions.logging.consoleloggerextensions)允许传入一个最低日志级别、一个筛选器函数，以及一个用于指示作用域是否受支持的布尔值。 另一个选项是传递 `IConfiguration` 对象，可通过它来指定支持的作用域及日志记录级别。

在为生产环境选择控制台提供程序时，请考虑到它将对性能产生重大影响。

在 Visual Studio 中创建新项目时，`AddConsole` 方法如下所示：

```csharp
loggerFactory.AddConsole(Configuration.GetSection("Logging"));
```

此代码引用 appSettings.json 文件的 `Logging` 部分：

[!code-json[](index/sample//appsettings.json)]

所显示的设置将框架日志限制为警告，并允许在调试级别记录应用日志，如[日志筛选](#log-filtering)部分所述。 有关详细信息，请参阅[配置](xref:fundamentals/configuration/index)。

::: moniker-end

### <a name="debug-provider"></a>调试提供程序

[Microsoft.Extensions.Logging.Debug](https://www.nuget.org/packages/Microsoft.Extensions.Logging.Debug) 提供程序包使用 [System.Diagnostics.Debug](/dotnet/api/system.diagnostics.debug) 类（`Debug.WriteLine` 方法调用）来写入日志输出。

在 Linux 中，此提供程序将日志写入 /var/log/message。

::: moniker range=">= aspnetcore-2.0"

```csharp
logging.AddDebug();
```

::: moniker-end

::: moniker range="< aspnetcore-2.0"

```csharp
loggerFactory.AddDebug();
```

[AddDebug 重载](/dotnet/api/microsoft.extensions.logging.debugloggerfactoryextensions)允许传入最低日志级别或筛选器函数。

::: moniker-end

### <a name="eventsource-provider"></a>EventSource 提供程序

对于面向 ASP.NET Core 1.1.0 或更高版本的应用，[Microsoft.Extensions.Logging.EventSource](https://www.nuget.org/packages/Microsoft.Extensions.Logging.EventSource) 提供程序包可实现事件跟踪。 在 Windows 中，它使用 [ETW](https://msdn.microsoft.com/library/windows/desktop/bb968803)。 提供程序可跨平台使用，但尚无支持 Linux 或 macOS 的事件集合和显示工具。

::: moniker range=">= aspnetcore-2.0"

```csharp
logging.AddEventSourceLogger();
```

::: moniker-end

::: moniker range="< aspnetcore-2.0"

```csharp
loggerFactory.AddEventSourceLogger();
```

::: moniker-end

可使用 [PerfView 实用工具](https://github.com/Microsoft/perfview)收集和查看日志。 虽然其他工具也可以查看 ETW 日志，但在处理由 ASP.NET 发出的 ETW 事件时，使用 PerfView 能获得最佳体验。

要将 PerfView 配置为收集此提供程序记录的事件，请向 Additional Providers 列表添加字符串 `*Microsoft-Extensions-Logging`。 （请勿遗漏字符串起始处的星号。）

![其他 Perfview 提供程序](index/_static/perfview-additional-providers.png)

### <a name="windows-eventlog-provider"></a>Windows EventLog 提供程序

[Microsoft.Extensions.Logging.EventLog](https://www.nuget.org/packages/Microsoft.Extensions.Logging.EventLog) 提供程序包向 Windows 事件日志发送日志输出。

::: moniker range=">= aspnetcore-2.0"

```csharp
logging.AddEventLog();
```

::: moniker-end

::: moniker range="< aspnetcore-2.0"

```csharp
loggerFactory.AddEventLog();
```

[AddEventLog 重载](/dotnet/api/microsoft.extensions.logging.eventloggerfactoryextensions)允许传入 `EventLogSettings` 或最低日志级别。

::: moniker-end

### <a name="tracesource-provider"></a>TraceSource 提供程序

[Microsoft.Extensions.Logging.TraceSource](https://www.nuget.org/packages/Microsoft.Extensions.Logging.TraceSource) 提供程序包使用 [System.Diagnostics.TraceSource](/dotnet/api/system.diagnostics.tracesource) 库和提供程序。

::: moniker range=">= aspnetcore-2.0"

```csharp
logging.AddTraceSource(sourceSwitchName);
```

::: moniker-end

::: moniker range="< aspnetcore-2.0"

```csharp
loggerFactory.AddTraceSource(sourceSwitchName);
```

::: moniker-end

[AddTraceSource 重载](/dotnet/api/microsoft.extensions.logging.tracesourcefactoryextensions) 允许传入资源开关和跟踪侦听器。

要使用此提供程序，应用程序必须在 .NET Framework（而非 .NET Core）上运行。 使用提供程序，可以将消息路由到多个[侦听器](/dotnet/framework/debug-trace-profile/trace-listeners)，例如示例应用程序中使用的 [TextWriterTraceListener](/dotnet/api/system.diagnostics.textwritertracelistenerr)。

以下示例配置了 `TraceSource` 提供程序，用于向控制台窗口记录 `Warning` 和更高级别的消息。

[!code-csharp[](index/sample/Startup.cs?name=snippet_TraceSource&highlight=9-12)]

### <a name="azure-app-service-provider"></a>Azure 应用服务提供程序

[Microsoft.Extensions.Logging.AzureAppServices](https://www.nuget.org/packages/Microsoft.Extensions.Logging.AzureAppServices) 提供程序包将日志写入 Azure App Service 应用的文件系统，以及 Azure 存储帐户中的 [blob 存储](https://azure.microsoft.com/documentation/articles/storage-dotnet-how-to-use-blobs/#what-is-blob-storage)。 面向 .NET Core 1.1 或更高版本的应用可使用该提供程序包。

::: moniker range=">= aspnetcore-2.0"

如果面向 .NET Core，请注意以下几点：

* 提供程序包包括在 ASP.NET Core [Microsoft.AspNetCore.All 元包](xref:fundamentals/metapackage) 中，但不包括在 [Microsoft.AspNetCore.App 元包](xref:fundamentals/metapackage-app)中。
* 请勿显示调用 [AddAzureWebAppDiagnostics](/dotnet/api/microsoft.extensions.logging.azureappservicesloggerfactoryextensions.addazurewebappdiagnostics) 将应用部署到 Azure 应用服务时，会自动使提供程序对应用可用。

如果面向 .NET Framework 或引用 `Microsoft.AspNetCore.App` 元包，请向项目添加提供程序包。 在 [ILoggerFactory](/dotnet/api/microsoft.extensions.logging.iloggerfactory) 实例上调用 `AddAzureWebAppDiagnostics`：

```csharp
logging.AddAzureWebAppDiagnostics();
```

::: moniker-end

::: moniker range="= aspnetcore-1.1"

```csharp
loggerFactory.AddAzureWebAppDiagnostics();
```

::: moniker-end

[AddAzureWebAppDiagnostics](/dotnet/api/microsoft.extensions.logging.azureappservicesloggerfactoryextensions.addazurewebappdiagnostics) 重载允许传入 [AzureAppServicesDiagnosticsSettings](/dotnet/api/microsoft.extensions.logging.azureappservices.azureappservicesdiagnosticssettings)，可用它替代默认设置，例如日志记录输出模板、blob 名称和文件大小限制等。 （输出模板是应用于所有日志的消息模板，其优先级高于调用 `ILogger` 方法时提供的模板。）

在部署 App Service 应用时，应用将遵循 Azure 门户中 App Service 页下[诊断日志](https://azure.microsoft.com/documentation/articles/web-sites-enable-diagnostic-log/#enablediag)部分的设置。 更新这些设置后，更改会立即生效，无需重新启动或重新部署应用。

![Azure 日志记录设置](index/_static/azure-logging-settings.png)

日志文件的默认位置是 D:\\home\\LogFiles\\Application 文件夹，默认文件名为 diagnostics-yyyymmdd.txt。 默认文件大小上限为 10 MB，默认最大保留文件数为 2。 默认 blob 名为 {app-name}{timestamp}/yyyy/mm/dd/hh/{guid}-applicationLog.txt。 有关默认行为的详细信息，请参阅 [AzureAppServicesDiagnosticsSettings](/dotnet/api/microsoft.extensions.logging.azureappservices.azureappservicesdiagnosticssettings)。

该提供程序仅当项目在 Azure 环境中运行时有效。 项目在本地运行时，该提供程序无效 &mdash; 它不会写入本地文件或 blob 的本地开发存储。

## <a name="third-party-logging-providers"></a>第三方日志记录提供程序

适用于 ASP.NET Core 的第三方日志记录框架：

* [elmah.io](https://elmah.io/)（[GitHub 存储库](https://github.com/elmahio/Elmah.Io.Extensions.Logging)）
* [Gelf](http://docs.graylog.org/en/2.3/pages/gelf.html)（[GitHub 存储库](https://github.com/mattwcole/gelf-extensions-logging)）
* [JSNLog](http://jsnlog.com/)（[GitHub 存储库](https://github.com/mperdeck/jsnlog)）
* [Loggr](http://loggr.net/)（[GitHub 存储库](https://github.com/imobile3/Loggr.Extensions.Logging)）
* [NLog](http://nlog-project.org/)（[GitHub 存储库](https://github.com/NLog/NLog.Extensions.Logging)）
* [Serilog](https://serilog.net/)（[GitHub 存储库](https://github.com/serilog/serilog-extensions-logging)）

某些第三方框架可以执行[语义日志记录（又称结构化日志记录）](https://softwareengineering.stackexchange.com/questions/312197/benefits-of-structured-logging-vs-basic-logging)。

使用第三方框架类似于使用以下内置提供程序之一：

1. 将 NuGet 包添加到你的项目。
1. 在 `ILoggerFactory` 上调用扩展方法。

有关详细信息，请参阅各框架的相关文档。

## <a name="azure-log-streaming"></a>Azure 日志流式处理

通过 Azure 日志流式处理，可从以下位置实时查看日志活动：

* 应用程序服务器
* Web 服务器
* 请求跟踪失败

要配置 Azure 日志流式处理，请执行以下操作：

* 从应用程序的门户页导航到“诊断日志”页
* 将“应用程序日志记录 (Filesystem)”设置为启用。

![Azure 门户诊断日志页](index/_static/azure-diagnostic-logs.png)

导航到“日志流式处理”页，查看应用程序消息。 它们由应用程序通过 `ILogger` 接口记录。

![Azure 门户应用程序日志流式处理](index/_static/azure-log-streaming.png)

## <a name="azure-application-insights-trace-logging"></a>Azure Application Insights 跟踪日志记录

[Application Insights](https://azure.microsoft.com/services/application-insights/) SDK 可从通过 ASP.NET Core日志记录基础结构生成的日志中收集跟踪遥测数据。 有关详细信息，请参阅 [Application Insights for ASP.NET Core](https://docs.microsoft.com/azure/application-insights/app-insights-asp-net-core) 和 [Microsoft/ApplicationInsights-aspnetcore Wiki：日志记录](https://github.com/Microsoft/ApplicationInsights-aspnetcore/wiki/Logging)。

## <a name="additional-resources"></a>其他资源

* <xref:fundamentals/logging/loggermessage>
