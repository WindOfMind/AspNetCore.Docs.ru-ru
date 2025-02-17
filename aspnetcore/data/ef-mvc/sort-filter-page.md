---
title: Учебник. Использование ASP.NET MVC с EF Core. Добавление сортировки, фильтрации и разбиения на страницы
description: Из этого руководства вы узнаете, как добавить на страницу указателя учащихся сортировку, фильтрацию и разбиение на страницы. Здесь также описывается создание страницы с простой группировкой.
author: rick-anderson
ms.author: tdykstra
ms.date: 03/27/2019
ms.topic: tutorial
uid: data/ef-mvc/sort-filter-page
ms.openlocfilehash: 921e27bf56587813f835357c9090c91a155c087b
ms.sourcegitcommit: b508b115107e0f8d7f62b25cfcc8ad45e1373459
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 05/07/2019
ms.locfileid: "65212549"
---
# <a name="tutorial-add-sorting-filtering-and-paging---aspnet-mvc-with-ef-core"></a>Учебник. Использование ASP.NET MVC с EF Core. Добавление сортировки, фильтрации и разбиения на страницы

В предыдущем руководстве был создан набор веб-страниц для основных операций CRUD для сущностей Student. Из этого руководства вы узнаете, как добавить на страницу указателя учащихся сортировку, фильтрацию и разбиение на страницы. Здесь также описывается создание страницы с простой группировкой.

На следующем рисунке изображен вид страницы после выполнения задания. Заголовки столбцов являются ссылками, при нажатии на которые происходит сортировка по этому столбцу. При повторном нажатии на заголовок происходит переключение между сортировкой по возрастанию и убыванию.

![Страница указателя учащихся](sort-filter-page/_static/paging.png)

В этом учебнике рассмотрены следующие задачи.

> [!div class="checklist"]
> * Добавление ссылок для сортировки столбцов
> * Добавление поля поиска
> * Добавление разбиения по страницам в указатель учащихся
> * Добавление разбиения по страницам в метод Index
> * Добавление ссылок для разбиения по страницам
> * Создание страницы сведений

## <a name="prerequisites"></a>Предварительные требования

* [Реализация функциональности CRUD](crud.md)

## <a name="add-column-sort-links"></a>Добавление ссылок для сортировки столбцов

Для добавления сортировки на страницу указателя учащихся изменим метод `Index` контроллера Students и добавим код в представление указателя учащихся.

### <a name="add-sorting-functionality-to-the-index-method"></a>Добавление сортировки в метод Index

В файле *StudentsController.cs* замените код метода `Index` следующим кодом:

[!code-csharp[](intro/samples/cu/Controllers/StudentsController.cs?name=snippet_SortOnly)]

Этот код принимает параметр `sortOrder` из строки запроса в URL. Значение строки запроса в ASP.NET Core MVC передается как параметр метода действия. Имя параметра представляет собой строку, состоящую из "Name" или "Date" с возможным добавлением знака подчеркивания и строки "desc" для указания убывающего порядка сортировки. По умолчанию используется порядок сортировки по возрастанию.

При первом запросе страницы Index строка запроса отсутствует. Список студентов отсортирован по фамилиям по возрастанию, порядок сортировки по умолчанию задается в выражении `switch`. Когда пользователь щелкает гиперссылку заголовка столбца, в строку запроса подставляется соответствующее значение параметра `sortOrder`.

Для формирования гиперссылок в заголовках столбцов в представлении используются два элемента `ViewData` (NameSortParm и DateSortParm) с соответствующими значениями строки запроса.

[!code-csharp[](intro/samples/cu/Controllers/StudentsController.cs?name=snippet_SortOnly&highlight=3-4)]

Это тернарные условные операторы. Первое выражение означает, что если параметр `sortOrder` пуст или равен null, то параметр NameSortParm должен принять значение "name_desc", в противном случае параметру NameSortParm присваивается пустая строка. Следующие два оператора устанавливают гиперссылки в заголовках столбцов в представлении следующим образом:

|  Текущий порядок сортировки  | Гиперссылка "Last Name" (Фамилия) | Гиперссылка "Date" (Дата) |
|:--------------------:|:-------------------:|:--------------:|
| "Last Name" (Фамилия) по возрастанию  | по убыванию          | по возрастанию      |
| "Last Name" (Фамилия) по убыванию | по возрастанию           | по возрастанию      |
| "Date" (Дата) по возрастанию       | по возрастанию           | по убыванию     |
| "Date" (Дата) по убыванию      | по возрастанию           | по возрастанию      |

Для указания столбца, по которому выполняется сортировка, этот метод использует LINQ to Entities. Данный код создает переменную `IQueryable` перед оператором switch, изменяет ее значение внутри оператора switch и вызывает метод `ToListAsync` после `switch`. После создания и изменения переменных `IQueryable` запрос в базу данных не отправляется. Запрос не выполнится, пока вы не преобразуете объект `IQueryable` в коллекцию, вызвав соответствующий метод, такой как `ToListAsync`. Таким образом, этот код создает одиночный запрос, который не будет выполнен до выполнения выражения `return View`.

Этот код можно расширить на случай большого числа столбцов. [В последнем руководстве серии](advanced.md#dynamic-linq) вы найдете пример кода, который позволяет передавать имя столбца `OrderBy` в строковой переменной.

### <a name="add-column-heading-hyperlinks-to-the-student-index-view"></a>Добавление гиперссылок для заголовков столбцов в представлении индекса учащихся

Для добавления гиперссылок в заголовки столбцов замените код в файле *Views/Students/Index.cshtml* следующим кодом. Измененные строки выделены.

[!code-html[](intro/samples/cu/Views/Students/Index2.cshtml?highlight=16,22)]

Для формирования гиперссылок с соответствующей строкой запроса этот код использует информацию из свойств `ViewData`.

Для проверки работы сортировки запустите приложение, выберите вкладку **Students** и нажимайте на заголовки столбцов **Last Name** и **Enrollment Date**.

![Страница указателя учащихся с отсортированным по фамилиям списком](sort-filter-page/_static/name-order.png)

## <a name="add-a-search-box"></a>Добавление поля поиска

Для добавления фильтра на страницу указателя учащихся необходимо добавить в представление текстовое поле и кнопку отправки и внести изменения в метод `Index`. Текстовое поле необходимо для ввода строки для поиска в полях имени и фамилии.

### <a name="add-filtering-functionality-to-the-index-method"></a>Добавление функций фильтрации в метод Index

В файле *StudentsController.cs* замените метод `Index` следующим кодом (изменения выделены).

[!code-csharp[](intro/samples/cu/Controllers/StudentsController.cs?name=snippet_SortFilter&highlight=1,5,9-13)]

Мы добавили в метод `Index` параметр `searchString`. Значение строки поиска получается из текстового поля, которое мы добавили в представление Index. Мы также добавили в запрос LINQ предложение where, которое отбирает только студентов, чье имя или фамилия содержат строку поиска. Выражение с предложением where выполняется только в том случае, если задано значение для поиска.

> [!NOTE]
> В этом коде мы вызываем метод `Where` объекта `IQueryable`, при этом фильтр будет обработан на сервере. В некоторых случаях может потребоваться вызов метода `Where` как метода расширения коллекции в памяти. (Предположим, например, что вы изменили ссылку на `_context.Students` таким образом, что вместо объекта EF `DbSet` она ссылается на метод репозитория, который возвращает коллекцию `IEnumerable`.) Обычно результат остается прежним, но в некоторых случаях он может отличаться.
>
>Например, в .NET Framework метод `Contains` по умолчанию выполняет сравнение с учетом регистра, а в SQL Server это определяется параметром сортировки конкретного экземпляра SQL сервера. По умолчанию параметр установлен на сравнение без учета регистра. Можно вызвать метод `ToUpper`, чтобы явно настроить тест не учитывать регистр:  *Where(s => s.LastName.ToUpper().Contains(searchString.ToUpper())*. Это гарантирует, что поведение программы не изменится, если вы измените код на использование репозитория, который возвращает `IEnumerable`, а не объект `IQueryable`. (При вызове метода `Contains` коллекции `IEnumerable` выполняется реализация .NET Framework; при вызове этого же метода у объекта `IQueryable` выполняется реализация поставщика базы данных.) Однако такое решение снижает производительность. Метод `ToUpper` добавляет функцию в предложение WHERE TSQL-выражения SELECT. Это не позволяет оптимизатору использовать индекс. Учитывая, что SQL обычно настраивается на то, чтобы не учитывать регистр, рекомендуется не использовать код `ToUpper` до миграции на хранилище данных, учитывающее регистр.

### <a name="add-a-search-box-to-the-student-index-view"></a>Добавление поля поиска на страницу индекса учащихся

В файле *Views/Student/Index.cshtml* добавьте выделенный код непосредственно перед открывающим тегом table для создания заголовка, текстового поля и кнопки **Search**.

[!code-html[](intro/samples/cu/Views/Students/Index3.cshtml?range=9-23&highlight=5-13)]

Для добавления кнопки и поля поиска этот код использует [вспомогательную функцию тега](xref:mvc/views/tag-helpers/intro) `<form>`. По умолчанию вспомогательная функция тега `<form>` отправляет данные формы с помощью запроса POST, это означает, что параметры передаются в теле сообщения HTTP, а не в URL-адресе в виде строки запросов. При указании метода HTTP GET данные формы передаются в URL-адресе в виде строк запроса, что позволяет добавлять URL-адреса в закладки. Руководства консорциума W3C рекомендуют использовать метод GET, когда действие не приводит к обновлению.

Для проверки работы фильтра запустите приложение, выберите вкладку **Students**, введите строку поиска и нажмите Search.

![Страница указателя учащихся с фильтрацией](sort-filter-page/_static/filtering.png)

Обратите внимание, что URL-адрес содержит строку поиска.

```html
http://localhost:5813/Students?SearchString=an
```

Если вы добавите эту страницу в закладки, то при открытии закладки будет открываться уже отфильтрованный список. Формирование строки запроса обеспечивает добавление `method="get"` в тег `form`.

На данном этапе, если нажать ссылку сортировки в заголовке столбца, то значение фильтра, которое мы ввели в поле **Search**, будет потеряно. Мы исправим это в следующем разделе.

## <a name="add-paging-to-students-index"></a>Добавление разбиения по страницам в указатель учащихся

Чтобы добавить на страницу указателя учащихся разбиение на страницы, следует создать класс `PaginatedList`, который использует операторы `Skip` и `Take` для фильтрации данных на сервере вместо того, чтобы каждый раз получать все строки таблицы. Затем мы внесем дополнительные изменения в метод `Index` и добавим в представление `Index` кнопки перелистывания страниц. На следующем рисунке показаны кнопки перелистывания.

![Страница указателя учащихся со ссылками для перелистывания](sort-filter-page/_static/paging.png)

В папке проекта создайте файл `PaginatedList.cs` и замените код шаблона на следующий код.

[!code-csharp[](intro/samples/cu/PaginatedList.cs)]

В этом коде метод `CreateAsync` принимает размер и номер страницы и вызывает соответствующие методы `Skip` и `Take` объекта `IQueryable`. Метод `ToListAsync` объекта `IQueryable` при вызове возвратит список, содержащий только запрошенную страницу. Для включения и отключения кнопок перелистывания страниц **Previous** и **Next** можно использовать свойства `HasPreviousPage` и `HasNextPage`.

Для создания объекта `PaginatedList<T>` вместо конструктора используется метод `CreateAsync`, поскольку конструкторы не могут выполнять асинхронный код.

## <a name="add-paging-to-index-method"></a>Добавление разбиения по страницам в метод Index

В файле *StudentsController.cs* замените код метода `Index` следующим кодом.

[!code-csharp[](intro/samples/cu/Controllers/StudentsController.cs?name=snippet_SortFilterPage&highlight=1-5,7,11-18,45-46)]

Этот код добавляет к сигнатуре метода параметры с номером страницы, текущим порядком сортировки и текущим фильтром.

```csharp
public async Task<IActionResult> Index(
    string sortOrder,
    string currentFilter,
    string searchString,
    int? pageNumber)
```

При первом отображении страницы или если пользователь еще не нажимал на ссылки сортировки и перелистывания, все параметры будут иметь значение null.  При нажатии на кнопку перелистывания переменная page будет содержать номер страницы для отображения.

Элемент `ViewData` с именем CurrentSort передает в представление порядок сортировки, поскольку он должен быть включен в ссылки перелистывания, чтобы сохранить порядок сортировки при переходе по страницам.

Элемент `ViewData` с именем CurrentFilter передает в представление текущую строку фильтра. Это значение необходимо включить в ссылки для перелистывания, чтобы при смене страницы сохранить настройки фильтра, кроме того, необходимо восстановить значение фильтра в текстовом поле после обновления страницы.

Если строка поиска изменяется во время перелистывания, то номер страницы должен быть сброшен на 1, так как с новым фильтром изменится состав отображаемых данных. Изменение строки поиска происходит при вводе в текстовое поле значения и нажатии на кнопку отправки. В этом случае значение параметра `searchString` не null.

```csharp
if (searchString != null)
{
    pageNumber = 1;
}
else
{
    searchString = currentFilter;
}
```

В конце метода `Index` метод `PaginatedList.CreateAsync` преобразует результат запроса студентов в страницу коллекции, поддерживающую разбиение на страницы. Это страница со студентами затем передается в представление.

```csharp
return View(await PaginatedList<Student>.CreateAsync(students.AsNoTracking(), pageNumber ?? 1, pageSize));
```

Метод `PaginatedList.CreateAsync` принимает номер страницы. Два вопросительных знака являются оператором объединения с null. Этот оператор определяет значение по умолчанию для значения null; выражение `(pageNumber ?? 1)` возвращает значение переменной `pageNumber`, если она имеет значение, и возвращает 1, если переменная `pageNumber` имеет значение null.

## <a name="add-paging-links"></a>Добавление ссылок для разбиения по страницам

Замените код в файле *Views/Students/Index.cshtml* следующим кодом. Изменения выделены.

[!code-html[](intro/samples/cu/Views/Students/Index.cshtml?highlight=1,27,30,33,61-79)]

Оператор `@model` в начале страницы указывает на то, что теперь представление принимает объект `PaginatedList<T>`, а не объект `List<T>`.

Ссылки в заголовках столбцов передают в контроллер при помощи строки запроса текущее значение строки поиска, чтобы пользователь мог сортировать отфильтрованные данные:

```html
<a asp-action="Index" asp-route-sortOrder="@ViewData["DateSortParm"]" asp-route-currentFilter ="@ViewData["CurrentFilter"]">Enrollment Date</a>
```

Кнопки перелистывания отображаются вспомогательными функциями тегов:

```html
<a asp-action="Index"
   asp-route-sortOrder="@ViewData["CurrentSort"]"
   asp-route-pageNumber="@(Model.PageIndex - 1)"
   asp-route-currentFilter="@ViewData["CurrentFilter"]"
   class="btn btn-default @prevDisabled">
   Previous
</a>
```

Запустите приложение и перейдите на страницу Students.

![Страница указателя учащихся со ссылками для перелистывания](sort-filter-page/_static/paging.png)

Чтобы убедиться, что постраничный просмотр работает, нажимайте кнопки перелистывания при различном порядке сортировки. Затем введите строку поиска и повторите перелистывание, чтобы убедиться, что разбиение на страницы работает корректно вместе с сортировкой и фильтрацией.

## <a name="create-an-about-page"></a>Создание страницы сведений

На странице **About** веб-сайта "Университет Contoso" будет отображаться количество зачисленных студентов по дням. Для этого понадобится группировка и выполнение простых расчетов в группах. Для выполнения этой задачи нам потребуется следующее:

* Создать класс модели представления для данных, которые необходимо передать в представление.
* Создать метод About в контроллере Home.
* Создать представление About.

### <a name="create-the-view-model"></a>Создание модели представления

Создайте папку *SchoolViewModels* в папке *Models*.

В новой папке добавьте файл класса *EnrollmentDateGroup.cs* и замените код шаблона следующим кодом:

[!code-csharp[](intro/samples/cu/Models/SchoolViewModels/EnrollmentDateGroup.cs)]

### <a name="modify-the-home-controller"></a>Изменение контроллера Home

Добавьте следующие директивы using в начало файла *HomeController.cs*:

[!code-csharp[](intro/samples/cu/Controllers/HomeController.cs?name=snippet_Usings1)]

Добавьте переменную класса для контекста базы данных сразу же после открывающей фигурной скобки описания класса и получите экземпляр контекста из ASP.NET Core DI:

[!code-csharp[](intro/samples/cu/Controllers/HomeController.cs?name=snippet_AddContext&highlight=3,5,7)]

Добавьте метод `About` со следующим кодом.

[!code-csharp[](intro/samples/cu/Controllers/HomeController.cs?name=snippet_UseDbSet)]

Запрос LINQ группирует записи из таблицы студентов по дате зачисления, вычисляет число записей в каждой группе и сохраняет результаты в коллекцию объектов моделей представления `EnrollmentDateGroup`.

### <a name="create-the-about-view"></a>Создание представления About

Добавьте файл *Views/Home/About.cshtml* со следующим кодом.

[!code-html[](intro/samples/cu/Views/Home/About.cshtml)]

Запустите приложение и перейдите на страницу About. Количество зачисленных студентов по дням отображается в таблице.

## <a name="get-the-code"></a>Получение кода

[Скачайте или ознакомьтесь с готовым приложением.](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/data/ef-mvc/intro/samples/cu-final)

## <a name="next-steps"></a>Следующие шаги

В этом учебнике рассмотрены следующие задачи.

> [!div class="checklist"]
> * Добавление ссылок для сортировки столбцов
> * Добавление поля поиска
> * Добавление разбиения по страницам в указатель учащихся
> * Добавление разбиения по страницам в метод Index
> * Добавление ссылок для разбиения по страницам
> * Создание страницы сведений

В следующем учебнике описано, как с помощью миграций обрабатывать изменения в модели данных.

> [!div class="nextstepaction"]
> [Далее: Обработка изменений модели данных](migrations.md)
