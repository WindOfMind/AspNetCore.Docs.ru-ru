---
title: Выбор между ASP.NET 4.x и ASP.NET Core
author: rick-anderson
description: Что такое ASP.NET Core и ASP.NET 4.x и как выбрать между ними.
ms.author: riande
ms.custom: mvc, seodec18
ms.date: 05/02/2019
uid: fundamentals/choose-between-aspnet-and-aspnetcore
ms.openlocfilehash: 97d65df8df1e27694d818098958288874e8ab92e
ms.sourcegitcommit: 47cc13ab90913af9a2887cef0896bb4e9aba4dd5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 06/26/2019
ms.locfileid: "67399166"
---
# <a name="choose-between-aspnet-4x-and-aspnet-core"></a>Выбор между ASP.NET 4.x и ASP.NET Core

ASP.NET Core является переработанной версией ASP.NET 4.x. В этой статье перечислены различия между ними.

## <a name="aspnet-core"></a>ASP.NET Core

ASP.NET Core — это кроссплатформенная среда с открытым кодом для создания современных облачных веб-приложений в Windows, macOS или Linux.

[!INCLUDE[](~/includes/benefits.md)]

## <a name="aspnet-4x"></a>ASP.NET 4.x

ASP.NET 4.x — это развитая платформа, предоставляющая необходимые службы для создания серверных веб-приложений корпоративного класса в Windows.

## <a name="framework-selection"></a>Выбор платформы

В следующей таблице сравниваются ASP.NET Core и ASP.NET 4.x.

| ASP.NET Core | ASP.NET 4.x |
|---|---|
|Предназначена для Windows, macOS или Linux|Предназначена для Windows|
|[Razor Pages](xref:razor-pages/index) — рекомендуемый метод создания пользовательского веб-интерфейса в ASP.NET Core 2.x. См. также [MVC](xref:mvc/overview), [веб-API](xref:tutorials/first-web-api), и [SignalR](xref:signalr/introduction).|Использование [веб-форм](/aspnet/web-forms), [SignalR](/aspnet/signalr), [MVC](/aspnet/mvc), [веб-API](/aspnet/web-api/), [веб-перехватчиков](/aspnet/webhooks/) или [веб-страниц](/aspnet/web-pages)|
|Несколько версий для одного компьютера|Одна версия для одного компьютера|
|Разработка в [Visual Studio](https://visualstudio.microsoft.com/vs/), [Visual Studio для Mac](https://visualstudio.microsoft.com/vs/mac/) или [Visual Studio Code](https://code.visualstudio.com/) с использованием C# или F#|Разработка с [Visual Studio](https://visualstudio.microsoft.com/vs/) с использованием C#, VB или F#|
|Более высокая производительность, чем в ASP.NET 4.x|Хорошая производительность|
|[Выбор среды выполнения .NET Framework или .NET Core](/dotnet/standard/choosing-core-framework-server)|Использование среды выполнения .NET Framework|

Дополнительные сведения о поддержке ASP.NET Core 2.x на платформе .NET Framework см. в разделе [ASP.NET Core для платформы .NET Framework](xref:index#target-framework).

## <a name="aspnet-core-scenarios"></a>Сценарии ASP.NET Core

* [Веб-сайты](xref:tutorials/first-mvc-app/index)
* [API-интерфейсы](xref:tutorials/first-web-api)
* [Режим реального времени](xref:signalr/index)
* [Развертывание приложения ASP.NET Core в Azure](/azure/app-service/app-service-web-get-started-dotnet)

## <a name="aspnet-4x-scenarios"></a>Сценарии ASP.NET 4.x

* [Веб-сайты](/aspnet/mvc)
* [API-интерфейсы](/aspnet/web-api)
* [Режим реального времени](/aspnet/signalr)
* [Создание веб-приложение ASP.NET 4.x в Azure](/azure/app-service/app-service-web-get-started-dotnet-framework)

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Введение в ASP.NET](/aspnet/overview)
* [Введение в ASP.NET Core](xref:index)
* <xref:host-and-deploy/azure-apps/index>
