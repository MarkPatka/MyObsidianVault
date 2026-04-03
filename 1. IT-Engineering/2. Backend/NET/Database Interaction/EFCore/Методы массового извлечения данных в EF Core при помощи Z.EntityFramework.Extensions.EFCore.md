_Время чтения: 6 минут_

Работая с Entity Framework Core, разработчикам часто требуется извлекать несколько сущностей из базы данных на основе коллекции идентификаторов или значений.

Типичный подход — использование метода `Contains`:
```csharp
var productIds = GetProductIdsForSync();

var products = await dbContext.Products
    .Where(p => productIds.Contains(p.Id))
    .ToListAsync();
```
Этот подход хорошо работает для небольших коллекций, но создает серьезные проблемы при работе с большими наборами данных:

- **Снижение производительности** — Даже при наличии правильных индексов SQL-запрос `WHERE IN` замедляется с увеличением количества параметров.
  
- **Превышение лимита параметров** — SQL Server имеет жесткое ограничение в 2100 параметров на запрос. Хотя можно обойти это, разбивая запросы на пакеты в SQL, это не всегда практично при использовании EF Core, особенно в сложных сценариях с дополнительными фильтрами и соединениями.
  
- **Проблемы с памятью и подключениями** — Множественные обращения к базе данных потребляют больше памяти и дольше удерживают подключения открытыми, что может привести к исчерпанию пула подключений.

Существует лучшее решение: библиотека **Entity Framework Extensions** предоставляет специализированные методы для массового извлечения данных, которые решают все эти проблемы.
## Проблема с методом `Contains`

Все примеры в этой статье тестировались на базе данных SQL Server.

Метод `Contains` в EF Core транслируется в SQL-запрос с `WHERE IN`.

Когда вы передаете коллекцию из 10 000 идентификаторов товаров, EF Core генерирует запрос с 10 000 параметрами:

```csharp
var productIds = await GetProductIdsFromExternalSystem();

var products = await dbContext.Products
    .Where(p => productIds.Contains(p.Id))
    .ToListAsync();
```

```sql
SELECT * FROM Products
WHERE Id IN (@p0, @p1, @p2, ... @p9999)
```

Этот подход имеет критические ограничения:
**Лимит параметров SQL Server** — SQL Server допускает максимум 2100 параметров в одном запросе. Если вы попытаетесь запросить более 2100 товаров, EF Core выбросит исключение:

```
System.Data.SqlClient.SqlException: The incoming request has too many parameters.
The server supports a maximum of 2100 parameters.
```

В качестве обходного пути можно разбить запрос на пакеты:

```csharp
var batchSize = 2000;
var allProducts = new List<Product>();

for (int i = 0; i < productIds.Count; i += batchSize)
{
    var batch = productIds.Skip(i).Take(batchSize).ToList();
    var products = await dbContext.Products
        .Where(p => batch.Contains(p.Id))
        .ToListAsync();
    
    allProducts.AddRange(products);
}
```

Однако это решение создает множественные обращения к базе данных, дольше удерживает подключения открытыми и затрудняет применение дополнительных фильтров или `Include` ко всем пакетам.

В SQL можно использовать временную таблицу или параметр табличного типа, чтобы обойти ограничение. Тем не менее, эти подходы требуют написания "сырого" SQL и лишают преимуществ строго типизированных запросов EF Core, отслеживания изменений и поддержки навигационных свойств.

**Entity Framework Extensions** — это библиотека, расширяющая EF Core высокопроизводительными массовыми операциями. Хотя она хорошо известна операциями массовой вставки, обновления и удаления, она также предоставляет мощные методы для массового извлечения данных.

Чтобы начать работу с Entity Framework Extensions, установите следующий пакет NuGet:

```bash
dotnet add package Z.EntityFramework.Extensions.EFCore
```

Библиотека предоставляет специализированные методы, которые внутренне используют временные таблицы для обхода ограничений на параметры и повышения производительности. Вместо передачи тысяч параметров в `WHERE IN` эти методы:

1.  Создают временную таблицу в базе данных.
2.  Вставляют значения для фильтрации в эту временную таблицу.
3.  Соединяют таблицу ваших сущностей с временной таблицей.
4.  Возвращают отфильтрованные результаты.
5.  Автоматически очищают временную таблицу.

Этот подход значительно улучшает производительность запросов для больших наборов данных.

Библиотека Entity Framework Extensions предлагает пять основных методов для массового извлечения данных:

1. `WhereBulkContains` — фильтрует сущности с помощью LINQ-запроса, используя все элементы из существующего списка. Будут возвращены только элементы базы данных, соответствующие элементам в вашем списке.
2. `WhereBulkNotContains` — фильтрует сущности с помощью LINQ-запроса, исключая все элементы из существующего списка.
3. `BulkRead` — фильтрует сущности с помощью LINQ-запроса, включая все элементы из существующего списка, и затем немедленно возвращает результат.
4. `WhereBulkContainsFilterList` — возвращает элементы из списка, которые уже существуют в базе данных.
5.  `WhereBulkNotContainsFilterList` — возвращает элементы из списка, которых нет в базе данных. Это обратный метод к `WhereBulkContainsFilterList`.

## Метод 1: `WhereBulkContains`

Метод `WhereBulkContains` является наиболее часто используемым методом массового извлечения. Он извлекает сущности, у которых указанное свойство соответствует любому значению в коллекции.

Этот метод особенно полезен для сценариев синхронизации товаров, где вам нужно получить тысячи товаров из вашей базы данных для сравнения с данными из внешней системы, API или импорта файла.

```csharp
// @nuget: Z.EntityFramework.Extensions.EFCore
using Z.EntityFramework.Extensions;

var productIds = await GetProductIdsFromExternalSystem();

var products = await dbContext.Products
    .WhereBulkContains(productIds)
    .ToListAsync();
```

Метод автоматически использует первичный ключ сущности для сопоставления элементов. В этом примере он соединяет по `Id`.

Вы также можете фильтровать, указав любое свойство в качестве второго параметра:

```csharp
// @nuget: Z.EntityFramework.Extensions.EFCore
using Z.EntityFramework.Extensions;

var products = await GetProductsFromExternalSystem();

var products = await dbContext.Products
    .WhereBulkContains(products, x => x.Id)
    .ToListAsync();
```

Первый параметр — это коллекция значений для фильтрации, а второй параметр — лямбда-выражение, указывающее, какое свойство сопоставлять.

Поддерживаются все виды списков. Единственное требование — ваш список должен быть простого типа или содержать свойство с тем же именем, что и ключ:

*   Простой тип, например `List<int>` и `List<Guid>`
*   Тип сущности, например `List<Customer>`
*   Анонимный тип
*   Список объектов `ExpandoObject`

**Фильтрация по свойствам, не являющимся первичным ключом:**
Вы можете использовать `WhereBulkContains` с любым свойством, а не только с первичным ключом:

```csharp
// @nuget: Z.EntityFramework.Extensions.EFCore
using Z.EntityFramework.Extensions;

var productCodes = new List<string>
{
    "SKU-001", "SKU-002", "SKU-003", // ...
};

var products = await dbContext.Products
    .WhereBulkContains(productCodes, x => x.ProductCode)
    .ToListAsync();
```

Вы можете фильтровать сразу по нескольким свойствам:

```csharp
// @nuget: Z.EntityFramework.Extensions.EFCore
using Z.EntityFramework.Extensions;

var products = context.Products
    .WhereBulkContains(deserializedProducts, x => new { x.SupplierCode, x.ProductCode })
    .ToListAsync();
```

`WhereBulkContains` возвращает `IQueryable<T>`, который можно объединять с другими LINQ-методами, такими как `Where`, `Select` и `Include`:

```csharp
// @nuget: Z.EntityFramework.Extensions.EFCore
using Z.EntityFramework.Extensions;

var productIds = GetProductIdsForSync();

var activeProducts = await dbContext.Products
    .Where(p => p.IsActive && p.Stock > 0)
    .WhereBulkContains(productIds, x => x.Id)
    .Select(p => new
    {
        p.Id,
        p.Name,
        p.Price,
        CategoryName = p.Category.Name
    })
    .ToListAsync();
```

Это намного более гибко, чем разделение запросов на пакеты, где применение дополнительных фильтров и `Include` между пакетами становится сложным и подверженным ошибкам.

## Метод 2: `WhereBulkNotContains`

Метод `WhereBulkNotContains` является противоположностью `WhereBulkContains`. Он извлекает сущности, у которых указанное свойство **не** соответствует ни одному значению в коллекции.

Этот метод полезен, когда вам нужно найти сущности, отсутствующие во внешней системе, или когда вы хотите исключить большой набор элементов из результатов запроса.

```csharp
// @nuget: Z.EntityFramework.Extensions.EFCore
using Z.EntityFramework.Extensions;

var discontinuedProductIds = await GetDiscontinuedProductIdsFromExternalSystem();

var activeProducts = await dbContext.Products
    .WhereBulkNotContains(discontinuedProductIds, x => x.Id)
    .ToListAsync();
```

Это извлекает все товары, кроме тех, что находятся в коллекции `discontinuedProductIds`, не превышая лимит параметров.

Как и `WhereBulkContains`, этот метод можно объединять с другими LINQ-операциями:

```csharp
// @nuget: Z.EntityFramework.Extensions.EFCore
using Z.EntityFramework.Extensions;

var processedProductIds = await GetProcessedProductIds();

var remainingProducts = await dbContext.Products
    .Where(p => p.RequiresProcessing)
    .WhereBulkNotContains(processedProductIds, x => x.Id)
    .OrderBy(p => p.Priority)
    .Take(1000)
    .ToListAsync();
```

## Метод 3: `BulkRead`

Метод `BulkRead` — это удобный метод, который объединяет `WhereBulkContains` с немедленным выполнением.

Он извлекает из базы данных сущности, соответствующие элементам вашего списка, и немедленно возвращает результаты в виде `List<T>`.

Он работает так же, как `WhereBulkContains`, но не требует объединения с другими методами для возврата результатов:

```csharp
// @nuget: Z.EntityFramework.Extensions.EFCore
using Z.EntityFramework.Extensions;

var deserializedProducts = DeserializeProductsFromJson(); // Возвращает 10 000 товаров

// Немедленно извлечь соответствующие товары из базы данных
var products = context.Products.BulkRead(deserializedProducts);
```

Разница в том, что `BulkRead` немедленно возвращает результаты, в то время как `WhereBulkContains` возвращает `IQueryable<T>`, который можно объединять с дополнительными LINQ-операциями.

Доступна также асинхронная версия:

```csharp
var deserializedProducts = DeserializeProductsFromJson();
var products = await context.Products.BulkReadAsync(deserializedProducts);
```

## Метод 4: `WhereBulkContainsFilterList`

Метод `WhereBulkContainsFilterList` фильтрует ваш входной список, возвращая только те элементы, которые уже существуют в базе данных.

Это отличается от `WhereBulkContains`, который возвращает сущности из базы данных. Этот метод возвращает элементы из вашего входного списка (не сущности базы данных).

Этот метод особенно полезен, когда вам нужно разделить ваши данные на существующие и несуществующие элементы, например, отделить записи, которые нужно вставить, от тех, которые нужно обновить.

```csharp
// @nuget: Z.EntityFramework.Extensions.EFCore
using Z.EntityFramework.Extensions;

var deserializedProducts = DeserializeProductsFromJson(); // Возвращает 20 000 товаров

// Возвращает элементы из deserializedProducts, которые существуют в базе данных
var existingProducts = context.Products
    .WhereBulkContainsFilterList(deserializedProducts)
    .ToList();
```

Метод возвращает элементы из вашего входного списка (не сущности базы данных), которые соответствуют строкам в базе данных.

Вы можете указать, какое свойство использовать для сопоставления:

```csharp
// @nuget: Z.EntityFramework.Extensions.EFCore
using Z.EntityFramework.Extensions;

var existingProducts = context.Products
    .WhereBulkContainsFilterList(deserializedProducts, x => x.ProductCode)
    .ToList();

// Или использовать строку
var existingProducts = context.Products
    .WhereBulkContainsFilterList(deserializedProducts, "ProductCode")
    .ToList();

// Или использовать список строк
var existingProducts = context.Products
    .WhereBulkContainsFilterList(deserializedProducts, new List<string> { "ProductCode" })
    .ToList();
```

Этот метод поддерживает фильтрацию по нескольким свойствам.

### Метод 5: `WhereBulkNotContainsFilterList`

Метод `WhereBulkNotContainsFilterList` является обратным к `WhereBulkContainsFilterList`. Он фильтрует ваш входной список, возвращая только те элементы, которых **нет** в базе данных.

Этот метод возвращает элементы из вашего входного списка (не сущности базы данных).

Этот метод полезен, когда вы хотите определить новые записи, которые нужно вставить, или когда вам нужно найти элементы из вашего импорта, отсутствующие в базе данных.

```csharp
// @nuget: Z.EntityFramework.Extensions.EFCore
using Z.EntityFramework.Extensions;

var deserializedProducts = DeserializeProductsFromJson(); // Возвращает 20 000 товаров

// Возвращает элементы из deserializedProducts, которых нет в базе данных
var notExistingProducts = context.Products
    .WhereBulkNotContainsFilterList(deserializedProducts)
    .ToList();
```

Метод возвращает элементы из вашего входного списка (не сущности базы данных), которые **не** соответствуют строкам в базе данных.

Вы можете указать, какое свойство использовать для сопоставления.

Этот метод поддерживает фильтрацию по нескольким свойствам.