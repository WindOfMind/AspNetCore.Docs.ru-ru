---
title: Начало работы с MVC ASP.NET Core
author: rick-anderson
description: Сведения о начале работы с MVC ASP.NET Core.
ms.author: riande
ms.date: 04/24/2019
uid: tutorials/first-mvc-app/start-mvc
ms.openlocfilehash: 91564bac02df77632a3b56b6dbddeb3b622f6438
ms.sourcegitcommit: d6e51c60439f03a8992bda70cc982ddb15d3f100
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/03/2019
ms.locfileid: "67555849"
---
# <a name="get-started-with-aspnet-core-mvc"></a>Начало работы с MVC ASP.NET Core

Автор: [Рик Андерсон](https://twitter.com/RickAndMSFT) (Rick Anderson)

[!INCLUDE [consider RP](~/includes/razor.md)]

В этом учебнике приводятся основные сведения о веб-приложении MVC ASP.NET Core.

Это приложение служит для управления базой данных названий фильмов. Вы научитесь:

> [!div class="checklist"]
> * Создание веб-приложения.
> * Добавление модели и формирование шаблона.
> * Работа с базой данных.
> * Добавление поиска и проверки.

В конечном итоге вы получите приложение, позволяющее управлять данными фильмов и отображать их.

[!INCLUDE[](~/includes/mvc-intro/download.md)]

## <a name="prerequisites"></a>Предварительные требования

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

[!INCLUDE[](~/includes/net-core-prereqs-vs2019-2.2.md)]

# <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code.](#tab/visual-studio-code)

[!INCLUDE[](~/includes/net-core-prereqs-vsc-2.2.md)]

# <a name="visual-studio-for-mactabvisual-studio-mac"></a>[Visual Studio для Mac](#tab/visual-studio-mac)

[!INCLUDE[](~/includes/net-core-prereqs-mac-2.2.md)]

---
## <a name="create-a-web-app"></a>Создание веб-приложения

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

* В Visual Studio выберите **Создать проект**.

* Выберите пункт **Веб-приложение ASP.NET Core** и нажмите кнопку **Далее**.

![Новое веб-приложение ASP.NET Core](start-mvc/_static/np_2.1.png)

* Присвойте проекту имя **MvcMovie** и нажмите кнопку **Создать**. Имя **MvcMovie** необходимо присвоить для того, чтобы при копировании кода пространства имен совпали.

  ![Новое веб-приложение ASP.NET Core](start-mvc/_static/config.png)


* Выберите **Веб-приложение (модель-представление-контроллер)** и нажмите кнопку **Создать**.

![Диалоговое окно нового проекта, .NET Core в левой области, веб-узел ASP.NET Core ](start-mvc/_static/new_project22-21.png)

В Visual Studio используется только что созданный вами шаблон по умолчанию для проекта MVC. Чтобы приложение стало рабочим, осталось только указать имя проекта и выбрать несколько параметров. Это простой начальный проект и хорошая стартовая точка.

# <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code.](#tab/visual-studio-code)

Для работы с этим руководством требуется знание VS Code. Дополнительные сведения см. в разделах [Начало работы с VS Code](https://code.visualstudio.com/docs) и [Справка по Visual Studio Code](#visual-studio-code-help).

* Откройте [Интегрированный терминал](https://code.visualstudio.com/docs/editor/integrated-terminal).
* Смените каталог (`cd`) на папку, в которой будет содержаться проект.
* Выполните следующую команду:

   ```console
   dotnet new mvc -o MvcMovie
   code -r MvcMovie
   ```

  * Появится диалоговое окно с предупреждением **В MvcMovie отсутствуют необходимые ресурсы для сборки и отладки. Добавить их?**  Выберите ответ **Да**.

  * `dotnet new mvc -o MvcMovie`: создает новый проект MVC ASP.NET Core в папке *MvcMovie*.
  * `code -r MvcMovie`: загружает файл проекта *MvcMovie.csproj* в Visual Studio Code.

# <a name="visual-studio-for-mactabvisual-studio-mac"></a>[Visual Studio для Mac](#tab/visual-studio-mac)

* Выберите **Файл** > **Новое решение**.

  ![Новое решение macOS](./start-mvc/_static/new_project_vsmac.png)

* Выберите **.NET Core** > **Приложение** > **Веб-приложение (модель — представление — контроллер)**  > **Далее**.

  ![Диалоговое окно "Новый проект" в macOS](./start-mvc/_static/new_project_mvc_vsmac.png)

* В диалоговом окне **Настройка нового веб-API ASP.NET Core** оставьте установленное по умолчанию значение для параметра **Целевая платформа**, то есть **.NET Core 2.2**.

  ![Выбор .NET Core 2.2 для macOS](./start-mvc/_static/new_project_22_vsmac.png)

* Присвойте проекту имя **MvcMovie** и нажмите кнопку **Создать**.

---

### <a name="run-the-app"></a>Запуск приложения

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

Нажмите **CTRl+F5**, чтобы запустить приложение без отладки.

[!INCLUDE[](~/includes/trustCertVS.md)]

* Visual Studio запускает [IIS Express](/iis/extensions/introduction-to-iis-express/iis-express-overview), а затем приложение. Обратите внимание на то, что в адресной строке указывается `localhost:port#`, а не что-либо типа `example.com`. Это связано с тем, что `localhost` — стандартное имя узла для локального компьютера. Когда Visual Studio создает веб-проект, для веб-сервера используется случайный порт.
* Запуск приложения с помощью клавиш CTRL+F5 (режим без отладки) позволяет внести изменения в код, сохранить файл, обновить браузер и увидеть изменения в коде. Многие разработчики предпочитают использовать режим без отладки, чтобы быстро запустить приложение и просмотреть изменения.
* Из меню **Отладка** можно запустить приложение в режиме с отладкой или без.

  ![Меню отладки](start-mvc/_static/debug_menu.png)

* Чтобы выполнить отладку приложения, нажмите кнопку **IIS Express**.

  ![IIS Express](start-mvc/_static/iis_express.png)

* Нажмите **Принять**, чтобы согласиться на отслеживание. Это приложение не отслеживает персональные данные. Созданный шаблоном код включает ресурсы для соблюдения [Общего регламента по защите данных (GDPR)](xref:security/gdpr).

  ![Домашняя или индексная страница](start-mvc/_static/privacy.png)

  На следующем рисунке показано приложение после принятия отслеживания:

  ![Домашняя или индексная страница](start-mvc/_static/home2.2.png)

# <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code.](#tab/visual-studio-code)

Нажмите клавиши CTRL+F5, чтобы выполнить запуск без отладчика.

[!INCLUDE[](~/includes/trustCertVSC.md)]

  Visual Studio Code запускает [Kestrel](xref:fundamentals/servers/kestrel), открывает браузер и переходит к `https://localhost:5001`. В адресной строке указывается `localhost:port:5001`, а не что-либо типа `example.com`. Это связано с тем, что `localhost` — стандартное имя узла для локального компьютера. Localhost обслуживает только веб-запросы с локального компьютера.

  Запуск приложения с помощью клавиш CTRL+F5 (режим без отладки) позволяет внести изменения в код, сохранить файл, обновить браузер и увидеть изменения в коде. Многие разработчики предпочитают использовать режим без отладки, чтобы быстро обновлять страницу и просматривать изменения.

* Нажмите **Принять**, чтобы согласиться на отслеживание. Это приложение не отслеживает персональные данные. Созданный шаблоном код включает ресурсы для соблюдения [Общего регламента по защите данных (GDPR)](xref:security/gdpr).

  ![Домашняя или индексная страница](start-mvc/_static/privacy.png)

  На следующем рисунке показано приложение после принятия отслеживания:

  ![Домашняя или индексная страница](start-mvc/_static/home2.2.png)

# <a name="visual-studio-for-mactabvisual-studio-mac"></a>[Visual Studio для Mac](#tab/visual-studio-mac)

Выберите **Выполнить** > **Запуск без отладки**, чтобы запустить приложение. Visual Studio для Mac запустит сервер [Kestrel](xref:fundamentals/servers/index#kestrel), откроет браузер и перейдет к `http://localhost:port`, где *port* — это номер порта, выбранный случайным образом.

[!INCLUDE[](~/includes/trustCertMac.md)]

* В адресной строке указывается `localhost:port#`, а не что-либо типа `example.com`. Это связано с тем, что `localhost` — стандартное имя узла для локального компьютера. Когда Visual Studio создает веб-проект, для веб-сервера используется случайный порт. При запуске приложения вы увидите другой номер порта.
* В меню **Запуск** можно запустить приложение в режиме с отладкой или без нее.

* Нажмите **Принять**, чтобы согласиться на отслеживание. Это приложение не отслеживает персональные данные. Созданный шаблоном код включает ресурсы для соблюдения [Общего регламента по защите данных (GDPR)](xref:security/gdpr).

  ![Домашняя или индексная страница](./start-mvc/_static/output_privacy_macos.png)

  На следующем рисунке показано приложение после принятия отслеживания:

  ![Домашняя или индексная страница](./start-mvc/_static/output_macos.png)

---

[!INCLUDE[](~/includes/vs-vsc-vsmac-help.md)]

В следующей части этого учебника мы поговорим об MVC и приступим к написанию кода.

> [!div class="step-by-step"]
> [Вперед](adding-controller.md)
