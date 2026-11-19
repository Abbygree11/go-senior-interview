# `error`: typed nil, цепочки, `errors.Is/As`, `Join` и потерянные ошибки

## 1. `error` — интерфейс

`error` — интерфейс с методом `Error() string`. Конкретная ошибка может быть структурой или указателем на неё. Отсутствие ошибки обозначается **nil интерфейсом**, а не typed nil.

```go
type MyError struct{}
func (*MyError) Error() string { return "failed" }

func f() error {
    var p *MyError
    return p // ненулевой интерфейс (*MyError,nil)
}
fmt.Println(f() == nil) // false
```

Правильно: если ошибки нет, `return nil`; при наличии — `return &MyError{}`.

## 2. Обёртка ошибки и `%w`

```go
var ErrNotFound = errors.New("not found")
func repo() error {
    return fmt.Errorf("query user: %w", ErrNotFound)
}
fmt.Println(errors.Is(repo(), ErrNotFound)) // true
fmt.Println(repo() == ErrNotFound)          // false
```

`%w` создаёт unwrap-связь. `%v` форматирует текст, но не сохраняет идентичность ошибки. Нельзя строить обработку по сравнению строк `err.Error()`.

## 3. `errors.As` проверяет тип

`errors.As` извлекает ошибку подходящего типа из цепочки:

```go
var pathErr *os.PathError
if errors.As(err, &pathErr) {
    fmt.Println(pathErr.Path)
}
```

Ошибка может быть обёрнута несколькими слоями — type assertion `err.(*os.PathError)` проверяет только непосредственный динамический тип и может не сработать.

## 4. Несколько причин: `errors.Join`

```go
err := errors.Join(ErrNotFound, context.DeadlineExceeded)
fmt.Println(errors.Is(err, ErrNotFound))             // true
fmt.Println(errors.Is(err, context.DeadlineExceeded)) // true
```

`errors.Join(nil, nil) == nil`. `Join` полезен для аккуратного накопления ошибки работы и ошибки закрытия ресурса. Но не каждая ошибка должна размножаться: сначала реши, какая информация действительно нужна вызывающему коду.

## 5. Потерянная ошибка `Close`

```go
func writeFile(f *os.File) (err error) {
    defer func() {
        err = errors.Join(err, f.Close())
    }()
    _, err = f.WriteString("data")
    return err
}
```

Здесь именованный результат позволяет defer добавить ошибку закрытия к основной. Если закрытие важно для сохранности данных, `defer f.Close()` без проверки может скрыть проблему. Для read-only файла политика может быть проще.

## 6. Sentinel, typed и wrapped errors

Sentinel (`var ErrX = errors.New(...)`) удобен для стабильной машинной идентичности, но публичное API с большим количеством sentinel может слишком связывать слои. Typed error даёт структурированные поля и возможность `errors.As`. Wrapped error даёт контекст. Выбирай механизм по **публичному контракту**, не по моде.

## 7. Ошибки контекста

`ctx.Err()` возвращает `context.Canceled` либо `context.DeadlineExceeded` после отмены. `context.Cause(ctx)` при создании через cause-aware API может давать более подробную причину. Важно не заменять пользовательскую ошибку на context cancellation без анализа, что является первопричиной.

## 8. `panic` ≠ `error`

Ошибки ожидаемых сбоев (I/O, validation, network) обычно возвращаются как `error`. `panic` уместна для нарушенного инварианта/невозможного состояния и не должна заменять нормальный контроль ошибок. `recover` обрабатывает panic только в текущей горутине и не восстанавливает автоматически корректность разделяемого состояния.

## 9. Вопросы

Почему `return (*MyError)(nil)` даёт `err != nil`? Чем `errors.Is` отличается от `==`? Когда `errors.As` лучше type assertion? Зачем `errors.Join`? Почему игнорировать `Close` иногда опасно?

Источники: [errors](https://pkg.go.dev/errors), [fmt.Errorf](https://pkg.go.dev/fmt#Errorf), [context](https://pkg.go.dev/context).