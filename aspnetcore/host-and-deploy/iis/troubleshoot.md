---
title: Устранение неполадок ASP.NET Core в службах IIS
author: guardrex
description: Сведения о диагностике проблем с развертываниями приложений ASP.NET Core на платформе IIS.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 06/19/2019
uid: host-and-deploy/iis/troubleshoot
ms.openlocfilehash: 4df370dd9b1a5a651bcf767b8b9ace4220bdc345
ms.sourcegitcommit: 9f11685382eb1f4dd0fb694dea797adacedf9e20
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 06/21/2019
ms.locfileid: "67313658"
---
# <a name="troubleshoot-aspnet-core-on-iis"></a>Устранение неполадок ASP.NET Core в службах IIS

Автор [Люк Латэм](https://github.com/guardrex) (Luke Latham)

Эта статья содержит инструкции по диагностике проблемы запуска приложения ASP.NET Core, размещенного в [службах IIS](/iis). Информация в этой статье относится к размещению в службах IIS на Windows Server и Windows Desktop.

Дополнительные статьи по устранению неполадок:

* Служба приложений Azure также использует [модуль ASP.NET Core](xref:host-and-deploy/aspnet-core-module) и службы IIS для размещения приложений. Советы по устранению неполадок, касающиеся Службы приложений Azure, см. в статье <xref:host-and-deploy/azure-apps/troubleshoot>.
* В статье <xref:fundamentals/error-handling> описывается, как обрабатывать ошибки в приложениях ASP.NET Core при разработке в локальной системе.
* Статья [Знакомство с отладчиком Visual Studio](/visualstudio/debugger/getting-started-with-the-debugger) описывает возможности отладчика Visual Studio.
* [Статья об отладке с помощью Visual Studio Code](https://code.visualstudio.com/docs/editor/debugging) описывает поддержку отладки, встроенную в Visual Studio Code.

[!INCLUDE[](~/includes/azure-iis-startup-errors.md)]

## <a name="troubleshoot-app-startup-errors"></a>Устранение неполадок при запуске приложения

### <a name="enable-the-aspnet-core-module-debug-log"></a>Включение журнала отладки модулей ASP.NET Core

Добавьте следующие параметры обработчика в файл *web.config* приложения, чтобы включить журналы отладки модулей ASP.NET Core:

```xml
<aspNetCore ...>
  <handlerSettings>
    <handlerSetting name="debugLevel" value="file" />
    <handlerSetting name="debugFile" value="c:\temp\ancm.log" />
  </handlerSettings>
</aspNetCore>
```

Убедитесь, что указанный для журнала путь существует и удостоверение пула приложений имеет разрешения на запись в это расположение.

### <a name="application-event-log"></a>Журнал событий приложений

Доступ к журналу событий приложений

1. Откройте меню "Пуск", выполните поиск по фразе **Просмотр событий** и выберите приложение **Просмотр событий**.
1. В **средстве просмотра событий** откройте узел **Журналы Windows**.
1. Выберите **Приложение**, чтобы открыть журнал событий приложения.
1. Проверьте здесь наличие ошибок, связанных с проблемным приложением. Нам нужны ошибки со значениями *Модуль IIS AspNetCore* или *Модуль IIS Express AspNetCore* в столбце *Источник*.

### <a name="run-the-app-at-a-command-prompt"></a>Запуск приложения в командной строке

Многие ошибки запуска не создают полезные сведения в журнале событий приложения. Для некоторых ошибок можно найти причину, запустив приложение в командной строке на компьютере размещения.

#### <a name="framework-dependent-deployment"></a>развертывание, зависящее от платформы;

Если развертываемое приложение [зависит от платформы](/dotnet/core/deploying/#framework-dependent-deployments-fdd), сделайте следующее:

1. В командной строке перейдите в папку развертывания и запустите приложение, выполнив команду *dotnet.exe* для сборки приложения. В следующей команде укажите нужное имя сборки вместо заполнителя \<assembly_name>: `dotnet .\<assembly_name>.dll`.
1. Выходные данные приложения, в том числе любые ошибки, будут выведены в окно консоли.
1. Если ошибки возникают при выполнении запроса к приложению, выполните запрос к узлу и порту, где Kestrel ожидает передачи данных. Создайте запрос к `http://localhost:5000/`, используя узел и порт по умолчанию. Если приложение отвечает на запросы по адресу конечной точки Kestrel как обычно, то проблема, вероятнее, связана с конфигурацией размещения, а не с самим приложением.

#### <a name="self-contained-deployment"></a>автономное развертывание;

Если приложение [развертывается автономно](/dotnet/core/deploying/#self-contained-deployments-scd), сделайте следующее:

1. В командной строке перейдите в папку развертывания и запустите исполняемый файл приложения. В следующей команде укажите нужное имя сборки вместо заполнителя \<assembly_name>: `<assembly_name>.exe`.
1. Выходные данные приложения, в том числе любые ошибки, будут выведены в окно консоли.
1. Если ошибки возникают при выполнении запроса к приложению, выполните запрос к узлу и порту, где Kestrel ожидает передачи данных. Создайте запрос к `http://localhost:5000/`, используя узел и порт по умолчанию. Если приложение отвечает на запросы по адресу конечной точки Kestrel как обычно, то проблема, вероятнее, связана с конфигурацией размещения, а не с самим приложением.

### <a name="aspnet-core-module-stdout-log"></a>Журнал stdout модуля ASP.NET Core

Чтобы включить и просмотреть журналы stdout, сделайте следующее:

1. Перейдите в папку развертывания сайта на компьютере размещения.
1. Если здесь нет папки *logs*, создайте ее. Сведения о том, как настроить в MSBuild автоматическое создание папки *logs* в развертывании, см. в статье [о структуре каталогов](xref:host-and-deploy/directory-structure).
1. Измените файл *web.config*. Задайте для параметра **stdoutLogEnabled** значение `true` и измените путь **stdoutLogFile** так, чтобы он указывал на папку *logs* (например, `.\logs\stdout`). В этом пути `stdout` обозначает префикс имени для файла журнала. При создании файла журнала автоматически добавляются метка времени, идентификатор процесса и расширение файла. Если указан префикс файла `stdout`, стандартный файл журнала будет иметь примерно такое имя: *stdout_20180205184032_5412.log*.
1. Убедитесь, что удостоверение пула приложений имеет разрешения на запись в папку *logs*.
1. Сохраните обновленный файл *web.config*.
1. Сделайте запрос к приложению.
1. Перейдите в папку *logs*. Найдите и откройте последний журнал вывода stdout.
1. Проверьте, нет ли в нем ошибок.

> [!IMPORTANT]
> Завершив устранение неполадок, отключите ведение журнала stdout.

1. Измените файл *web.config*.
1. Задайте параметру **stdoutLogEnabled** значение `false`.
1. Сохраните файл.

> [!WARNING]
> Ошибка при отключении журнала stdout может привести к сбоям приложения или сервера. Ни размер файла журнала, ни количество создаваемых файлов журналов ничем не ограничены.
>
> Для обычного ведения журналов для приложения ASP.NET Core используйте библиотеку, которая ограничивает размер файла журнала и выполняет циклический сдвиг журналов. Дополнительные сведения см. в разделе [Сторонние поставщики ведения журналов](xref:fundamentals/logging/index#third-party-logging-providers).

## <a name="enable-the-developer-exception-page"></a>Включение страницы исключений для разработчика

Можно добавить переменную среды `ASPNETCORE_ENVIRONMENT` [в файл web.config](xref:host-and-deploy/aspnet-core-module#setting-environment-variables), чтобы запустить приложение в среде разработки. Если среда не переопределяется при запуске приложения использованием `UseEnvironment` в конструкторе узла, эта переменная среды позволяет отображать [страницу исключения для разработчика](xref:fundamentals/error-handling) при запуске приложения.

::: moniker range=">= aspnetcore-2.2"

```xml
<aspNetCore processPath="dotnet"
      arguments=".\MyApp.dll"
      stdoutLogEnabled="false"
      stdoutLogFile=".\logs\stdout"
      hostingModel="InProcess">
  <environmentVariables>
    <environmentVariable name="ASPNETCORE_ENVIRONMENT" value="Development" />
  </environmentVariables>
</aspNetCore>
```

::: moniker-end

::: moniker range="< aspnetcore-2.2"

```xml
<aspNetCore processPath="dotnet"
      arguments=".\MyApp.dll"
      stdoutLogEnabled="false"
      stdoutLogFile=".\logs\stdout">
  <environmentVariables>
    <environmentVariable name="ASPNETCORE_ENVIRONMENT" value="Development" />
  </environmentVariables>
</aspNetCore>
```

::: moniker-end

Настройка переменной среды `ASPNETCORE_ENVIRONMENT` рекомендуется только на промежуточных и тестовых серверах, доступ к которым из Интернета закрыт. Завершив устранение неполадок, удалите эту переменную среды из файла *web.config*. Сведения о настройке переменных среды в файле *web.config* см. в статье о [дочернем элементе environmentVariables в aspNetCore](xref:host-and-deploy/aspnet-core-module#setting-environment-variables).

## <a name="common-startup-errors"></a>Стандартные ошибки запуска

См. раздел <xref:host-and-deploy/azure-iis-errors-reference>. В этой статье рассматривается большинство распространенных ошибок, препятствующих запуску приложений.

## <a name="obtain-data-from-an-app"></a>Получение данных из приложения

Если приложение способно отвечать на запросы, получите данные о запросе, подключении и дополнительные данные из приложений с помощью встроенного терминала ПО промежуточного слоя. Дополнительные сведения и примеры с кодом см. здесь: <xref:test/troubleshoot#obtain-data-from-an-app>.

## <a name="create-a-dump"></a>Создание дампа

*Дамп* представляет собой моментальный снимок системной памяти и может помочь определить причину аварийного завершения, сбоя запуска или медленной работы приложения.

### <a name="app-crashes-or-encounters-an-exception"></a>Аварийное завершение работы приложения или исключение

Получите и проанализируйте дамп из [отчетов об ошибках Windows (WER)](/windows/desktop/wer/windows-error-reporting):

1. Создайте папку для хранения файлов аварийного дампа в `c:\dumps`. Пул приложений должен иметь доступ на запись к папке.
1. Запустите [скрипт PowerShell EnableDumps](https://github.com/aspnet/AspNetCore.Docs/blob/master/aspnetcore/host-and-deploy/iis/troubleshoot/scripts/EnableDumps.ps1):
   * Если приложение использует [модель размещения в процессе](xref:host-and-deploy/iis/index#in-process-hosting-model), выполните скрипт для *w3wp.exe*:

     ```console
     .\EnableDumps w3wp.exe c:\dumps
     ```

   * Если приложение использует [модель размещения вне процесса](xref:host-and-deploy/iis/index#out-of-process-hosting-model), выполните скрипт для *dotnet.exe*:

     ```console
     .\EnableDumps dotnet.exe c:\dumps
     ```

1. Запустите приложение в условиях, вызывающих аварийное завершение.
1. После аварийного завершения запустите [скрипт PowerShell DisableDumps](https://github.com/aspnet/AspNetCore.Docs/blob/master/aspnetcore/host-and-deploy/iis/troubleshoot/scripts/DisableDumps.ps1):
   * Если приложение использует [модель размещения в процессе](xref:host-and-deploy/iis/index#in-process-hosting-model), выполните скрипт для *w3wp.exe*:

     ```console
     .\DisableDumps w3wp.exe
     ```

   * Если приложение использует [модель размещения вне процесса](xref:host-and-deploy/iis/index#out-of-process-hosting-model), выполните скрипт для *dotnet.exe*:

     ```console
     .\DisableDumps dotnet.exe
     ```

Когда приложение аварийно завершит работу и сбор дампов будет выполнен, приложение сможет завершить работу обычным образом. Скрипт PowerShell настраивает отчеты об ошибках Windows для сбора до пяти дампов для приложения.

> [!WARNING]
> Аварийные дампы могут занимать много места на диске (до нескольких гигабайтов каждый).

### <a name="app-hangs-fails-during-startup-or-runs-normally"></a>Приложение перестает отвечать на запросы, не запускается или работает в обычном режиме

Когда приложение *перестает отвечать на запросы* (но аварийное завершение не происходит), не запускается или работает в обычном режиме, см. раздел [Файлы дампа пользовательского режима: выбор лучшего инструмента](/windows-hardware/drivers/debugger/user-mode-dump-files#choosing-the-best-tool), чтобы выбрать подходящий инструмент для создания дампа.

### <a name="analyze-the-dump"></a>Анализ дампа

Дамп можно проанализировать несколькими способами. Дополнительные сведения см. в разделе [Анализ файла дампа пользовательского режима](/windows-hardware/drivers/debugger/analyzing-a-user-mode-dump-file).

## <a name="remote-debugging"></a>Удаленная отладка

Изучите раздел документации по Visual Studio [об удаленной отладке ASP.NET Core на удаленном компьютере IIS в Visual Studio 2017](/visualstudio/debugger/remote-debugging-aspnet-on-a-remote-iis-computer).

## <a name="application-insights"></a>Application Insights

[Application Insights](/azure/application-insights/) предоставляет данные телеметрии из приложений, размещенных в IIS, в том числе возможности регистрации ошибок и создания отчетов. Application Insights может сообщать только об ошибках, возникающих после запуска приложения, когда становятся доступными функции ведения журнала приложения. Дополнительные сведения см. в статье [Application Insights для ASP.NET Core](/azure/application-insights/app-insights-asp-net-core).

## <a name="additional-advice"></a>Дополнительные советы

Иногда приложения-функции перестают работать сразу после обновления пакета SDK для .NET Core на компьютере разработки или обновления пакетов в самом приложении. В некоторых случаях в результате важного обновления несогласованные версии пакетов могут привести к нарушению работы приложения. Большинство этих проблем можно исправить следующим образом:

1. Удалите папки *bin* и *obj*.
1. Очистите кэши пакетов, размещенные в папках *%UserProfile%\\.nuget\\packages* и *%LocalAppData%\\Nuget\\v3-cache*.
1. Восстановите и перестройте проект.
1. Убедитесь, что предыдущее развертывание на сервере полностью удалено, прежде чем развертывать его снова.

> [!TIP]
> Очистить кэши пакета проще всего командой `dotnet nuget locals all --clear` из командной строки.
>
> Также очистку кэшей пакетов можно выполнить средством [nuget.exe](https://www.nuget.org/downloads) или командой `nuget locals all -clear`. *NuGet.exe* не входит в пакет установки операционной системы Windows для настольных компьютеров и его нужно получить отдельно на [веб-сайте NuGet](https://www.nuget.org/downloads).

## <a name="additional-resources"></a>Дополнительные ресурсы

* <xref:test/troubleshoot>
* <xref:fundamentals/error-handling>
* <xref:host-and-deploy/azure-iis-errors-reference>
* <xref:host-and-deploy/aspnet-core-module>
* <xref:host-and-deploy/azure-apps/troubleshoot>
