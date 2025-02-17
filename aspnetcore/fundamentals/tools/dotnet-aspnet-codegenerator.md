---
title: Команда dotnet aspnet-codegenerator
author: rick-anderson
ms.author: riande
description: Команда dotnet aspnet-codegenerator формирует шаблоны для проектов ASP.NET Core
ms.date: 07/04/2019
uid: fundamentals/tools/dotnet-aspnet-codegenerator
ms.openlocfilehash: 55b592d9d203970777c84438e210519957abb35d
ms.sourcegitcommit: f6e6730872a7d6f039f97d1df762f0d0bd5e34cf
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/04/2019
ms.locfileid: "67561687"
---
# <a name="dotnet-aspnet-codegenerator"></a>dotnet aspnet-codegenerator

Автор: [Рик Андерсон](https://twitter.com/RickAndMSFT) (Rick Anderson)

Команда `dotnet aspnet-codegenerator` запускает подсистему формирования шаблонов ASP.NET Core. Команду `dotnet aspnet-codegenerator` достаточно применить для формирования шаблонов из командной строки. Ее не нужно использовать для формирования шаблонов с помощью Visual Studio.

Эта статья относится к [пакету SDK для .NET Core 2.2.x](https://dotnet.microsoft.com/download/dotnet-core/2.2) и более поздних версий.

## <a name="installing-aspnet-codegenerator"></a>Установка aspnet-codegenerator

`aspnet-codegenerator` — [глобальное средство](/dotnet/core/tools/global-tools), которое нужно установить. Следующая команда позволяет установить последнюю стабильную версию средства `aspnet-codegenerator`:

```console
dotnet tool install -g aspnet-codegenerator
```

Приведенная ниже команда позволяет обновить `aspnet-codegenerator` до последней стабильной версии, доступной из установленных пакетов SDK для .NET Core.

```console
dotnet tool update -g aspnet-codegenerator
```

## <a name="synopsis"></a>Краткий обзор

```
dotnet aspnet-codegenerator [arguments] [-p|--project] [-n|--nuget-package-dir] [-c|--configuration] [-tfm|--target-framework] [-b|--build-base-path] [--no-build] 
dotnet aspnet-codegenerator [-h|--help]
```

## <a name="description"></a>ОПИСАНИЕ

Глобальная команда `dotnet aspnet-codegenerator ` запускает генератор кода и подсистему формирования шаблонов ASP.NET Core.

## <a name="arguments"></a>Аргументы

`generator`

Запускаемый генератор кода. Доступны такие генераторы:

| Generator | Операция |
| ----------------- | ------------ | 
| площадь      | [Формирует шаблон области](/aspnet/core/mvc/controllers/areas) |
  контроллер| [Формирует шаблон контроллера](/aspnet/core/tutorials/first-mvc-app/adding-model) |
  identity  | [Формирует шаблон удостоверения](/aspnet/core/security/authentication/scaffold-identity) |
  razorpage | [Формирует шаблоны страниц Razor Pages](/aspnet/core/tutorials/razor-pages/model) |
  view      | [Формирует шаблон представления](/aspnet/core/mvc/views/overview) |

## <a name="options"></a>Параметры

`-n|--nuget-package-dir`

Позволяет указать каталог пакета NuGet.

`-c|--configuration {Debug|Release}`

Определяет конфигурацию сборки. Значение по умолчанию — `Debug`.

`-tfm|--target-framework`

[Целевая платформа](/dotnet/standard/frameworks) для использования. Например, `net46`.

`-b|--build-base-path`

Базовый путь сборки.

`-h|--help`

Выводит краткую справку по команде.

`--no-build`

Не выполняет сборку проекта перед запуском. Он также неявно задает флаг `--no-restore`.

`-p|--project <PATH>`

Задает путь к запускаемому файлу проекта (имя папки или полный путь). Если значение не задано, по умолчанию используется текущий каталог.

## <a name="generator-options"></a>Параметры генератора

В следующих разделах подробно описаны параметры, доступные для поддерживаемых генераторов.

* Область
* Контроллер
* идентификации  
* Razorpage
* Просмотр

<a name="area"></a>

### <a name="area-options"></a>Параметры области

Это средство предназначено для веб-проектов ASP.NET Core с контроллерами и представлениями. Оно не предназначено для приложений Razor Pages.

Использование: `dotnet aspnet-codegenerator area AreaNameToGenerate`

Приведенная выше команда создает такие папки:

* *Области*
  * *AreaNameToGenerate*
    * *Контроллеры*
    * *Данные*
    * *Models*
    * *Представления*

<a name="ctl"></a>

### <a name="controller-options"></a>Параметры контроллера

В таблице ниже перечислены параметры для `aspnet-codegenerator`, `controller` и `razorpage`.

[!INCLUDE [aspnet-codegenerator-args-md.md](~/includes/aspnet-codegenerator-args-md.md)]

В таблице ниже перечислены параметры только для `aspnet-codegenerator controller`.

| Параметр               | ОПИСАНИЕ|
| ----------------- | ------------ |
| --controllerName или -name | Имя контроллера. |
| --useAsyncActions или -async | Создание асинхронных действий контроллера. |
| --noViews или -nv | Представления **не** создаются. |
| --restWithNoViews или -api  | Создание контроллера с API в стиле REST. Используется `noViews`, а все параметры, связанные с представлением, игнорируются. |
| --readWriteActions или -actions | Создание контроллера с действиями чтения и записи без модели. |

Чтобы получить справку по команде `aspnet-codegenerator controller`, используйте параметр `-h`.

```console
dotnet aspnet-codegenerator controller -h
```

Пример `dotnet aspnet-codegenerator controller` см. в разделе [Создание модели фильма](/aspnet/core/tutorials/razor-pages/model).

### <a name="razorpage"></a>Razorpage

<a name="rp"></a>

Шаблоны для страниц Razor Pages можно формировать по отдельности. Для этого нужно указать имя новой страницы и используемый шаблон. Поддерживаются такие шаблоны:

* `Empty`
* `Create`
* `Edit`
* `Delete`
* `Details`
* `List`

Например, в следующей команде используется шаблон Edit для создания *MyEdit.cshtml* и *MyEdit.cshtml.cs*:

```console
dotnet aspnet-codegenerator razorpage MyEdit Edit -m Movie -dc RazorPagesMovieContext -outDir Pages/Movies
```

Создаются следующие шаблоны, а шаблон и имя созданного файла обычно не указываются:

* `Create`
* `Edit`
* `Delete`
* `Details`
* `List`

В таблице ниже перечислены параметры для `aspnet-codegenerator`, `razorpage` и `controller`.

[!INCLUDE [aspnet-codegenerator-args-md.md](~/includes/aspnet-codegenerator-args-md.md)]

В таблице ниже перечислены параметры только для `aspnet-codegenerator razorpage`.

| Параметр               | ОПИСАНИЕ|
| ----------------- | ------------ |
|   --namespaceName или -namespace | Имя пространства имен, используемого для созданной модели PageModel. |
| --partialView или -partial | Создание частичного представления. Если указан этот параметр, параметры макета -l и -udl игнорируются. |
| --noPageModel или -npm | Параметр, при использовании которого не создается класс PageModel для пустого шаблона. |

Чтобы получить справку по команде `aspnet-codegenerator razorpage`, используйте параметр `-h`.

```console
dotnet aspnet-codegenerator razorpage -h
```

Пример `dotnet aspnet-codegenerator razorpage` см. в разделе [Создание модели фильма](/aspnet/core/tutorials/razor-pages/model).

### <a name="identity"></a>идентификации

См. статью [Удостоверение шаблона](/aspnet/core/security/authentication/scaffold-identity).