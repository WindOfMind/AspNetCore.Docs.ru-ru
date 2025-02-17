---
title: Использовать потоковую передачу в ASP.NET Core SignalR
author: bradygaster
description: Узнайте, как выполнять потоковый обмен данными между клиентом и сервером.
monikerRange: '>= aspnetcore-2.1'
ms.author: bradyg
ms.custom: mvc
ms.date: 06/05/2019
uid: signalr/streaming
ms.openlocfilehash: a75156f398e113393ddb891d16eec3f09de80c09
ms.sourcegitcommit: e7e04a45195d4e0527af6f7cf1807defb56dc3c3
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 06/06/2019
ms.locfileid: "66750187"
---
# <a name="use-streaming-in-aspnet-core-signalr"></a>Использовать потоковую передачу в ASP.NET Core SignalR

По [Бреннан Конрой](https://github.com/BrennanConroy)

::: moniker range=">= aspnetcore-3.0"

ASP.NET Core SignalR поддерживает потоковой передачи от клиента к серверу и от сервера клиенту. Это полезно в сценариях, где фрагментов данных поступают по времени. При потоковой передаче, каждый фрагмент отправляется на клиенте или сервере как только она становится доступны, вместо ожидания возвращения всех данных станут доступны.

::: moniker-end

::: moniker range="< aspnetcore-3.0"

ASP.NET Core SignalR поддерживает потоковой передачи возвращаемые значения методов сервера. Это полезно в сценариях, где фрагментов данных поступают по времени. При возвращаемое значение передается клиенту, каждый фрагмент отправляется клиенту, как только она становится доступны, вместо ожидания возвращения всех данные станут доступны.

::: moniker-end

[Просмотреть или скачать образец кода](https://github.com/aspnet/AspNetCore.Docs/tree/live/aspnetcore/signalr/streaming/samples/) ([как скачивать](xref:index#how-to-download-a-sample))

## <a name="set-up-a-hub-for-streaming"></a>Настройка концентратора для потоковой передачи

::: moniker range=">= aspnetcore-3.0"

Метод концентратора автоматически становится потоковой передачи метода концентратора, если он возвращает <xref:System.Collections.Generic.IAsyncEnumerable`1>, <xref:System.Threading.Channels.ChannelReader%601>, `Task<IAsyncEnumerable<T>>`, или `Task<ChannelReader<T>>`.

::: moniker-end

::: moniker range="< aspnetcore-3.0"

Метод концентратора автоматически становится потоковой передачи метода концентратора, если он возвращает <xref:System.Threading.Channels.ChannelReader%601> или `Task<ChannelReader<T>>`.

::: moniker-end

### <a name="server-to-client-streaming"></a>Клиентом и сервером потоковой передачи

::: moniker range=">= aspnetcore-3.0"

Потоковая передача методов концентратора может возвращать `IAsyncEnumerable<T>` в дополнение к `ChannelReader<T>`. Самый простой способ возврата `IAsyncEnumerable<T>` заключается в изменении метода концентратора асинхронный метод итератора, как показано в следующем примере. Методы итератора async центра может принять `CancellationToken` параметр, который активируется, когда клиент отменяет подписку из потока. Асинхронные методы итератора избежать распространенных проблем с каналами, например не возвращают `ChannelReader` достаточно рано или выход из метода, не завершая <xref:System.Threading.Channels.ChannelWriter`1>.

[!INCLUDE[](~/includes/csharp-8-required.md)]

[!code-csharp[Streaming hub async iterator method](streaming/samples/3.0/Hubs/AsyncEnumerableHub.cs?name=snippet_AsyncIterator)]

::: moniker-end

В следующем примере показано с основами потоковой передачи данных на клиент с помощью каналов. Каждый раз, когда объект записывается <xref:System.Threading.Channels.ChannelWriter%601>, объект немедленно отправляется клиенту. В конце `ChannelWriter` завершения, чтобы сообщить клиенту поток закрыт.

> [!NOTE]
> Запись `ChannelWriter<T>` в фоновом потоке и возврат `ChannelReader` как можно скорее. Другие вызовы концентратора будут заблокированы, пока `ChannelReader` возвращается.
>
> Перенос логики в `try ... catch`. Завершить `Channel` в `catch` так и вне `catch` чтобы убедиться, что концентратор вызов метода завершается должным образом.

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[Streaming hub method](streaming/samples/3.0/Hubs/StreamHub.cs?name=snippet1)]

::: moniker-end

::: moniker range="= aspnetcore-2.2"

[!code-csharp[Streaming hub method](streaming/samples/2.2/Hubs/StreamHub.cs?name=snippet1)]

::: moniker-end

::: moniker range="= aspnetcore-2.1"

[!code-csharp[Streaming hub method](streaming/samples/2.1/Hubs/StreamHub.cs?name=snippet1)]

::: moniker-end

::: moniker range=">= aspnetcore-2.2"

Может принимать-клиентом и сервером потоковой передачи методов концентратора `CancellationToken` параметр, который активируется, когда клиент отменяет подписку из потока. Используйте этот маркер для остановки работы сервера и освободить все ресурсы, если клиент отключается до окончания потока.

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

### <a name="client-to-server-streaming"></a>Клиент сервер потоковой передачи

Метод концентратора, автоматически становится клиентом и сервером потоковой передачи метода концентратора, если он принимает один или несколько объектов типа <xref:System.Threading.Channels.ChannelReader%601> или <xref:System.Collections.Generic.IAsyncEnumerable%601>. В следующем примере показано основные данные потоковой передачи, отправленные клиентом. Каждый раз, когда клиент записывает <xref:System.Threading.Channels.ChannelWriter%601>, данные записываются в `ChannelReader` на сервере, из которой считывает метода концентратора.

[!code-csharp[Streaming upload hub method](streaming/samples/3.0/Hubs/StreamHub.cs?name=snippet2)]

<xref:System.Collections.Generic.IAsyncEnumerable%601> Соответствует версии метода.

[!INCLUDE[](~/includes/csharp-8-required.md)]

```csharp
public async Task UploadStream(IAsyncEnumerable<Stream> stream) 
{
    await foreach (var item in stream)
    {
        Console.WriteLine(item);
    }
}
```

::: moniker-end

## <a name="net-client"></a>Клиент .NET

### <a name="server-to-client-streaming"></a>Клиентом и сервером потоковой передачи


::: moniker range=">= aspnetcore-3.0"

`StreamAsync` И `StreamAsChannelAsync` методы `HubConnection` используются для вызова методов потоковой передачи-клиентом и сервером. Передайте имя метода концентратора и аргументы, заданные в методе концентратора, чтобы `StreamAsync` или `StreamAsChannelAsync`. Универсальный параметр на `StreamAsync<T>` и `StreamAsChannelAsync<T>` указывает тип объектов, возвращаемых методом потоковой передачи. Объект типа `IAsyncEnumerable<T>` или `ChannelReader<T>` возвращается из вызова потока и представляет собой поток на стороне клиента.

Объект `StreamAsync` пример, который возвращает `IAsyncEnumerable<int>`:

```csharp
// Call "Cancel" on this CancellationTokenSource to send a cancellation message to
// the server, which will trigger the corresponding token in the hub method.
var cancellationTokenSource = new CancellationTokenSource();
var stream = await hubConnection.StreamAsync<int>(
    "Counter", 10, 500, cancellationTokenSource.Token);

await foreach (var count in stream)
{
    Console.WriteLine($"{count}");
}

Console.WriteLine("Streaming completed");
```

Соответствующий `StreamAsChannelAsync` пример, который возвращает `ChannelReader<int>`:

```csharp
// Call "Cancel" on this CancellationTokenSource to send a cancellation message to
// the server, which will trigger the corresponding token in the hub method.
var cancellationTokenSource = new CancellationTokenSource();
var channel = await hubConnection.StreamAsChannelAsync<int>(
    "Counter", 10, 500, cancellationTokenSource.Token);

// Wait asynchronously for data to become available
while (await channel.WaitToReadAsync())
{
    // Read all currently available data synchronously, before waiting for more data
    while (channel.TryRead(out var count))
    {
        Console.WriteLine($"{count}");
    }
}

Console.WriteLine("Streaming completed");
```

::: moniker-end

::: moniker range=">= aspnetcore-2.2"

`StreamAsChannelAsync` Метод `HubConnection` используется для вызова метода клиентом и сервером потоковой передачи. Передайте имя метода концентратора и аргументы, заданные в методе концентратора, чтобы `StreamAsChannelAsync`. Универсальный параметр на `StreamAsChannelAsync<T>` указывает тип объектов, возвращаемых методом потоковой передачи. Объект `ChannelReader<T>` возвращается из вызова потока и представляет собой поток на стороне клиента.

```csharp
// Call "Cancel" on this CancellationTokenSource to send a cancellation message to
// the server, which will trigger the corresponding token in the hub method.
var cancellationTokenSource = new CancellationTokenSource();
var channel = await hubConnection.StreamAsChannelAsync<int>(
    "Counter", 10, 500, cancellationTokenSource.Token);

// Wait asynchronously for data to become available
while (await channel.WaitToReadAsync())
{
    // Read all currently available data synchronously, before waiting for more data
    while (channel.TryRead(out var count))
    {
        Console.WriteLine($"{count}");
    }
}

Console.WriteLine("Streaming completed");
```

::: moniker-end

::: moniker range="= aspnetcore-2.1"

`StreamAsChannelAsync` Метод `HubConnection` используется для вызова метода клиентом и сервером потоковой передачи. Передайте имя метода концентратора и аргументы, заданные в методе концентратора, чтобы `StreamAsChannelAsync`. Универсальный параметр на `StreamAsChannelAsync<T>` указывает тип объектов, возвращаемых методом потоковой передачи. Объект `ChannelReader<T>` возвращается из вызова потока и представляет собой поток на стороне клиента.

```csharp
var channel = await hubConnection
    .StreamAsChannelAsync<int>("Counter", 10, 500, CancellationToken.None);

// Wait asynchronously for data to become available
while (await channel.WaitToReadAsync())
{
    // Read all currently available data synchronously, before waiting for more data
    while (channel.TryRead(out var count))
    {
        Console.WriteLine($"{count}");
    }
}

Console.WriteLine("Streaming completed");
```

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

### <a name="client-to-server-streaming"></a>Клиент сервер потоковой передачи

Существует два способа вызова клиентом и сервером потоковой передачи метода концентратора от клиента .NET. Вы можете либо передайте в `IAsyncEnumerable<T>` или `ChannelReader` как аргумент `SendAsync`, `InvokeAsync`, или `StreamAsChannelAsync`в зависимости от вызова метода концентратора.

Каждый раз, когда данные записываются в `IAsyncEnumerable` или `ChannelWriter` объекта, метод концентратора на сервере получает новый элемент с данными от клиента.

При использовании `IAsyncEnumerable` объекта потока заканчивается после метода, возвращающего поток элементов завершает работу.

[!INCLUDE[](~/includes/csharp-8-required.md)]

```csharp
async IAsyncEnumerable<string> clientStreamData()
{
    for (var i = 0; i < 5; i++)
    {
        var data = await FetchSomeData();
        yield return data;
    }
    //After the for loop has completed and the local function exits the stream completion will be sent.
}

await connection.SendAsync("UploadStream", clientStreamData());
```

Или если вы используете `ChannelWriter`, выполнения канал с `channel.Writer.Complete()`:

```csharp
var channel = Channel.CreateBounded<string>(10);
await connection.SendAsync("UploadStream", channel.Reader);
await channel.Writer.WriteAsync("some data");
await channel.Writer.WriteAsync("some more data");
channel.Writer.Complete();
```

::: moniker-end

## <a name="javascript-client"></a>Клиент JavaScript

### <a name="server-to-client-streaming"></a>Клиентом и сервером потоковой передачи

Клиенты JavaScript вызывать-клиентом и сервером потоковой передачи методов Hub с помощью `connection.stream`. `stream` Метод принимает два аргумента:

* Имя метода концентратора. В следующем примере, является имя метода концентратора `Counter`.
* Аргументы, определенный в методе концентратора. В следующем примере аргументы являются подсчет числа элементов потока для получения и задержку между элементами потока.

`connection.stream` Возвращает `IStreamResult`, который содержит `subscribe` метод. Передайте `IStreamSubscriber` для `subscribe` и задайте `next`, `error`, и `complete` обратные вызовы для получения уведомлений из `stream` вызова.

::: moniker range=">= aspnetcore-2.2"

[!code-javascript[Streaming javascript](streaming/samples/2.2/wwwroot/js/stream.js?range=19-36)]

Чтобы завершить поток, из клиента, вызовите `dispose` метод `ISubscription` , возвращаемый методом `subscribe` метод. Вызов этого метода приводит к отмене `CancellationToken` параметр метода концентратора, если вы указали один.

::: moniker-end

::: moniker range="= aspnetcore-2.1"

[!code-javascript[Streaming javascript](streaming/samples/2.1/wwwroot/js/stream.js?range=19-36)]

Чтобы завершить поток, из клиента, вызовите `dispose` метод `ISubscription` , возвращаемый методом `subscribe` метод.

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

### <a name="client-to-server-streaming"></a>Клиент сервер потоковой передачи

Клиенты JavaScript вызывать клиентом и сервером потоковой передачи методов концентраторов, передав `Subject` как аргумент `send`, `invoke`, или `stream`в зависимости от вызова метода концентратора. `Subject` — Это класс, который выглядит как `Subject`. Например в RxJS, можно использовать [субъекта](https://rxjs-dev.firebaseapp.com/api/index/class/Subject) класс из этой библиотеки.

[!code-javascript[Upload javascript](streaming/samples/3.0/wwwroot/js/stream.js?range=41-51)]

Вызов `subject.next(item)` с элемент записывает элемент в поток, а метод концентратора получает элемента на сервере.

Чтобы завершить поток, вызовите `subject.complete()`.

## <a name="java-client"></a>Клиент Java

### <a name="server-to-client-streaming"></a>Клиентом и сервером потоковой передачи

Клиент SignalR Java использует `stream` метод для вызова методов потоковой передачи. `stream` принимает три или более аргументов:

* Ожидаемый тип элементов потока.
* Имя метода концентратора.
* Аргументы, определенный в методе концентратора.

```java
hubConnection.stream(String.class, "ExampleStreamingHubMethod", "Arg1")
    .subscribe(
        (item) -> {/* Define your onNext handler here. */ },
        (error) -> {/* Define your onError handler here. */},
        () -> {/* Define your onCompleted handler here. */});
```

`stream` Метод `HubConnection` Возвращает наблюдаемый объект типа элемента потока. Наблюдаемого типа `subscribe` метод именно `onNext`, `onError` и `onCompleted` определены обработчики.

::: moniker-end

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Центры](xref:signalr/hubs)
* [Клиент .NET](xref:signalr/dotnet-client)
* [Клиент JavaScript](xref:signalr/javascript-client)
* [Публикация в Azure](xref:signalr/publish-to-azure-web-app)
