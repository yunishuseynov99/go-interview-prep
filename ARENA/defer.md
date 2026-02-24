

# Мой ответ на собеседовании: «Расскажи про defer»

---

**Defer** — это встроенный механизм Go, который **откладывает выполнение функции до момента возврата из окружающей функции**. Используется для гарантированной очистки ресурсов, освобождения блокировок, закрытия файлов — всего, что должно произойти «при выходе», независимо от того, каким путём функция завершилась.

---

**По внутреннему устройству** — каждая горутина хранит **связный список defer-записей** в структуре `runtime._defer`:

```go
// runtime/runtime2.go (упрощённо)
type _defer struct {
    siz     int32    // размер аргументов
    started bool
    sp      uintptr  // stack pointer вызывающей функции
    pc      uintptr  // program counter
    fn      *funcval // отложенная функция
    _panic  *_panic  // паника, вызвавшая defer (если есть)
    link    *_defer  // → следующий defer в цепочке (стек)
}
```

С Go 1.14 компилятор применяет **open-coded defers** — для простых случаев defer инлайнится прямо в функцию с битовой маской, без аллокации `_defer` структуры. Это сделало defer практически **бесплатным** по производительности.

---

## Три главных правила defer

### Правило 1: Аргументы вычисляются **немедленно**

```go
func main() {
    x := 10
    defer fmt.Println(x)  // x вычисляется СЕЙЧАС (= 10)
    x = 20
}
// Вывод: 10 (не 20!)
```

Аргументы **копируются в момент регистрации** defer, а не в момент вызова. Но если нужно «поймать» значение на момент выхода — используем замыкание или указатель:

```go
// Замыкание — захватывает переменную по ссылке:
func main() {
    x := 10
    defer func() { fmt.Println(x) }()  // замыкание видит x
    x = 20
}
// Вывод: 20

// Указатель:
func main() {
    x := 10
    defer fmt.Println(&x)  // адрес копируется, но данные по нему — актуальные
    x = 20
}
```

### Правило 2: Порядок — **LIFO** (стек)

```go
func main() {
    defer fmt.Println("first")
    defer fmt.Println("second")
    defer fmt.Println("third")
}
// Вывод:
// third
// second
// first
```

Последний зарегистрированный defer выполняется **первым**. Логика простая — стек. Это важно например при вложенных блокировках:

```go
mu1.Lock()
defer mu1.Unlock()  // разблокируется последним
mu2.Lock()
defer mu2.Unlock()  // разблокируется первым
// Правильный порядок разблокировки: mu2 → mu1
```

### Правило 3: Defer может **читать и изменять** именованные возвращаемые значения

```go
func example() (result int) {
    defer func() {
        result = result * 2  // модифицирует возвращаемое значение!
    }()
    return 10
}
// Возвращает 20, а не 10
```

Порядок выполнения `return`:
1. Вычисляется выражение, результат присваивается в именованную переменную (`result = 10`)
2. Выполняются defer-функции (`result = result * 2` → `result = 20`)
3. Функция реально возвращает `result`

Это активно используется для **модификации ошибок**:
```go
func doWork() (err error) {
    tx, err := db.Begin()
    if err != nil {
        return err
    }
    defer func() {
        if err != nil {
            tx.Rollback()  // откатываем при ошибке
        } else {
            err = tx.Commit()  // коммитим, и если Commit вернёт ошибку — она станет результатом
        }
    }()
    // работа с tx...
}
```

---

## Defer и паника

Defer выполняется **даже при panic**. Именно поэтому `recover` работает только внутри defer:

```go
func safeFunction() {
    defer func() {
        if r := recover(); r != nil {
            fmt.Println("recovered:", r)
        }
    }()
    panic("something went wrong")
}
// Не крашится, выводит: recovered: something went wrong
```

Порядок при панике:
1. Функция паникует
2. Все defer'ы выполняются в LIFO-порядке
3. Если ни один defer не вызвал `recover` — паника **пробрасывается выше**
4. Если дошла до верха горутины — `fatal`, программа падает

---

## Типичные подводные камни

**Первое** — defer **в цикле** не выполняется до выхода из функции, а не из итерации:

```go
// ❌ Плохо — все файлы откроются, но закроются только при выходе из функции
func processFiles(names []string) {
    for _, name := range names {
        f, _ := os.Open(name)
        defer f.Close()  // накапливается! Утечка ресурсов
    }
}

// ✅ Хорошо — вынести в отдельную функцию
func processFiles(names []string) {
    for _, name := range names {
        processOne(name)
    }
}
func processOne(name string) {
    f, _ := os.Open(name)
    defer f.Close()  // закроется при выходе из processOne
}
```

**Второе** — defer с **методом на nil**:
```go
var f *os.File  // nil
defer f.Close() // аргумент (f=nil) вычисляется сейчас
                // panic при вызове Close в момент выхода
```

**Третье** — **os.Exit** не вызывает defer'ы:
```go
func main() {
    defer fmt.Println("bye")  // НЕ выполнится!
    os.Exit(0)
}
```

**Четвёртое** — не путать регистрацию и вызов замыкания:
```go
defer myFunc()   // ✅ вызовет myFunc при выходе
defer myFunc     // ❌ ошибка компиляции — это не вызов
```

---

## Типичные паттерны

```go
// 1. Освобождение ресурсов
f, err := os.Open("file.txt")
if err != nil { return err }
defer f.Close()

// 2. Unlock мьютексов
mu.Lock()
defer mu.Unlock()

// 3. Замер времени
func measure() {
    start := time.Now()
    defer func() {
        fmt.Println("elapsed:", time.Since(start))
    }()
    // работа...
}

// 4. Recover из паники
defer func() {
    if r := recover(); r != nil {
        log.Printf("panic: %v\n%s", r, debug.Stack())
    }
}()

// 5. Трассировка входа-выхода
func bigOperation() {
    defer trace("bigOperation")()  // обрати внимание: ()() !
    // ...
}
func trace(name string) func() {
    start := time.Now()
    log.Printf("enter %s", name)
    return func() { log.Printf("exit %s (%v)", name, time.Since(start)) }
}
```

---

## Производительность

| Версия Go | Стоимость defer |
|---|---|
| Go 1.12 и раньше | ~35 нс (heap-аллокация `_defer`) |
| Go 1.13 | ~6 нс (stack-allocated defers) |
| Go 1.14+ | **~0 нс** (open-coded defers, инлайн с битовой маской) |

В Go 1.14+ простой defer компилируется в обычный код с проверкой флага — **нулевой оверхед** в happy path.

---

*Вот в целом основное. Если нужно, могу углубиться — open-coded defers, взаимодействие defer/panic/recover в деталях, escape analysis для defer-замыканий.*
