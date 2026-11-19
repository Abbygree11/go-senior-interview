# Go под капотом — подробные разборы для Senior Go Backend

Здесь **каждой теме соответствует отдельный файл**. Это не список терминов и не краткие ответы, а учебные главы: **внутреннее устройство → конкретная семантика → неожиданные кейсы → объяснение результата → правильный приём → контрольные вопросы**. Большинство примеров можно перенести в `main.go` (добавив очевидные `package main` и импорты) и проверить.

**Граница точности.** Отделяем гарантии спецификации/API от деталей конкретного runtime. Для устаревающих кейсов используем версионные оговорки (Go 1.22+, Go 1.24+, Go 1.25+, Go 1.26–1.27). Не нужно учить наизусть число полей внутренней структуры, если оно не часть контракта.

## Язык и типы

| Отдельная глава | Что разобрано |
|---|---|
| [Слайсы](slices.md) | Заголовок `ptr/len/cap`, базовый массив, копирование, `append`, `copy`, `Clone`, full slice, aliasing, удержание памяти, `range` и гонки |
| [Массивы](arrays.md) | `[N]T` против `[]T`, копирование, указатели, `range` по массиву и сравнение |
| [Map](maps.md) | Хеш-таблица, `nil`, `comma-ok`, неадресуемость значений, итерация, runtime Swiss Tables, конкурентный доступ |
| [Интерфейсы](interfaces.md) | Динамический тип/значение, typed `nil`, `error`, method sets, pointer receivers, assertions, сравнение |
| [Строки](strings.md) | Байты, UTF-8, `rune`, графемы, конвертация, Unicode-нормализация, память |
| [Defer, panic, recover](defer-panic.md) | LIFO, захват аргументов, замыкания, именованные возвраты, циклы, cleanup, границы `recover` |

## Конкурентность и примитивы

| Отдельная глава | Что разобрано |
|---|---|
| [Горутины](goroutines.md) | Создание, завершение `main`, управление жизненным циклом, отмена, утечки, panic, семантика циклов |
| [GMP scheduler](scheduler.md) | G/M/P, локальные очереди, work-stealing, блокировки, netpoller, preemption, `GOMAXPROCS` |
| [Каналы](channels.md) | Runtime модель, буфер, блокировка, `nil`, закрытие, память, multiple writers, `cancel`, ownership |
| [Select](select.md) | Готовность веток, вычисление выражений, `default`, `nil`, закрытые каналы, fairness, приоритет отмены |
| [Mutex](mutex.md) | Lock/Unlock и happens-before, реентерабельность, копирование, методы-значения, deadlock, I/O под lock |
| [RWMutex](rwmutex.md) | Параллельные читатели, ожидающий writer, рекурсивный `RLock`, невозможность upgrade, benchmark |
| [Atomics](atomics.md) | SC memory ordering, `Load/Store/Add`, CAS, ABA, `atomic.Pointer`, `atomic.Value`, immutable snapshots |
| [WaitGroup](waitgroup.md) | Регистрация до запуска, `Wait`, double Done, `WaitGroup.Go`, канал results и deadlock |
| [Context](context.md) | Дерево отмены, `Done`/`Err`/`Cause`, deadline, утечки, контекст и отправка |
| [sync.Map](sync-map.md) | `LoadOrStore` и двойное вычисление, нет транзакции `Load+Store`, `Range`, выбор против map+mutex |
| [sync.Once](sync-once.md) | Инициализация, публикация, panic, ошибки, retry, рекурсивный Do |
| [sync.Cond](sync-cond.md) | Lock + predicate, цикл `Wait`, потерянный сигнал, Signal/Broadcast, отмена |
| [sync.Pool](sync-pool.md) | Время жизни объекта, GC, владение после Put, очистка данных, реальные бенчмарки |

## Память

| Отдельная глава | Что разобрано |
|---|---|
| [GC и escape analysis](garbage-collector.md) | Stack/heap, достижимость, write barriers, Green Tea GC (1.26+), `GOGC`, `GOMEMLIMIT`, retainers |

## Как готовиться

Читай одну тему за раз. Для каждого примера перед проверкой ответь себе:

1. **Что копируется** — данные, заголовок, указатель, сама структура?
2. **Кто владеет состоянием**, и кто вправе его менять/закрывать?
3. **Какая операция может заблокировать**, вызвать panic или data race?
4. **Что гарантирует Go**, а что лишь типичное поведение текущего runtime?
5. **Как изменится результат** при другой ёмкости, числе горутин или версии Go?

Проверяй непредсказуемые сценарии на работающем коде; для гонок используй `go test -race`. Одного удачного запуска недостаточно для доказательства корректности.

## Первоисточники

- [Go Language Specification](https://go.dev/ref/spec) — поведение типов, `range`, `select`, `defer` и выражений.
- [Go Memory Model](https://go.dev/ref/mem) — синхронизация и happens-before.
- [sync](https://pkg.go.dev/sync), [sync/atomic](https://pkg.go.dev/sync/atomic) — контракты примитивов.
- [Runtime HACKING](https://go.dev/src/runtime/HACKING) — модель G/M/P, внутренние механизмы.
- [Go 1.22](https://go.dev/doc/go1.22), [Go 1.24](https://go.dev/doc/go1.24), [Go 1.25](https://go.dev/doc/go1.25), [Go 1.26](https://go.dev/doc/go1.26), [Go 1.27](https://go.dev/doc/go1.27) — версионные изменения.

Существующий каталог [`topics/`](../topics/) сохраняется как **отдельная коллекция вопросов с эталонными ответами**, и не заменяет эти самостоятельные главы.