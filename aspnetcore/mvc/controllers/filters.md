---
title: Фильтры в ASP.NET Core
author: ardalis
description: Из этой статьи вы узнаете, как работают фильтры и как их использовать в ASP.NET Core.
ms.author: riande
ms.custom: mvc
ms.date: 5/08/2019
uid: mvc/controllers/filters
ms.openlocfilehash: cdf121b97396cb23103d49cd141b9ef19b8c0cc6
ms.sourcegitcommit: e1623d8279b27ff83d8ad67a1e7ef439259decdf
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 05/25/2019
ms.locfileid: "66223024"
---
# <a name="filters-in-aspnet-core"></a>Фильтры в ASP.NET Core

Авторы: [Кирк Ларкин](https://github.com/serpent5) (Kirk Larkin) [Рик Андерсон](https://twitter.com/RickAndMSFT) (Rick Anderson), [Том Дайкстра](https://github.com/tdykstra/) (Tom Dykstra) и [Стив Смит](https://ardalis.com/) (Steve Smith)

*Фильтры* в ASP.NET Core позволяют выполнять код до или после определенных этапов в конвейере обработки запросов.

Встроенные фильтры обрабатывают следующие задачи:

* Авторизация (предотвращение несанкционированного доступа к ресурсам).
* Кэширование откликов (замыкание конвейера обработки запросов для возврата кэшированного ответа).

Для обработки сквозной функциональности можно создавать настраиваемые фильтры. Примеры сквозной функциональности включают обработку ошибок, кэширование, конфигурирование, авторизацию и ведение журнала.  Фильтры предотвращают дублирование кода. Например, можно объединить обработку ошибок с помощью фильтра исключений обработки ошибок.

Этот документ применим к Razor Pages, контроллерам API и контроллерам с представлениями.

[Просмотреть или скачать пример](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/controllers/filters/sample) ([как скачивать](xref:index#how-to-download-a-sample)).

## <a name="how-filters-work"></a>Как работают фильтры

Фильтры выполняются в *конвейере вызова действий ASP.NET Core*, который иногда называют *конвейером фильтров*.  Конвейер фильтров запускается после того, как платформа ASP.NET Core выбирает выполняемое действие.

![Запрос обрабатывается с использованием прочего ПО промежуточного слоя, ПО промежуточного слоя маршрутизации, выбора действия и конвейера вызова действий ASP.NET Core. Обработка запроса продолжается в обратном порядке посредством выбора действия, ПО промежуточного слоя маршрутизации и прочего ПО промежуточного слоя, пока не будет получен запрос, отправляемый клиенту.](filters/_static/filter-pipeline-1.png)

### <a name="filter-types"></a>Типы фильтров

Фильтр каждого типа выполняется на определенном этапе конвейера фильтров:

* [Фильтры авторизации](#authorization-filters). Применяются в первую очередь и служат для определения того, авторизован ли пользователь для выполнения запроса. Эти фильтры не могут сократить выполнение конвейера, если запрос не авторизован.

* [Фильтры ресурсов](#resource-filters):

  * Выполняются после авторизации.  
  * <xref:Microsoft.AspNetCore.Mvc.Filters.IResourceFilter.OnResourceExecuting*> может выполнять код до остальной части конвейера фильтров. Например, `OnResourceExecuting` может выполнять код до привязки модели.
  * <xref:Microsoft.AspNetCore.Mvc.Filters.IResourceFilter.OnResourceExecuted*> может выполнять код после завершения остальной части конвейера.

* [Фильтры действий](#action-filters) могут выполнять код непосредственно до и после вызова отдельного метода действия. С их помощью можно управлять аргументами, передаваемыми в действие, и возвращаемым из него результатом. Фильтры действий **не** поддерживаются в Razor Pages.

* [Фильтры исключений](#exception-filters) служат для применения глобальных политик к необработанным исключениям, которые происходят до записи данных в тело ответа.

* [Фильтры результатов](#result-filters) могут выполнять код непосредственно до и после выполнения результатов отдельного действия. Они выполняются только в том случае, если метод действия выполнен успешно. Они полезны для логики, которая должна включать исключения для представлений или модуля форматирования.

На приведенной ниже схеме показано, как типы фильтров взаимодействуют друг с другом в конвейере фильтров.

![Запрос обрабатывается посредством фильтров авторизации, фильтров ресурсов, привязки модели, фильтров действий, выполнения действия и преобразования результата действия, фильтров исключений, фильтров результатов и выполнения результатов. На обратном пути запрос обрабатывается только фильтрами результатов и фильтрами ресурсов, прежде чем стать ответом, отправляемым клиенту.](filters/_static/filter-pipeline-2.png)

## <a name="implementation"></a>Реализация

Фильтры поддерживают как синхронные, так и асинхронные реализации посредством различных определений интерфейсов.

Синхронные фильтры могут выполнять код до (`On-Stage-Executing`) и после (`On-Stage-Executed`) этапа конвейера. Например, `OnActionExecuting` вызывается перед вызовом метода действия, а `OnActionExecuted` — после возврата метода действия.

[!code-csharp[](./filters/sample/FiltersSample/Filters/MySampleActionFilter.cs?name=snippet_ActionFilter)]

Асинхронные фильтры определяют метод `On-Stage-ExecutionAsync`:

[!code-csharp[](./filters/sample/FiltersSample/Filters/SampleAsyncActionFilter.cs?name=snippet)]

В приведенном выше коде `SampleAsyncActionFilter` включает <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutionDelegate> (`next`) для выполнения метода действия.  Каждый из методов `On-Stage-ExecutionAsync` принимает `FilterType-ExecutionDelegate` для выполнения этапа конвейера фильтра.

### <a name="multiple-filter-stages"></a>Несколько этапов фильтра

Реализовать интерфейсы для нескольких этапов фильтра можно в одном классе. Например, класс <xref:Microsoft.AspNetCore.Mvc.Filters.ActionFilterAttribute> реализует интерфейсы `IActionFilter` и `IResultFilter`, а также их асинхронные эквиваленты.

Реализуйте синхронный **или** асинхронный интерфейс фильтра, но **не** оба варианта. Среда выполнения сначала проверяет, реализует ли фильтр асинхронный интерфейс. Если да, вызывается именно он. В противном случае вызываются методы синхронного интерфейса. Если асинхронный и синхронный интерфейсы реализованы в одном классе, вызывается только асинхронный метод. При использовании абстрактных классов, таких как <xref:Microsoft.AspNetCore.Mvc.Filters.ActionFilterAttribute>, следует переопределять только синхронные методы или асинхронный метод каждого типа фильтра.

### <a name="built-in-filter-attributes"></a>Встроенные атрибуты фильтров

ASP.NET Core включает встроенные фильтры на основе атрибутов, с помощью которых можно создавать подклассы и которые можно настраивать. Например, следующий фильтр результатов добавляет заголовок к ответу:

<a name="add-header-attribute"></a>

[!code-csharp[](./filters/sample/FiltersSample/Filters/AddHeaderAttribute.cs?name=snippet)]

Атрибуты позволяют фильтрам принимать аргументы, как показано в примере выше. Примените `AddHeaderAttribute` к методу контроллера или действия, а затем укажите имя и значение заголовка HTTP:

[!code-csharp[](./filters/sample/FiltersSample/Controllers/SampleController.cs?name=snippet_AddHeader&highlight=1)]

<!-- `https://localhost:5001/Sample` -->

Некоторые интерфейсы фильтров имеют соответствующие атрибуты, которые можно использовать как базовые классы для пользовательских реализаций.

Атрибуты фильтров:

* <xref:Microsoft.AspNetCore.Mvc.Filters.ActionFilterAttribute>
* <xref:Microsoft.AspNetCore.Mvc.Filters.ExceptionFilterAttribute>
* <xref:Microsoft.AspNetCore.Mvc.Filters.ResultFilterAttribute>
* <xref:Microsoft.AspNetCore.Mvc.FormatFilterAttribute>
* <xref:Microsoft.AspNetCore.Mvc.ServiceFilterAttribute>
* <xref:Microsoft.AspNetCore.Mvc.TypeFilterAttribute>

## <a name="filter-scopes-and-order-of-execution"></a>Области и порядок выполнения фильтров

Фильтр можно добавить в конвейер в одной из трех *областей*:

* С помощью атрибута в действии.
* С помощью атрибута в контроллере.
* Глобально для всех контроллеров и действий, как показано в следующем коде:

[!code-csharp[](./filters/sample/FiltersSample/StartupGF.cs?name=snippet_ConfigureServices)]

Предыдущий код глобально добавляет три фильтра с помощью коллекции [MvcOptions.Filters](xref:Microsoft.AspNetCore.Mvc.MvcOptions.Filters).

### <a name="default-order-of-execution"></a>Порядок выполнения по умолчанию

Если для определенного этапа конвейера имеется несколько фильтров, область определяет порядок их выполнения по умолчанию.  Глобальные фильтры заключают в себя фильтры классов, которые, в свою очередь, заключают в себя фильтры методов.

В результате такого вложения *последующий* код фильтров выполняется в порядке, обратном выполнению *предшествующего* кода. Последовательность фильтров:

* *Предшествующий* код глобальных фильтров.
  * *Предшествующий* код фильтров контроллера.
    * *Предшествующий* код фильтров методов действий.
    * *Последующий* код фильтров методов действий.
  * *Последующий* код фильтров контроллера.
* *Последующий* код глобальных фильтров.
  
В следующем примере показан порядок вызова методов фильтров для синхронных фильтров действий.

| Sequence | Область фильтра | Метод фильтра |
|:--------:|:------------:|:-------------:|
| 1 | Global | `OnActionExecuting` |
| 2 | Контроллер | `OnActionExecuting` |
| 3 | Метод | `OnActionExecuting` |
| 4 | Метод | `OnActionExecuted` |
| 5 | Контроллер | `OnActionExecuted` |
| 6 | Global | `OnActionExecuted` |

Эта последовательность показывает:

* Метод фильтра вкладывается в фильтр контроллера.
* Фильтр контроллера вкладывается в глобальный фильтр.

### <a name="controller-and-razor-page-level-filters"></a>Фильтры уровня контроллера и страницы Razor

Каждый контроллер, наследуемый от базового класса <xref:Microsoft.AspNetCore.Mvc.Controller> включает методы [Controller.OnActionExecuting](xref:Microsoft.AspNetCore.Mvc.Controller.OnActionExecuting*), [Controller.OnActionExecutionAsync](xref:Microsoft.AspNetCore.Mvc.Controller.OnActionExecutionAsync*) и [Controller.OnActionExecuted](xref:Microsoft.AspNetCore.Mvc.Controller.OnActionExecuted*)
`OnActionExecuted`. Эти методы:

* Заключают фильтры, которые выполняются для указанного действия.
* `OnActionExecuting` вызывается перед всеми фильтрами действий.
* `OnActionExecuted` вызывается после всех фильтров действий.
* `OnActionExecutionAsync` вызывается перед всеми фильтрами действий. Код в фильтре после `next` выполняется после метода действия.

Например, в скачиваемом примере `MySampleActionFilter` применяется глобально при запуске.

`TestController`:

* Применяет `SampleActionFilterAttribute` (`[SampleActionFilter]`) для действия `FilterTest2`:
* Переопределяет `OnActionExecuting` и `OnActionExecuted`.

[!code-csharp[](./filters/sample/FiltersSample/Controllers/TestController.cs?name=snippet)]

Переходит к `https://localhost:5001/Test/FilterTest2` для выполнения следующего кода:

* `TestController.OnActionExecuting`
  * `MySampleActionFilter.OnActionExecuting`
    * `SampleActionFilterAttribute.OnActionExecuting`
      * `TestController.FilterTest2`
    * `SampleActionFilterAttribute.OnActionExecuted`
  * `MySampleActionFilter.OnActionExecuted`
* `TestController.OnActionExecuted`

См. подробнее о [реализации фильтров страницы Razor путем переопределения методов фильтра](xref:razor-pages/filter#implement-razor-page-filters-by-overriding-filter-methods).

### <a name="overriding-the-default-order"></a>Переопределение порядка по умолчанию

Чтобы переопределить порядок выполнения по умолчанию, можно реализовать <xref:Microsoft.AspNetCore.Mvc.Filters.IOrderedFilter>. `IOrderedFilter` предоставляет свойство <xref:Microsoft.AspNetCore.Mvc.Filters.IOrderedFilter.Order>, которое имеет приоритет над областью и определяет порядок выполнения. Фильтр со значением меньше `Order`:

* Выполняется *перед* кодом, выполняемым до фильтра с более высоким значением `Order`.
* Выполняется *после* кода, выполняемого после фильтра с более высоким значением `Order`.

Свойство `Order` можно задать с помощью параметра конструктора:

```csharp
[MyFilter(Name = "Controller Level Attribute", Order=1)]
```

Рассмотрим три фильтра действий, показанные в предыдущем примере. Если свойство `Order` контроллера и глобальные фильтры имеют значения 1 и 2 соответственно, порядок выполнения инвертируется.

| Sequence | Область фильтра | Свойство`Order`  | Метод фильтра |
|:--------:|:------------:|:-----------------:|:-------------:|
| 1 | Метод | 0 | `OnActionExecuting` |
| 2 | Контроллер | 1  | `OnActionExecuting` |
| 3 | Global | 2  | `OnActionExecuting` |
| 4 | Global | 2  | `OnActionExecuted` |
| 5 | Контроллер | 1  | `OnActionExecuted` |
| 6 | Метод | 0  | `OnActionExecuted` |

Свойство `Order` переопределяет область при определении порядка выполнения фильтров. Фильтры сначала сортируются по порядку, а затем очередность окончательно определяется по области. Все встроенные фильтры реализуют `IOrderedFilter` и задают значение по умолчанию `Order`, равное 0. Во встроенных фильтрах область определяет порядок, если для `Order` не задано ненулевое значение.

## <a name="cancellation-and-short-circuiting"></a>Отмена и сокращенное выполнение

Вы можете настроить сокращенное выполнение конвейера фильтров, задав свойство <xref:Microsoft.AspNetCore.Mvc.Filters.ResourceExecutingContext.Result> для параметра <xref:Microsoft.AspNetCore.Mvc.Filters.ResourceExecutingContext>, передаваемого в метод фильтра. Например, приведенный ниже фильтр ресурсов предотвращает выполнение остальной части конвейера:

<a name="short-circuiting-resource-filter"></a>

[!code-csharp[](./filters/sample/FiltersSample/Filters/ShortCircuitingResourceFilterAttribute.cs?name=snippet)]

В приведенном ниже коде как фильтр `ShortCircuitingResourceFilter`, так и фильтр `AddHeader` нацелены на метод действия `SomeResource`. `ShortCircuitingResourceFilter`:

* Выполняется первым, поскольку это фильтр ресурсов, а `AddHeader` — фильтр действий.
* Замыкает оставшуюся часть конвейера.

Поэтому фильтр `AddHeader` никогда не выполняется для действия `SomeResource`. Поведение будет аналогичным при применении обоих фильтров на уровне метода действия при условии, что фильтр `ShortCircuitingResourceFilter` выполняется первым. Фильтр `ShortCircuitingResourceFilter` выполняется в первую очередь из-за его типа или в связи с явным указанием свойства `Order`.

[!code-csharp[](./filters/sample/FiltersSample/Controllers/SampleController.cs?name=snippet_AddHeader&highlight=1,9)]

## <a name="dependency-injection"></a>Внедрение зависимостей

Фильтры можно добавлять по типу или экземпляру. Добавляемый экземпляр используется для каждого запроса. Если добавляется тип, он является активированным. Активация фильтра означает, что:

* Экземпляр создается для каждого запроса.
* Зависимости конструктора заполняются путем [внедрения зависимостей](xref:fundamentals/dependency-injection).

Фильтры, которые реализуются как атрибуты и добавляются непосредственно в классы контроллеров или методы действий, не могут иметь зависимости конструктора, предоставленные посредством [внедрения зависимостей](xref:fundamentals/dependency-injection). Зависимости конструктора не могут предоставляться путем внедрения зависимостей, так как:

* Параметры конструктора должны предоставляться для атрибутов в месте их применения. 
* Это ограничение, налагаемое на использование атрибутов.

Следующие фильтры поддерживают зависимости конструктора, предоставленные путем внедрения зависимостей:

* <xref:Microsoft.AspNetCore.Mvc.ServiceFilterAttribute>
* <xref:Microsoft.AspNetCore.Mvc.TypeFilterAttribute>
* <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory> реализуется в атрибуте.

Предыдущие фильтры могут применяться к контроллеру или методу действия.

Средства ведения журнала доступны путем внедрения зависимостей. Но следует избегать создания и использования фильтров исключительно для целей ведения журнала. [Встроенное средство ведения журнала](xref:fundamentals/logging/index) обычно предоставляет все необходимое для ведения журнала. При добавлении ведения журналов в фильтры следует учитывать следующее:

* Следует сосредоточиться на бизнес-среде или поведении в привязке к фильтру.
* **Не** следует регистрировать действия или другие события платформы, так как это делают встроенные фильтры.

### <a name="servicefilterattribute"></a>ServiceFilterAttribute

Типы реализации фильтра службы регистрируются в `ConfigureServices`. Атрибут <xref:Microsoft.AspNetCore.Mvc.ServiceFilterAttribute> извлекает экземпляр фильтра из внедрения зависимостей.

В следующем коде используется `AddHeaderResultServiceFilter`:

[!code-csharp[](./filters/sample/FiltersSample/Filters/LoggingAddHeaderFilter.cs?name=snippet_ResultFilter)]

В следующем коде в контейнер внедрения зависимостей добавляется `AddHeaderResultServiceFilter`:

[!code-csharp[](./filters/sample/FiltersSample/Startup.cs?name=snippet_ConfigureServices&highlight=4)]

В следующем коде атрибут `ServiceFilter` извлекает экземпляр фильтра `AddHeaderResultServiceFilter` из внедрения зависимостей:

[!code-csharp[](./filters/sample/FiltersSample/Controllers/HomeController.cs?name=snippet_ServiceFilter&highlight=1)]

[ServiceFilterAttribute.IsReusable](xref:Microsoft.AspNetCore.Mvc.ServiceFilterAttribute.IsReusable):

* Указывает, что экземпляр фильтра *можно* многократно использовать за пределами области запроса, в которой он был создан. Среда выполнения ASP.NET Core не гарантирует:

  * что будет создан хотя бы один экземпляр фильтра;
  * что фильтр не будет повторно запрошен из контейнера внедрения зависимостей на более позднем этапе.

* Не следует использовать с фильтром, который зависит от служб со временем существования, кроме элементов singleton.

 Объект <xref:Microsoft.AspNetCore.Mvc.ServiceFilterAttribute> реализует интерфейс <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory>. `IFilterFactory` предоставляет метод <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory.CreateInstance*> для создания экземпляра <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterMetadata>. `CreateInstance` загружает указанный тип из внедрения зависимостей.

### <a name="typefilterattribute"></a>TypeFilterAttribute

Атрибут <xref:Microsoft.AspNetCore.Mvc.TypeFilterAttribute> похож на <xref:Microsoft.AspNetCore.Mvc.ServiceFilterAttribute>, но его тип не разрешается напрямую из контейнера внедрения зависимостей. Он создает экземпляр типа с помощью <xref:Microsoft.Extensions.DependencyInjection.ObjectFactory?displayProperty=fullName>.

Так как типы `TypeFilterAttribute` не разрешаются напрямую из контейнера внедрения зависимостей:

* Типы, на которые ссылаются с помощью `TypeFilterAttribute`, не нужно регистрировать в контейнере внедрения зависимостей.  Их зависимости выполняются самим контейнером.
* Атрибут `TypeFilterAttribute` может также принимать аргументы конструктора для типа.

При использовании `TypeFilterAttribute` задание `IsReusable` является указанием, что экземпляр фильтра *можно* многократно использовать за пределами области запроса, в которой он был создан. Среда выполнения ASP.NET Core не предоставляет никаких гарантий, что будет создан хотя бы один экземпляр фильтра. `IsReusable` не следует использовать с фильтром, который зависит от служб со временем существования, кроме элементов singleton.

В следующем примере показано, как передавать аргументы в тип с помощью `TypeFilterAttribute`:

[!code-csharp[](../../mvc/controllers/filters/sample/FiltersSample/Controllers/HomeController.cs?name=snippet_TypeFilter&highlight=1,2)]
[!code-csharp[](../../mvc/controllers/filters/sample/FiltersSample/Filters/LogConstantFilter.cs?name=snippet_TypeFilter_Implementation&highlight=6)]

<!-- 
https://localhost:5001/home/hi?name=joe
VS debug window shows 
FiltersSample.Filters.LogConstantFilter:Information: Method 'Hi' called
-->

## <a name="authorization-filters"></a>Фильтры авторизации

Фильтры авторизации:

* Выполняются в первую очередь в конвейере фильтров.
* Контролируют доступ к методам действий.
* Имеют предшествующий, но не последующий метод.

Для работы пользовательских фильтров авторизации требуется настраиваемая платформа авторизации. Настройка политик авторизации или определение пользовательской политики авторизации предпочтительнее создания пользовательского фильтра. Встроенный фильтр авторизации:

* Вызывает систему авторизации.
* Не выполняет авторизацию запросов.

**Не** вызывайте исключения в фильтрах авторизации:

* Исключение не будет обработано.
* Фильтры исключений не будут обрабатывать исключение.

При возникновении исключения в фильтре авторизации попробуйте создать запрос.

Дополнительные сведения об [авторизации](xref:security/authorization/introduction).

## <a name="resource-filters"></a>Фильтры ресурсов

Фильтры ресурсов:

* Реализуют либо интерфейс <xref:Microsoft.AspNetCore.Mvc.Filters.IResourceFilter>, либо интерфейс <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncResourceFilter>.
* Выполнение охватывает большую часть конвейера фильтров.
* До фильтров ресурсов применяются только [фильтры авторизации](#authorization-filters).

Фильтры ресурсов полезны для сокращенного выполнения большей части конвейера. Например, фильтр кэширования предотвращает выполнение остальной части конвейера при попадании в кэше.

Примеры фильтров ресурсов:

* [Сокращенное выполнение фильтра ресурсов](#short-circuiting-resource-filter) показано ранее.
* [DisableFormValueModelBindingAttribute](https://github.com/aspnet/Entropy/blob/rel/2.0.0-preview2/samples/Mvc.FileUpload/Filters/DisableFormValueModelBindingAttribute.cs):

  * Предотвращает доступ привязки модели к данным формы.
  * Используется для загрузки больших файлов, если необходимо предотвратить считывание данных формы в память.

## <a name="action-filters"></a>Фильтры действий

> [!IMPORTANT]
> Фильтры действий **неприменимы** к Razor Pages. Razor Pages поддерживает <xref:Microsoft.AspNetCore.Mvc.Filters.IPageFilter> и <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncPageFilter>. Дополнительные сведения см. в разделе [Методы фильтрации для Razor Pages](xref:razor-pages/filter).

Фильтры действий:

* Реализуют либо интерфейс <xref:Microsoft.AspNetCore.Mvc.Filters.IActionFilter>, либо интерфейс <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncActionFilter>.
* Их выполнение охватывает выполнение методов действия.

В следующем фрагменте кода показан пример фильтра действий:

[!code-csharp[](./filters/sample/FiltersSample/Filters/MySampleActionFilter.cs?name=snippet_ActionFilter)]

<xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutingContext> предоставляет следующие свойства:

* <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutingContext.ActionArguments> позволяет считать входные данные в метод действия.
* <xref:Microsoft.AspNetCore.Mvc.Controller> позволяет управлять экземпляром контроллера.
* <xref:System.Web.Mvc.ActionExecutingContext.Result> позволяет при задании свойства `Result` сократить выполнение метода действия и последующих фильтров действий.

Вызов исключения в методе действия:

* Предотвращает выполнение последующих фильтров.
* В отличие от параметра `Result` рассматривается как сбой, а не успешный результат.

<xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutedContext> предоставляет `Controller` и `Result`, а также следующие свойства:

* <xref:System.Web.Mvc.ActionExecutedContext.Canceled> будет иметь значение true, если выполнение действия было сокращено другим фильтром.
* <xref:System.Web.Mvc.ActionExecutedContext.Exception> будет иметь ненулевое значение, если предыдущее выполнение фильтра действий вызвало исключение. При присвоении этому свойству ненулевого значения:

  * Будет эффективно обрабатываться исключение.
  * `Result` будет выполняться так, как при возвращении методом действия.

Для `IAsyncActionFilter` вызов <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutionDelegate>:

* Приводит к выполнению последующих фильтров действий и метода действия.
* Возвращает `ActionExecutedContext`.

Для сокращенного выполнения присвойте <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutingContext.Result?displayProperty=fullName> экземпляру результата и не вызывайте `next` (`ActionExecutionDelegate`).

Платформа предоставляет абстрактный класс <xref:Microsoft.AspNetCore.Mvc.Filters.ActionFilterAttribute>, для которого можно создавать подклассы.

Фильтр действий `OnActionExecuting` можно использовать:

* Для проверки состояния модели.
* Для получения ошибки, если состояние является недопустимым.

[!code-csharp[](./filters/sample/FiltersSample/Filters/ValidateModelAttribute.cs?name=snippet)]

Метод `OnActionExecuted` выполняется после метода действия:

* Он имеет доступ к результатам действия и может управлять ими с помощью свойства <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutedContext.Result>.
* <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutedContext.Canceled> будет иметь значение true, если выполнение действия было сокращено другим фильтром.
* <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutedContext.Exception> будет иметь ненулевое значение, если действие или последующий фильтр действий вызвали исключение. Если установить для `Exception` значение NULL:

  * Будет эффективно обрабатываться исключение.
  * `ActionExecutedContext.Result` будет выполняться так, как если бы метод действия вернул его обычным образом.

[!code-csharp[](./filters/sample/FiltersSample/Filters/ValidateModelAttribute.cs?name=snippet2&higlight=12-99)]

## <a name="exception-filters"></a>Фильтры исключений

Фильтры исключений:

* Реализуют <xref:Microsoft.AspNetCore.Mvc.Filters.IExceptionFilter> или <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncExceptionFilter>. 
* Можно использовать для реализации политик обработки стандартных ошибок.

В следующем примере фильтра исключений используется пользовательское представление ошибок для отображения подробных сведений об исключениях, которые происходят во время разработки приложения:

[!code-csharp[](./filters/sample/FiltersSample/Filters/CustomExceptionFilterAttribute.cs?name=snippet_ExceptionFilter&highlight=16-19)]

Фильтры исключений:

* Не имеют предшествующих и последующих событий.
* Реализуют <xref:Microsoft.AspNetCore.Mvc.Filters.IExceptionFilter.OnException*> или <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncExceptionFilter.OnExceptionAsync*>.
* Обрабатывают необработанные исключения, которые возникают при создании страницы Razor или контроллера, в [привязке модели](xref:mvc/models/model-binding), фильтрах действий или методах действий.
* **Не** перехватывают исключения, которые возникают в фильтрах ресурсов, фильтрах результатов или при выполнении результата MVC.

Для обработки исключения присвойте свойству <xref:System.Web.Mvc.ExceptionContext.ExceptionHandled> значение `true` или напишите ответ. Это предотвратит распространение исключения. Фильтр исключений не может преобразовать исключение в успешное выполнение. Это может сделать только фильтр действий.

Фильтры исключений:

* Хорошо подходят для перехвата исключений, возникающих в действиях.
* Не обладает такой гибкостью, как ПО промежуточного слоя обработки ошибок.

ПО промежуточного слоя хорошо подходит для обработки исключений. Используйте фильтры исключений, только если обработка ошибок осуществляется *по-разному* с учетом вызванного метода действия. Например, в приложении могут использоваться методы действий как для конечных точек API, так и для представлений или HTML. Конечные точки API могут возвращать сведения об ошибках в формате JSON, в то время как действия на основе представлений могут возвращать страницу ошибки в формате HTML.

## <a name="result-filters"></a>Фильтры результатов

Фильтры результатов:

* Реализация интерфейса:
  * <xref:Microsoft.AspNetCore.Mvc.Filters.IResultFilter> или <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncResultFilter>
  * <xref:Microsoft.AspNetCore.Mvc.Filters.IAlwaysRunResultFilter> или <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncAlwaysRunResultFilter>
* Их выполнение охватывает выполнение результатов действий.

### <a name="iresultfilter-and-iasyncresultfilter"></a>IResultFilter и IAsyncResultFilter

В следующем фрагменте кода показан фильтр результатов, который добавляет заголовок HTTP:

[!code-csharp[](./filters/sample/FiltersSample/Filters/LoggingAddHeaderFilter.cs?name=snippet_ResultFilter)]

Тип выполняемого результата зависит от соответствующего действия. Действие, возвращающее представление, будет включать всю обработку Razor в рамках выполняемого объекта <xref:Microsoft.AspNetCore.Mvc.ViewResult>. В процессе выполнения результата метод API может производить сериализацию. Дополнительные сведения о [результатах действий](xref:mvc/controllers/actions)

Фильтры результатов выполняются только в случае успешных результатов — когда действие или фильтры действий предоставляют результат действия. Фильтры результатов не выполняются, когда фильтры исключений обрабатывают исключение.

Метод <xref:Microsoft.AspNetCore.Mvc.Filters.IResultFilter.OnResultExecuting*?displayProperty=fullName> может сокращать выполнение результата действия и последующих фильтров результатов, присваивая свойству <xref:Microsoft.AspNetCore.Mvc.Filters.ResultExecutingContext.Cancel?displayProperty=fullName> значение `true`. При сокращении выполнения выполните запись в объект ответа, чтобы избежать формирования пустого ответа. Вызов исключения в `IResultFilter.OnResultExecuting`:

* Предотвращает выполнение результата действия и последующих фильтров.
* Рассматривается как сбой, а не успешный результат.

При выполнении метода <xref:Microsoft.AspNetCore.Mvc.Filters.IResultFilter.OnResultExecuted*?displayProperty=fullName>:

* Ответ, скорее всего, был отправлен клиенту и его нельзя изменить.
* Если возникло исключение, текст ответа не отправляется.

<!-- Review preceding "If an exception was thrown: Original 
When the OnResultExecuted method runs, the response has likely been sent to the client and cannot be changed further (unless an exception was thrown).

SHould that be , 
If an exception was thrown **IN THE RESULT FILTER**, the response body is not sent.

 -->

`ResultExecutedContext.Canceled` будет иметь значение `true`, если выполнение результата действия было сокращено другим фильтром.

`ResultExecutedContext.Exception` будет иметь ненулевое значение, если результат действия или последующий фильтр результатов вызвали исключение. Присвоение `Exception` значения NULL приводит к обработке исключения и предотвращает его последующий вызов ASP.NET Core на дальнейших этапах конвейера. Надежный способ записи данных в ответ при обработке исключения в фильтре результатов отсутствует. Если результат действия вызывает исключение в процессе выполнения и заголовки уже были переданы в клиент, надежного механизма отправки кода сбоя не существует.

Для <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncResultFilter> вызов к `await next` для <xref:Microsoft.AspNetCore.Mvc.Filters.ResultExecutionDelegate> приводит к выполнению последующих фильтров результатов и результата действия. Для сокращения выполнения задайте [ResultExecutingContext.Cancel](xref:Microsoft.AspNetCore.Mvc.Filters.ResultExecutingContext.Cancel) для `true` и не вызывайте `ResultExecutionDelegate`:

[!code-csharp[](./filters/sample/FiltersSample/Filters/MyAsyncResponseFilter.cs?name=snippet)]

Платформа предоставляет абстрактный класс `ResultFilterAttribute`, для которого можно создавать подклассы. Представленный ранее класс [AddHeaderAttribute](#add-header-attribute) — это пример атрибута фильтра результатов.

### <a name="ialwaysrunresultfilter-and-iasyncalwaysrunresultfilter"></a>IAlwaysRunResultFilter и IAsyncAlwaysRunResultFilter

Интерфейсы <xref:Microsoft.AspNetCore.Mvc.Filters.IAlwaysRunResultFilter> и <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncAlwaysRunResultFilter> объявляют реализацию <xref:Microsoft.AspNetCore.Mvc.Filters.IResultFilter>, которая выполняется для всех результатов действий. Фильтр применяется для всех результатов действий, если:

* <xref:Microsoft.AspNetCore.Mvc.Filters.IExceptionFilter> или <xref:Microsoft.AspNetCore.Mvc.Filters.IAuthorizationFilter> применяется и сокращает ответ.
* Фильтр исключений обрабатывает исключение и выдает результат действия.

Фильтры, отличающиеся от `IExceptionFilter` и `IAuthorizationFilter`, не сокращают `IAlwaysRunResultFilter` и `IAsyncAlwaysRunResultFilter`.

Например, следующий фильтр всегда выполняется, задавая результат действия (<xref:Microsoft.AspNetCore.Mvc.ObjectResult>) с кодом состояния *422 Unprocessable Entity* при сбое согласования содержимого:

[!code-csharp[](./filters/sample/FiltersSample/Filters/UnprocessableResultFilter.cs?name=snippet)]

### <a name="ifilterfactory"></a>IFilterFactory

Объект <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory> реализует интерфейс <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterMetadata>. Поэтому экземпляр `IFilterFactory` можно использовать в качестве экземпляра `IFilterMetadata` в любом месте конвейера фильтров. Когда среда выполнения готовится вызвать фильтр, она пытается привести его к `IFilterFactory`. Если приведение проходит успешно, вызывается метод <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory.CreateInstance*> для создания вызываемого экземпляра `IFilterMetadata`. Это обеспечивает высокую степень гибкости, так как при запуске приложения нет необходимости в явном и точном определении конвейера фильтров.

Еще один подход к созданию фильтров заключается в реализации `IFilterFactory` с помощью настраиваемых атрибутов:

[!code-csharp[](./filters/sample/FiltersSample/Filters/AddHeaderWithFactoryAttribute.cs?name=snippet_IFilterFactory&highlight=1,4,5,6,7)]

Приведенный выше код можно проверить, выполнив [скачиваемый пример](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/controllers/filters/sample):

* Вызовите средства для разработчика (F12).
* Перейдите к папке `https://localhost:5001/Sample/HeaderWithFactory`.

В средствах для разработчика (F12) отобразятся следующие заголовки ответа, добавленные примером кода:

* **author:** `Joe Smith`
* **globaladdheader:** `Result filter added to MvcOptions.Filters`
* **internal:** `My header`

Приведенный выше код создает заголовок ответа **internal:** `My header`.

### <a name="ifilterfactory-implemented-on-an-attribute"></a>Реализация IFilterFactory в атрибуте

<!-- Review 
This section needs to be rewritten.
What's a non-named attribute?
-->

Фильтры, реализующие `IFilterFactory`, используются для фильтров, которые:

* Не требуют передачи параметров.
* Содержат зависимости конструктора, которые должны заполняться путем внедрения зависимостей.

Объект <xref:Microsoft.AspNetCore.Mvc.TypeFilterAttribute> реализует интерфейс <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory>. `IFilterFactory` предоставляет метод <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory.CreateInstance*> для создания экземпляра <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterMetadata>. `CreateInstance` загружает указанный тип из контейнера служб (внедрение зависимостей).

[!code-csharp[](./filters/sample/FiltersSample/Filters/SampleActionFilterAttribute.cs?name=snippet_TypeFilterAttribute&highlight=1,3,7)]

В коде ниже приведено три примера применения `[SampleActionFilter]`:

[!code-csharp[](./filters/sample/FiltersSample/Controllers/HomeController.cs?name=snippet&highlight=1)]

В приведенном выше коде декорирование метода с использованием `[SampleActionFilter]` является рекомендуемым способом применения `SampleActionFilter`.

## <a name="using-middleware-in-the-filter-pipeline"></a>Использование ПО промежуточного слоя в конвейере фильтров

Фильтры ресурсов по принципу работы похожи на [ПО промежуточного слоя](xref:fundamentals/middleware/index) тем, что они заключают в себя выполнение всех объектов, находящихся далее в конвейере. При этом фильтры отличаются от ПО промежуточного слоя тем, что они являются частью среды выполнения ASP.NET Core, то есть они имеют доступ к контексту и конструкциям ASP.NET Core.

Чтобы использовать ПО промежуточного слоя в качестве фильтра, создайте тип с методом `Configure`, определяющим ПО промежуточного слоя, которое нужно внедрить в конвейер фильтров. В примере ниже ПО промежуточного слоя локализации применяется для определения текущих языка и региональных параметров для запроса:

[!code-csharp[](./filters/sample/FiltersSample/Filters/LocalizationPipeline.cs?name=snippet_MiddlewareFilter&highlight=3,21)]

Используйте <xref:Microsoft.AspNetCore.Mvc.MiddlewareFilterAttribute> для выполнения ПО промежуточного слоя:

[!code-csharp[](./filters/sample/FiltersSample/Controllers/HomeController.cs?name=snippet_MiddlewareFilter&highlight=2)]

Фильтры ПО промежуточного слоя выполняются на том же этапе конвейера фильтров, что и фильтры ресурсов, перед привязкой модели и после остальной части конвейера.

## <a name="next-actions"></a>Дальнейшие действия

* Дополнительные сведения см. в статье [Filter methods for Razor Pages in ASP.NET Core](xref:razor-pages/filter) (Методы фильтрации для Razor Pages в ASP.NET Core).
* Чтобы поэкспериментировать с фильтрами, [скачайте, протестируйте и измените пример с GitHub](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/controllers/filters/sample).
