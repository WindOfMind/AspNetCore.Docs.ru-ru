---
title: Добавление проверки на страницу Razor в ASP.NET Core
author: rick-anderson
description: Практическое руководство по добавлению проверки на страницу Razor в ASP.NET Core.
ms.author: riande
ms.custom: mvc
ms.date: 12/5/2018
uid: tutorials/razor-pages/validation
ms.openlocfilehash: 38e1fff9c7a212af992951dbf57e124cae69d36f
ms.sourcegitcommit: ccbb84ae307a5bc527441d3d509c20b5c1edde05
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 05/19/2019
ms.locfileid: "65874991"
---
# <a name="add-validation-to-an-aspnet-core-razor-page"></a>Добавление проверки на страницу Razor в ASP.NET Core

Автор: [Рик Андерсон](https://twitter.com/RickAndMSFT) (Rick Anderson)

В этом разделе к модели `Movie` добавляется логика проверки. Правила проверки применяются каждый раз, когда пользователь создает или редактирует фильм.

## <a name="validation"></a>Проверка

Ключевой принцип разработки программного обеспечения называется [DRY](https://wikipedia.org/wiki/Don%27t_repeat_yourself) (от английского "**D**on't **R**epeat **Y**ourself" — не повторяйся). При разработке страниц Razor Pages рекомендуется задавать любые функциональные возможности лишь один раз и затем при необходимости отражать их в рамках всего приложения. Принцип "Не повторяйся" может помочь:

* сократить объем кода в приложении;
* снизить вероятность возникновения ошибки в коде и упростить его тестирование и поддержку.

Ярким примером применения принципа "Не повторяйся" является поддержка проверки, реализуемая на страницах Razor и на платформе Entity Framework. Правила проверки декларативно определяются в одном месте (в классе модели) и затем применяются в рамках всего приложения.

[!INCLUDE[](~/includes/RP-MVC/validation.md)]

### <a name="validation-error-ui-in-razor-pages"></a>Пользовательский интерфейс проверки ошибок на страницах Razor

Запустите приложение и перейдите в раздел "Pages/Movies" (Страницы/фильмы).

Щелкните ссылку **Create New** (Создать). Введите в форму какие-либо недопустимые значения. Если функция проверки jQuery на стороне клиента обнаруживает ошибку, сведения о ней отображаются в соответствующем сообщении.

![Форма просмотра фильма с несколькими ошибками проверки jQuery на стороне клиента](validation/_static/val.png)

[!INCLUDE[](~/includes/currency.md)]

Обратите внимание, что для каждого поля, содержащего недопустимое значение, в форме автоматически отображается сообщение об ошибке проверки. Эти ошибки применяются как на стороне клиента (с помощью JavaScript и jQuery), так и на стороне сервера (если пользователь отключает JavaScript).

Основным преимуществом является то, что на страницах создания или редактирования **не требуется** изменять код. После применения класса DataAnnotations к модели активируется пользовательский интерфейс проверки. На страницах Razor, создаваемых в рамках этого руководства, автоматически применяются правила проверки (для этого к свойствам класса модели `Movie` применяются атрибуты). При проверке страницы редактирования применяются те же правила.

Данные формы передаются на сервер только после того, как будут устранены все ошибки проверки на стороне клиента. Чтобы убедиться, что данные формы не отправляются, используйте любой из следующих способов.

* Поместите точку останова в метод `OnPostAsync`. Отправьте форму с помощью команды **Create** (Создать) или **Save** (Сохранить). Точка останова не достигается ни при каких обстоятельствах.
* Используйте [инструмент Fiddler](http://www.telerik.com/fiddler).
* Проследите сетевой трафик с помощью инструментов разработчика для браузера.

### <a name="server-side-validation"></a>Проверка на стороне сервера

Если в браузере отключен JavaScript, форма с ошибками отправляется на сервер.

Реализация проверки на стороне сервера:

* Отключите JavaScript в браузере. Это можно сделать с помощью средств разработчика в браузере. Если сделать это не удается, попробуйте использовать другой браузер.
* Поместите точку останова в метод `OnPostAsync` страниц создания или редактирования.
* Отправьте форму с ошибками проверки.
* Проверка недопустимого состояния модели:

  ```csharp
   if (!ModelState.IsValid)
   {
      return Page();
   }
  ```

В следующем коде демонстрируется часть страницы *Create.cshtml*, сформированной ранее в рамках этого руководства. Она используется на страницах создания и редактирования для отображения исходной формы и повторного вывода формы в случае ошибки.

[!code-cshtml[](razor-pages-start/sample/RazorPagesMovie/Pages/Movies/Create.cshtml?range=14-20)]

[Вспомогательная функция тега Input](xref:mvc/views/working-with-forms) использует атрибуты [DataAnnotations](/aspnet/mvc/overview/older-versions/mvc-music-store/mvc-music-store-part-6) и создает HTML-атрибуты, необходимые для проверки jQuery на стороне клиента. [Вспомогательная функция тега Validation](xref:mvc/views/working-with-forms#the-validation-tag-helpers) отображает ошибки проверки. Дополнительные сведения см. в разделе [Проверка](xref:mvc/models/validation).

На страницах создания и редактирования не определены правила проверки. Правила проверки и строки ошибок указываются только в классе `Movie`. Они автоматически применяются к страницам Razor, которые редактируют модель `Movie`.

Любые необходимые изменения логики проверки осуществляются исключительно в модели. Проверка применяется согласованно на уровне всего приложения, для чего логика проверки определяется в одном месте. Такой подход позволяет максимально оптимизировать код и упростить его поддержку и обновление.

## <a name="using-datatype-attributes"></a>Использование атрибутов DataType

Проверьте класс `Movie`. В пространстве имен `System.ComponentModel.DataAnnotations` в дополнение к набору встроенных атрибутов проверки предоставляются атрибуты форматирования. Атрибут `DataType` применяется к свойствам `ReleaseDate` и `Price`.

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie/Models/MovieDateRatingDA.cs?highlight=2,6&name=snippet2)]

Атрибуты `DataType` предоставляют модулю просмотра только рекомендации по форматированию данных, а также другие атрибуты, например `<a>` для URL-адресов и `<a href="mailto:EmailAddress.com">` для электронной почты. Используйте атрибут `RegularExpression` для проверки формата данных. Атрибут `DataType` позволяет указать тип данных с более точным определением по сравнению со встроенным типом базы данных. Атрибуты `DataType` не предназначены для проверки. В том же приложении отображается только дата (без времени).

В перечислении `DataType` представлено множество типов данных, таких как Date, Time, PhoneNumber, Currency, EmailAddress и других. Атрибут `DataType` также обеспечивает автоматическое предоставление функций для определенных типов в приложении. Например, можно создавать ссылку `mailto:` для `DataType.EmailAddress`. Для `DataType.Date` в браузерах с поддержкой HTML5 можно предоставить селектор даты. Атрибут `DataType` создает атрибуты HTML 5 `data-`, которые используются браузерами с поддержкой HTML 5. Атрибуты `DataType` **не предназначены** для проверки.

`DataType.Date` не задает формат отображаемой даты. По умолчанию поле данных отображается с использованием форматов, установленных в параметрах `CultureInfo` сервера.

Требуются заметки к данным `[Column(TypeName = "decimal(18, 2)")]`, чтобы Entity Framework Core корректно сопоставила `Price` с валютой в базе данных. Дополнительные сведения см. в разделе [Типы данных](/ef/core/modeling/relational/data-types).

С помощью атрибута `DisplayFormat` можно явно указать формат даты:

```csharp
[DisplayFormat(DataFormatString = "{0:yyyy-MM-dd}", ApplyFormatInEditMode = true)]
public DateTime ReleaseDate { get; set; }
```

Параметр `ApplyFormatInEditMode` указывает, что формат должен применяться при отображении значения для редактирования. Для некоторых полей такое поведение нежелательно. Например, в полях валюты в пользовательском интерфейсе редактирования использовать символ денежной единицы, как правило, не требуется.

Атрибут `DisplayFormat` может использоваться отдельно, однако чаще всего его рекомендуется применять вместе с атрибутом `DataType`. Атрибут `DataType` передает семантику данных (в отличие от способа их вывода на экран) и дает следующие преимущества по сравнению с атрибутом DisplayFormat:

* Поддержка функций HTML5 в браузере (отображение элемента управления календарем, соответствующего языковому стандарту символа валюты, ссылок электронной почты и т. д.)
* По умолчанию формат отображения данных в браузере определяется в соответствии с установленным языковым стандартом.
* Атрибут `DataType` обеспечивает поддержку платформы ASP.NET Core для выбора соответствующего шаблона поля, применяемого при отображении данных. При отдельном использовании атрибут `DisplayFormat` базируется на строковом шаблоне.

Примечание. Проверка jQuery не работает с атрибутом `Range` и `DateTime`. Например, следующий код всегда приводит к возникновению ошибки проверки на стороне клиента, даже если дата попадает в указанный диапазон:

```csharp
[Range(typeof(DateTime), "1/1/1966", "1/1/2020")]
   ```

Как правило, не рекомендуется компилировать модели с фиксированными датами, поэтому использовать атрибуты `Range` и `DateTime` следует крайне осторожно.

В следующем коде демонстрируется объединение атрибутов в одной строке:

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Models/MovieDateRatingDAmult.cs?name=snippet1)]

Дополнительные операции EF Core с Razor Pages см. в статье [Начало работы с Razor Pages и EF Core](xref:data/ef-rp/intro).

### <a name="publish-to-azure"></a>Публикация в Azure

Сведения о развертывании в Azure, см. в разделе [Учебник: Создание приложения ASP.NET в Azure с подключением к базе данных SQL](/azure/app-service/app-service-web-tutorial-dotnet-sqldatabase). Эти инструкции приведены для приложения ASP.NET, а не ASP.NET Core, но шаги совпадают.

Благодарим вас за изучение общих сведений о страницах Razor. Отличным дополнением к этому руководству является руководство по [началу работы с Razor Pages и EF Core](xref:data/ef-rp/intro).

## <a name="additional-resources"></a>Дополнительные ресурсы

* <xref:mvc/views/working-with-forms>
* <xref:fundamentals/localization>
* <xref:mvc/views/tag-helpers/intro>
* <xref:mvc/views/tag-helpers/authoring>
* [Версия руководства на YouTube](https://youtu.be/b63m66eu7us)

> [!div class="step-by-step"]
> [Предыдущая статья. Добавление нового поля](xref:tutorials/razor-pages/new-field)
