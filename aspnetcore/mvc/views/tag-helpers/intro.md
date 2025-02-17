---
title: Вспомогательные функции тегов в ASP.NET Core
author: rick-anderson
description: Сведения о вспомогательных функциях тегов и их использовании в ASP.NET Core.
ms.author: riande
ms.custom: H1Hack27Feb2017
ms.date: 03/18/2019
uid: mvc/views/tag-helpers/intro
ms.openlocfilehash: 11d2914b5797735fb6a262a31bdb49f58391579f
ms.sourcegitcommit: 5b0eca8c21550f95de3bb21096bd4fd4d9098026
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/27/2019
ms.locfileid: "64884059"
---
# <a name="tag-helpers-in-aspnet-core"></a>Вспомогательные функции тегов в ASP.NET Core

Автор: [Рик Андерсон](https://twitter.com/RickAndMSFT) (Rick Anderson)

## <a name="what-are-tag-helpers"></a>Что собой представляют вспомогательные функции тегов

Вспомогательные функции тегов позволяют серверному коду участвовать в создании и отрисовке HTML-элементов в файлах Razor. Например, встроенная функция `ImageTagHelper` может добавить номер версии к имени изображения. При каждом изменении изображения сервер генерирует новую уникальную версию изображения, поэтому клиенты гарантированно получат текущую версию изображения (вместо устаревшего кэшированного изображения). Существует множество встроенных вспомогательных функций тегов для общих задач — например, для создания форм, ссылок, загрузки ресурсов и т. д. Кроме того, огромное количество функций доступно в общедоступных репозиториях GitHub и в качестве пакетов NuGet. Вспомогательные функции тегов разрабатываются на C# и предназначены для HTML-элементов на основе имени элемента, имени атрибута или родительского тега. Например, встроенная функция `LabelTagHelper` может работать с HTML-элементом `<label>`, если применены атрибуты `LabelTagHelper`. Если вы знакомы со [вспомогательными методами HTML](http://stephenwalther.com/archive/2009/03/03/chapter-6-understanding-html-helpers), то поймете, что вспомогательные функции тегов сокращают количество явных переходов между HTML и C# в представлениях Razor. Во многих случаях вспомогательные методы HTML располагают альтернативными вариантами для определенной вспомогательной функции тега, но следует отметить, что вспомогательные функции тегов не заменяют вспомогательные методы HTML и для каждого вспомогательного метода HTML не существует конкретной вспомогательной функции тега. Более подробно различия описаны в разделе о [сравнении вспомогательных методов HTML и вспомогательных функций тегов](#tag-helpers-compared-to-html-helpers).

## <a name="what-tag-helpers-provide"></a>Возможности и преимущества вспомогательных функций тегов

**Большая схожесть с HTML**. В большинстве случаев разметка Razor, где используются вспомогательные функции тегов, выглядит как стандартный HTML. Разработчики интерфейсной части, работающие с HTML, CSS и JavaScript, могут редактировать Razor без изучения синтаксиса C# Razor.

**Полнофункциональная среда IntelliSense для создания разметки HTML и Razor**. Этим вспомогательные функции тегов существенно отличаются от вспомогательных методов HTML, более раннего подхода для создания серверной разметки в представлениях Razor. Более подробно различия описаны в разделе о [сравнении вспомогательных методов HTML и вспомогательных функций тегов](#tag-helpers-compared-to-html-helpers). Возможности среды IntelliSense объясняются в статье [Поддержка IntelliSense для вспомогательных функций тегов](#intellisense-support-for-tag-helpers). Даже разработчики с опытом в области синтаксиса Razor C# работают более продуктивно, используя вспомогательные функции тегов, нежели создавая разметку C# Razor.

**Вы сможете работать более продуктивно и создавать более надежный, устойчивый и безопасный код, используя информацию, которая доступна только на сервере**. Например, когда ранее требовалось изменить изображение, нужно было изменить имя изображения. Изображения нужно было обязательно кэшировать по причинам эффективности, и до тех пор, пока не менялось имя изображения, клиенты могли получать его устаревшую копию. Исторически после редактирования изображения нужно было обязательно менять его имя и обновлять каждую ссылку на него в веб приложении. Мало того, что это трудоемкий процесс, так он еще подвержен возникновению ошибок (можно пропустить ссылку, случайно ввести неправильную строку и т. д.). Встроенная функция `ImageTagHelper` может сделать это автоматически. Функция `ImageTagHelper` добавляет номер версии к имени изображения, поэтому при каждом изменении изображения сервер автоматически генерирует новую уникальную версию изображения. Клиенты гарантированно получают текущее изображение. Все это достигается с помощью функции `ImageTagHelper`.

Большинство встроенных вспомогательных функций тегов работают со стандартными HTML-элементами и предоставляют для них атрибуты на стороне сервера. Например, элемент `<input>`, используемый во многих представлениях в папке *Представления/учетная запись*, содержит атрибут `asp-for`. Этот атрибут извлекает имя свойства указанной модели в готовый для просмотра HTML-код. Рассмотрим представление Razor с помощью следующей модели:

```csharp
public class Movie
{
    public int ID { get; set; }
    public string Title { get; set; }
    public DateTime ReleaseDate { get; set; }
    public string Genre { get; set; }
    public decimal Price { get; set; }
}
```

Следующая разметка Razor:

```cshtml
<label asp-for="Movie.Title"></label>
```

Генерирует следующий HTML:

```html
<label for="Movie_Title">Title</label>
```

Атрибут `asp-for` предоставляется свойством `For` в [LabelTagHelper](/dotnet/api/microsoft.aspnetcore.mvc.taghelpers.labeltaghelper?view=aspnetcore-2.0). Дополнительные сведения см. в статье [Создание вспомогательных функций тегов](xref:mvc/views/tag-helpers/authoring).

## <a name="managing-tag-helper-scope"></a>Управление областью действия вспомогательной функции тега

Управление областью действия вспомогательных функций тегов осуществляется с помощью сочетания `@addTagHelper`, `@removeTagHelper` и символа отказа "!".

<a name="add-helper-label"></a>

### <a name="addtaghelper-makes-tag-helpers-available"></a>Директива `@addTagHelper` обеспечивает доступность вспомогательных функций тегов

При создании веб-приложения ASP.NET Core с именем *AuthoringTagHelpers* в проект будет добавлен следующий файл *Views/_ViewImports.cshtml*:

[!code-cshtml[](../../../mvc/views/tag-helpers/authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/Views/_ViewImportsCopy.cshtml?highlight=2&range=2-3)]

Директива `@addTagHelper` делает вспомогательные функции тегов доступными в представлении. В этом случае файлом представления является *Pages/_ViewImports.cshtml*, который по умолчанию наследуется всеми файлами в папке *Pages* и вложенными папками. Таким образом, вспомогательные функции тегов доступны для всех. В приведенном выше коде используется синтаксис знаков подстановки ("\*") для указания того, что все вспомогательные функции тегов в указанной сборке (*Microsoft.AspNetCore.Mvc.TagHelpers*) будут доступны для каждого файла представления в каталоге *Views* или вложенном каталоге. Первый параметр после директивы `@addTagHelper` указывает загружаемые вспомогательные функции тегов (мы используем "\*" для всех вспомогательных функций тегов), а второй параметр "Microsoft.AspNetCore.Mvc.TagHelpers" указывает сборку, содержащую вспомогательные функции тегов. *Microsoft.AspNetCore.Mvc.TagHelpers* — это сборка для встроенных вспомогательных функций тегов ASP.NET Core.

Чтобы использовать все вспомогательные функции тегов в этом проекте (где создается сборка *AuthoringTagHelpers*), нужно сделать следующее:

[!code-cshtml[](../../../mvc/views/tag-helpers/authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/Views/_ViewImportsCopy.cshtml?highlight=3)]

Если проект содержит `EmailTagHelper` с пространством имен по умолчанию (`AuthoringTagHelpers.TagHelpers.EmailTagHelper`), можно указать полное имя (FQN) вспомогательной функции тега:

```cshtml
@using AuthoringTagHelpers
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
@addTagHelper AuthoringTagHelpers.TagHelpers.EmailTagHelper, AuthoringTagHelpers
```

Чтобы добавить вспомогательную функцию тега в представление с помощью FQN, сначала добавьте FQN (`AuthoringTagHelpers.TagHelpers.EmailTagHelper`), а затем — имя сборки (*AuthoringTagHelpers*). Большинство разработчиков предпочитают использовать синтаксис знаков подстановки "\*". Синтаксис знаков подстановки позволяет вставлять знак подстановки "\*" в FQN в качестве суффикса. Например, любая из следующих директив добавит `EmailTagHelper`:

```cshtml
@addTagHelper AuthoringTagHelpers.TagHelpers.E*, AuthoringTagHelpers
@addTagHelper AuthoringTagHelpers.TagHelpers.Email*, AuthoringTagHelpers
```

Как упоминалось ранее, добавление директивы `@addTagHelper` в файл *Views/_ViewImports.cshtml* делает вспомогательную функцию тега доступной для всех файлов представлений в каталоге *Views* и вложенных каталогах. Директиву `@addTagHelper` можно использовать в конкретных файлах представлений, если вы хотите, чтобы вспомогательная функция тега находилась только в этих представлениях.

<a name="remove-razor-directives-label"></a>

### <a name="removetaghelper-removes-tag-helpers"></a>Директива `@removeTagHelper` удаляет вспомогательные функций тегов

У директивы `@removeTagHelper` есть те же два параметра, что и у директивы `@addTagHelper`, и она удаляет ранее добавленную вспомогательную функцию тега. Например, если применить директиву `@removeTagHelper` к определенному представлению, она удалит из него указанную вспомогательную функцию тега. Использование директивы `@removeTagHelper` в файле *Views/Folder/_ViewImports.cshtml* приводит к удалению указанной вспомогательной функции тега из всех представлений в *папке*.

### <a name="controlling-tag-helper-scope-with-the-viewimportscshtml-file"></a>Контроль области действия вспомогательной функции тега с помощью файла *_ViewImports.cshtml*

Можно добавить файл *_ViewImports.cshtml* в любую папку представления, и подсистема представлений применит директивы из этого файла и файла *Views/_ViewImports.cshtml*. Если вы добавили пустой файл *Views/Home/_ViewImports.cshtml* для представлений *Home*, ничего не изменится, поскольку файл *_ViewImports.cshtml* является дополнительным. Любая директива `@addTagHelper`, добавляемая в файл *Views/Home/_ViewImports.cshtml* (и не входящая в файл по умолчанию *Views/_ViewImports.cshtml*), будет предоставлять эти вспомогательные функции тегов для представлений только в папке *Home*.

<a name="opt-out"></a>

### <a name="opting-out-of-individual-elements"></a>Отказ от использования отдельных элементов

Можно отключить вспомогательную функцию тега на уровне элемента с помощью символа отказа от использования ("!"). Например, с помощью данного символа отключена проверка `Email` в `<span>`:

```cshtml
<!span asp-validation-for="Email" class="text-danger"></!span>
```

Символ отказа от использования вспомогательной функции тега необходимо применять в открывающем и закрывающем тегах. (Редактор Visual Studio автоматически добавляет символ отказа в закрывающий тег, если такой символ добавлен в открывающий тег.) После добавления символа отказа атрибуты элемента и вспомогательной функции больше не будут отображаться отдельным шрифтом.

<a name="prefix-razor-directives-label"></a>

### <a name="using-taghelperprefix-to-make-tag-helper-usage-explicit"></a>Применение `@tagHelperPrefix` для явного использования вспомогательной функции тега

Директива `@tagHelperPrefix` позволяет указать строку префикса тега для включения поддержки вспомогательной функции тега и явного использования этой функции. Например, можно добавить следующую разметку в файл *Views/_ViewImports.cshtml*:

```cshtml
@tagHelperPrefix th:
```

На приведенном ниже рисунке кода задан префикс `th:` вспомогательной функции тега, поэтому вспомогательную функцию тега поддерживают только элементы с префиксом `th:` (элементы с поддержкой вспомогательной функции тега выделены особым шрифтом). У элементов `<label>` и `<input>` есть префикс вспомогательной функции тега и поддержка этой функции, а у элемента `<span>` этого префикса и поддержки нет.

![изображение](intro/_static/thp.png)

Правила иерархии, которые применяются к `@addTagHelper`, также применяются и к `@tagHelperPrefix`.

## <a name="self-closing-tag-helpers"></a>Самозакрывающиеся вспомогательные функции тегов

Многие вспомогательные функции тегов нельзя использовать как самозакрывающиеся теги. Некоторые вспомогательные функции тегов созданы как самозакрывающиеся теги. При использовании вспомогательной функции тегов, которая не создана как самозакрывающаяся, подавляются выводимые данные. При самостоятельном закрывании вспомогательной функции тегов в выводимых данных появляется самозакрывающийся тег. Дополнительные сведения см. в [этом примечании](xref:mvc/views/tag-helpers/authoring#self-closing) в статье [Создание вспомогательных функций тегов](xref:mvc/views/tag-helpers/authoring).

## <a name="intellisense-support-for-tag-helpers"></a>Поддержка Intellisense для вспомогательных функций тегов

Веб-приложение ASP.NET Core, создаваемое в Visual Studio, добавляет пакет NuGet "Microsoft.AspNetCore.Razor.Tools". Это пакет добавляет инструментарий вспомогательной функции тега.

Рекомендуется написать элемент HTML `<label>`. Когда вы введете `<l` в редакторе Visual Studio, IntelliSense отобразит подходящие элементы:

![изображение](intro/_static/label.png)

Выводится не только справка по HTML, но и значок ("@" symbol with "<>" под ней).

![изображение](intro/_static/tagSym.png)

Он показывает элемент, на который нацелены вспомогательные функции тегов. Чистые элементы HTML (например, `fieldset`) отображают значок "<>".

Чистый тег HTML `<label>` отображается (с помощью базовой цветовой гаммы Visual Studio) коричневым цветом, атрибуты — красным, а значения атрибутов — синим.

![изображение](intro/_static/LableHtmlTag.png)

После того как вы введете `<label`, IntelliSense выведет перечень доступных атрибутов HTML и CSS и атрибутов для вспомогательной функции тега:

![изображение](intro/_static/labelattr.png)

Функция завершения операторов IntelliSense позволяет заполнить выражение выбранным значением с помощью клавиши TAB:

![изображение](intro/_static/stmtcomplete.png)

После ввода атрибута вспомогательной функции тега шрифты тега и атрибута изменяются. При использовании базовых цветовых схем Visual Studio "Синяя" или "Светлая" шрифт будет пурпурным с полужирным начертанием. Если используется тема "Темная", шрифт будет сине-зеленым с полужирным начертанием. Изображения в этом документе были созданы с помощью темы по умолчанию.

![изображение](intro/_static/labelaspfor2.png)

Вы можете использовать сочетание клавиш Visual Studio *CompleteWord* (CTRL+пробел [по умолчанию](/visualstudio/ide/default-keyboard-shortcuts-in-visual-studio) внутри двойных кавычек (""), чтобы переключиться на C# так же, как если бы вы находились в классе C#. IntelliSense отобразит все методы и свойства на модели страницы. Методы и свойства доступны, поскольку типом свойства является `ModelExpression`. На рисунке ниже выполняется редактирование представления `Register`, поэтому доступен `RegisterViewModel`.

![изображение](intro/_static/intellemail.png)

IntelliSense выводит свойства и методы, доступные для модели на странице. Благодаря широким возможностям IntelliSense помогает выбрать класс CSS:

![изображение](intro/_static/iclass.png)

![изображение](intro/_static/intel3.png)

## <a name="tag-helpers-compared-to-html-helpers"></a>Сравнение вспомогательных функций тегов со вспомогательными методами HTML

Вспомогательные функции тегов присоединяются к HTML-элементам в представлениях Razor, тогда как [вспомогательные методы HTML](http://stephenwalther.com/archive/2009/03/03/chapter-6-understanding-html-helpers) вызываются как методы, смешанные с HTML, в представлениях Razor. Рассмотрим следующую разметку Razor, которая создает метку HTML с классом CSS "caption":

```cshtml
@Html.Label("FirstName", "First Name:", new {@class="caption"})
```

Символ at (`@`) указывает Razor, что это начало кода. Следующие два параметра ("FirstName" и "First Name:") представляют собой строки, поэтому [IntelliSense](/visualstudio/ide/using-intellisense) помочь здесь не может. Последний аргумент:

```cshtml
new {@class="caption"}
```

является анонимным объектом, который используется для представления атрибутов. Так как `class` — это зарезервированное ключевое слово в C#, используется символ `@`, чтобы в C# интерпретировать `@class=` как символ (имя свойства). Для разработчиков интерфейсной части (тех, кто знаком с HTML, CSS, JavaScript и другими клиентскими технологиями, но не знаком с C# и Razor), большая часть этой строки непонятна. И вся строка должны быть написана без помощи IntelliSense.

С помощью функции `LabelTagHelper` та же разметка может быть написана следующим образом:

```cshtml
<label class="caption" asp-for="FirstName"></label>
```

При использовании вспомогательной функции тега, как только вы вводите `<l` в редакторе Visual Studio, IntelliSense отобразит подходящие элементы:

![изображение](intro/_static/label.png)

IntelliSense помогает написать всю строку.

На приведенном ниже изображении с кодом показана часть Form представления Razor *Views/Account/Register.cshtml*, которое создано с помощью шаблона MVC 4.5.x ASP.NET, входящего в состав Visual Studio.

![изображение](intro/_static/regCS.png)

Редактор Visual Studio отображает код C# на сером фоне. Например, вспомогательный метод HTML `AntiForgeryToken`:

```cshtml
@Html.AntiForgeryToken()
```

отображается на сером фоне. Большая часть разметки в представлении Register является кодом C#. Сравните это с эквивалентным подходом, где используются вспомогательные функции тегов:

![изображение](intro/_static/regTH.png)

Разметка здесь более чистая, ее проще читать, редактировать и поддерживать, чем при использовании вспомогательных методов HTML. Количество C# кода уменьшено до минимума — здесь есть только то, что необходимо знать серверу. Редактор Visual Studio отображает разметку, с которой работает вспомогательная функция тега, особым шрифтом.

Рассмотрим группу *Email*:

[!code-csharp[](intro/sample/Register.cshtml?range=12-18)]

У каждого из атрибутов "asp-" есть значение "Email", но "Email" не является строкой. В данном контексте "Email" — это свойство модельного выражения C# для `RegisterViewModel`.

Редактор Visual Studio помогает написать **всю** разметку, если вы используете вспомогательную функцию тега, а если вы применяете вспомогательные методы HTML, помощи Visual Studio для большей части кода можно не ожидать. Дополнительные сведения о работе со вспомогательными функциями тегов в редакторе Visual Studio см. в статье [Поддержка IntelliSense для вспомогательных функций тегов](#intellisense-support-for-tag-helpers).

## <a name="tag-helpers-compared-to-web-server-controls"></a>Сравнение вспомогательных функций тегов с серверными веб-элементами управления

* Вспомогательным функциям тегов не принадлежит элемент, с которыми они связаны. Они просто участвуют в отрисовке элемента и содержимого. [Серверные веб-элементы управления](https://msdn.microsoft.com/library/7698y1f0.aspx) ASP.NET объявляются и вызываются на странице.

* [Серверные веб-элементы управления](https://msdn.microsoft.com/library/zsyt68f1.aspx) имеют нестандартный жизненный цикл, что усложняет разработку и отладку.

* Серверные веб-элементы управления позволяют добавлять функционал в элементы DOM с помощью клиентского элемента управления. У вспомогательных функций тегов отсутствует DOM.

* Серверные веб-элементы управления включают в себя автоматическое обнаружение браузера. Вспомогательные функции тегов ничего не знают о браузере.

* Несколько вспомогательных функций тегов могут работать с одним и тем же элементом (см. статью о [предотвращении конфликтов вспомогательной функции тега](xref:mvc/views/tag-helpers/authoring#avoid-tag-helper-conflicts)), серверные веб-элементы управления обычно нельзя комбинировать.

* Вспомогательные функции тегов могут менять тег и содержимое HTML-элементов, с которыми они связаны, но напрямую они не влияют больше ни на какие элементы на странице. Серверные веб-элементы управления имеют менее конкретную область действия и могут выполнять действия, которые влияют на другие части страницы, и это иногда может привести к побочным эффектам.

* Серверные веб-элементы управления используют преобразователи типов для преобразования строк в объекты. Со вспомогательными функциями тегов вы работаете изначально в C#, поэтому преобразование типов не требуется.

* Серверные веб-элементы управления используют [System.ComponentModel](/dotnet/api/system.componentmodel), чтобы реализовать поведение компонентов и элементов управления во время разработки и выполнения. `System.ComponentModel` содержит базовые классы и интерфейсы для реализации атрибутов и преобразователей типов, привязки к источникам данных и лицензирования компонентов. Сравните это со вспомогательными функциями тегов, которые обычно являются производными от `TagHelper`, а базовый класс `TagHelper` предоставляет только два метода — `Process` и `ProcessAsync`.

## <a name="customizing-the-tag-helper-element-font"></a>Настройка шрифта элемента вспомогательной функции тега

Вы можете настроить шрифт и цвет, последовательно выбрав **Сервис** > **Параметры** > **Среда** > **Шрифты и цвета**:

![изображение](intro/_static/fontoptions2.png)

[!INCLUDE[](~/includes/built-in-TH.md)]

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Создание вспомогательных функций тегов](xref:mvc/views/tag-helpers/authoring)
* [Работа с формами](xref:mvc/views/working-with-forms)
* [TagHelperSamples на GitHub](https://github.com/dpaquette/TagHelperSamples) содержит примеры вспомогательной функции тега для работы с [Bootstrap](http://getbootstrap.com/).
