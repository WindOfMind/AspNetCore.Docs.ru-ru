---
title: Кэширование ответов в ASP.NET Core
author: rick-anderson
description: Узнайте, как использовать кэширование ответов, чтобы снизить требования к пропускной способности и повысить производительность приложений ASP.NET Core.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.date: 02/28/2019
uid: performance/caching/response
ms.openlocfilehash: 2e247dcff2cbaa3711a9206d7237a061ae351e1d
ms.sourcegitcommit: 5b0eca8c21550f95de3bb21096bd4fd4d9098026
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/27/2019
ms.locfileid: "64892471"
---
# <a name="response-caching-in-aspnet-core"></a>Кэширование ответов в ASP.NET Core

По [Luo Джон](https://github.com/JunTaoLuo), [Рик Андерсон](https://twitter.com/RickAndMSFT), [Стив Смит](https://ardalis.com/), и [Люк Лэтем](https://github.com/guardrex)

[Просмотреть или скачать образец кода](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/performance/caching/response/samples) ([как скачивать](xref:index#how-to-download-a-sample))

Кэширование ответов уменьшает количество запросов, выполненных клиентского приложения или прокси-сервера веб-сервера. Кэширование ответов также сокращает время работы веб-сервер выполняет для создания ответа. Кэширование ответов управляется заголовки, указывающие способ клиента, прокси-сервера и по промежуточного слоя для кэширования ответов.

[Атрибута ResponseCache](#responsecache-attribute) участвует в параметр кэширования заголовков, в которых клиенты могут поддерживать при кэшировании ответов ответов. [По промежуточного слоя для кэширования ответа](xref:performance/caching/middleware) может использоваться для кэширования ответов на сервере. Можно использовать по промежуточного слоя <xref:Microsoft.AspNetCore.Mvc.ResponseCacheAttribute> свойства, которые влияют на поведение кэширования на стороне сервера.

## <a name="http-based-response-caching"></a>Кэширование ответов на основе HTTP

[Спецификации кэширования HTTP 1.1](https://tools.ietf.org/html/rfc7234) описано поведение Internet кэшей. Основной заголовок HTTP, используемый для кэширования [Cache-Control](https://tools.ietf.org/html/rfc7234#section-5.2), который используется для указания кэша *директивы*. Директивы Управление поведением кэширования, как запросы все же встречаются несущественные от клиентов к серверам и ответы сделать свой путь с серверов обратно клиентам. Запросы и ответы перемещения между прокси-серверами и прокси-серверы также должны соответствовать спецификации кэширования HTTP 1.1.

Распространенные `Cache-Control` директивы показаны в следующей таблице.

| Директива                                                       | Действие |
| --------------------------------------------------------------- | ------ |
| [public](https://tools.ietf.org/html/rfc7234#section-5.2.2.5)   | Кэш может хранить ответ. |
| [private](https://tools.ietf.org/html/rfc7234#section-5.2.2.6)  | Ответ не должны храниться в общий кэш. Частный кэш может хранить и повторно использовать ответ. |
| [max-age](https://tools.ietf.org/html/rfc7234#section-5.2.1.1)  | Клиент не принимает ответ, возраст которых больше, чем указанное число секунд. Примеры `max-age=60` (60 секунд), `max-age=2592000` (1 месяц) |
| [без кэша](https://tools.ietf.org/html/rfc7234#section-5.2.1.4) | **При запросах**: Кэш не должны использовать хранимые ответ для удовлетворения запроса. Повторно создает на сервере-источнике ответа для клиента, и по промежуточного слоя обновляет ответов в своем кэше.<br><br>**При ответах**: Ответ не должны использоваться для последующих запросов без проверки на исходном сервере. |
| [no-store](https://tools.ietf.org/html/rfc7234#section-5.2.1.5) | **При запросах**: Кэш не должен хранить запроса.<br><br>**При ответах**: Кэш не должен хранить любую часть ответа. |

В следующей таблице показаны другие заголовки кэша, которые влияют на кэширование.

| Header                                                     | Функция |
| ---------------------------------------------------------- | -------- |
| [Срок действия](https://tools.ietf.org/html/rfc7234#section-5.1)     | Приблизительное количество времени в секундах с момента ответа сформирован или создан успешно прошли проверку на исходном сервере. |
| [Срок действия истекает](https://tools.ietf.org/html/rfc7234#section-5.3) | Время, после чего ответ считается устаревшей. |
| [Директивы pragma](https://tools.ietf.org/html/rfc7234#section-5.4)  | Существует для обеспечения обратной совместимости с HTTP/1.0 кэширует параметра `no-cache` поведение. Если `Cache-Control` заголовок присутствует, `Pragma` заголовка учитывается. |
| [Различаются](https://tools.ietf.org/html/rfc7231#section-7.1.4)  | Указывает, что кэшированный ответ должен быть отправляется, только если все из `Vary` заголовок поля согласуются в кэшированный ответ исходного запроса и новый запрос. |

## <a name="http-based-caching-respects-request-cache-control-directives"></a>Отношениях кэширования на основе HTTP запроса директивы Cache-Control

[Спецификации кэширования HTTP 1.1 для заголовка Cache-Control](https://tools.ietf.org/html/rfc7234#section-5.2) требует кэша соблюдать является допустимым для `Cache-Control` заголовка, отправленные клиентом. Клиент может выполнять запросы с `no-cache` значение заголовка и принудительного создания нового ответа для каждого запроса сервером.

Всегда учитывает клиента `Cache-Control` заголовки запроса смысл, если вы считаете цель HTTP-кэширования. В официальной спецификации кэширование призвана сократить расходы на задержку и сети для удовлетворения запросов по сети, прокси-серверов, серверов и клиентов. Это не обязательно для контроля нагрузки на сервер-источник.

Отсутствует контроль разработчика над это поведение кэширования при использовании [по промежуточного слоя для кэширования ответа](xref:performance/caching/middleware) так, как по промежуточного слоя, соответствует официальной спецификации кэширования. [Запланированный усовершенствования в по промежуточного слоя](https://github.com/aspnet/AspNetCore/issues/2612) – это возможность для настройки по промежуточного слоя, чтобы игнорировать запроса `Cache-Control` заголовка при принятии решения о обслуживать кэшированный ответ. Плановое улучшения предоставляют возможность лучше нагрузка на сервер управления.

## <a name="other-caching-technology-in-aspnet-core"></a>Другие технология кэширования в ASP.NET Core

### <a name="in-memory-caching"></a>Кэширование в памяти

Кэширование в памяти использует память сервера для хранения кэшированных данных. Этот тип кэширования подходит для одного или нескольких серверов с использованием *прикрепленные сеансы*. Прикрепленные сеансы означает, что запросы от клиента всегда направляются на один и тот же сервер для обработки.

Дополнительные сведения см. в разделе <xref:performance/caching/memory>.

### <a name="distributed-cache"></a>Распределенный кэш

Используйте распределенный кэш для хранения данных в памяти, когда приложение будет размещено в облаке или сервер фермы. Кэш распределяется по всем серверам, которые обрабатывают запросы. Клиент может отправить запрос, обрабатываемый любой сервер в группе, при наличии кэшированных данных для клиента. ASP.NET Core предлагает SQL Server и кэши Redis распределенных.

Дополнительные сведения см. в разделе <xref:performance/caching/distributed>.

### <a name="cache-tag-helper"></a>Вспомогательная функция тега кэша

Кэширование содержимого из представления MVC или страница Razor со вспомогательной функцией тега кэша. Вспомогательная функция тега кэша использует кэширование в памяти для хранения данных.

Дополнительные сведения см. в разделе <xref:mvc/views/tag-helpers/builtin-th/cache-tag-helper>.

### <a name="distributed-cache-tag-helper"></a>Вспомогательная функция тега распределенного кэша

Кэшировать содержимое из представления MVC или страницу Razor в распределенных сценариях облака или веб-фермы с вспомогательная функция тега распределенного кэша. Для хранения данных кэша вспомогательной функции тега распределенного использует SQL Server или Redis.

Дополнительные сведения см. в разделе <xref:mvc/views/tag-helpers/builtin-th/distributed-cache-tag-helper>.

## <a name="responsecache-attribute"></a>Атрибута ResponseCache

<xref:Microsoft.AspNetCore.Mvc.ResponseCacheAttribute> Указывает параметры, необходимые для задания соответствующих заголовков в кэширование ответов.

> [!WARNING]
> Отключите кэширование для содержимого, которое содержит сведения о прошедших проверку подлинности клиентов. Кэширование можно включать только для содержимого, которое не меняется на основе удостоверения пользователя или ли пользователь выполнил вход.

<xref:Microsoft.AspNetCore.Mvc.CacheProfile.VaryByQueryKeys> хранимые ответ зависит от значения указанного списка ключей запроса. Если одно значение `*` не указан, по промежуточного слоя, зависит от ответов для всех запросов параметров строки запроса.

[По промежуточного слоя для кэширования ответа](xref:performance/caching/middleware) необходимо включить для задания <xref:Microsoft.AspNetCore.Mvc.CacheProfile.VaryByQueryKeys> свойство. В противном случае возникает исключение времени выполнения. Нет соответствующего заголовка HTTP для <xref:Microsoft.AspNetCore.Mvc.CacheProfile.VaryByQueryKeys> свойство. Свойство — это функция HTTP, обрабатываемых по промежуточного слоя для кэширования ответа. Для по промежуточного слоя для обслуживания кэшированный ответ строку запроса и значения строки запроса, должны совпадать предыдущего запроса. Например рассмотрим последовательность запросов и результаты, показанные в следующей таблице.

| Запрос                          | Результат                    |
| -------------------------------- | ------------------------- |
| `http://example.com?key1=value1` | Возвращаемые с сервера. |
| `http://example.com?key1=value1` | Возвращаемые по промежуточного слоя. |
| `http://example.com?key1=value2` | Возвращаемые с сервера. |

Первый запрос возвращается сервером и кэшируются в по промежуточного слоя. Второй запрос возвращается по промежуточного слоя, так как строка запроса совпадает с предыдущим запросом. Третий запрос не в кэше по промежуточного слоя, поскольку значение строки запроса не соответствует предыдущего запроса.

<xref:Microsoft.AspNetCore.Mvc.ResponseCacheAttribute> Используется для настройки и создания (с помощью <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory>) <xref:Microsoft.AspNetCore.Mvc.Internal.ResponseCacheFilter>. <xref:Microsoft.AspNetCore.Mvc.Internal.ResponseCacheFilter> Выполняет обновление соответствующие заголовки HTTP и функции ответа. Фильтр:

* Удаляет любые существующие заголовки для `Vary`, `Cache-Control`, и `Pragma`.
* Записывает соответствующие заголовки, в зависимости от свойств, заданных <xref:Microsoft.AspNetCore.Mvc.ResponseCacheAttribute>.
* Обновляет ответ HTTP функции кэширования, если <xref:Microsoft.AspNetCore.Mvc.CacheProfile.VaryByQueryKeys> имеет значение.

### <a name="vary"></a>Различаются

Этот заголовок записывается только когда <xref:Microsoft.AspNetCore.Mvc.CacheProfile.VaryByHeader> свойству. Это свойство имеет значение `Vary` значение свойства. В следующем примере используется <xref:Microsoft.AspNetCore.Mvc.CacheProfile.VaryByHeader> свойство:

[!code-csharp[](response/samples/2.x/ResponseCacheSample/Pages/Cache1.cshtml.cs?name=snippet)]

Пример приложения, просмотрите заголовки ответа с помощью средства обозревателя сети. Следующие заголовки ответа отправляются вместе с ответом Cache1 страницы:

```
Cache-Control: public,max-age=30
Vary: User-Agent
```

### <a name="nostore-and-locationnone"></a>NoStore и Location.None

<xref:Microsoft.AspNetCore.Mvc.CacheProfile.NoStore> переопределяет большинство других свойств. Если присвоить этому свойству `true`, `Cache-Control` заголовок имеет значение `no-store`. Если <xref:Microsoft.AspNetCore.Mvc.CacheProfile.Location> присваивается `None`:

* Параметру `Cache-Control` задается значение `no-store,no-cache`.
* Параметру `Pragma` задается значение `no-cache`.

Если <xref:Microsoft.AspNetCore.Mvc.CacheProfile.NoStore> — `false` и <xref:Microsoft.AspNetCore.Mvc.CacheProfile.Location> — `None`, `Cache-Control`, и `Pragma` присваивается `no-cache`.

<xref:Microsoft.AspNetCore.Mvc.CacheProfile.NoStore> обычно устанавливается равным `true` для страниц ошибок. На странице Cache2 в пример приложения создает заголовки ответа клиенту не хранить ответ.

[!code-csharp[](response/samples/2.x/ResponseCacheSample/Pages/Cache2.cshtml.cs?name=snippet)]

Пример приложения возвращает страницу Cache2 со следующими заголовками:

```
Cache-Control: no-store,no-cache
Pragma: no-cache
```

### <a name="location-and-duration"></a>Расположение и длительность

Чтобы включить кэширование, <xref:Microsoft.AspNetCore.Mvc.CacheProfile.Duration> должно быть присвоено положительное значение и <xref:Microsoft.AspNetCore.Mvc.CacheProfile.Location> должен быть либо `Any` (по умолчанию) или `Client`. В этом случае `Cache-Control` заголовок присваивается значение расположения, за которым следует `max-age` ответа.

> [!NOTE]
> <xref:Microsoft.AspNetCore.Mvc.CacheProfile.Location>в параметры `Any` и `Client` переводиться `Cache-Control` значения заголовка `public` и `private`, соответственно. Как отмечалось ранее, параметр <xref:Microsoft.AspNetCore.Mvc.CacheProfile.Location> для `None` задает оба `Cache-Control` и `Pragma` заголовки `no-cache`.

В следующем примере показано Cache3 страничной модели из примера приложения и заголовки, создаваемое путем установки <xref:Microsoft.AspNetCore.Mvc.CacheProfile.Duration> и оставить значение по умолчанию <xref:Microsoft.AspNetCore.Mvc.CacheProfile.Location> значение:

[!code-csharp[](response/samples/2.x/ResponseCacheSample/Pages/Cache3.cshtml.cs?name=snippet)]

Пример приложения возвращает страницу Cache3 с следующий заголовок:

```
Cache-Control: public,max-age=10
```

### <a name="cache-profiles"></a>Профили кэша

Вместо повторения параметры кэша ответа на многие атрибуты действий контроллера, профили кэша можно настроить в качестве параметров при настройке MVC и Razor Pages в `Startup.ConfigureServices`. Значения в конфигурации кэша, на которую указывает ссылка, используются значения по умолчанию, <xref:Microsoft.AspNetCore.Mvc.ResponseCacheAttribute> и переопределяются все свойства, заданные в атрибуте.

Настройка профиля кэша. В следующем примере показано 30 второй профиль кэша в примере приложения `Startup.ConfigureServices`:

[!code-csharp[](response/samples/2.x/ResponseCacheSample/Startup.cs?name=snippet1)]

Ссылки на модели страницы Cache4 примера приложения `Default30` кэша профиля:

[!code-csharp[](response/samples/2.x/ResponseCacheSample/Pages/Cache4.cshtml.cs?name=snippet)]

<xref:Microsoft.AspNetCore.Mvc.ResponseCacheAttribute> Применим к:

* Обработчики страницы Razor (классы) &ndash; атрибуты не могут применяться к методам обработчиков.
* Контроллеры MVC (классы).
* Действия MVC (методы) &ndash; атрибуты на уровне метода переопределяют параметры, указанные в атрибуты уровня класса.

Полученный заголовок, примененные к ответу Cache4 страницы с `Default30` кэша профиля:

```
Cache-Control: public,max-age=30
```

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Сохранения ответов в кэше](https://tools.ietf.org/html/rfc7234#section-3)
* [Cache-Control](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.9)
* <xref:performance/caching/memory>
* <xref:performance/caching/distributed>
* <xref:fundamentals/change-tokens>
* <xref:performance/caching/middleware>
* <xref:mvc/views/tag-helpers/builtin-th/cache-tag-helper>
* <xref:mvc/views/tag-helpers/builtin-th/distributed-cache-tag-helper>
