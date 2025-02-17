---
title: Razor Pages с EF Core в ASP.NET Core — параллелизм — 8 из 8
author: rick-anderson
description: Это руководство описывает, как обрабатывать конфликты, когда несколько пользователей одновременно изменяют одну сущность.
ms.author: riande
ms.custom: mvc
ms.date: 05/31/2019
uid: data/ef-rp/concurrency
ms.openlocfilehash: 8430f8e720870a7b541655ea8bcfe2f67c942bb3
ms.sourcegitcommit: c5339594101d30b189f61761275b7d310e80d18a
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 06/02/2019
ms.locfileid: "66458426"
---
# <a name="razor-pages-with-ef-core-in-aspnet-core---concurrency---8-of-8"></a>Razor Pages с EF Core в ASP.NET Core — параллелизм — 8 из 8

Авторы: [Рик Андерсон](https://twitter.com/RickAndMSFT) (Rick Anderson), [Том Дайкстра](https://github.com/tdykstra) (Tom Dykstra) и [Йон П. Смит](https://twitter.com/thereformedprog) (Jon P Smith)

[!INCLUDE [about the series](../../includes/RP-EF/intro.md)]

Это руководство описывает, как обрабатывать конфликты, когда несколько пользователей параллельно (одновременно) изменяют одну сущность. При возникновении проблем, которые вам не удается устранить, [скачайте или просмотрите готовое приложение.](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/data/ef-rp/intro/samples) [Указания по скачиванию](xref:index#how-to-download-a-sample).

## <a name="concurrency-conflicts"></a>Конфликты параллелизма

Конфликт параллелизма возникает в следующих условиях:

* Пользователь переходит на страницу редактирования для сущности.
* Другой пользователь обновляет ту же сущность до того, как изменение первого пользователя записывается в базу данных.

Если обнаружение параллелизма отключено, возникают параллельные изменения:

* Побеждает последнее изменение. Таким образом, в базу данных заносятся значения из последнего изменения.
* Первые из текущих изменений утрачиваются.

### <a name="optimistic-concurrency"></a>Оптимистическая блокировка

Оптимистическая блокировка допускает появление конфликтов параллелизма, а затем обрабатывает их соответствующим образом. Например, Мария посещает страницу изменения кафедры и изменяет бюджет кафедры английской языка с 350 000,00 USD на 0,00 USD.

![Изменение бюджета на 0](concurrency/_static/change-budget.png)

Прежде чем Мария нажимает кнопку **Save** (Сохранить), Дмитрий заходит на ту же страницу и изменяет значение в поле "Start Date" (Дата начала) с 9/1/2007 на 9/1/2013.

![Изменение начальной даты на 2013 г.](concurrency/_static/change-date.png)

Сначала Мария нажимает кнопку **Save** (Сохранить) и видит свое изменение, когда браузер отображает страницу индекса.

![Бюджет изменен на нуль](concurrency/_static/budget-zero.png)

Дмитрий нажимает кнопку **Save** (Сохранить) на странице редактирования, где все еще отображается бюджет 350 000,00 USD. Дальнейший ход событий определяется порядком обработки конфликтов параллелизма.

Оптимистическая блокировка включает в себя следующие параметры:

* Вы можете отслеживать, для какого свойства пользователь изменил и обновил только соответствующие столбцы в базе данных.

  В этом сценарии никакие данные не теряются. Разные свойства были обновлены двумя пользователями. Когда какой-либо пользователь просмотрит кафедру английского языка в следующий раз, он увидит изменения, внесенные как Марией, так и Дмитрием. Этот способ обновления позволяет снизить количество конфликтов, способных привести к потере данных. Особенности этого подхода:
 
  * Он не позволяет избежать потери данных, если конкурирующие изменения вносятся для одного свойства.
  * Он не слишком хорошо подходит для веб-приложений, так как требует поддерживать значительный объем состояний, чтобы отслеживать все извлеченные и новые значения. Обслуживание большого объема состояний может негативно повлиять на производительность приложения.
  * Он может повысить сложность приложений по сравнению с обнаружением параллелизма для сущности.

* Вы можете позволить изменению Дмитрия перезаписать изменение Марии.

  Когда какой-либо пользователь просмотрит кафедру английского языка в следующий раз, он увидит дату 9/1/2013 и значение 350 000,00 USD. Такой подход называется *победой клиента* или *сохранением последнего внесенного изменения*. (Все значения из клиента имеют приоритет над данными в хранилище.) Если не писать код для обработки параллелизма, автоматически применяется победа клиента.

* Вы можете запретить обновление изменения Дмитрия в базе данных. Как правило приложение будет:

  * отображать сообщение об ошибке;
  * отображать текущее состояние данных;
  * разрешать пользователю повторно применять изменения.

  Это называется *победой хранилища*. (Значения в хранилище имеют приоритет над данными, передаваемыми клиентом.) В этом руководстве вы реализуете сценарий победы хранилища. Данный метод гарантирует, что никакие изменения не перезаписываются без оповещения пользователя.

## <a name="handling-concurrency"></a>Обработка параллелизма 

Если свойство настроено как [токен параллелизма](/ef/core/modeling/concurrency):

* EF Core проверяет, что свойство не было изменено после его получения. Эта проверка выполняется при вызове [SaveChanges](/dotnet/api/microsoft.entityframeworkcore.dbcontext.savechanges?view=efcore-2.0#Microsoft_EntityFrameworkCore_DbContext_SaveChanges) или [SaveChangesAsync](/dotnet/api/microsoft.entityframeworkcore.dbcontext.savechangesasync?view=efcore-2.0#Microsoft_EntityFrameworkCore_DbContext_SaveChangesAsync_System_Threading_CancellationToken_).
* Если свойство было изменено после получения, возникает исключение [DbUpdateConcurrencyException](/dotnet/api/microsoft.entityframeworkcore.dbupdateconcurrencyexception?view=efcore-2.0). 

Нужно настроить базу данных и модель данных для поддержки исключения `DbUpdateConcurrencyException`.

### <a name="detecting-concurrency-conflicts-on-a-property"></a>Обнаружение конфликтов параллелизма для свойства

Конфликты параллелизма можно обнаружить на уровне свойств с помощью атрибута [ConcurrencyCheck](/dotnet/api/system.componentmodel.dataannotations.concurrencycheckattribute?view=netcore-2.0). Этот атрибут можно применить к нескольким свойствам в модели. Дополнительные сведения см. в [описании ConcurrencyCheck в подразделе "Заметки к данным"](/ef/core/modeling/concurrency#data-annotations).

Атрибут`[ConcurrencyCheck]` в этом руководстве не используется.

### <a name="detecting-concurrency-conflicts-on-a-row"></a>Обнаружение конфликтов параллелизма для строки

Чтобы обнаружить конфликты параллелизма, в модель добавлен столбец отслеживания [rowversion](/sql/t-sql/data-types/rowversion-transact-sql).  `rowversion`:

* Относится к SQL Server. Другие базы данных могут не предоставлять подобную функцию.
* Используется для определения того, что сущность не была изменена после ее получения из базы данных. 

База данных формирует последовательный номер `rowversion`, увеличивающийся при каждом обновлении строки. В команде `Update` или `Delete` предложение `Where` включает извлеченное значение из `rowversion`. Если обновляемая строка изменились:

* `rowversion` не соответствует полученному значению.
* Команда `Update` или `Delete` не находит строку, так как предложение `Where` включает полученное значение `rowversion`.
* Возникает исключение `DbUpdateConcurrencyException`.

Когда в EF Core нет строк, обновленных командой `Update` или `Delete`, возникает исключение параллелизма.

### <a name="add-a-tracking-property-to-the-department-entity"></a>Добавление свойства отслеживания в сущность Department

В *Models/Department.cs* добавьте свойство отслеживания RowVersion:

[!code-csharp[](intro/samples/cu/Models/Department.cs?name=snippet_Final&highlight=26,27)]

Атрибут [Timestamp](/dotnet/api/system.componentmodel.dataannotations.timestampattribute) указывает, что этот столбец входит в предложение `Where` для команд `Update` и `Delete`. Этот атрибут называется `Timestamp`, так как предыдущие версии SQL Server использовали тип данных `timestamp` SQL, пока его не сменил тип `rowversion`.

Текучий API также может задавать свойство отслеживания:

```csharp
modelBuilder.Entity<Department>()
  .Property<byte[]>("RowVersion")
  .IsRowVersion();
```

В следующем коде показана часть кода T-SQL, созданного EF Core при обновлении названия кафедры:

[!code-sql[](intro/samples/sql.txt?highlight=2-3)]

Предыдущий выделенный код показывает предложение `WHERE`, содержащее `RowVersion`. Если база данных `RowVersion` не соответствует параметру `RowVersion` (`@p2`), никакие строки не обновляются.

Следующий выделенный код показывает код T-SQL, который подтверждает, что была обновлена всего одна строка.

[!code-sql[](intro/samples/sql.txt?highlight=4-6)]

[@@ROWCOUNT](/sql/t-sql/functions/rowcount-transact-sql) возвращает число строк, затронутых при выполнении последнего оператора. Если никакие строки не обновляются, EF Core выдает исключение `DbUpdateConcurrencyException`.

Вы можете просмотреть код T-SQL, создаваемый EF Core в окне вывода Visual Studio.

### <a name="update-the-db"></a>Обновление базы данных

Добавление свойства `RowVersion` изменяет модель базы данных, которая требует миграции.

Выполните построение проекта. Введите в командном окне следующее:

```console
dotnet ef migrations add RowVersion
dotnet ef database update
```

Предыдущие команды:

* Добавляет файл миграций *Migrations/{метка времени}_RowVersion.cs*.
* Изменяет файл *Migrations/SchoolContextModelSnapshot.cs*. Это изменение добавляет следующий выделенный код в метод `BuildModel`:

  [!code-csharp[](intro/samples/cu/Migrations/SchoolContextModelSnapshot2.cs?name=snippet&highlight=14-16)]

* Запускает миграции для обновления базы данных.

<a name="scaffold"></a>

## <a name="scaffold-the-departments-model"></a>Формирование шаблона для модели кафедр

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio) 

Следуйте инструкциям в разделе [Формирование шаблона для модели Student](xref:data/ef-rp/intro#scaffold-the-student-model) и используйте `Department` для класса модели.

# <a name="net-core-clitabnetcore-cli"></a>[Интерфейс командной строки .NET Core](#tab/netcore-cli)

 Выполните следующую команду:

  ```console
  dotnet aspnet-codegenerator razorpage -m Department -dc SchoolContext -udl -outDir Pages\Departments --referenceScriptLibraries
  ```

---

Предыдущая команда формирует шаблон для модели `Department`. Откройте проект в Visual Studio.

Выполните построение проекта.

### <a name="update-the-departments-index-page"></a>Изменение страницы индекса кафедр

Подсистема формирования шаблонов создала столбец `RowVersion` для страницы индекса, однако это поле не должно отображаться. В этом руководстве отображается последний байт `RowVersion` для лучшего понимания параллелизма. Этот последний байт необязательно является уникальным. Реальное приложение не будет отображать `RowVersion` или последний байт `RowVersion`.

Обновите страницу индекса:

* Замените Index на Departments.
* Замените разметку, содержащую `RowVersion`, на последний байт `RowVersion`.
* Замените FirstMidName на FullName.

Следующая разметка показывает обновленную страницу:

[!code-html[](intro/samples/cu/Pages/Departments/Index.cshtml?highlight=5,8,29,47,50)]

### <a name="update-the-edit-page-model"></a>Обновление модели страницы "Edit" (Редактирование)

Измените *Pages\Departments\Edit.cshtml.cs*, используя следующий код:

[!code-csharp[](intro/samples/cu/Pages/Departments/Edit.cshtml.cs?name=snippet)]

Для обнаружения проблемы параллелизма [OriginalValue](/dotnet/api/microsoft.entityframeworkcore.changetracking.propertyentry.originalvalue?view=efcore-2.0#Microsoft_EntityFrameworkCore_ChangeTracking_PropertyEntry_OriginalValue) обновляется с помощью значения `rowVersion` из сущности, откуда он был получен. EF Core создает команду SQL UPDATE с предложением WHERE, содержащим исходное значение `RowVersion`. Если команда UPDATE не затрагивает никакие строки (нет строк, имеющих исходное значение `RowVersion`), возникает исключение `DbUpdateConcurrencyException`.

[!code-csharp[](intro/samples/cu/Pages/Departments/Edit.cshtml.cs?name=snippet_rv&highlight=24-999)]

В приведенном выше коде `Department.RowVersion` является значением на момент извлечения сущности. `OriginalValue` является значением в базе данных на момент вызова `FirstOrDefaultAsync` в этом методе.

Следующий код возвращает значения клиента (значения, переданные в этот метод) и значения базы данных:

[!code-csharp[](intro/samples/cu/Pages/Departments/Edit.cshtml.cs?name=snippet_try&highlight=9,18)]

Следующий код добавляет пользовательское сообщение об ошибке для каждого столбца, у которого значения базы данных отличаются в переданных в `OnPostAsync`:

[!code-csharp[](intro/samples/cu/Pages/Departments/Edit.cshtml.cs?name=snippet_err)]

Следующий выделенный код задает для `RowVersion` новое значение, полученное из базы данных. Когда пользователь в следующий раз нажимает кнопку **Save** (Сохранить), перехватываются только те ошибки параллелизма, возникшие с момента последнего отображения страницы "Edit" (Редактирование).

[!code-csharp[](intro/samples/cu/Pages/Departments/Edit.cshtml.cs?name=snippet_try&highlight=23)]

Оператор `ModelState.Remove` является обязательным, так как `ModelState` имеет старое значение `RowVersion`. На странице Razor значение `ModelState` для поля имеет приоритет над значениями свойств модели, если они присутствуют вместе.

## <a name="update-the-edit-page"></a>Обновление страницы редактирования

Обновите *Pages/Departments/Edit.cshtml*, используя следующую разметку:

[!code-html[](intro/samples/cu/Pages/Departments/Edit.cshtml?highlight=1,14,16-17,37-39)]

Предыдущая разметка:

* Изменяет директиву `page` с `@page` на `@page "{id:int}"`.
* Добавляет версию скрытых строк. Нужно добавить `RowVersion`, чтобы обратная передача привязывала значение.
* Отображает последний байт `RowVersion` в целях отладки.
* Заменяет `ViewData` на строго типизированный `InstructorNameSL`.

## <a name="test-concurrency-conflicts-with-the-edit-page"></a>Тестирование конфликтов параллелизма с использованием страницы "Edit" (Редактирование)

Откройте в браузере два экземпляра страницы "Edit" (Редактирование) для кафедры английского языка:

* Запустите приложение и выберите "Departments" (Кафедры).
* Щелкните правой кнопкой мыши гиперссылку **Edit** (Изменить) для кафедры английского языка и выберите пункт **Открыть на новой вкладке**.
* На первой вкладке щелкните гиперссылку **Edit** (Изменить) для кафедры английского языка.

На обеих вкладках браузера отображаются одинаковые сведения.

Измените имя на первой вкладке браузера и нажмите кнопку **Save** (Сохранить).

![Страница "Edit" (Редактирование) кафедры 1 после изменения](concurrency/_static/edit-after-change-1.png)

В браузере отображается страница индекса с измененным значением и обновленным индикатором rowVersion. Обратите внимание на обновленный индикатор rowVersion, он отображается при второй обратной передаче на другой вкладке.

Измените другое поле на второй вкладке браузера.

![Страница "Edit" (Редактирование) кафедры 2 после изменения](concurrency/_static/edit-after-change-2.png)

Нажмите кнопку **Сохранить**. Отображаются сообщения об ошибках для всех полей, которые не соответствуют значениям базы данных:

![Сообщение об ошибке для страницы "Edit" (Редактирование) кафедры](concurrency/_static/edit-error.png)

В этом окне браузера не планировалось изменять поле "Name" (Имя). Скопируйте и вставьте текущее значение "Languages" (Языки) в поле "Name" (Имя). Выйдите из поля с помощью клавиши TAB. Проверка на стороне клиента удаляет сообщение об ошибке.

![Сообщение об ошибке для страницы "Edit" (Редактирование) кафедры](concurrency/_static/cv.png)

Снова нажмите кнопку **Save** (Сохранить). Сохраняется значение, введенное на второй вкладке браузера. Сохраненные значения отображаются на странице индекса.

## <a name="update-the-delete-page"></a>Обновление страницы удаления

Обновите страницу "Delete" (Удаление) с помощью следующего кода:

[!code-csharp[](intro/samples/cu/Pages/Departments/Delete.cshtml.cs)]

Страница "Delete" (Удаление) обнаруживает конфликты параллелизма при изменении сущности после ее получения. `Department.RowVersion` является версией строки при получении сущности. Когда EF Core создает команду SQL DELETE, она включает предложение WHERE с `RowVersion`. Если команда SQL DELETE не затрагивает ни одной строки:

* `RowVersion` в команде SQL DELETE не соответствует `RowVersion` в базе данных.
* Возникает исключение DbUpdateConcurrencyException.
* Вызывается `OnGetAsync` с `concurrencyError`.

### <a name="update-the-delete-page"></a>Обновление страницы удаления

Измените *Pages/Departments/Delete.cshtml*, используя следующий код:

[!code-html[](intro/samples/cu/Pages/Departments/Delete.cshtml?highlight=1,10,39,51)]

Приведенная выше разметка вносит следующие изменения:

* Изменяет директиву `page` с `@page` на `@page "{id:int}"`.
* Добавляет сообщение об ошибке.
* Заменяет FirstMidName на FullName в поле **Administrator** (Администратор).
* Изменяет `RowVersion` для отображения последнего байта.
* Добавляет версию скрытых строк. Нужно добавить `RowVersion`, чтобы обратная передача привязывала значение.

### <a name="test-concurrency-conflicts-with-the-delete-page"></a>Тестирование конфликтов параллелизма с использованием страницы "Delete" (Удаление)

Создайте тестовую кафедру.

Откройте в браузере два экземпляра страницы "Delete" (Удаление) для тестовой кафедры:

* Запустите приложение и выберите "Departments" (Кафедры).
* Щелкните правой кнопкой мыши гиперссылку **Delete** (Удалить) для тестовой кафедры и выберите пункт **Открыть на новой вкладке**.
* Щелкните гиперссылку **Edit** (Изменить) для тестовой кафедры.

На обеих вкладках браузера отображаются одинаковые сведения.

Измените бюджет на первой вкладке браузера и нажмите кнопку **Save** (Сохранить).

В браузере отображается страница индекса с измененным значением и обновленным индикатором rowVersion. Обратите внимание на обновленный индикатор rowVersion, он отображается при второй обратной передаче на другой вкладке.

Удалите тестовую кафедру со второй закладки. Отображается ошибка параллелизма с текущими значениями из базы данных. При нажатии кнопки **Delete** (Удалить) сущность удаляется, если только не был обновлен `RowVersion` или не была удалена кафедра.

Сведения о том, как наследовать модель данных, см. в разделе [Наследование](xref:data/ef-mvc/inheritance).

### <a name="additional-resources"></a>Дополнительные ресурсы

* [Токены параллелизма в EF Core](/ef/core/modeling/concurrency)
* [Обработка параллелизма в EF Core](/ef/core/saving/concurrency)
* [Версия руководства на YouTube (обработка конфликтов параллелизма)](https://youtu.be/EosxHTFgYps)
* [Версия руководства на YouTube (часть 2)](https://www.youtube.com/watch?v=kcxERLnaGO0)
* [Версия руководства на YouTube (часть 3)](https://www.youtube.com/watch?v=d4RbpfvELRs)

> [!div class="step-by-step"]
> [Назад](xref:data/ef-rp/update-related-data)
