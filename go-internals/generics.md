# Generics: type parameters, constraints, type sets и реальные каверзы

## 1. Обобщения в Go — проверка ограничений

`func Max[T cmp.Ordered](a, b T) T` компилируется лишь для типов, удовлетворяющих constraint. Constraint описывает допустимый **набор типов и операций**, а не просто «набор методов» обычного runtime-интерфейса.

```go
func First[T any](s []T) (T, bool) {
    var zero T
    if len(s) == 0 { return zero, false }
    return s[0], true
}
```

`zero` имеет zero value **конкретного T**, а не обязательно nil. Для `int` это 0, для `*User` — nil.

## 2. `any` vs `comparable`

`any` — любой тип; `comparable` ограничивает допустимые операции с использованием `==/!=`. Не забывай, что динамическое содержимое интерфейсного значения может быть несравнимым и вызвать panic при сравнении, даже когда статические правила позволяют использовать interface как comparable.

Для ключей generic map обычно `K comparable`:

```go
func Has[K comparable, V any](m map[K]V, k K) bool {
    _, ok := m[k]
    return ok
}
```

## 3. `~` в type set

`~int` включает типы, у которых underlying type — `int`. Например:

```go
type UserID int
type Integer interface { ~int | ~int64 }
func Double[T Integer](v T) T { return v + v }
fmt.Println(Double(UserID(3))) // 6
```

Без `~` constraint `int | int64` не включает пользовательские именованные типы с таким underlying type.

## 4. Методы и type parameters зависят от версии

До Go 1.27 объявления методов не могли вводить собственные дополнительные type parameters; Go 1.27 добавил поддержку generic methods. **На собеседовании всегда уточняй версию**, если обсуждаете синтаксис новых возможностей. Это не значит, что любая произвольная форма higher-kinded types теперь появилась: ограничения спецификации остаются.

## 5. Мономорфизация и runtime costs — не догма

Не утверждай, что Go всегда генерирует полностью отдельный машинный код для каждой комбинации типов или всегда реализует generics через interface boxing: стратегия компилятора и оптимизации могут использовать dictionaries, shape-based специализацию и инлайнинг. Семантика задаётся спецификацией, а эффективность проверяется `-benchmem` и профилированием.

## 6. Type assertion на параметре типа

Непосредственное `v.(T)`, где `v` — значение type parameter, не всегда допустимо так, как для интерфейсного выражения; если нужен runtime type switch, можно рассматривать `any(v).(type)` и затем аккуратно вести себя с именованными типами. Но часто лучше решать задачу ограничениями типа, а не рефлексией.

## 7. Не всегда generics — лучший API

Если нужна runtime-полиморфия по поведению объектов, интерфейсы часто лучше. Generics удобнее для алгоритмов/контейнеров с типизированными значениями. Избыток type parameters может сделать API сложнее — оцени реальную пользу типизации.

## 8. Вопросы

Чем `any` отличается от `comparable`? Почему нужен `~int` для `UserID`? Почему `var zero T` не всегда nil? Когда generics лучше интерфейса? Какие возможности generic methods зависят от версии Go?

Источники: [Go spec — type parameters](https://go.dev/ref/spec#Type_parameter_declarations), [Go 1.27 release notes](https://go.dev/doc/go1.27).

## Новые каверзы: type sets и generic methods в Go 1.27

### 9. Generic method теперь допустим, но не реализует метод интерфейса

Go 1.27 разрешает **собственные type parameters в методах конкретных типов**, но интерфейсный method set по-прежнему не содержит generic methods.

```go
type Box struct{}
func (Box) Identity[T any](v T) T { return v } // Go 1.27+
func main() {
    var b Box
    fmt.Println(b.Identity(42)) // 42
}
```

**Каверза:** нельзя объявить interface с `Identity[T any]` как контракт обычного интерфейса; generic concrete method нельзя считать реализацией «такого» интерфейсного метода. Для полиморфизма над поведением нужны обычные методы интерфейса или generic функции/типы.

### 10. `comparable` — не гарантия «никогда не паниковать» при динамическом сравнении

Generic функция с `T comparable` может быть инстанцирована типом интерфейса, в котором динамически лежит `[]int`. Тогда сравнение двух таких значений может вызвать panic: статическое удовлетворение constraint не делает **динамическое содержимое** сравнимым.

### 11. Type set — не runtime union

`interface{ ~int | ~int64 }` используется **как constraint**, но не как обычный interface для произвольной runtime переменной `var x interface{ ~int | ~int64 }`. Это ограничение языка: non-basic interfaces, содержащие union/type terms, применяются только в соответствующих позициях ограничений.

### 12. Inference не безграничен

`func Zero[T any]() T { var z T; return z }` не выводит `T` по отсутствующим аргументам в произвольной вызовной позиции: `Zero()` требует конкретизации, например `Zero[int]()` (если контекст сам не предоставляет разрешённого правила вывода). С Go 1.27 расширено inference generic function values в assignment contexts; это **не отменяет** всех требований к указанию типа.

### 13. Не обещать более быстрый машинный код

У Go generics нет универсальной гарантии «полная мономорфизация каждой инстанциации». Инлайнинг, shape-specialization, dictionaries и аллокации — вопросы версии компилятора. `go test -benchmem` и профилирование решают, что выгоднее для реального кода.

Источники: [Generic Methods, Go team](https://go.dev/blog/generic-methods), [Go 1.27 release notes](https://go.dev/doc/go1.27), [Go spec](https://go.dev/ref/spec#Interface_types).
