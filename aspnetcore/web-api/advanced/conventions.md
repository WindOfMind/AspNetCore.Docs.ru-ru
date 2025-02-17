---
title: Использование соглашений веб-API
author: pranavkm
description: Сведения о соглашениях веб-API в ASP.NET Core.
monikerRange: '>= aspnetcore-2.2'
ms.author: scaddie
ms.custom: mvc
ms.date: 12/13/2018
uid: web-api/advanced/conventions
ms.openlocfilehash: ae409158defe15ffaa7454039b302cf3c6437e54
ms.sourcegitcommit: 6d9cf728465cdb0de1037633a8b7df9a8989cccb
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 06/28/2019
ms.locfileid: "67463278"
---
# <a name="use-web-api-conventions"></a>Использование соглашений веб-API

Авторы: [Пранав Кришнамурти (Pranav Krishnamoorthy)](https://github.com/pranavkm) и [Скот Эдди (Scott Addie)](https://github.com/scottaddie)

В ASP.NET Core 2.2 и более поздних версий добавлен способ извлечения распространенной [документации по API](xref:tutorials/web-api-help-pages-using-swagger) и ее применения к нескольким действиям, контроллерам или всем контроллерам в рамках сборки. Соглашения веб-API заменяют применение атрибута [[ProducesResponseType]](xref:Microsoft.AspNetCore.Mvc.ProducesResponseTypeAttribute) к отдельным действиям.

Соглашение позволяет следующее.

* Определять наиболее распространенные типы возврата и коды состояния, возвращаемые из определенного типа действия.
* Выявлять действия, отличающиеся от определенного стандарта.

ASP.NET Core MVC 2.2 и более поздних версий включает в себя набор соглашений по умолчанию в <xref:Microsoft.AspNetCore.Mvc.DefaultApiConventions?displayProperty=fullName>. Соглашения основаны на контроллере (*ValuesController.cs*), предоставляемым в шаблоне проекта **API** ASP.NET Core. Если ваши действия соответствуют схеме в шаблоне, вы сможете успешно использовать соглашения по умолчанию. Если соглашения по умолчанию не соответствуют вашим потребностям, см. раздел [Создание соглашений веб-API](#create-web-api-conventions).

Во время выполнения <xref:Microsoft.AspNetCore.Mvc.ApiExplorer> понимает соглашения. `ApiExplorer` является абстракцией MVC для взаимодействия с генераторами документов [OpenAPI](https://www.openapis.org/) (также называется Swagger). Атрибуты из примененного соглашения связываются с действием и включаются в документацию OpenAPI для действия. [Анализаторы API](xref:web-api/advanced/analyzers) также понимают соглашения. Если ваше действие является нестандартным (например, возвращает код состояния, который не задокументирован примененным соглашением), выводится предупреждение, позволяющее задокументировать код состояния.

[Просмотреть или скачать образец кода](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/web-api/advanced/conventions/sample) ([как скачивать](xref:index#how-to-download-a-sample))

## <a name="apply-web-api-conventions"></a>Применение соглашений веб-API

Соглашения не являются составными, каждое действие может быть связано только с одним соглашением. Более конкретные соглашения имеют приоритет над менее определенными. Если к действию применяются два или более соглашения с одинаковым приоритетом, выбор осуществляется недетерминированным образом. Существуют следующие варианты применения соглашения к действию (от наиболее конкретного до наименее конкретного):

1. `Microsoft.AspNetCore.Mvc.ApiConventionMethodAttribute` &mdash; Применяется к отдельным действиям и указывает тип соглашения и применимый метод соглашения.

    В следующем примере метод соглашения `Microsoft.AspNetCore.Mvc.DefaultApiConventions.Put` типа соглашения по умолчанию применяется к действию `Update`:

    [!code-csharp[](conventions/sample/Controllers/ContactsConventionController.cs?name=snippet_ApiConventionMethod&highlight=3)]

    Метод соглашения `Microsoft.AspNetCore.Mvc.DefaultApiConventions.Put` применяет следующие атрибуты к действию:

    ```csharp
    [ProducesDefaultResponseType]
    [ProducesResponseType(StatusCodes.Status204NoContent)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    ```

    Дополнительные сведения о `[ProducesDefaultResponseType]` см. в описании [ответа по умолчанию](https://swagger.io/docs/specification/describing-responses/#default).

1. `Microsoft.AspNetCore.Mvc.ApiConventionTypeAttribute`, примененный к контроллеру &mdash;, применяет определенный тип соглашения ко всем действиям в контроллере. Метод соглашения снабжен указаниями, определяющими действия, к которым применяется метод соглашения. Дополнительные сведения об указаниях см. в разделе [Создание соглашений веб-API](#create-web-api-conventions).

    В следующем примере набор соглашений по умолчанию применяется ко всем действиям в *ContactsConventionController*:

    [!code-csharp[](conventions/sample/Controllers/ContactsConventionController.cs?name=snippet_ApiConventionTypeAttribute&highlight=2)]

1. `Microsoft.AspNetCore.Mvc.ApiConventionTypeAttribute`, примененный к сборке &mdash;, применяет определенный тип соглашения ко всем контроллерам в текущей сборке. Атрибуты уровня сборки рекомендуется применять в файле *Startup.cs*.

    В следующем примере набор соглашений по умолчанию применяется ко всем контроллерам в сборке:

    [!code-csharp[](conventions/sample/Startup.cs?name=snippet_ApiConventionTypeAttribute&highlight=1)]

## <a name="create-web-api-conventions"></a>Создание соглашений веб-API

Если соглашения API по умолчанию не соответствуют вашим потребностям, создайте собственные соглашения. Соглашение:

* Это статический тип с методами.
* Может определять [типы ответов](#response-types) и [требования к именованию](#naming-requirements) для действий.

### <a name="response-types"></a>Типы ответов

Эти методы помечаются атрибутами `[ProducesResponseType]` или `[ProducesDefaultResponseType]`. Например:

```csharp
public static class MyAppConventions
{
    [ProducesResponseType(StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public static void Find(int id)
    {
    }
}
```

Если отсутствуют более конкретные атрибуты метаданных, применение этого соглашения к сборке указывает, что:

* Метод соглашение применяется к любому действию с именем `Find`.
* Параметр с именем `id` присутствует для действия `Find`.

### <a name="naming-requirements"></a>Требования к именованию

Атрибуты `[ApiConventionNameMatch]` и `[ApiConventionTypeMatch]` можно применить к методу соглашения, определяющему действия, к которым они применяются. Например:

```csharp
[ProducesResponseType(StatusCodes.Status200OK)]
[ProducesResponseType(StatusCodes.Status404NotFound)]
[ApiConventionNameMatch(ApiConventionNameMatchBehavior.Prefix)]
public static void Find(
    [ApiConventionNameMatch(ApiConventionNameMatchBehavior.Suffix)]
    int id)
{ }
```

В предшествующем примере:

* Параметр `Microsoft.AspNetCore.Mvc.ApiExplorer.ApiConventionNameMatchBehavior.Prefix`, примененный к методу, указывает, что соглашение соответствует любому действию с префиксом Find. Примеры соответствующих действий: `Find`, `FindPet` и `FindById`.
* `Microsoft.AspNetCore.Mvc.ApiExplorer.ApiConventionNameMatchBehavior.Suffix`, примененный к параметру, указывает, что соглашение соответствует методам только с одним параметром, заканчивающимся на идентификатор суффикса. Это такие параметры, как `id` или `petId`. `ApiConventionTypeMatch` может аналогичным образом применяться к типам для ограничения типа параметра. Аргумент `params[]` указывает на остальные параметры, которым не требуется явное сопоставление.

## <a name="additional-resources"></a>Дополнительные ресурсы

* <xref:web-api/advanced/analyzers>
* <xref:tutorials/web-api-help-pages-using-swagger>
