---
title: Активация ПО промежуточного слоя с помощью контейнера сторонних разработчиков в ASP.NET Core
author: guardrex
description: В этой статье приводятся сведения о том, как использовать строго типизированное ПО промежуточного слоя с активацией на основе фабрики и контейнером сторонних разработчиков в ASP.NET Core.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 07/03/2019
uid: fundamentals/middleware/extensibility-third-party-container
ms.openlocfilehash: 4bc99b4c336aba611287c9fbe03d4252f8abee5b
ms.sourcegitcommit: f6e6730872a7d6f039f97d1df762f0d0bd5e34cf
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/04/2019
ms.locfileid: "67561650"
---
# <a name="middleware-activation-with-a-third-party-container-in-aspnet-core"></a>Активация ПО промежуточного слоя с помощью контейнера сторонних разработчиков в ASP.NET Core

Автор [Люк Латэм](https://github.com/guardrex) (Luke Latham)

В этой статье демонстрируется использование <xref:Microsoft.AspNetCore.Http.IMiddlewareFactory> и <xref:Microsoft.AspNetCore.Http.IMiddleware> в качестве точки расширяемости для активации [ПО промежуточного слоя](xref:fundamentals/middleware/index) с помощью контейнера сторонних разработчиков. Вводные сведения о `IMiddlewareFactory` и `IMiddleware` см. в статье <xref:fundamentals/middleware/extensibility>.

[Просмотреть или скачать образец кода](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/middleware/extensibility-third-party-container/samples/) ([как скачивать](xref:index#how-to-download-a-sample))

В примере приложения показана активация ПО промежуточного слоя путем реализации `IMiddlewareFactory`, `SimpleInjectorMiddlewareFactory`. В этом образце используется контейнер внедрения зависимостей [Simple Injector](https://simpleinjector.org).

Реализация ПО промежуточного слоя в примере записывает значение, предоставленное в параметре строки запроса (`key`). ПО промежуточного слоя использует внедренный контекст базы данных (служба с заданной областью) для записи значения строки запроса в базу данных, выполняющуюся в памяти.

> [!NOTE]
> В примере приложения [Simple Injector](https://github.com/simpleinjector/SimpleInjector) используется исключительно для демонстрации. Использование Simple Injector не означает поддержку данного инструмента. Подходы к активации ПО промежуточного слоя, описанные в документации по Simple Injector и статьях на GitHub, рекомендованы командой Simple Injector. Дополнительные сведения см. в [документации по Simple Injector](https://simpleinjector.readthedocs.io/en/latest/index.html) и [в репозитории Simple Injector в GitHub](https://github.com/simpleinjector/SimpleInjector).

## <a name="imiddlewarefactory"></a>IMiddlewareFactory

<xref:Microsoft.AspNetCore.Http.IMiddlewareFactory> предоставляет методы для создания ПО промежуточного слоя.

В примере приложения реализуется фабрика ПО промежуточного слоя для создания экземпляра `SimpleInjectorActivatedMiddleware`. Фабрика ПО промежуточного слоя использует контейнер Simple Injector для разрешения по промежуточного слоя:

[!code-csharp[](extensibility-third-party-container/samples/2.x/SampleApp/Middleware/SimpleInjectorMiddlewareFactory.cs?name=snippet1&highlight=5-8,12)]

## <a name="imiddleware"></a>IMiddleware

<xref:Microsoft.AspNetCore.Http.IMiddleware> определяет ПО промежуточного слоя для конвейера запросов приложения.

ПО промежуточного слоя, активированное реализацией `IMiddlewareFactory` (*Middleware/SimpleInjectorActivatedMiddleware.cs*):

[!code-csharp[](extensibility-third-party-container/samples/2.x/SampleApp/Middleware/SimpleInjectorActivatedMiddleware.cs?name=snippet1)]

Для ПО промежуточного слоя создается расширение (*Middleware/MiddlewareExtensions.cs*):

[!code-csharp[](extensibility-third-party-container/samples/2.x/SampleApp/Middleware/MiddlewareExtensions.cs?name=snippet1)]

`Startup.ConfigureServices` должен выполнять несколько задач:

* Настройка контейнера Simple Injector.
* Регистрация фабрики и ПО промежуточного слоя.
* Предоставление контекста базы данных приложения из контейнера Simple Injector.

[!code-csharp[](extensibility-third-party-container/samples/2.x/SampleApp/Startup.cs?name=snippet1)]

ПО промежуточного слоя регистрируется в конвейере обработки запросов в `Startup.Configure`:

[!code-csharp[](extensibility-third-party-container/samples/2.x/SampleApp/Startup.cs?name=snippet2&highlight=13)]

## <a name="additional-resources"></a>Дополнительные ресурсы

* [ПО промежуточного слоя](xref:fundamentals/middleware/index)
* [Активация фабричного ПО промежуточного слоя](xref:fundamentals/middleware/extensibility)
* [Репозиторий Simple Injector в GitHub](https://github.com/simpleinjector/SimpleInjector)
* [Документация по Simple Injector](https://simpleinjector.readthedocs.io/en/latest/index.html)
