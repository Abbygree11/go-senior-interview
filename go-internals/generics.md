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