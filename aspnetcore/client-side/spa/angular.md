---
title: Использование шаблона проекта Angular с ASP.NET Core
author: SteveSandersonMS
description: Дополнительные сведения о начале работы с шаблоном проекта одностраничного приложения (SPA) ASP.NET Core для Angular и Angular CLI.
monikerRange: '>= aspnetcore-2.1'
ms.author: stevesa
ms.custom: mvc
ms.date: 03/07/2019
uid: spa/angular
ms.openlocfilehash: 6d0107ef52d63a0f6f5713c518ddc54ac4230d53
ms.sourcegitcommit: 5b0eca8c21550f95de3bb21096bd4fd4d9098026
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/27/2019
ms.locfileid: "64893671"
---
# <a name="use-the-angular-project-template-with-aspnet-core"></a>Использование шаблона проекта Angular с ASP.NET Core

Обновленный шаблон проекта Angular обеспечивает удобную отправную точку для приложений ASP.NET Core с использованием Angular и Angular CLI для реализации богатого пользовательского интерфейса (UI) на стороне клиента.

Шаблон является эквивалентом созданию проекта ASP.NET Core, выступающего в качестве серверного API, и проекта Angular CLI в качестве пользовательского интерфейса. В шаблоне предлагается удобное размещения обоих типов проектов в проекте отдельного приложения. Что в свою очередь позволит создать и опубликовать проект приложения как единое целое.

## <a name="create-a-new-app"></a>Создание нового приложения

Если установлена ASP.NET Core 2.1, не нужно устанавливать проект шаблона Angular.

Создайте из командной строки новый проект в пустом каталоге с помощью команды `dotnet new angular`. Например, следующие команды позволяют создать приложение в каталоге *my-new-app* и перейти к этому каталогу:

```console
dotnet new angular -o my-new-app
cd my-new-app
```

Запустите приложение в Visual Studio или в .NET Core CLI.

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio/)

Откройте созданный файл проекта *.csproj* и запустите в нем приложение в обычном режиме.

В процессе сборки при первом запуске восстанавливаются зависимости npm. Это может занять несколько минут. Последующие сборки выполняются гораздо быстрее.

# <a name="net-core-clitabnetcore-cli"></a>[Интерфейс командной строки .NET Core](#tab/netcore-cli/)

Убедитесь, что у вас существует переменная среда `ASPNETCORE_Environment`, которой соответствует значение `Development`. Запустите команду `SET ASPNETCORE_Environment=Development` в ОС Windows (в приглашениях, отличных от PowerShell). А в ОС Linux или macOS запустите команду `export ASPNETCORE_Environment=Development`.

Чтобы проверить наличие ошибок при создании приложения, запустите команду [dotnet-build](/dotnet/core/tools/dotnet-build). При первом запуске процесса создания будут восстановлены зависимости npm, на что может потребоваться несколько минут. Последующие сборки выполняются гораздо быстрее.

Чтобы запустить приложение, выполните команду [dotnet run](/dotnet/core/tools/dotnet-run). Следующее сообщение будет записано в журнал.

```console
Now listening on: http://localhost:<port>
```

Перейдите по этому URL-адресу в браузере.

Приложение запускает экземпляр сервера Angular CLI в фоновом режиме. Следующее сообщение будет записано в журнал. *NG Live Development Server прослушивает localhost:&lt;otherport&gt;, откройте браузер на http://localhost:&lt; otherport&gt;/*. Игнорируйте это сообщение — этот URL-адрес **не соответствует** объединенному приложению ASP.NET Core и Angular CLI.

---

Шаблон проекта создает приложение ASP.NET Core и приложение Angular. Приложение ASP.NET Core предназначено для использования в таких сферах как: получение доступа к данным, авторизация и других проблемных вопросах на стороне сервера.Приложение ASP.NET Core предназначено для доступа к данным, авторизации и других задач на стороне сервера. Приложение Angular, размещенное в подкаталоге *ClientApp*, предназначено для всех задач, связанных с пользовательским интерфейсом.

## <a name="add-pages-images-styles-modules-etc"></a>Добавление страниц, изображений, стилей, модулей и т. д.

Каталог *ClientApp* содержит стандартное приложение Angular CLI. дополнительные сведения в [официальной документации Angular](https://github.com/angular/angular-cli/wiki).

Существуют небольшие различия между приложениями Angular, создаваемыми этим шаблоном и создаваемыми непосредственно Angular CLI (командой `ng new`), однако возможности приложений одинаковые. Приложение, созданное с помощью шаблона, содержит основанный на [Bootstrap](https://getbootstrap.com/) макет и простой пример маршрутизации.

## <a name="run-ng-commands"></a>Выполнение команд ng

В командной строке перейдите в подкаталог *ClientApp*:

```console
cd ClientApp
```

Если средство `ng` установлено глобально, вы можете запустить любую из его команд. Например, можно запустить `ng lint`, `ng test` или любую другую [команду Angular CLI](https://github.com/angular/angular-cli/wiki#additional-commands). Выполнять `ng serve` не требуется, так как приложения ASP.NET Core обслуживают как серверную, так и клиентскую части приложения. На внутреннем уровне они используют `ng serve` в процессе разработки.

Если у вас не установлено средство `ng`, запустите вместо него `npm run ng`. Например, можно запустить `npm run ng lint` или `npm run ng test`.

## <a name="install-npm-packages"></a>Установка пакетов npm

Для установки npm-пакетов сторонних разработчиков используйте в командной строке подкаталог *ClientApp*. Пример:

```console
cd ClientApp
npm install --save <package_name>
```

## <a name="publish-and-deploy"></a>Публикация и развертывание

В процессе разработки приложение запускается в режиме, оптимизированном для удобства разработчиков. Например, пакеты JavaScript включают сопоставления с исходным кодом (благодаря чему при отладке вы сможете видеть исходный код TypeScript). Приложение следит за изменениями файлов TypeScript, HTML и CSS на диске и при обнаружении изменений автоматически производит перекомпиляцию и перезагрузку страницы.

На рабочем этапе используйте версию приложения, оптимизированную для производительности. На нее уже настроено автоматическое переключение. При публикации конфигурация сборки выдает уменьшенные, предварительно скомпилированные (AoT) сборки клиентского кода. В отличие от построения разработки в рабочей среде сборка не требует Node.js для установки на сервере (Если вы включили отрисовки на стороне сервера (SSR)).

Можно использовать стандартные [методы размещения и развертывания ASP.NET Core](xref:host-and-deploy/index).

## <a name="run-ng-serve-independently"></a>Запуск "ng serve" отдельно

Проект настроен запускать свой собственный экземпляр сервера Angular CLI в фоновом режиме, когда приложение ASP.NET Core запускается в режиме разработки. Это удобно, так как нет необходимости вручную запускать отдельный сервер.

У этой настройки по умолчанию есть недостатки. Каждый раз при изменении кода на C# и необходимости перезапустить приложение ASP.NET Core также перезагружается и сервер Angular CLI. Перезапуск может занять около 10 секунд. Если вы часто вносите изменения в код C# и не хотите ждать перезапуска сервера Angular CLI, запустите сервер Angular CLI отдельно, независимо от процесса ASP.NET Core. Для этого сделайте следующее:

1. В командной строке перейдите в подкаталог *ClientApp* и запустите сервер разработки Angular CLI следующей командой:

    ```console
    cd ClientApp
    npm start
    ```

    > [!IMPORTANT]
    > Используйте `npm start`, а не `ng serve` для запуска сервера разработки Angular CLI, в результате чего будет учитываться конфигурация в файле *package.json*. Чтобы передать серверу Angular CLI дополнительные параметры, добавьте их в *package.json* в нужную строку `scripts`.

2. Измените свои приложения ASP.NET Core так, чтобы они использовали внешний экземпляр сервера Angular CLI, а не запускали собственный. В вашем классе *Startup* замените вызов `spa.UseAngularCliServer` на следующий:

    ```csharp
    spa.UseProxyToSpaDevelopmentServer("http://localhost:4200");
    ```

При запуске приложения ASP.NET Core экземпляр сервера Angular CLI запускаться не будет. Вместо него будет использоваться экземпляр, запускаемый вручную. Это ускорит запуск и перезагрузку. Теперь не придется каждый раз дожидаться повторной сборки клиентского приложения в Angular CLI.

### <a name="pass-data-from-net-code-into-typescript-code"></a>Передача данных от .NET к TypeScript

Во время работы SSR может возникнуть необходимость передать данные отдельных запросов из ASP.NET Core в Angular. Например, можно передать сведения из файла cookie или данные, полученные из базы данных. Чтобы это сделать, необходимо внести изменения в класс *Startup*. В обратном вызове `UseSpaPrerendering` для `options.SupplyData` установите указанное ниже значение.

```csharp
options.SupplyData = (context, data) =>
{
    // Creates a new value called isHttpsRequest that's passed to TypeScript code
    data["isHttpsRequest"] = context.Request.IsHttps;
};
```

С помощью обратного вызова `SupplyData` можно передавать произвольные данные, отдельные запросы, а также JSON-сериализируемые данные (например, строки, логические значения или числа). Для кода в файле *main.server.ts* эти данные будут получены в качестве `params.data`. Например, в предыдущем примере кода логическое значение передается как `params.data.isHttpsRequest` в обратный вызов `createServerRenderer`. Также эти данные можно передавать другим частям приложения любым способом, поддерживаемым Angular. Например, ознакомьтесь с тем, как *main.server.ts* передает значения `BASE_URL` любому компоненту, конструктор которого объявлен получателем.

### <a name="drawbacks-of-ssr"></a>Недостатки SSR

Не все приложения получают преимущество от использования SSR. Основное преимущество — воспринимаемая производительность. При использовании вашего приложения посетители, у которых низкая скорость сетевого подключения или медленные мобильные устройства, быстро обнаруживают первоначальный пользовательский интерфейс, даже если для получения или анализа пакетов JavaScript требуется некоторое время. Тем не менее в основном многие одностраничные приложения (SPA) используются в быстрых внутренних сетях компании на быстрых компьютерах, где приложение появляется почти мгновенно.

В тоже время при использовании SSR существует ряд недостатков. В процесс разработки вносится дополнительная сложность. Код должен быть запущен в двух разных средах как на стороне клиента, так и на стороне сервера (в среде Node.js, вызываемой из ASP.NET Core). Ниже приведены некоторые аспекты, которые необходимо учитывать.

* Для SSR необходимо, чтобы на рабочих серверах был установлен Node.js. В некоторых сценариях это происходит автоматически (например, службы приложений Azure), а в других — нет (например, Azure Service Fabric).
* Установка флага сборки `BuildServerSideRenderer` приведет к публикации каталога *node_modules*. В этой папке находятся более 20 000 файлов, что повлечет за собой увеличение времени развертывания.
* Для запуска кода в среде Node.js нельзя полагаться на такие существующие API JavaScript как `window` или `localStorage`. Если во время выполнения SSR для кода (или некоторых используемых библиотек сторонних разработчиков) будут использованы эти API, выполнение SSR завершится сбоем. Например, не используйте jQuery, так как часто он использует на браузерные API. Чтобы предотвратить возникновение ошибки, необходимо избегать использовать SSR или браузерных API-интерфейсов и библиотек. Любые вызовы могут быть перенесены в проверки API, чтобы гарантировать, что они не будут вызваны во время SSR. Например, в коде JavaScript или TypeScript можно использовать следующую проверку:

    ```javascript
    if (typeof window !== 'undefined') {
        // Call browser-specific APIs here
    }
    ```

## <a name="additional-resources"></a>Дополнительные ресурсы

* <xref:security/authentication/identity/spa>
