---
title: Учебник. Начало работы с Razor Pages в ASP.NET Core
author: rick-anderson
description: В этой серии руководств объясняется, как использовать Razor Pages в ASP.NET Core. Узнайте, как создать модель, сгенерировать код для Razor Pages, использовать Entity Framework Core и SQL Server для доступа к данным, добавлять функции поиска и проверки ввода, а также использовать возможность миграции для обновления модели.
ms.author: riande
ms.date: 6/3/2019
uid: tutorials/razor-pages/razor-pages-start
ms.openlocfilehash: 207f5a5ab5e21d8cd8f6cf2f63641b94d8077ae5
ms.sourcegitcommit: d6e51c60439f03a8992bda70cc982ddb15d3f100
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/03/2019
ms.locfileid: "67555798"
---
# <a name="tutorial-get-started-with-razor-pages-in-aspnet-core"></a>Учебник. Начало работы с Razor Pages в ASP.NET Core

Автор: [Рик Андерсон](https://twitter.com/RickAndMSFT) (Rick Anderson)

Это первый учебник из серии. В этой [серии](xref:tutorials/razor-pages/index) приводятся основные сведения о сборке веб-приложения Razor Pages в ASP.NET Core.

[!INCLUDE[](~/includes/advancedRP.md)]

Пройдя всю серию, вы получите приложение, которое управляет базой данных фильмов.  

[!INCLUDE[View or download sample code](~/includes/rp/download.md)]

В этом учебнике рассмотрены следующие задачи.

> [!div class="checklist"]
> * Создание веб-приложения Razor Pages.
> * Запустите приложение.
> * Анализ файлов проекта.

Пройдя это руководство, вы получите рабочее веб-приложение Razor Pages, сборка которого описана в последующих руководствах.

![Домашняя или индексная страница](razor-pages-start/_static/home2.2.png)

## <a name="prerequisites"></a>Предварительные требования

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

[!INCLUDE[](~/includes/net-core-prereqs-vs2019-2.2.md)]

# <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code.](#tab/visual-studio-code)

[!INCLUDE[](~/includes/net-core-prereqs-vsc-2.2.md)]

# <a name="visual-studio-for-mactabvisual-studio-mac"></a>[Visual Studio для Mac](#tab/visual-studio-mac)

[!INCLUDE[](~/includes/net-core-prereqs-mac-2.2.md)]

---

## <a name="create-a-razor-pages-web-app"></a>Создание веб-приложения Razor Pages

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

* В меню **Файл** Visual Studio откройте меню **Создать** > **Проект**.

* Создайте веб-приложение ASP.NET Core и нажмите кнопку **Далее**.

  ![Новое веб-приложение ASP.NET Core](razor-pages-start/_static/np_2.1.png)

* Назовите проект **RazorPagesMovie**. Очень важно, чтобы проект имел имя *RazorPagesMovie*, так как пространства имен при копировании и вставке кода должны совпасть.

  ![Новое веб-приложение ASP.NET Core](razor-pages-start/_static/config.png)

* Выберите в раскрывающемся списке пункт **ASP.NET Core 2.2**, затем — **Веб-приложение** и нажмите кнопку **Создать**.

![Новое веб-приложение ASP.NET Core](razor-pages-start/_static/np_2_2.2.png)

  Создается следующий начальный проект:

  ![обозреватель решений](razor-pages-start/_static/se2.2.png)

# <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code.](#tab/visual-studio-code)

* Откройте [Интегрированный терминал](https://code.visualstudio.com/docs/editor/integrated-terminal).

* Перейдите в каталог `cd`, в котором находится проект.

* Выполните следующие команды:

  ```console
  dotnet new webapp -o RazorPagesMovie
  code -r RazorPagesMovie
  ```

  * Команда `dotnet new` создает новый проект Razor Pages в папке *RazorPagesMovie*.
  * Команда `code` открывает папку *RazorPagesMovie* в текущем экземпляре Visual Studio Code.

* Когда значок строки состояния OmniSharp станет зеленым, появится диалоговое окно с предупреждением **Required assets to build and debug are missing from 'RazorPagesMovie'. (В RazorPagesMovie отсутствуют необходимые ресурсы для сборки и отладки.) Добавить их?** Выберите ответ **Да**.

  Каталог *.vscode*, содержащий файлы *launch.json* и *tasks.json*, добавляется в корневой каталог проекта.

# <a name="visual-studio-for-mactabvisual-studio-mac"></a>[Visual Studio для Mac](#tab/visual-studio-mac)

В терминале выполните следующую команду:

<!-- TODO: update these instruction once mac support 2.2 projects -->

```console
dotnet new webapp -o RazorPagesMovie
```

Указанные выше команды используют [интерфейс командной строки .NET Core](/dotnet/core/tools/dotnet) для создания проекта Razor Pages.

## <a name="open-the-project"></a>Открытие проекта

В Visual Studio откройте меню **Файл > Открыть файл** и выберите файл *RazorPagesMovie.csproj*.

<!-- End of VS tabs -->

---

## <a name="run-the-app"></a>Запуск приложения

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

* Нажмите клавиши CTRL+F5, чтобы выполнить запуск без отладчика.

  [!INCLUDE[](~/includes/trustCertVS.md)]

  Visual Studio запускает [IIS Express](/iis/extensions/introduction-to-iis-express/iis-express-overview), а затем приложение. В адресной строке указывается `localhost:port#`, а не что-либо типа `example.com`. Это связано с тем, что `localhost` — стандартное имя узла для локального компьютера. Localhost обслуживает только веб-запросы с локального компьютера. Когда Visual Studio создает веб-проект, для веб-сервера используется случайный порт.

* На домашней странице нажмите **Принять**, чтобы согласиться на отслеживание.

  Это приложение не отслеживает персональные данные, но в шаблон проекта входит компонент согласия на случай, если приложение должно соответствовать [общему регламенту по защите данных (GDPR)](xref:security/gdpr) Европейского Союза.

  ![Домашняя или индексная страница](razor-pages-start/_static/homeGDPR2.2.png)

  На следующем рисунке показано приложение после принятия отслеживания:

  ![Домашняя или индексная страница](razor-pages-start/_static/home2.2.png)
  
# <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code.](#tab/visual-studio-code)

  [!INCLUDE[](~/includes/trustCertVSC.md)]

* Нажмите клавиши **CTRL-F5**, чтобы выполнить запуск без отладчика.

  Visual Studio Code запускает [Kestrel](xref:fundamentals/servers/kestrel), открывает браузер и переходит к `http://localhost:5001`. В адресной строке указывается `localhost:port#`, а не что-либо типа `example.com`. Это связано с тем, что `localhost` — стандартное имя узла для локального компьютера. Localhost обслуживает только веб-запросы с локального компьютера.

* На домашней странице нажмите **Принять**, чтобы согласиться на отслеживание.

  Это приложение не отслеживает персональные данные, но в шаблон проекта входит компонент согласия на случай, если приложение должно соответствовать [общему регламенту по защите данных (GDPR)](xref:security/gdpr) Европейского Союза.

  ![Домашняя или индексная страница](razor-pages-start/_static/homeGDPR2.2.png)

  На следующем рисунке показано приложение после принятия отслеживания:

  ![Домашняя или индексная страница](razor-pages-start/_static/home2.2.png)
  
# <a name="visual-studio-for-mactabvisual-studio-mac"></a>[Visual Studio для Mac](#tab/visual-studio-mac)

  [!INCLUDE[](~/includes/trustCertMac.md)]

* Нажмите клавиши **Cmd-Opt-F5**, чтобы выполнить запуск без отладчика.

  Visual Studio запускает [Kestrel](xref:fundamentals/servers/kestrel), открывает браузер и переходит к `http://localhost:5001`.

* На домашней странице нажмите **Принять**, чтобы согласиться на отслеживание.

  Это приложение не отслеживает персональные данные, но в шаблон проекта входит компонент согласия на случай, если приложение должно соответствовать [общему регламенту по защите данных (GDPR)](xref:security/gdpr) Европейского Союза.

  ![Домашняя или индексная страница](razor-pages-start/_static/homeGDPR2.2_safari.png)

  На следующем рисунке показано приложение после принятия отслеживания:

  ![Домашняя или индексная страница](razor-pages-start/_static/home2.2_safari.png)

<!-- End of VS tabs -->

---

## <a name="examine-the-project-files"></a>Анализ файлов проекта

Ниже приведен обзор основных папок и файлов проекта, с которыми вы будете работать в последующих учебниках.

### <a name="pages-folder"></a>Папка Pages

Содержит страницы Razor и вспомогательные файлы. Каждая страница Razor — это пара файлов.

* Файл *.cshtml*, содержащий HTML-разметку с кодом C# и синтаксисом Razor.
* Файл *. cshtml.cs*, содержащий код C#, который обрабатывает события страницы.

Имена вспомогательных файлов начинаются с символа подчеркивания. Например, файл *_Layout.cshtml* настраивает элементы пользовательского интерфейса, общие для всех страниц. Этот файл настраивает меню навигации в верхней части страницы и уведомление об авторских правах в нижней части страницы. Дополнительные сведения можно найти по адресу: <xref:mvc/views/layout>.

### <a name="wwwroot-folder"></a>Папка wwwroot

Содержит статические файлы, такие как HTML-файлы, файлы JavaScript и CSS-файлы. Дополнительные сведения можно найти по адресу: <xref:fundamentals/static-files>.

### <a name="appsettingsjson"></a>appSettings.json

Содержит данные конфигурации, например строки подключения. Дополнительные сведения можно найти по адресу: <xref:fundamentals/configuration/index>.

### <a name="programcs"></a>Program.cs

Содержит точку входа для программы. Дополнительные сведения можно найти по адресу: <xref:fundamentals/host/generic-host>.

### <a name="startupcs"></a>Startup.cs

Содержит код, который настраивает поведение приложения, например, требуется ли согласие для файлов cookie. Дополнительные сведения можно найти по адресу: <xref:fundamentals/startup>.

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Версия руководства на YouTube](https://www.youtube.com/watch?v=F0SP7Ry4flQ&feature=youtu.be)

## <a name="next-steps"></a>Следующие шаги

В этом учебнике рассмотрены следующие задачи.

> [!div class="checklist"]
> * Создание веб-приложения Razor Pages.
> * Запуск приложения.
> * Анализ файлов проекта.

Перейдите к следующему учебнику в серии:

> [!div class="step-by-step"]
> [Добавление модели](xref:tutorials/razor-pages/model)
