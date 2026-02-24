# Мой ответ на собеседовании: «Расскажи про select»

---

**Select** — это встроенная конструкция Go для **мультиплексирования операций с каналами**. Позволяет горутине ожидать на нескольких каналах одновременно и реагировать на тот, который станет готов первым. По сути — это `switch`, но для каналов.

---

**По внутреннему устройству** — select компилируется в вызовы `runtime.selectgo()`:

```go
// runtime/select.go (упрощённо)
func selectgo(cas0 *scase, order0 *uint16, ncases int) (int, bool)

type scase struct {
    c    *hchan         // канал
    elem unsafe.Pointer // указатель на данные для send/recv
    kind uint16         // send, recv, или default
}
```

Runtime делает следующее:
1. **Перемешивает** порядок проверки case'ов (для fairness)
2. **Сортирует** каналы по адресу (для консистентного порядка блокировки — избежание deadlock)
3. Проходит по всем case'ам — если какой-то готов, выполняет его
4. Если ни один не готов и есть `default` — выполняет default
5. Если ни один не готов и нет `default` — ставит горутину в **очереди ожидания всех каналов** и засыпает

---

## Базовый синтаксис

```go
select {
case v := <-ch1:
    // получили v из ch1
case ch2 <- 42:
    // отправили 42 в ch2
case v, ok := <-ch3:
    // получили v, ok показывает открыт ли канал
default:
    // ни один канал не готов
}
```

---

## Ключевые свойства

### Первое — случайный выбор при нескольких готовых case'ах

```go
ch1 := make(chan int, 1)
ch2 := make(chan int, 1)
ch1 <- 1
ch2 <- 2

select {
case <-ch1:
    fmt.Println("ch1")
case <-ch2:
    fmt.Println("ch2")
}
// Выведет ch1 ИЛИ ch2 — случайно!
```

Это **намеренно** — предотвращает starvation. Если бы выбирался первый готовый в порядке написания, нижние case'ы могли бы никогда не выполниться.

### Второе — без default select блокируется

```go
// Блокирующий select — ждёт пока хоть один канал будет готов
select {
case v := <-ch1:
    // ...
case v := <-ch2:
    // ...
}
// Горутина спит, пока ch1 или ch2 не станут готовы
```

### Третье — с default становится non-blocking

```go
// Non-blocking проверка
select {
case v := <-ch:
    fmt.Println("got:", v)
default:
    fmt.Println("channel not ready")
}
// Никогда не блокируется!
```

Это идиоматичный способ **попробовать** операцию без блокировки.

### Четвёртое — nil каналы игнорируются

```go
var ch1 chan int  // nil
ch2 := make(chan int, 1)
ch2 <- 42

select {
case <-ch1:
    // НИКОГДА не выберется — nil канал блокирует навсегда
case <-ch2:
    fmt.Println("ch2")  // всегда сюда
}
```

Это **фича**, а не баг — позволяет **динамически отключать case'ы**:

```go
func merge(ch1, ch2 <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for ch1 != nil || ch2 != nil {
            select {
            case v, ok := <-ch1:
                if !ok {
                    ch1 = nil  // отключаем case — больше не будет выбираться
                    continue
                }
                out <- v
            case v, ok := <-ch2:
                if !ok {
                    ch2 = nil
                    continue
                }
                out <- v
            }
        }
    }()
    return out
}
```

### Пятое — пустой select блокируется навсегда

```go
select {}  // блокирует горутину навсегда
```

Иногда используется в `main()` чтобы не дать программе завершиться, пока другие горутины работают. Но обычно лучше использовать `sync.WaitGroup` или done-канал.

### Шестое — каждый case должен быть канальной операцией

```go
select {
case <-ch:      // ✅ receive
case ch <- v:   // ✅ send
case x := <-ch: // ✅ receive with assignment
default:        // ✅ default
case x > 5:     // ❌ ошибка компиляции — не канальная операция
}
```

---

## Типичные паттерны

### 1. Таймаут

```go
select {
case result := <-ch:
    fmt.Println("got:", result)
case <-time.After(5 * time.Second):
    fmt.Println("timeout!")
}
```

### 2. Отмена через context

```go
func worker(ctx context.Context, jobs <-chan Job) {
    for {
        select {
        case <-ctx.Done():
            fmt.Println("cancelled:", ctx.Err())
            return
        case job := <-jobs:
            process(job)
        }
    }
}
```

### 3. Non-blocking send/receive

```go
// Non-blocking send
select {
case ch <- v:
    // отправили
default:
    // канал полон, делаем что-то другое
}

// Non-blocking receive
select {
case v := <-ch:
    // получили
default:
    // канал пуст
}
```

### 4. Heartbeat / периодические действия

```go
func worker(ctx context.Context) {
    ticker := time.NewTicker(1 * time.Second)
    defer ticker.Stop()
    
    for {
        select {
        case <-ctx.Done():
            return
        case <-ticker.C:
            doPeriodicWork()
        case job := <-jobs:
            processJob(job)
        }
    }
}
```

### 5. Fan-in (слияние каналов)

```go
func fanIn(channels ...<-chan int) <-chan int {
    out := make(chan int)
    var wg sync.WaitGroup
    
    for _, ch := range channels {
        wg.Add(1)
        go func(c <-chan int) {
            defer wg.Done()
            for v := range c {
                out <- v
            }
        }(ch)
    }
    
    go func() {
        wg.Wait()
        close(out)
    }()
    
    return out
}
```

### 6. Приоритетный select (workaround)

Select не имеет приоритетов, но можно эмулировать:

```go
// Сначала пробуем высокоприоритетный канал
select {
case v := <-highPriority:
    handle(v)
default:
    // Если high пуст — проверяем оба
    select {
    case v := <-highPriority:
        handle(v)
    case v := <-lowPriority:
        handle(v)
    }
}
```

---

## Подводные камни

**Первое** — **утечка горутин** при забытом таймауте/отмене:

```go
// ❌ Плохо — если канал никогда не получит данные, горутина висит вечно
go func() {
    result := <-ch
    process(result)
}()

// ✅ Хорошо — с отменой
go func() {
    select {
    case result := <-ch:
        process(result)
    case <-ctx.Done():
        return
    }
}()
```

**Второе** — **time.After в цикле** создаёт новый таймер каждую итерацию:

```go
// ❌ Плохо — утечка таймеров (GC очистит, но нагрузка на runtime)
for {
    select {
    case <-ch:
        // ...
    case <-time.After(5 * time.Second):
        // ...
    }
}

// ✅ Хорошо — переиспользуем таймер
timer := time.NewTimer(5 * time.Second)
defer timer.Stop()

for {
    select {
    case <-ch:
        if !timer.Stop() {
            <-timer.C
        }
        timer.Reset(5 * time.Second)
    case <-timer.C:
        // timeout
        timer.Reset(5 * time.Second)
    }
}
```

**Третье** — **закрытый канал** всегда готов к чтению:

```go
ch := make(chan int)
close(ch)

select {
case v := <-ch:
    // Сразу сюда! v = 0, это zero value
}
```

Поэтому проверяем `ok`:

```go
select {
case v, ok := <-ch:
    if !ok {
        // канал закрыт
        return
    }
    process(v)
}
```

---

## Сравнение с другими конструкциями

| Конструкция | Назначение |
|-------------|------------|
| `switch` | Выбор по значению |
| `select` | Выбор по готовности каналов |
| `for` + `range ch` | Последовательное чтение из одного канала |
| `select` | Параллельное ожидание нескольких каналов |

---

*Вот в целом основное. Если нужно, могу углубиться — внутренности runtime.selectgo, reflect.Select для динамического количества case'ов, паттерны graceful shutdown.*
