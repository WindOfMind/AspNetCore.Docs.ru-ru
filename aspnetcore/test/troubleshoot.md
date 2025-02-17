---
title: Устранение неполадок с проектов ASP.NET Core
author: Rick-Anderson
description: Устранение неполадок при возникновении ошибок и предупреждений в проектах ASP.NET Core.
ms.author: riande
ms.custom: mvc
ms.date: 06/19/2019
uid: test/troubleshoot
ms.openlocfilehash: bcec8a55a5111e1f3acf53ae2f57b45e6e609d25
ms.sourcegitcommit: 9f11685382eb1f4dd0fb694dea797adacedf9e20
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 06/21/2019
ms.locfileid: "67313672"
---
# <a name="troubleshoot-aspnet-core-projects"></a>Устранение неполадок с проектов ASP.NET Core

Автор: [Рик Андерсон](https://twitter.com/RickAndMSFT) (Rick Anderson)

Рекомендации по устранению неполадок по следующим ссылкам:

* <xref:host-and-deploy/azure-apps/troubleshoot>
* <xref:host-and-deploy/iis/troubleshoot>
* <xref:host-and-deploy/azure-iis-errors-reference>
* [Представленная им ndc Прошедшей конференции (Лондон, 2018 г.): Диагностика проблем в приложениях ASP.NET Core](https://www.youtube.com/watch?v=RYI0DHoIVaA)
* [Блог разработчиков ASP.NET. Устранение неполадок производительности ASP.NET Core](https://blogs.msdn.microsoft.com/webdev/2018/05/23/asp-net-core-performance-improvements/)

## <a name="net-core-sdk-warnings"></a>Пакет SDK для .NET core предупреждения

### <a name="both-the-32-bit-and-64-bit-versions-of-the-net-core-sdk-are-installed"></a>При установке 32-разрядных и 64-разрядных версий пакета SDK для .NET Core

В **новый проект** диалоговое окно для ASP.NET Core, может появиться следующее предупреждение:

> При установке 32-разрядных и 64-разрядных версий пакета SDK для .NET Core. Только шаблоны из 64-разрядные версии, установленной на "C:\\Program Files\\dotnet\\sdk\\" отображаются.

Это предупреждение появляется, когда 32-(x86) и 64-разрядные (x 64) версии [пакет SDK для .NET Core](https://www.microsoft.com/net/download/all) установлены. Распространенные причины, которые могут быть установлены обе версии включают:

* Изначально загрузить установщик пакета SDK для .NET Core, используя 32-разрядном компьютере, но затем скопировали его через и его установки на 64-разрядном компьютере.
* Пакет SDK для 32-разрядной .NET Core был установлен другим приложением.
* Неправильная версия была загружаются и устанавливаются.

Удалите пакет SDK 32-разрядной .NET Core для, чтобы устранить это предупреждение. Удалить из **панели управления** > **программы и компоненты** > **удаление или изменение программы**. Если вы понимаете, почему было создано предупреждение и ее особенности, предупреждение можно игнорировать.

### <a name="the-net-core-sdk-is-installed-in-multiple-locations"></a>Пакет SDK для .NET Core устанавливается в нескольких местах

В **новый проект** диалоговое окно для ASP.NET Core, может появиться следующее предупреждение:

> Пакет SDK для .NET Core устанавливается в нескольких местах. Только шаблоны из пакетов SDK, установленные в "C:\\Program Files\\dotnet\\sdk\\" отображаются.

Вы видите это сообщение при наличии по крайней мере по одному разу на пакет SDK для .NET Core в каталоге за пределами *C:\\Program Files\\dotnet\\sdk\\* . Обычно это происходит после развертывания пакета SDK для .NET Core на компьютере, с помощью копирования и вставки вместо установщика MSI.

Удалите все 32-разрядных пакетов SDK для .NET Core и среды выполнения для предотвращения этого предупреждения. Удалить из **панели управления** > **программы и компоненты** > **удаление или изменение программы**. Если вы понимаете, почему было создано предупреждение и ее особенности, предупреждение можно игнорировать.

### <a name="no-net-core-sdks-were-detected"></a>Нет пакетов SDK для .NET Core не обнаружены

* В Visual Studio **новый проект** диалоговое окно для ASP.NET Core, может появиться следующее предупреждение:

  > Нет пакетов SDK для .NET Core были обнаружены, убедитесь, они включаются в переменной среды `PATH`.

* При выполнении `dotnet` команды, предупреждение появляется в том, как:

  > Не удалось найти все установленные dotnet пакеты SDK.

Эти предупреждения отображаются, когда переменная среды `PATH` не указывает на пакеты SDK для Core любой .NET на компьютере. Чтобы устранить эту проблему:

* Установите пакет SDK для .NET Core. Получить последнюю версию установщика из [загрузки .NET](https://dotnet.microsoft.com/download).
* Убедитесь, что `PATH` переменной среды указывает на расположение, где установлен пакет SDK (`C:\Program Files\dotnet\` для 64 — бит/x64 или `C:\Program Files (x86)\dotnet\` для 32 бит x86 /). Установщик пакета SDK для обычно задает `PATH`. Всегда устанавливайте же разрядности пакеты SDK и среды выполнения на одном компьютере.

### <a name="missing-sdk-after-installing-the-net-core-hosting-bundle"></a>Отсутствует пакет SDK после установкой пакета размещения .NET Core

Установка [пакет размещения .NET Core](xref:host-and-deploy/iis/index#install-the-net-core-hosting-bundle) изменяет `PATH` при установке среды выполнения .NET Core, чтобы он указывал на 32-разрядной (x 86) версию .NET Core (`C:\Program Files (x86)\dotnet\`). Это может привести отсутствующие пакеты SDK при 32-разрядной (x 86) .NET Core `dotnet` используется команда ([SDK Core .NET не были обнаружены](#no-net-core-sdks-were-detected)). Чтобы устранить эту проблему, переместите `C:\Program Files\dotnet\` на позицию перед `C:\Program Files (x86)\dotnet\` на `PATH`.

## <a name="obtain-data-from-an-app"></a>Получение данных из приложения

Если приложение может отвечать на запросы, можно получить следующие данные из приложения, с помощью по промежуточного слоя:

* Запросить &ndash; строка заголовков запроса метода, схему, узел, pathbase, путь,
* Подключение &ndash; удаленный IP-адрес, удаленный порт, локальный IP-адрес, локальный порт, сертификат клиента
* Удостоверение &ndash; имя, отображаемое имя
* Параметры конфигурации
* Переменные среды

Поместите следующий [по промежуточного слоя](xref:fundamentals/middleware/index#create-a-middleware-pipeline-with-iapplicationbuilder) кода в начале `Startup.Configure` конвейера обработки запросов метода. Среде проверяется перед запуском по промежуточного слоя, чтобы убедиться, что код выполняется только в среде разработки.

Чтобы получить среду, используйте один из следующих подходов:

* Внедрить `IHostingEnvironment` в `Startup.Configure` метод и проверьте среды с локальной переменной. В следующем примере кода демонстрируется этот подход.

* Присвоить свойству в среде `Startup` класса. Проверьте среду, с помощью свойства (например, `if (Environment.IsDevelopment())`).

```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env, 
    IConfiguration config)
{
    if (env.IsDevelopment())
    {
        app.Run(async (context) =>
        {
            var sb = new StringBuilder();
            var nl = System.Environment.NewLine;
            var rule = string.Concat(nl, new string('-', 40), nl);
            var authSchemeProvider = app.ApplicationServices
                .GetRequiredService<IAuthenticationSchemeProvider>();

            sb.Append($"Request{rule}");
            sb.Append($"{DateTimeOffset.Now}{nl}");
            sb.Append($"{context.Request.Method} {context.Request.Path}{nl}");
            sb.Append($"Scheme: {context.Request.Scheme}{nl}");
            sb.Append($"Host: {context.Request.Headers["Host"]}{nl}");
            sb.Append($"PathBase: {context.Request.PathBase.Value}{nl}");
            sb.Append($"Path: {context.Request.Path.Value}{nl}");
            sb.Append($"Query: {context.Request.QueryString.Value}{nl}{nl}");

            sb.Append($"Connection{rule}");
            sb.Append($"RemoteIp: {context.Connection.RemoteIpAddress}{nl}");
            sb.Append($"RemotePort: {context.Connection.RemotePort}{nl}");
            sb.Append($"LocalIp: {context.Connection.LocalIpAddress}{nl}");
            sb.Append($"LocalPort: {context.Connection.LocalPort}{nl}");
            sb.Append($"ClientCert: {context.Connection.ClientCertificate}{nl}{nl}");

            sb.Append($"Identity{rule}");
            sb.Append($"User: {context.User.Identity.Name}{nl}");
            var scheme = await authSchemeProvider
                .GetSchemeAsync(IISDefaults.AuthenticationScheme);
            sb.Append($"DisplayName: {scheme?.DisplayName}{nl}{nl}");

            sb.Append($"Headers{rule}");
            foreach (var header in context.Request.Headers)
            {
                sb.Append($"{header.Key}: {header.Value}{nl}");
            }
            sb.Append(nl);

            sb.Append($"Websockets{rule}");
            if (context.Features.Get<IHttpUpgradeFeature>() != null)
            {
                sb.Append($"Status: Enabled{nl}{nl}");
            }
            else
            {
                sb.Append($"Status: Disabled{nl}{nl}");
            }

            sb.Append($"Configuration{rule}");
            foreach (var pair in config.AsEnumerable())
            {
                sb.Append($"{pair.Path}: {pair.Value}{nl}");
            }
            sb.Append(nl);

            sb.Append($"Environment Variables{rule}");
            var vars = System.Environment.GetEnvironmentVariables();
            foreach (var key in vars.Keys.Cast<string>().OrderBy(key => key, 
                StringComparer.OrdinalIgnoreCase))
            {
                var value = vars[key];
                sb.Append($"{key}: {value}{nl}");
            }

            context.Response.ContentType = "text/plain";
            await context.Response.WriteAsync(sb.ToString());
        });
    }
}
```
