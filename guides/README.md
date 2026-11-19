# Под капотом Go: каверзы для Senior-собеседований

Этот раздел — **справочник по механизмам**, а не журнал ответов. Формат: **устройство → неожиданный эффект → как избежать**. Ориентация: Senior Go Backend, собеседования в российских компаниях.

## Навигация

1. [Runtime, горутины, память и GC](runtime-memory.md) — GMP, блокировки, scheduler, escape analysis, GC, finalizers.
2. [Concurrency primitives](concurrency-primitives.md) — channels, select, Mutex/RWMutex, atomics, sync.Map, WaitGroup, Once, Cond, Pool.
3. [Типы и коллекции](types-collections.md) — slices, arrays, maps, interface/typed nil, strings, defer, range.
4. [Backend, сеть и эксплуатация](backend-pitfalls.md) — context, HTTP, SQL, ресурсы, таймауты, goroutine leaks, profiling.

**Как читать:** объяснение описывает устойчивый контракт языка/API; детали внутренней реализации приведены для интуиции и могут измениться между версиями.

**Важно о версиях:** в Go 1.22 изменена семантика переменных цикла (в модулях с `go 1.22+`), в Go 1.24 изменена внутренняя реализация встроенной `map`. Не путать «как было реализовано когда-то» с гарантией языка.

## Основные первоисточники

- [Go language specification](https://go.dev/ref/spec)
- [Go memory model](https://go.dev/ref/mem)
- [sync](https://pkg.go.dev/sync), [sync/atomic](https://pkg.go.dev/sync/atomic), [context](https://pkg.go.dev/context)
- [runtime docs](https://pkg.go.dev/runtime), [runtime/HACKING](https://go.dev/src/runtime/HACKING)
- [Race detector](https://go.dev/doc/articles/race_detector)
- [Go 1.22 loop changes](https://go.dev/blog/loopvar-preview)
- [Go 1.24 release notes](https://go.dev/doc/go1.24)

## Рекомендуемая проверка

```bash
go test ./...
go test -race ./...
go test -run '^TestName$' -count=100 ./...
go test -bench=. -benchmem ./...
go vet ./...
```

`-race` обнаруживает многие реальные гонки **в исполнявшихся путях**, но отсутствие отчёта не доказывает отсутствие гонок. Бенчмарки не заменяют анализ корректности.
