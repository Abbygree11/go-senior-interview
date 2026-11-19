# Интерфейсы (`interface`, `any`, `error`): динамический тип, typed nil, методы, сравнение

## 1. Модель: динамический тип + значение

Интерфейсное значение концептуально описывается парой `(dynamic type, dynamic value)`. `nil`-интерфейс означает **отсутствие динамического типа и значения**. Конкретная реализация runtime может содержать метаданные и указатели, но не стоит обещать на собеседовании, что «оба поля всегда указатели на пользовательские объекты».

```go
var x any
fmt.Println(x == nil) // true

var p *int = nil
x = p
fmt.Println(x == nil) // false: dynamic type = *int
```

## 2. Typed nil — ловушка с `error`

```go
type MyError struct{ msg string }
func (e *MyError) Error() string { return e.msg }

func run() error {
    var p *MyError = nil
    return p
}
func main() {
    fmt.Println(run() == nil) // false
}
```

В `error` лежит `(*MyError, nil)`, а не `(nil,nil)`. Явное преобразование `return error(p)` **ничего не исправляет**. Правильно:

```go
func run(ok bool) error {
    if ok { return nil }
    return &MyError{msg: "failed"}
}
```

**Дополнительная опасность:** вызов метода на typed nil может запаниковать при разыменовании receiver; сама проверка `err != nil` допустима, но логически вводит в заблуждение.

## 3. `T` и `*T`: method set

```go
type N struct{}
func (N) A() {}
func (*N) B() {}

type I interface{ B() }
var _ I = (*N)(nil) // OK
// var _ I = N{}     // не компилируется
```

У `T` method set содержит методы с value receiver; у `*T` — методы с value и pointer receiver. Даже если `n.B()` компилируется для адресуемой переменной `n`, это **не доказывает**, что `N` реализует `I`. Элемент map и динамическое интерфейсное значение не всегда адресуемы для подобного автоматического преобразования.

## 4. Реализация интерфейса не требует явного `implements`

Go использует структурную реализацию: тип удовлетворяет интерфейсу по множеству методов. Учитывай имена, сигнатуры и пакетную принадлежность неэкспортируемых методов.

`any` не означает отсутствие динамического типа. Type assertion `v.(T)` проверяет именно динамическое значение; `v.(T)` без `, ok` при несовпадении вызывает panic.

## 5. `switch v := x.(type)` и nil

```go
var x any = (*int)(nil)
switch v := x.(type) {
case nil:
    fmt.Println("nil interface")
case *int:
    fmt.Println("typed *int", v == nil) // typed *int true
}
```

`case nil` срабатывает только для **nil-интерфейса**, не для typed nil.

## 6. Сравнение интерфейсов может паниковать

```go
var a any = []int{1}
var b any = []int{1}
// fmt.Println(a == b) // panic: comparing uncomparable type []int
```

Интерфейсы сравнимы синтаксически, но если их одинаковые динамические типы несравнимы (например, slice/map), при сравнении может произойти panic. Поэтому `map[any]V` тоже может паниковать при попытке использовать несравнимый динамический ключ.

## 7. Value receiver копирует значение

Если метод объявлен как `func (x T) M()`, на вызове receiver передаётся по значению. Если `T` содержит `sync.Mutex`, копирование после начала использования может нарушить синхронизацию. Обычно для мутируемых структур и типов с lock используют pointer receiver.

## 8. Приведение к интерфейсу и аллокации

Передача значения в интерфейс **не гарантирует** heap allocation. Escape analysis решает размещение по контексту использования; конкретное представление и boxing — оптимизационные детали. Измерять `-gcflags=-m=2` и `-benchmem`, а не судить по одной сигнатуре.

## 9. Ошибки `errors.Is/As`

`err == target` не проходит через цепочку обёрток. `errors.Is(err, target)` проверяет совместимость с целевой ошибкой по unwrap-цепочке; `errors.As` находит ошибку подходящего типа. Обёртывай через `fmt.Errorf("...: %w", err)` при необходимости сохранить идентичность причины. `%v` не создаёт unwrap-связь.

## 10. Контрольные случаи

- Почему `return (*MyError)(nil)` не равно `return nil` при типе результата `error`?
- Когда `T` реализует интерфейс, а только `*T` — нет?
- Почему `var x any = []int{1}; x == x` паникует?
- Чем проверка `case nil` в type switch отличается от проверки значения под `case *T`?
- Почему у конвертации к интерфейсу нет универсального правила «всегда аллокация»?

Источники: [Go spec — interface types](https://go.dev/ref/spec#Interface_types), [method sets](https://go.dev/ref/spec#Method_sets), [comparisons](https://go.dev/ref/spec#Comparison_operators), [errors](https://pkg.go.dev/errors).