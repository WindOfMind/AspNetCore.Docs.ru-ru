---
title: Операции запросов и ответов в ASP.NET Core
author: jkotalik
description: Узнайте, как читать текст запроса и писать текст ответа в ASP.NET Core.
monikerRange: '>= aspnetcore-3.0'
ms.author: jukotali
ms.custom: mvc
ms.date: 02/26/2019
uid: fundamentals/middleware/request-response
ms.openlocfilehash: c9f6509738ef6290666a58268fbb0584913db9d6
ms.sourcegitcommit: 357a7120632b20465801c093e4e5bd4a315496a8
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/08/2019
ms.locfileid: "67649230"
---
# <a name="request-and-response-operations-in-aspnet-core"></a>Операции запросов и ответов в ASP.NET Core

Автор [Джастин Коталик (Justin Kotalik)](https://github.com/jkotalik)

В этой статье объясняется, как читать текст запроса и писать текст ответа. Возможно, вам потребуется написать код для этих операций при создании ПО промежуточного слоя. В других случаях писать такой код обычно не нужно, так как эти операции обрабатывают MVC и Razor Pages.

В ASP.NET Core 3.0 есть две абстракции для текста запросов и ответов: <xref:System.IO.Stream> и <xref:System.IO.Pipelines.Pipe>. При чтении запроса [HttpRequest.Body](xref:Microsoft.AspNetCore.Http.HttpRequest.Body) — это <xref:System.IO.Stream>, а `HttpRequest.BodyPipe` — это <xref:System.IO.Pipelines.PipeReader>. При записи ответа [HttpResponse.Body](xref:Microsoft.AspNetCore.Http.HttpResponse.Body) — это `HttpResponse.BodyPipe` и <xref:System.IO.Pipelines.PipeWriter>.

Мы рекомендуем использовать конвейеры, а не потоки. Потоки удобнее использовать для некоторых простых операций, но производительность конвейеров выше и с ними проще работать в большинстве сценариев. Начиная с ASP.NET Core 3.0 преимущество отдается внутреннему использованию конвейеров вместо потоков. Примеры:

- `FormReader`
- `TextReader`
- `TextWriter`
- `HttpResponse.WriteAsync`

Потоки по-прежнему используются. С ними продолжают работать в .NET, и многие типы потоков не имеют эквивалентного конвейера, например `FileStreams` и `ResponseCompression`.

## <a name="stream-examples"></a>Примеры потоков

Предположим, вам необходимо создать ПО промежуточного слоя, которое считывает весь текст запроса как список строк с разделением на новые строки. Реализация простого потока может выглядеть следующим образом:

[!code-csharp[](request-response/samples/3.x/RequestResponseSample/Startup.cs?name=GetListOfStringsFromStream)]

Этот код работает, но есть определенные проблемы:

- Перед добавлением `StringBuilder` в коде создается другая строка (`encodedString`), которая сразу отбрасывается. Этот процесс выполняется для всех байтов в потоке, и в результате выделяется дополнительный объем памяти для всего текста запроса.
- Код считывает всю строку перед разделением на новые строки. Было бы эффективнее искать новые строки в массиве байтов.

Ниже приведен пример, в котором устранены некоторые из этих проблем.

[!code-csharp[](request-response/samples/3.x/RequestResponseSample/Startup.cs?name=GetListOfStringsFromStreamMoreEfficient)]

В этом примере:

- Не создается буфер для всего текста запроса в `StringBuilder`, если нет символов новой строки.
- В строке не вызывается `Split`.

Но некоторые проблемы при этом остаются:

- Если символы новой строки разрежены, большая часть текста запроса буферизуется в строке.
- Строки (`remainingString`) по-прежнему создаются и добавляются в буфер строки, что приводит к выделению дополнительной памяти.

Эти проблемы можно решить, но ценой усложнения кода при незначительных его улучшениях. Конвейеры позволяют решить эти проблемы при минимальной сложности кода.

## <a name="pipelines"></a>Конвейеры

В следующем примере показано, как тот же сценарий можно обработать с помощью `PipeReader`.

[!code-csharp[](request-response/samples/3.x/RequestResponseSample/Startup.cs?name=GetListOfStringFromPipe)]

В этом примере устранены многие проблемы, характерные для реализации посредством потоков.

- В буфере строки теперь нет необходимости, так как `PipeReader` обрабатывает байты, которые не использовались.
- Кодированные строки напрямую добавляются в список возвращенных строк.
- Создание строк не связано с выделением памяти, кроме памяти, используемой строкой (за исключением вызова `ToArray()`).

## <a name="adapters"></a>Адаптеры

Теперь, когда свойства `Body` и `BodyPipe` доступны для `HttpRequest` и `HttpResponse`, что произойдет, если назначить `Body` другому потоку? В версии 3.0 новый набор адаптеров автоматически адаптирует каждый тип к другому. Например, если назначить `HttpRequest.Body` новому потоку, `HttpRequest.BodyPipe` автоматически назначается новому `PipeReader`, который создает оболочку для `HttpRequest.Body`. То же относится и к заданию свойства `BodyPipe`. Если `HttpResponse.BodyPipe` назначить новому `PipeWriter`, `HttpResponse.Body` автоматически назначается новому потоку, который создает оболочку для `HttpResponse.BodyPipe`.

## <a name="startasync"></a>StartAsync

Метод `HttpResponse.StartAsync` впервые появился в версии 3.0. Он используется для указания того, что заголовки являются неизменяемыми, а также для запуска обратных вызовов `OnStarting`. В предварительной версии 3.0, необходимо вызвать `StartAsync` перед использованием `HttpRequest.BodyPipe`. В последующих выпусках это станет рекомендацией. При использования Kestrel в качестве сервера вызов StartAsync перед применением `PipeReader` гарантирует, что память, возвращенная `GetMemory`, относится к внутреннему методу <xref:System.IO.Pipelines.Pipe> Kestrel, а не к внешнему буферу.

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Ознакомительная статья о библиотеке System.IO.Pipelines](https://devblogs.microsoft.com/dotnet/system-io-pipelines-high-performance-io-in-net/)
- <xref:fundamentals/middleware/write>
