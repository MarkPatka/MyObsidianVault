### Пример 1

1. Положить в вазочку N конфет. 
2. Загнуть на левой руке 3 пальца. 
3. Бросить монетку на стол. 
4. Если выпал орёл, увеличить количество конфет в вазочке в А раз, иначе добавить в вазочку В конфет. 
5. Разогнуть один палец на левой руке. 
6. Если на левой руке остались загнутые пальцы, перейти к п.3.

```plantuml
@startuml
actor "Человек" as Human
participant "Вазочка" as Bowl
participant "Монетка" as Coin
participant "Левая рука" as LeftHand

Human -> Bowl : Положить N конфет
Human -> LeftHand : Загнуть 3 пальца

loop Пока есть загнутые пальцы
    Human -> Coin : Бросить на стол
    alt Выпал орёл
        Human -> Bowl : Увеличить конфеты в A раз
    else Выпала решка
        Human -> Bowl : Добавить B конфет
    end
    Human -> LeftHand : Разогнуть один палец
end

@enduml
```

### Пример 2

Вы принц. И для выбора наиболее подходящей невесты вы организовали смотр кандидаток. Придумайте и опишите в виде блок-схемы алгоритм для решения этой задачи. 
- На вход подается N кандидаток. 
- Необходимо выбрать одну - самую красивую. 
- Операция сравнения красоты двух претенденток задана функцией `SelectMoreAttractiveCandidate(candidate_x, candidate_y)`.

```plantuml
@startuml
start

:Get list of N candidates;
if (N == 0) then (yes)
  :Bride is not chosen - there are no candidates;
  stop
endif

:Set first candidate as currentCandidate;
:Set the following candidate index i = 2;

while (i <= N?) is (true)
  :Take candidate by index i;
  :Set current = SelectMoreAttractiveCandidate(
    candidate_i,
    currentCandidate
  );
  :i = i + 1;
endwhile (i > N?)

:Return current;

stop
@enduml
```

### Пример 3
Создать UML для алгоритма сортировки яиц

Дано: 
- две калибровочных лунки (маленькая и большая), 
- лоток с яйцами

Надо разделить яйца на высшую, первую и вторую категории 
Псевдокод алгоритма:
```
Нач 
	Пока в лотке есть яйца 
Нц 
	Взять яйцо из лотка 
	Опустить в большую лунку 
	Если яйцо не пролезает То Положить яйцо в корзину для высшей категории Иначе Опустить яйцо в маленькую лунку 
		Если яйцо не пролезает То Положить яйцо в корзину для первой категории 
		Иначе Положить яйцо в корзину для второй категории 
		Всё 
	Всё 
Кц 
Кон
```

UML:
```plantuml
@startuml
start

:Get list of N eggs (Egg Tray);
if (N == 0) then (yes)
    :There are no eggs in the tray;
    stop
endif

:Set HighestCategoryEggsList as bucket for biggest eggs;
:Set FirstCategoryEggsList as bucket for medium eggs;
:Set SecondCategoryEggsList as bucket for small eggs;
:Set egg index i = 0;

while (i < N)
    if (FitBigHole(egg[i])?) then (yes)
        :Add egg[i] to HighestCategoryEggsList;
    else (no)
        if (FitMediumHole(egg[i])?) then (yes)
            :Add egg[i] to FirstCategoryEggsList;
        else (no)
            :Add egg[i] to SecondCategoryEggsList;
        endif
    endif
    :i = i + 1;
endwhile

stop
@enduml
```

