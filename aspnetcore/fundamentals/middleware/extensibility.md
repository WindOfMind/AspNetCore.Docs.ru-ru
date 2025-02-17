---
title: Активация ПО промежуточного слоя на основе фабрики в ASP.NET Core
author: guardrex
description: В этой статье приводятся сведения о том, как использовать строго типизированное ПО промежуточного слоя с реализацией активации на основе фабрики в ASP.NET Core.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 03/31/2019
uid: fundamentals/middleware/extensibility
ms.openlocfilehash: b4d71c2c7f09acb58b73e84080e8574d77f8b326
ms.sourcegitcommit: dd9c73db7853d87b566eef136d2162f648a43b85
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 05/06/2019
ms.locfileid: "65087003"
---
# <a name="factory-based-middleware-activation-in-aspnet-core"></a>Активация ПО промежуточного слоя на основе фабрики в ASP.NET Core

Автор [Люк Латэм](https://github.com/guardrex) (Luke Latham)

<xref:Microsoft.AspNetCore.Http.IMiddlewareFactory>/<xref:Microsoft.AspNetCore.Http.IMiddleware> — это точка расширяемости для активации [ПО промежуточного слоя](xref:fundamentals/middleware/index).

Методы расширения <xref:Microsoft.AspNetCore.Builder.UseMiddlewareExtensions.UseMiddleware*> проверяют, реализует ли зарегистрированный тип ПО промежуточного слоя <xref:Microsoft.AspNetCore.Http.IMiddleware>. Если да, то экземпляр <xref:Microsoft.AspNetCore.Http.IMiddlewareFactory>, зарегистрированный в контейнере, используется для разрешения реализации <xref:Microsoft.AspNetCore.Http.IMiddleware> вместо стандартной логики активации ПО промежуточного слоя. ПО промежуточного слоя регистрируется как [ограниченная (scoped) или временная (temporary) служба](xref:fundamentals/dependency-injection#service-lifetimes) в контейнере служб приложения.

Преимущества:

* Активация по клиентскому запросу (внедрение ограниченных служб)
* Строгая типизация ПО промежуточного слоя

<xref:Microsoft.AspNetCore.Http.IMiddleware> активируется по клиентскому запросу (подключению), благодаря чему ограниченные (scoped) службы могут внедряться в конструктор ПО промежуточного слоя.

[Просмотреть или скачать образец кода](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/middleware/extensibility/samples) ([как скачивать](xref:index#how-to-download-a-sample))

## <a name="imiddleware"></a>IMiddleware

<xref:Microsoft.AspNetCore.Http.IMiddleware> определяет ПО промежуточного слоя для конвейера запросов приложения. Метод [InvokeAsync(HttpContext, RequestDelegate)](xref:Microsoft.AspNetCore.Http.IMiddleware.InvokeAsync*) обрабатывает запрос и возвращает объект <xref:System.Threading.Tasks.Task>, который представляет выполнение ПО промежуточного слоя.

Стандартная активация ПО промежуточного слоя:

[!code-csharp[](extensibility/samples/2.x/MiddlewareExtensibilitySample/Middleware/ConventionalMiddleware.cs?name=snippet1)]

Активация ПО промежуточного слоя с помощью <xref:Microsoft.AspNetCore.Http.MiddlewareFactory>:

[!code-csharp[](extensibility/samples/2.x/MiddlewareExtensibilitySample/Middleware/FactoryActivatedMiddleware.cs?name=snippet1)]

Расширения, создаваемые для ПО промежуточного слоя:

[!code-csharp[](extensibility/samples/2.x/MiddlewareExtensibilitySample/Middleware/MiddlewareExtensions.cs?name=snippet1)]

Невозможна передача объектов в ПО промежуточного слоя, активируемое на основе фабрики с помощью <xref:Microsoft.AspNetCore.Builder.UseMiddlewareExtensions.UseMiddleware*>:

```csharp
public static IApplicationBuilder UseFactoryActivatedMiddleware(
    this IApplicationBuilder builder, bool option)
{
    // Passing 'option' as an argument throws a NotSupportedException at runtime.
    return builder.UseMiddleware<FactoryActivatedMiddleware>(option);
}
```

Активируемое на основе фабрики ПО промежуточного слоя добавляется во встроенный контейнер в файле `Startup.ConfigureServices`.

[!code-csharp[](extensibility/samples/2.x/MiddlewareExtensibilitySample/Startup.cs?name=snippet1&highlight=6)]

Оба ПО промежуточного слоя регистрируются в конвейере обработки запросов в `Startup.Configure`:

[!code-csharp[](extensibility/samples/2.x/MiddlewareExtensibilitySample/Startup.cs?name=snippet2&highlight=13-14)]

## <a name="imiddlewarefactory"></a>IMiddlewareFactory

<xref:Microsoft.AspNetCore.Http.IMiddlewareFactory> предоставляет методы для создания ПО промежуточного слоя. Реализация ПО промежуточного слоя на основе фабрики регистрируется в контейнере в виде ограниченной (scoped) службы.

Реализация <xref:Microsoft.AspNetCore.Http.IMiddlewareFactory> по умолчанию, <xref:Microsoft.AspNetCore.Http.MiddlewareFactory>, находится в пакете [Microsoft.AspNetCore.Http](https://www.nuget.org/packages/Microsoft.AspNetCore.Http/).

## <a name="additional-resources"></a>Дополнительные ресурсы

* <xref:fundamentals/middleware/index>
* <xref:fundamentals/middleware/extensibility-third-party-container>
