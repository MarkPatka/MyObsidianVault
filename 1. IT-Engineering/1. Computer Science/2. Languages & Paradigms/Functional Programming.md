**Функциональное программирование** — это парадигма программирования, которая акцентируется на вычислении через функции в математическом стиле, неизменяемость, выразительность и уменьшение использования переменных и состояний ([ссылка](https://www.raywenderlich.com/82599/swift-functional-programming-tutorial)).  
  
Существует 6 основных концепций:  

1. концепция первого класса и функций высшего порядка
2. концепция чистых функций
3. концепция неизменяемого состояния
4. концепция опциональности и сопоставления с образом
5. концепция ленивости и бесконечных структур данных
6. концепция лямбда-исчислений

## Функция первого класса

#### Что это
  
Это сущность, которая поддерживает операции, обычно доступные для других сущностей. Эти операции, как правило. включают в себя: передачу сущности как аргумента, возвращение сущности из функции и присваивание оной в переменную.  
  
#### Чем полезна
  
Упрощает работу с функциями, давая больше возможностей и способов для их использования.  
  
#### Примеры использования

```fsharp
// Define the callback function type
type CallbackFunc = string -> unit

// Recursive function to list files
let rec listFiles (path: string) (callback: CallbackFunc) =
    // recursive read directory structure ...
    // Example implementation:
    let entries = System.IO.Directory.GetFileSystemEntries(path)
    for entry in entries do
        if System.IO.Directory.Exists(entry) then
            listFiles entry callback
        else
            callback entry

// Function to print file path
let printFilePath (filePath: string) =
    printfn "%s" filePath

// Function that returns the print function
let getPrintFunc () : CallbackFunc =
    printFilePath

// Call listFiles with the callback
listFiles "/tmp/" (getPrintFunc())
```

В приведенном примере функция `get_print_func` создает переменную, в которой хранит ссылку на функцию, а затем возвращает ее. А ниже по коду мы передаем результат, возвращенный нам функцией `get_print_func` в другую функцию. Это и есть операции, доступные нам благодаря функциям первого класса.

## Функция высшего порядка

#### Что это
  
Это функция, которая оперирует другими функциями. Оперирует, получая их в качестве параметра или возвращая их.  
  
#### Чем полезна

Как и в случае функций первого порядка, эта концепция дает нам больше возможностей для работы с функциями. Также эта концепция открывает нам возможность использования функций как обработчиков событий, о которых система или какая-либо библиотека может нам сообщать посредством вызова переданной ей функции первого класса.  
  
#### Примеры использования

Cм. предыдущий пример. Здесь есть функция, которая читает директорию. И рекурсивно читает все подпапки. Для каждого найденного файла вызывает функцию, которую мы передали — `callback`.  


## Чистая функция

#### Что это
  
Это функция, которая выполняет три условия.
1. ✅ Всегда возвращать одинаковый результат для одинаковых аргументов
2. ✅ Не иметь побочных эффектов (не изменять внешнее состояние, не выполнять I/O)
3. ✅ Не зависеть от внешнего изменяемого состояния
  
#### Чем полезна

Чистая функция, обычно, является показателем хорошо написанного кода, так как такие функции легко покрывать тестами, их можно легко переносить и повторно использовать.  
  
#### Примеры использования
  
В приведенном ниже фрагменте показаны примеры чистых функций.

```fsharp
// quad1 с вложенной функцией, которая захватывает x из внешней области
let quad1 (x: int) : int =
    let square () : int =
        x * x
    
    square () * square ()

// quad2 с вложенной чистой функцией, которая принимает параметр
let quad2 (x: int) : int =
    let square (y: int) : int =
        y * y
    
    square x * square x

// Чистая функция возведения в квадрат
let square (x: int) : int =
    x * x

// Функция возведения в куб, использующая square
let cube (x: int) : int =
    square x * x

// Функция с побочным эффектом (печать)
let printSquareOf (x: int) : unit =
    printfn "%d" (square x)

// Изменяемая переменная на уровне модуля
let mutable screenScale = 2.0

// Функция, которая использует внешнее изменяемое состояние
let pixelsToScreenPixels (pixels: int) : int =
    pixels * int screenScale
```

##### ✅ Чистые функции:

1. **`quad1`** - чистая (всегда возвращает одинаковый результат для одинакового `x`)
2. **`quad2`** - чистая
3. **`square`** - чистая
4. **`cube`** - чистая

##### ❌ Не чистые функции:

5. **`printSquareOf`** - **нечистая**, потому что:
    - Выполняет побочный эффект (вывод в консоль)
    - Возвращает `unit` (void), что часто признак нечистоты
    - При вызове с одним и тем же аргументом изменяет внешний мир
6. **`pixelsToScreenPixels`** - **нечистая**, потому что:
    - Зависит от внешнего изменяемого состояния (`screenScale`)
    - Если `screenScale` изменится, результат для того же `pixels` будет другим
    - Нарушает принцип "одинаковый вход → одинаковый выход

## Неизменяемое состояние

#### Что это

Неизменяемое состояние — состояние объекта, которое не может быть изменено после того, как объект был создан. Под состоянием объекта здесь, подразумевается набор значений его свойств.  
  
#### Чем полезно

Так как неизменяемые объекты гарантируют нам, что на протяжении своего жизненного цикла они не могут менять свое состояние, то мы можем быть уверены, что использование или передача таких объектов в другие места программы не приведет к каким либо непредвиденным последствиям. Это особенно важно при работе в многопоточном окружении.

#### Примеры использования

Пример использования неизменяемых значений в F#:
```fsharp
// Простые неизменяемые значения
let one = 1
// one <- 2  // Ошибка компиляции: "This value is not mutable"

let hello = "hello"
// hello <- "bye"  // Ошибка компиляции

// Неизменяемый список
let argv = ["uptime"; "--help"]
// argv <- ["man"; "reboot"]  // Ошибка компиляции
// argv.[0] <- "man"  // Ошибка компиляции: списки в F# неизменяемы

// Неизменяемый массив
let numbers = [| 1; 2; 3 |]
// numbers <- [| 4; 5; 6 |]  // Ошибка: нельзя переназначить
// НО! Элементы массива можно изменять:
numbers.[0] <- 10  // Это работает для массивов!

// Неизменяемая запись (record)
type Person = { Name: string; Age: int }
let john = { Name = "John"; Age = 30 }
// john <- { Name = "Jane"; Age = 25 }  // Ошибка компиляции
// john.Name <- "Jane"  // Ошибка: поля записи неизменяемы по умолчанию

// Создание нового значения на основе старого (не мутация!)
let two = one + 1  // one остается 1
let goodbye = hello + " and goodbye"  // hello остается "hello"
let newArgv = "man" :: argv  // argv не изменился, создан новый список
```
**По умолчанию всё неизменяемо**: Нужно явно указать `mutable` для изменяемости
```fsharp
// Если нужна изменяемость - явно указываем mutable
let mutable counter = 0
counter <- counter + 1  // Теперь работает!
```


## Pattern Matching
  
Pattern Matching — акт проверки последовательности токенов на совпадение с определенным шаблоном.  

#### Чем полезен
  
Позволяет нам писать более краткий, сосредоточенный на решении проблемы код.  
  
#### Примеры использования
```fsharp
// 1. Базовый пример - сопоставление с конкретными значениями
let describNumber x =
    match x with
    | 0 -> "ноль"
    | 1 -> "один"
    | 2 -> "два"
    | _ -> "другое число"  // _ это "всё остальное"

printfn "%s" (describNumber 1)  // "один"
printfn "%s" (describNumber 5)  // "другое число"


// 2. Сопоставление со списками
let describeList list =
    match list with
    | [] -> "пустой список"
    | [x] -> sprintf "один элемент: %d" x
    | [x; y] -> sprintf "два элемента: %d и %d" x y
    | x :: xs -> sprintf "первый элемент %d, остальных %d" x (List.length xs)

printfn "%s" (describeList [])           // "пустой список"
printfn "%s" (describeList [5])          // "один элемент: 5"
printfn "%s" (describeList [1; 2; 3])    // "первый элемент 1, остальных 2"


// 3. Сопоставление с условиями (guards)
let classifyAge age =
    match age with
    | x when x < 0 -> "некорректный возраст"
    | x when x < 18 -> "ребенок"
    | x when x < 65 -> "взрослый"
    | _ -> "пенсионер"

printfn "%s" (classifyAge 15)   // "ребенок"
printfn "%s" (classifyAge 30)   // "взрослый"


// 4. Сопоставление с кортежами
let describePoint (x, y) =
    match (x, y) with
    | (0, 0) -> "начало координат"
    | (0, _) -> "на оси Y"
    | (_, 0) -> "на оси X"
    | (x, y) when x = y -> "на диагонали"
    | _ -> "обычная точка"

printfn "%s" (describePoint (0, 0))   // "начало координат"
printfn "%s" (describePoint (3, 3))   // "на диагонали"


// 5. Практический пример - светофор
type TrafficLight = Red | Yellow | Green

let whatToDo light =
    match light with
    | Red -> "Стоп!"
    | Yellow -> "Приготовься"
    | Green -> "Езжай"

printfn "%s" (whatToDo Red)     // "Стоп!"
printfn "%s" (whatToDo Green)   // "Езжай"
```

#### Почему pattern matching лучше чем if/else:
```fsharp
// Вместо этого:
let describeNumberOld x =
    if x = 0 then "ноль"
    elif x = 1 then "один"
    elif x = 2 then "два"
    else "другое число"

// Используем pattern matching - читается как таблица соответствий!
let describeNumber x =
    match x with
    | 0 -> "ноль"
    | 1 -> "один"
    | 2 -> "два"
    | _ -> "другое число"
```

**Преимущества:**

- ✅ Более читаемый код
- ✅ Компилятор проверяет, что все случаи обработаны
- ✅ Можно сопоставлять сложные структуры данных
- ✅ Можно извлекать данные прямо в процессе сопоставления

## Ленивость или ленивые вычисления

#### Что это
  
Ленивое вычисление — стратегия вычисления, которая откладывает вычисления выражения до момента, когда значение этого выражения будет необходимо.    

#### Чем полезно

Позволяет отложить вычисление некоторого кода до определенного или заранее неопределенного момента времени.  
  
#### Пример использования
```fsharp
open System

// 1. Базовый пример с lazy
type Post = {
    Id: int
    Title: string
    CreationDate: string
    // lazy значение - вычисляется только при первом обращении
    CreatedAt: Lazy<DateTime option>
}

let createPost id title creationDate =
    {
        Id = id
        Title = title
        CreationDate = creationDate
        CreatedAt = lazy (
            printfn "Парсим дату..." // Выведется только при первом обращении!
            DateTime.TryParse(creationDate) 
            |> function | (true, date) -> Some date | _ -> None
        )
    }

let post = createPost 1 "Hello F#" "2024-11-24"
printfn "Пост создан"
// Дата ещё не распарсена!

let date1 = post.CreatedAt.Value  // Здесь произойдёт вычисление
// Выведет: "Парсим дату..."

let date2 = post.CreatedAt.Value  // Повторное обращение - просто возвращает результат
// "Парсим дату..." НЕ выведется


// 2. Более показательный пример - дорогие вычисления
type ExpensiveReport = {
    Name: string
    Data: Lazy<int list>
}

let createReport name =
    {
        Name = name
        Data = lazy (
            printfn "Выполняем ОЧЕНЬ дорогую операцию для отчета '%s'..." name
            System.Threading.Thread.Sleep(2000) // Имитация долгой работы
            [1..1000000] |> List.filter (fun x -> x % 2 = 0)
        )
    }

let report = createReport "Продажи"
printfn "Отчет создан мгновенно!"

// Данные вычислятся только здесь (через 2 секунды):
// let data = report.Data.Value


// 3. Ленивая последовательность (seq) - ещё мощнее!
let lazyNumbers = 
    seq {
        for i in 1..1000000 do
            printfn "Генерируем число %d" i
            yield i * i
    }

printfn "Последовательность создана, но числа ещё не сгенерированы!"

// Числа генерируются по требованию:
let first5 = lazyNumbers |> Seq.take 5 |> Seq.toList
// Выведет только "Генерируем число 1" ... "Генерируем число 5"


// 4. Практический пример - загрузка конфигурации
type AppConfig = {
    DatabaseConnection: Lazy<string>
    ApiKeys: Lazy<Map<string, string>>
}

let loadConfig () =
    {
        DatabaseConnection = lazy (
            printfn "Подключаемся к базе данных..."
            System.Threading.Thread.Sleep(1000)
            "Server=localhost;Database=mydb"
        )
        ApiKeys = lazy (
            printfn "Загружаем API ключи из файла..."
            System.Threading.Thread.Sleep(500)
            Map.ofList [("google", "key123"); ("azure", "key456")]
        )
    }

let config = loadConfig()
printfn "Конфигурация создана без задержек!"

// Подключение к БД произойдёт только здесь:
// let db = config.DatabaseConnection.Value

// API ключи загрузятся только здесь:
// let keys = config.ApiKeys.Value
```

## Бесконечная структура данных
  
Бесконечная структура данных — структура, чье определение дано в терминах бесконечных диапазонов или непрекращающейся рекурсии, но реальные значения вычисляются только в момент, когда они необходимы.  
  
#### Чем полезна

Позволяет нам определять структуры данных бесконечной или огромной величины, не затрачивая ресурсы на вычисление значений этой структуры.  
  
#### Примеры использования
```fsharp
// 1. Бесконечная последовательность натуральных чисел
let infiniteNumbers = Seq.initInfinite id
// 0, 1, 2, 3, 4, 5, ...

let first10 = infiniteNumbers |> Seq.take 10 |> Seq.toList
// [0; 1; 2; 3; 4; 5; 6; 7; 8; 9]


// 2. Бесконечная последовательность Фибоначчи
let fibonacci = 
    Seq.unfold (fun (a, b) -> Some(a, (b, a + b))) (0, 1)
// 0, 1, 1, 2, 3, 5, 8, 13, 21, 34, ...

let fib20 = fibonacci |> Seq.take 20 |> Seq.toList


// 3. Бесконечная последовательность случайных чисел
let randomNumbers =
    let rnd = System.Random()
    Seq.initInfinite (fun _ -> rnd.Next(100))

let random5 = randomNumbers |> Seq.take 5 |> Seq.toList


// 4. Бесконечный цикл значений
let colors = seq {
    while true do
        yield "Red"
        yield "Green"
        yield "Blue"
}
// Red, Green, Blue, Red, Green, Blue, ...

let first7Colors = colors |> Seq.take 7 |> Seq.toList
// ["Red"; "Green"; "Blue"; "Red"; "Green"; "Blue"; "Red"]


// 5. Бесконечная последовательность степеней двойки
let powersOf2 = Seq.initInfinite (fun n -> pown 2 n)
// 1, 2, 4, 8, 16, 32, 64, 128, ...

let first8Powers = powersOf2 |> Seq.take 8 |> Seq.toList


// 6. Практический пример - генератор ID
let idGenerator startFrom =
    Seq.initInfinite (fun i -> startFrom + i)

let userIds = idGenerator 1000
let nextUserId = userIds |> Seq.head  // 1000
let next5Ids = userIds |> Seq.take 5 |> Seq.toList
// [1000; 1001; 1002; 1003; 1004]


// 7. Бесконечный поток времени
let timestamps = 
    Seq.initInfinite (fun _ -> 
        System.Threading.Thread.Sleep(1000)
        System.DateTime.Now
    )

// Получить первые 3 метки времени с интервалом в 1 секунду:
// let first3 = timestamps |> Seq.take 3 |> Seq.toList


// 8. Сложный пример - простые числа (решето Эратосфена)
let rec sieve numbers = seq {
    let prime = Seq.head numbers
    yield prime
    yield! sieve (numbers |> Seq.filter (fun x -> x % prime <> 0))
}

let primes = sieve (Seq.initInfinite (fun i -> i + 2))
// 2, 3, 5, 7, 11, 13, 17, 19, 23, 29, ...

let first10Primes = primes |> Seq.take 10 |> Seq.toList


// 9. Рекурсивное определение бесконечной последовательности
let rec ones = seq {
    yield 1
    yield! ones
}
// 1, 1, 1, 1, 1, 1, ...


// 10. Практический пример - пагинация
type Page<'T> = {
    Items: 'T list
    PageNumber: int
}

let fetchPage pageNum =
    // Имитация запроса к API
    { Items = [pageNum * 10 .. pageNum * 10 + 9]; PageNumber = pageNum }

let allPages = Seq.initInfinite fetchPage
// Страница 0, Страница 1, Страница 2, ...

let first3Pages = allPages |> Seq.take 3 |> Seq.toList


// 11. Комбинирование бесконечных последовательностей
let evenNumbers = Seq.initInfinite (fun i -> i * 2)
let oddNumbers = Seq.initInfinite (fun i -> i * 2 + 1)

// Чередование четных и нечетных
let alternating = 
    Seq.zip evenNumbers oddNumbers
    |> Seq.collect (fun (even, odd) -> [even; odd])
// 0, 1, 2, 3, 4, 5, 6, 7, ...

let first12 = alternating |> Seq.take 12 |> Seq.toList
```

##### Ключевые функции для работы с бесконечными структурами:
```fsharp
// Создание бесконечных последовательностей:
Seq.initInfinite    // По индексу
Seq.unfold          // Разворачивание состояния
seq { while true }  // Явный бесконечный цикл

// Потребление бесконечных последовательностей:
Seq.take            // Взять N элементов
Seq.takeWhile       // Взять пока условие истинно
Seq.head            // Первый элемент
Seq.skip            // Пропустить N элементов
Seq.filter          // Фильтрация (осторожно!)
Seq.map             // Преобразование
```

## Лямбда-исчисления

#### Что это
  
Лямбда исчисление — формальная система в математической логике для выражения вычисления на основе операций аппликации и абстракции функций при помощи связывания и замены переменных.  
#### Чем полезны
  
Концепция лямбда-исчисления приносит в языки программирования понятие анонимных функций, которые могут захватывать внешние (по отношению к функции) переменные.  
#### Пример использования
```fsharp
// 1. Простые лямбда-функции (анонимные функции)
let square = fun x -> x * x
square 5  // 25

let add = fun x y -> x + y
add 3 7  // 10


// 2. Лямбды с функциями высшего порядка
let numbers = [1; 2; 3; 4; 5]

numbers |> List.map (fun x -> x * 2)
// [2; 4; 6; 8; 10]

numbers |> List.filter (fun x -> x > 3)
// [4; 5]

numbers |> List.fold (fun acc x -> acc + x) 0
// 15


// 3. Каррирование (currying) - частичное применение
let multiply = fun x -> fun y -> x * y
let double = multiply 2  // частично применили
double 5  // 10

// Короткая запись того же самого:
let add3 x y z = x + y + z
let add5 = add3 2 3  // частично применили два аргумента
add5 10  // 15


// 4. Композиция функций
let addOne = fun x -> x + 1
let timesTwo = fun x -> x * 2

// Композиция: сначала addOne, потом timesTwo
let addOneThenDouble = addOne >> timesTwo
addOneThenDouble 5  // (5 + 1) * 2 = 12

// Обратная композиция
let doubleThenAddOne = timesTwo >> addOne
doubleThenAddOne 5  // (5 * 2) + 1 = 11


// 5. Замыкания (closures)
let makeAdder n = 
    fun x -> x + n  // лямбда "захватывает" n

let add10 = makeAdder 10
let add100 = makeAdder 100

add10 5   // 15
add100 5  // 105


// 6. Лямбды как аргументы
let applyTwice f x = f (f x)

applyTwice (fun x -> x * 2) 3
// (3 * 2) * 2 = 12

applyTwice (fun s -> s + "!") "Hello"
// "Hello!!"


// 7. Y-комбинатор (рекурсия через лямбды)
let rec factorial n =
    if n <= 1 then 1
    else n * factorial (n - 1)

// Та же рекурсия через лямбду:
let fact = 
    fun self n ->
        if n <= 1 then 1
        else n * self self (n - 1)

fact fact 5  // 120


// 8. Комбинаторы (I, K, S)
// I комбинатор (identity) - возвращает аргумент как есть
let I = fun x -> x
I 42  // 42

// K комбинатор (constant) - возвращает первый аргумент, игнорирует второй
let K = fun x y -> x
K 5 999  // 5

// S комбинатор (substitution)
let S = fun f g x -> f x (g x)
S (fun x y -> x + y) (fun x -> x * 2) 3
// 3 + (3 * 2) = 9


// 9. Чистое лямбда-исчисление - булевы значения
let TRUE = fun x y -> x
let FALSE = fun x y -> y

let IF = fun cond then_val else_val -> cond then_val else_val

IF TRUE "yes" "no"   // "yes"
IF FALSE "yes" "no"  // "no"
```

##### Каррирование
Также концепция ламбда-исчисления привносит в языки программирования понятие каррирования. **Каррирование** позволяет нам разбить функцию с несколькими параметрами на несколько функций с одним параметром. Это дает нам возможность получать результат вычисления промежуточных функций и применять к этим функциям разные аргументы для получения нескольких результатов