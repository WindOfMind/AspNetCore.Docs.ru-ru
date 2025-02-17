---
title: Создание клиента и сервера gRPC .NET Core в ASP.NET Core
author: juntaoluo
description: В этом учебнике объясняется, как создать службу и клиента gRPC в ASP.NET Core. Узнайте, как создать проект службы gRPC, изменить файл proto и добавить дуплексный режим потоковой передачи вызовов.
monikerRange: '>= aspnetcore-3.0'
ms.author: johluo
ms.date: 06/12/2019
uid: tutorials/grpc/grpc-start
ms.openlocfilehash: 8507c3dcefeb61a4dd34a1ff967c082d287cf646
ms.sourcegitcommit: d6e51c60439f03a8992bda70cc982ddb15d3f100
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/03/2019
ms.locfileid: "67555895"
---
# <a name="tutorial-create-a-grpc-client-and-server-in-aspnet-core"></a>Учебник. Создание клиента и сервера gRPC в ASP.NET Core

Автор: [John Luo](https://github.com/juntaoluo) (Джон Луо)

В этом руководстве показано, как создать клиент [gRPC](https://grpc.io/docs/guides/) в .NET Core и сервер gRPC ASP.NET Core.

В итоге вы получите клиент gRPC, который взаимодействует со службой Greeter gRPC.

[Просмотреть или скачать пример кода](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/tutorials/grpc/grpc-start/sample) ([описание скачивания](xref:index#how-to-download-a-sample)).

В этом учебнике рассмотрены следующие задачи.

> [!div class="checklist"]
> * Создание сервера gRPC.
> * Создание клиента gRPC.
> * Тестирование службы клиента gRPC с помощью службы Greeter gRPC.

## <a name="prerequisites"></a>Предварительные требования

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

[!INCLUDE[](~/includes/net-core-prereqs-vs-3.0.md)]

# <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code.](#tab/visual-studio-code)

[!INCLUDE[](~/includes/net-core-prereqs-vsc-3.0.md)]

# <a name="visual-studio-for-mactabvisual-studio-mac"></a>[Visual Studio для Mac](#tab/visual-studio-mac)

[!INCLUDE[](~/includes/net-core-prereqs-mac-3.0.md)]

---

## <a name="create-a-grpc-service"></a>Создание службы gRPC

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

* В меню **Файл** Visual Studio откройте меню **Создать** > **Проект**.
* В диалоговом окне **Создать проект** выберите **Веб-приложение ASP.NET Core**.
* Выберите **Далее**
* Присвойте проекту имя **GrpcGreeter**. Для проекта необходимо установить имя *GrpcGreeter*, чтобы при копировании и вставке кода совпадали пространства имен.
* Выберите **Создать**.
* в диалоговом окне **Создание веб-приложения ASP.NET Core**:
  * В раскрывающемся меню выберите **.NET Core** и **ASP.NET Core 3.0**. 
  * Выберите шаблон **gRPC Service**.
  * Выберите **Создать**.

# <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code.](#tab/visual-studio-code)

* Откройте [Интегрированный терминал](https://code.visualstudio.com/docs/editor/integrated-terminal).
* Смените каталог (`cd`) на папку, в которой будет содержаться проект.
* Выполните следующие команды:

  ```console
  dotnet new grpc -o GrpcGreeter
  code -r GrpcGreeter
  ```

  * С помощью команды `dotnet new` в папке *GrpcGreeter* создается служба gRPC.
  * С помощью команды `code` папка *GrpcGreeter* открывается в новом экземпляре Visual Studio Code.

  Появится диалоговое окно с предупреждением **Required assets to build and debug are missing from 'GrpcGreeter'. Добавить их?**
* Выберите ответ **Да**.

# <a name="visual-studio-for-mactabvisual-studio-mac"></a>[Visual Studio для Mac](#tab/visual-studio-mac)

Из терминала выполните следующие команды:

```console
  dotnet new grpc -o GrpcGreeter
  cd GrpcGreeter
```

Эти команды позволяют создать службу gRPC с помощью [.NET Core CLI](/dotnet/core/tools/dotnet).

### <a name="open-the-project"></a>Открытие проекта

В Visual Studio откройте меню **Файл > Открыть файл** и выберите файл *GrpcGreeter.sln*.

<!-- End of VS tabs -->

---

### <a name="run-the-service"></a>Запуск службы

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

* Нажмите `Ctrl+F5`, чтобы запустить службу gRPC без отладчика.

  Служба запустится в командной строке Visual Studio.

# <a name="visual-studio-code--visual-studio-for-mactabvisual-studio-codevisual-studio-mac"></a>[Visual Studio Code/Visual Studio для Mac](#tab/visual-studio-code+visual-studio-mac)

* Выполните команду `dotnet run` в командной строке, чтобы запустить проект gRPC Greeter с именем *GrpcGreeter*.

<!-- End of combined VS/Mac tabs -->

---

В журналах отобразится служба, которая ожидает передачи данных через `http://localhost:50051`.

```console
info: Microsoft.Hosting.Lifetime[0]
      Now listening on: http://localhost:50051
info: Microsoft.Hosting.Lifetime[0]
      Application started. Press Ctrl+C to shut down.
info: Microsoft.Hosting.Lifetime[0]
      Hosting environment: Development
info: Microsoft.Hosting.Lifetime[0]
```

### <a name="examine-the-project-files"></a>Анализ файлов проекта

Файлы проекта *GrpcGreeter*:

* *greet.proto*: Файл *Protos/greet.proto* определяет службу gRPC `Greeter` и используется для создания ресурсов сервера gRPC. Дополнительные сведения см. в разделе [Введение в gRPC](xref:grpc/index).
* Папка *Services* содержит реализацию службы `Greeter`.
* *appSettings.json*: содержит данные конфигурации, такие как протокол, используемый в Kestrel. Дополнительные сведения можно найти по адресу: <xref:fundamentals/configuration/index>.
* *Program.cs*: содержит точку входа для службы gRPC. Дополнительные сведения можно найти по адресу: <xref:fundamentals/host/generic-host>.
* *Startup.cs*: содержит код, задающий поведение приложения. Дополнительные сведения: [Запуск приложения](xref:fundamentals/startup).

## <a name="create-the-grpc-client-in-a-net-console-app"></a>Создание клиента gRPC в консольном приложении .NET

## <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

* Откройте еще один экземпляр Visual Studio.
* Выберите **Файл** > **Создать** > **Проект** в меню.
* В диалоговом окне **Создать проект** выберите **Консольное приложение (.NET Core)** .
* Выберите **Далее**
* В текстовом поле **Имя** введите "GrpcGreeterClient".
* Выберите **Создать**.

# <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code.](#tab/visual-studio-code)

* Откройте [Интегрированный терминал](https://code.visualstudio.com/docs/editor/integrated-terminal).
* Смените каталог (`cd`) на папку, в которой будет содержаться проект.
* Выполните следующие команды:

```console
dotnet new console -o GrpcGreeterClient
code -r GrpcGreeterClient
```

# <a name="visual-studio-for-mactabvisual-studio-mac"></a>[Visual Studio для Mac](#tab/visual-studio-mac)

Следуйте инструкциям в [этой статье](/dotnet/core/tutorials/using-on-mac-vs-full-solution), чтобы создать консольное приложение с именем *GrpcGreeterClient*.

<!-- End of VS tabs -->

---

### <a name="add-required-packages"></a>Добавление необходимых пакетов

Для клиентского проекта gRPC требуются следующие пакеты:

* [Grpc.Net.Client](https://www.nuget.org/packages/Grpc.Net.Client), который содержит клиент .NET Core.
* [Google.Protobuf](https://www.nuget.org/packages/Google.Protobuf/), который содержит API сообщений protobuf для C#;
* [Grpc.Tools](https://www.nuget.org/packages/Grpc.Tools/), который содержит поддержку инструментов C# для файлов protobuf. Пакет инструментов не требуется во время выполнения, поэтому зависимость помечается `PrivateAssets="All"`.

### <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

Установка пакетов с помощью консоли диспетчера пакетов (PMC) или управления пакетами NuGet.

#### <a name="pmc-option-to-install-packages"></a>Установка пакетов с помощью консоли диспетчера пакетов

* В Visual Studio выберите пункты меню **Сервис** > **Диспетчер пакетов NuGet** > **Консоль диспетчера пакетов**.
* В **консоли диспетчера пакетов** перейдите в каталог, в котором содержится файл *GrpcGreeterClient.csproj*.
* Выполните следующие команды:

 ```powershell
Install-Package Grpc.Net.Client
Install-Package Google.Protobuf
Install-Package Grpc.Tools
```

#### <a name="manage-nuget-packages-option-to-install-packages"></a>Установка пакетов с помощью раздела управления пакетами NuGet

* Щелкните правой кнопкой мыши проект в **обозревателе решений** > **Управление пакетами NuGet**
* Выберите вкладку **Обзор**.
* В поле поиска введите **Grpc.Core**.
* Выберите пакет **Grpc.Core** на вкладке **Обзор** и нажмите кнопку **Установить**.
* Повторите все шаги для `Google.Protobuf` и `Grpc.Tools`.

### <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code.](#tab/visual-studio-code)

Во **встроенном терминале** выполните следующие команды:

```console
dotnet add GrpcGreeterClient.csproj package Grpc.Net.Client
dotnet add GrpcGreeterClient.csproj package Google.Protobuf
dotnet add GrpcGreeterClient.csproj package Grpc.Tools
```

### <a name="visual-studio-for-mactabvisual-studio-mac"></a>[Visual Studio для Mac](#tab/visual-studio-mac)

* Щелкните правой кнопкой мыши папку **Пакеты** на **панели решения** > **Добавление пакетов**
* В поле поиска введите **Grpc.Net.Client**.
* В области результатов выберите пакет **Grpc.Net.Client** и нажмите кнопку **Добавить пакет**
* Повторите все шаги для `Google.Protobuf` и `Grpc.Tools`.

---

### <a name="add-greetproto"></a>Добавление greet.proto

* Создайте папку **Protos** в клиентском проекте gRPC.
* Скопируйте файл **Protos\greet.proto** из службы Greeter gRPC в проект клиента gRPC.
* Измените файл проекта *GrpcGreeterClient.csproj*:

  # <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio) 

  Щелкните проект правой кнопкой мыши и выберите **Изменить файл проекта**.

  # <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code.](#tab/visual-studio-code) 

  Выберите файл *GrpcGreeterClient.csproj*.

  # <a name="visual-studio-for-mactabvisual-studio-mac"></a>[Visual Studio для Mac](#tab/visual-studio-mac)

  Щелкните проект правой кнопкой мыши и выберите **Сервис > Изменить файл**.

  ---

* Добавьте группу элементов с элементом `<Protobuf>`, ссылающимся на файл **greet.proto**:

  ```XML
  <ItemGroup>
    <Protobuf Include="Protos\greet.proto" GrpcServices="Client" />
  </ItemGroup>
  ```

### <a name="create-the-greeter-client"></a>Создание клиента Greeter

Скомпилируйте проект, чтобы создать типы в пространстве имен `GrpcGreeter`. Типы `GrpcGreeter` создаются автоматически в процессе сборки.

Добавьте в файл *Program.cs* клиента gRPC следующий код:

[!code-cs[](~/tutorials/grpc/grpc-start/sample/GrpcGreeterClient/Program.cs?name=snippet2)]

файл *Program.cs* содержит точку входа и логику для клиента gRPC.

Клиент Greeter создается следующим образом:

* Создание экземпляра `HttpClient` со сведениями для создания подключения к службе gRPC.
* Использование `HttpClient` для создания клиента Greeter:

[!code-cs[](~/tutorials/grpc/grpc-start/sample/GrpcGreeterClient/Program.cs?name=snippet&highlight=3-9)]

Клиент Greeter вызывает асинхронный метод `SayHello`. Отображается результат вызова `SayHello`:

[!code-cs[](~/tutorials/grpc/grpc-start/sample/GrpcGreeterClient/Program.cs?name=snippet&highlight=10-12)]

## <a name="test-the-grpc-client-with-the-grpc-greeter-service"></a>Тестирование клиента gRPC с помощью службы Greeter gRPC

### <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

* В службе Greeter нажмите `Ctrl+F5` для запуска сервера без отладчика.
* В проекте `GrpcGreeterClient` нажмите `Ctrl+F5` для запуска сервера без отладчика.

### <a name="visual-studio-code--visual-studio-for-mactabvisual-studio-codevisual-studio-mac"></a>[Visual Studio Code/Visual Studio для Mac](#tab/visual-studio-code+visual-studio-mac)

* Запустите службу Greeter.
* Запустите клиент.

<!-- End of combined VS/Mac tabs -->

---

Клиент отправляет приветствие в службу с сообщением, содержащим его имя "GreeterClient". Служба отправляет сообщение "Hello GreeterClient" в качестве ответа. Ответ "Hello GreeterClient" отображается в командной строке:

```console
Greeting: Hello GreeterClient
Press any key to exit...
```

Служба gRPC записывает сведения об успешном вызове в журналы, что отображается в командной строке.

```console
info: Microsoft.Hosting.Lifetime[0]
      Now listening on: http://localhost:50051
info: Microsoft.Hosting.Lifetime[0]
      Application started. Press Ctrl+C to shut down.
info: Microsoft.Hosting.Lifetime[0]
      Hosting environment: Development
info: Microsoft.Hosting.Lifetime[0]
      Content root path: C:\GH\aspnet\docs\4\Docs\aspnetcore\tutorials\grpc\grpc-start\sample\GrpcGreeter
info: Microsoft.AspNetCore.Hosting.Diagnostics[1]
      Request starting HTTP/2 POST http://localhost:50051/Greet.Greeter/SayHello application/grpc
info: Microsoft.AspNetCore.Routing.EndpointMiddleware[0]
      Executing endpoint 'gRPC - /Greet.Greeter/SayHello'
info: Microsoft.AspNetCore.Routing.EndpointMiddleware[1]
      Executed endpoint 'gRPC - /Greet.Greeter/SayHello'
info: Microsoft.AspNetCore.Hosting.Diagnostics[2]
      Request finished in 78.32260000000001ms 200 application/grpc
```

### <a name="next-steps"></a>Следующие шаги

* <xref:grpc/index>
* <xref:grpc/basics>
* <xref:grpc/migration>
