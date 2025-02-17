---
title: Учебник. Использование ASP.NET MVC с EF Core. Реализация наследования
description: В этом учебнике показано, как реализовать наследование в модели данных с использованием платформы Entity Framework Core в приложении ASP.NET Core.
author: rick-anderson
ms.author: tdykstra
ms.custom: mvc
ms.date: 03/27/2019
ms.topic: tutorial
uid: data/ef-mvc/inheritance
ms.openlocfilehash: f80de595fd23cc9c1065e5257ad1d2376ea40cf3
ms.sourcegitcommit: 5b0eca8c21550f95de3bb21096bd4fd4d9098026
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/27/2019
ms.locfileid: "64886299"
---
# <a name="tutorial-implement-inheritance---aspnet-mvc-with-ef-core"></a>Учебник. Использование ASP.NET MVC с EF Core. Реализация наследования

В предыдущем учебнике была показана обработка исключений параллелизма. В этом учебнике демонстрируется, как реализовать наследование в модели данных.

В объектно-ориентированном программировании наследование применяется для оптимизации повторного использования кода. В рамках этого учебника вы измените классы `Instructor` и `Student` таким образом, чтобы они были производными от базового класса `Person`, который содержит общие свойства для преподавателей и учащихся, такие как `LastName`. Изменения вносятся в коде, а не на веб-страницах, и автоматически отражаются в базе данных.

В этом учебнике рассмотрены следующие задачи.

> [!div class="checklist"]
> * Сопоставление наследования с базой данных
> * Создание класса Person
> * Обновление Instructor и Student
> * Добавление Person в модель
> * Создание и обновление миграций
> * Тестирование реализации

## <a name="prerequisites"></a>Предварительные требования

* [Обработка параллелизма](concurrency.md)

## <a name="map-inheritance-to-database"></a>Сопоставление наследования с базой данных

Классы `Instructor` и `Student` в модели данных School имеют несколько идентичных свойств:

![Классы Student и Instructor](inheritance/_static/no-inheritance.png)

Предположим, что вам требуется исключить повторяющийся код для свойств, которые являются общими для сущностей `Instructor` и `Student`. Кроме того, вам может потребоваться написать службу, которая может форматировать имена как преподавателей, так и учащихся. В таких сценариях можно создать базовый класс `Person`, который содержит только общие свойства, а затем унаследовать от него классы `Instructor` и `Student`, как показано на следующем рисунке:

![Классы Student и Instructor, производные от класса Person](inheritance/_static/inheritance.png)

Структура наследования может быть представлена в базе данных несколькими способами. Вы можете создать таблицу Person, которая будет содержать одновременно информацию о преподавателях и учащихся. Некоторые столбцы могут относиться только к преподавателям (HireDate), некоторые только к учащимся (EnrollmentDate), а некоторые одновременно к обеим сущностям (LastName, FirstName). Как правило, используется столбец дискриминатора, который указывает на тип, представленный соответствующей строкой. Например, в столбце дискриминатора может указываться значение "Instructor" для преподавателей и "Student" для учащихся.

![Пример наследования типа "одна таблица на иерархию"](inheritance/_static/tph.png)

Такая модель, описывающая формирование структуры наследования сущностей на основе одной таблицы базы данных, называется наследованием типа "одна таблица на иерархию".

В качестве альтернативы можно создать базу данных, которая будет иметь приближенный к структуре наследования вид. Например, можно хранить в таблице Person только поля с именами и создать отдельные таблицы Instructor и Student с полями дат.

![Наследование типа «одна таблица на тип»](inheritance/_static/tpt.png)

Такая модель создания таблицы базы данных для каждого класса сущности называется наследованием типа "одна таблица на тип".

Кроме того, можно сопоставить все не являющиеся абстрактными типы с отдельными таблицами. Все свойства класса, включая унаследованные, сопоставляются со столбцами в соответствующей таблице. Такая модель называется наследованием типа "одна таблица на конкретный класс". Если реализовать наследование типа "одна таблица на конкретный класс" для показанных выше классов Person, Student и Instructor, таблицы Student и Instructor после реализации наследования будут выглядеть так же, как и до этого.

Модели наследования "одна таблица на иерархию" и "одна таблица на конкретный класс", как правило, обеспечивают более высокую производительность по сравнению с моделью "одна таблица на тип", поскольку при использовании последней могут выполняться сложные запросы на соединение.

В этом учебнике демонстрируется реализация модели наследования "одна таблица на иерархию". Платформа Entity Framework поддерживает только модель наследования "одна таблица на иерархию".  Вам необходимо создать класс `Person`, изменить классы `Instructor` и `Student` так, чтобы они были производными от класса `Person`, добавить новый класс в `DbContext`, после чего создать миграцию.

> [!TIP]
> Перед внесением изменений рекомендуется сохранить копию проекта.  В этом случае, если возникнут проблемы, вы сможете вернуться к сохраненному проекту вместо того, чтобы отменять выполненные в рамках этого учебника действия или начинать всю серию сначала.

## <a name="create-the-person-class"></a>Создание класса Person

В папке Models создайте файл Person.cs и замените код шаблона следующим:

[!code-csharp[](intro/samples/cu/Models/Person.cs)]

## <a name="update-instructor-and-student"></a>Обновление Instructor и Student

В файле *Instructor.cs* измените класс Instructor так, чтобы он был производным от класса Person, и удалите поля ключа и имени. Код будет выглядеть следующим образом:

[!code-csharp[](intro/samples/cu/Models/Instructor.cs?name=snippet_AfterInheritance&highlight=8)]

Выполните те же изменения в файле *Student.cs*.

[!code-csharp[](intro/samples/cu/Models/Student.cs?name=snippet_AfterInheritance&highlight=8)]

## <a name="add-person-to-the-model"></a>Добавление Person в модель

Добавьте тип сущности Person в файл *SchoolContext.cs*. Новые строки выделены.

[!code-csharp[](intro/samples/cu/Data/SchoolContext.cs?name=snippet_AfterInheritance&highlight=19,30)]

Это все, что требуется платформе Entity Framework для настройки наследования типа "одна таблица на иерархию". Как видно, после обновления базы данных в ней будет присутствовать таблица Person вместо таблиц Student и Instructor.

## <a name="create-and-update-migrations"></a>Создание и обновление миграций

Сохраните изменения и выполните сборку проекта. Затем откройте командное окно в папке проекта и введите следующую команду:

```console
dotnet ef migrations add Inheritance
```

На этом этапе не выполняйте команду `database update`. Ее выполнение приведет к потере данных, поскольку будет удалена таблица Instructor, а таблица Student будет переименована в Person. Для сохранения существующих данных потребуется настраиваемый код.

Откройте файл *Migrations/\<метка_времени>_Inheritance.cs* и замените метод `Up` следующим кодом:

[!code-csharp[](intro/samples/cu/Migrations/20170216215525_Inheritance.cs?name=snippet_Up)]

Этот код выполняет следующие задачи по обновлению базы данных:

* Удаляет ограничения внешнего ключа и индексы, которые указывают на таблицу Student.

* Переименовывает таблицу Instructor в Person и вносит изменения, необходимые для сохранения в ней данных из таблицы Student:

* Добавляет допускающий значения NULL тип EnrollmentDate для учащихся.

* Добавляет столбец дискриминатора, который указывает, представляет ли строка учащегося или преподавателя.

* Изменяет тип HireDate на допускающий значения NULL, поскольку в строках для учащихся не будет указываться дата приема на работу.

* Добавляет временное поле, которое будет использоваться для обновления внешних ключей, указывающих на учащихся. При копировании записей учащихся в таблицу Person им назначаются новые значения первичного ключа.

* Копирует данные из таблицы Student в таблицу Person. При этом записям учащихся назначаются новые значения первичного ключа.

* Исправляет значения внешнего ключа, которые указывают на учащихся.

* Повторно создает ограничения внешнего ключа и индексы, которые после этого указывают на таблицу Person.

(Если вместо целочисленного типа первичного ключа используется GUID, значения первичного ключа для записей учащихся изменять не потребуется и некоторые из этих действий можно пропустить.)

Выполните команду `database update`:

```console
dotnet ef database update
```

(В рабочей системе необходимо внести соответствующие изменения в метод `Down` на случай, если вам потребуется вернуться к предыдущей версии базы данных. В этом учебнике метод `Down` не используется.)

> [!NOTE]
> При изменении схемы в базе, содержащей существующие данные, возможны другие ошибки. Если вы получаете ошибки миграции, которые не удается устранить, измените имя базы данных в строке подключения или удалите базу данных. В новой базе не будет данных, которые требуется перенести, в результате чего команда обновления базы данных с большей долей вероятности завершится без ошибок. Чтобы удалить базу данных, используйте средство SSOX или выполните команду `database drop` в интерфейсе командной строки.

## <a name="test-the-implementation"></a>Тестирование реализации

Запустите приложение и попробуйте открыть различные страницы. Все работает так же, как и раньше.

В **обозревателе объектов SQL Server** разверните узел **Data Connections/SchoolContext** и затем **Таблицы**. Вы увидите, что вместо таблиц Student и Instructor появилась таблица Person. Откройте таблицу Person в конструкторе и убедитесь, что в ней представлены все столбцы, содержавшиеся в таблицах Student и Instructor.

![Таблица Person в окне SSOX](inheritance/_static/ssox-person-table.png)

Щелкните таблицу Person правой кнопкой мыши и выберите команду **Показать данные таблицы**, чтобы просмотреть столбец дискриминатора.

![Таблица Person в окне SSOX — данные таблицы](inheritance/_static/ssox-person-data.png)

## <a name="get-the-code"></a>Получение кода

[Скачайте или ознакомьтесь с готовым приложением.](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/data/ef-mvc/intro/samples/cu-final)

## <a name="additional-resources"></a>Дополнительные ресурсы

Дополнительные сведения о наследовании на платформе Entity Framework Core см. в разделе [Наследование](/ef/core/modeling/inheritance).

## <a name="next-steps"></a>Следующие шаги

В этом учебнике рассмотрены следующие задачи.

> [!div class="checklist"]
> * Сопоставление наследования с базой данных
> * Создание класса Person
> * Обновление Instructor и Student
> * Добавление Person в модель
> * Создание и обновление миграций
> * Тестирование реализации

В следующем учебнике описано, как использовать несколько сценариев Entity Framework с расширенными возможностями.

> [!div class="nextstepaction"]
> [Далее: Дополнительные разделы](advanced.md)
