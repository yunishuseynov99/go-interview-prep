

# Go для разработчиков — Полное руководство

> Курс по языку Go для разработчиков, переходящих с других языков. Предполагается знание основ программирования.

---

## Оглавление

- [Установка и Hello World](#установка-и-hello-world)
- [Переменные](#переменные)
- [Типы данных](#типы-данных)
  - [Целые числа (int)](#целые-числа-int)
  - [Биты и байты](#биты-и-байты)
  - [Строки (string)](#строки-string)
  - [Байт (byte)](#байт-byte)
  - [Руна (rune)](#руна-rune)
  - [Булевы значения (bool)](#булевы-значения-bool)
  - [Числа с плавающей точкой (float)](#числа-с-плавающей-точкой-float)
  - [Комплексные числа (complex)](#комплексные-числа-complex)
- [Составные типы данных](#составные-типы-данных)
  - [Структуры (struct)](#структуры-struct)
  - [Массивы и слайсы](#массивы-и-слайсы)
  - [Строки как слайсы](#строки-как-слайсы)
  - [Мапы (map)](#мапы-map)
- [Конвертация типов](#конвертация-типов)
- [Управление потоком (Flow Control)](#управление-потоком-flow-control)
  - [Условные конструкции (if/else)](#условные-конструкции-ifelse)
  - [Циклы (for)](#циклы-for)
  - [Range](#range)
  - [Switch](#switch)
- [Функции](#функции)
  - [Объявление функций](#объявление-функций)
  - [Функции как first-class citizens](#функции-как-first-class-citizens)
  - [Возврат нескольких значений](#возврат-нескольких-значений)
- [Обработка ошибок](#обработка-ошибок)
  - [Основы](#основы-обработки-ошибок)
  - [Best practices](#best-practices-обработки-ошибок)
  - [Именованные ошибки](#именованные-ошибки)
- [Defer](#defer)
- [Практический пример: работа с файлами](#практический-пример-работа-с-файлами)
- [Указатели (Pointers)](#указатели-pointers)
  - [Основы указателей](#основы-указателей)
  - [Указатели и функции](#указатели-и-функции)
  - [Варианты использования](#варианты-использования-указателей)
- [ООП в Go](#ооп-в-go)
  - [Типы и структуры](#типы-и-структуры)
  - [Методы и ресиверы](#методы-и-ресиверы)
  - [Конструктор](#конструктор)
  - [Экспорт (видимость)](#экспорт-видимость)
  - [Best practices ООП](#best-practices-ооп)
  - [Чего нет в Go](#чего-нет-в-go)
- [Ссылочные типы](#ссылочные-типы)

---

## Установка и Hello World

### Установка

📥 **Ссылка для установки:** [https://go.dev/doc/install](https://go.dev/doc/install)

- **macOS / Windows** — скачать установщик и запустить
- **Linux** — выполнить несколько команд из инструкции

**Проверка установки:**

```bash
go version
```

### Создание проекта

```bash
mkdir hello-go
cd hello-go
go mod init hello-go
```

Команда `go mod init` создаёт файл `go.mod` — файл проекта (модуля), содержащий имя и версию Go. По мере разработки в нём появляются зависимости.

> **Примечание:** Для публичных проектов имя модуля должно быть уникальным (обычно — путь к репозиторию, например `github.com/username/project`).

### Hello World

Создаём файл `main.go` — точка входа в приложение:

```go
package main

import "fmt"

func main() {
    fmt.Println("Hello, World!")
}
```

**Запуск:**

```bash
go run main.go
# или
go run .
```

**Ключевые моменты:**

- Каждый файл начинается с объявления пакета (`package`)
- `package main` + `func main()` — стандартная точка входа
- Пакет `fmt` — стандартная библиотека для форматирования и вывода строк

---

## Переменные

### Три способа объявления

```go
// Способ 1: полное объявление
var s string = "Hello, World!"

// Способ 2: объявление с указанием типа, присвоение позже
var s string
s = "Hello, World!"

// Способ 3: короткое объявление (используется в 99% случаев)
s := "Hello, World!"
```

> ⚠️ Оператор `:=` — объявление + присвоение. Оператор `=` — только присвоение.

### Zero Values

Каждый тип данных в Go имеет **zero value** — значение по умолчанию для неинициализированной переменной:

| Тип | Zero Value |
|---|---|
| `int`, `float64` | `0` |
| `string` | `""` (пустая строка) |
| `bool` | `false` |
| указатели, слайсы, мапы, интерфейсы | `nil` |

```go
var n int
fmt.Println(n) // 0

var s string
fmt.Println(s) // "" (пустая строка)
```

---

## Типы данных

### Обзор

| Категория | Типы |
|---|---|
| **Простые** | `int`, `string`, `bool`, `byte`, `rune` |
| **Математические** | `float32`, `float64`, `complex64`, `complex128` |
| **Составные** | массивы, слайсы, мапы, структуры |
| **Особые** | `error`, каналы (`chan`) |

---

### Целые числа (int)

| Тип | Размер | Диапазон (со знаком) |
|---|---|---|
| `int8` | 1 байт | -128 … 127 |
| `int16` | 2 байта | -32 768 … 32 767 |
| `int32` | 4 байта | -2 147 483 648 … 2 147 483 647 |
| `int64` | 8 байт | -9.2×10¹⁸ … 9.2×10¹⁸ |
| `int` | 4 или 8 байт | зависит от платформы (32/64 бит) |

Для беззнаковых типов (`uint8`, `uint16`, `uint32`, `uint64`, `uint`) — диапазон только положительный, вдвое больший максимум.

#### Overflow (переполнение)

```go
var s uint8 = 255
s++
fmt.Println(s) // 0 — произошёл overflow, отсчёт начался сначала
```

#### Рекомендации по выбору типа

- **По умолчанию** — используйте `int`
- **Если нужна гарантия 64 бит** — явно указывайте `int64`
- **Для экономии памяти** (например, при работе с Protobuf `int32`) — используйте соответствующий тип

---

### Биты и байты

```
1 бит    = 0 или 1
1 байт   = 8 бит   → 256 комбинаций (0–255)
2 байта  = 16 бит  → 65 536 комбинаций
4 байта  = 32 бита → ~4.3 млрд комбинаций
8 байт   = 64 бита → ~18.4 квинтиллиона комбинаций
```

> Go — язык для высоконагруженных систем. Понимание байтов и размеров типов — необходимость.

---

### Строки (string)

```go
a := "Hello"
b := "World"

// Сравнение
fmt.Println(a == b)  // false
fmt.Println(a < b)   // true (лексикографически)

// Конкатенация (простой способ)
c := a + " " + b

// Строки иммутабельны
// a[0] = 'X'  // ❌ Ошибка компиляции
```

#### Эффективная конкатенация через Builder

```go
var buff strings.Builder
buff.WriteString(a)
buff.WriteString(" ")
buff.WriteString(b)
result := buff.String() // "Hello World"
```

> Под капотом `Builder` использует `[]byte` — меньше аллокаций памяти, чем при `+`.

#### Split и Join

```go
s := "1,2,3,4,5"

// Разбиение строки
parts := strings.Split(s, ",") // []string{"1", "2", "3", "4", "5"}

// Склейка обратно с другим разделителем
result := strings.Join(parts, " ") // "1 2 3 4 5"
```

---

### Байт (byte)

```go
// byte — это alias для uint8 (числа от 0 до 255)
// Чаще всего используется как []byte (слайс байт)
```

`byte` — это сырые бинарные данные. Отдельно используется редко, чаще в виде `[]byte`.

---

### Руна (rune)

**Rune** — символ Unicode. Это alias для `int32`.

```go
r := 'A'              // rune (одинарные кавычки)
fmt.Println(r)         // 65 (код символа в UTF-8)
fmt.Println(string(r)) // "A"

r2 := 'Б'
fmt.Println(r2)         // 1041
fmt.Println(string(r2)) // "Б"
```

**Почему `int32`, а не `uint32`?**

Для обработки битых символов функции возвращают `-1`:

```go
func decodeRune(data []byte) rune {
    // если символ битый:
    return -1
}
```

---

### Булевы значения (bool)

```go
var b bool       // false (zero value)
b = true
b = 1 > 2       // false
```

---

### Числа с плавающей точкой (float)

```go
var f float64 = 3.14
```

> ⚠️ **Важно:** Float-числа имеют ошибку округления! **Никогда не храните деньги в `float`!**

Для денег используйте библиотеки decimal:
- `shopspring/decimal`
- `cockroachdb/apd`
- `ericlagergren/decimal`

---

### Комплексные числа (complex)

```go
var c complex128 = complex(1, 2) // 1 + 2i
```

> Используются крайне редко, преимущественно в математических вычислениях.

---

## Составные типы данных

### Структуры (struct)

```go
// Анонимная структура
a := struct {
    B int
    C int
}{B: 10, C: 20}

// Именованный тип (основной способ)
type MyType struct {
    B int
    C string
}

var m MyType              // {0, ""} — zero values
m = MyType{B: 10, C: "hello"}
```

**Пустая структура:**

```go
a := struct{}{}
fmt.Println(unsafe.Sizeof(a)) // 0 — занимает 0 байт в памяти
```

---

### Массивы и слайсы

#### Массивы (практически не используются)

```go
// Массив фиксированного размера
a := [5]int{} // [0, 0, 0, 0, 0]
// Нельзя добавить или убрать элементы
```

#### Слайсы (основной инструмент)

```go
// Объявление
var a []int                          // nil слайс
a = []int{1, 2, 3}                   // литерал
a = make([]int, 0, 10)               // make(тип, длина, ёмкость)
a = make([]int, 10)                  // длина = ёмкость = 10

// Добавление элементов
a = append(a, 4)                     // один элемент
a = append(a, 5, 6, 7)              // несколько элементов
a = append(a, otherSlice...)         // spread оператор
```

#### Сабслайсы

```go
a := []int{1, 2, 3, 4, 5, 6}

b := a[1:3]   // [2, 3]  — от индекса 1 (вкл.) до 3 (не вкл.)
b = a[:3]      // [1, 2, 3]
b = a[3:]      // [4, 5, 6]
b = a[:]       // [1, 2, 3, 4, 5, 6] — полная копия

b = a[1:len(a)-1] // [2, 3, 4, 5] — от первого до предпоследнего
```

#### Полезные функции

```go
len(a)  // длина слайса

// Пакет slices (Go 1.21+)
slices.Sort(a)                    // сортировка
idx := slices.Index(a, 22)       // индекс элемента (-1 если нет)
slices.Insert(a, pos, elems...)   // вставка элементов
```

> ⚠️ Функций `map`, `reduce`, `filter` в стандартной библиотеке пока нет — приходится писать самостоятельно.

#### Zero value слайса

```go
var a []int
fmt.Println(a)        // []
fmt.Println(a == nil) // true — неинициализированный слайс равен nil
```

---

### Строки как слайсы

Строка в Go — это **слайс байт** (`[]byte`) с переменной длиной символов в кодировке UTF-8:

| Символ | Байт |
|---|---|
| Латиница (`A`) | 1 байт |
| Кириллица (`Б`) | 2 байта |
| Иероглифы | 3 байта |
| Эмодзи | 4 байта |

```go
fmt.Println(len("hello"))  // 5
fmt.Println(len("привет")) // 12 (6 символов × 2 байта)
```

#### Получение символа по индексу

```go
s := "привет"

// ❌ Неправильно — получим байт, а не символ
fmt.Println(s[0])           // 208 (первый байт буквы П)
fmt.Println(string(s[0]))   // невалидный символ

// ✅ Правильно — через []rune
runes := []rune(s)
fmt.Println(string(runes[0])) // "п"
```

#### Подсчёт символов (не байт)

```go
import "unicode/utf8"

utf8.RuneCountInString("привет") // 6
utf8.RuneCountInString("hello")  // 5
```

---

### Мапы (map)

```go
// Объявление
var m map[int]int                      // nil map
m = map[int]int{1: 10, 2: 20}         // литерал
m = make(map[int]int, 10)              // с предвыделением памяти

// Операции
m[1] = 10           // добавление / обновление
delete(m, 2)        // удаление
val := m[1]         // чтение (вернёт zero value, если ключа нет)
```

#### Безопасное чтение (value, ok)

```go
val, ok := m[2]
if !ok {
    fmt.Println("key does not exist")
} else {
    fmt.Println(val)
}
```

> Паттерн `value, ok` используется также для чтения из каналов и type assertions.

---

## Конвертация типов

### Между числовыми типами

```go
var s uint8 = 255
a := int(s)           // ✅ Явное приведение
fmt.Println(a)         // 255
```

### Между числами и строками

```go
import "strconv"

// Число → строка
s := strconv.Itoa(100)         // "100" (Integer to ASCII)

// Строка → число
n, err := strconv.Atoi("100") // 100, nil (ASCII to Integer)
if err != nil {
    fmt.Println("conversion failed")
}
```

> ⚠️ `string(100)` вернёт `"d"` (символ с кодом 100), а **не** `"100"`!

### Между строками и байтами/рунами

```go
s := "hello"

bytes := []byte(s)    // строка → слайс байт
s2 := string(bytes)   // слайс байт → строка

runes := []rune(s)    // строка → слайс рун
s3 := string(runes)   // слайс рун → строка
```

---

## Управление потоком (Flow Control)

### Условные конструкции (if/else)

```go
// Простое условие — без скобок
if x > 10 {
    fmt.Println("big")
} else if x > 5 {
    fmt.Println("medium")
} else {
    fmt.Println("small")
}

// С инициализацией переменной
if n, err := strconv.Atoi("100"); err != nil {
    fmt.Println("error")
} else {
    fmt.Println(n)
}
```

---

### Циклы (for)

В Go **только один** вид цикла — `for`. Нет `while`, `do-while`, `foreach`.

```go
// Классический for
for i := 0; i < 10; i++ {
    fmt.Println(i)
}

// Аналог while
for condition {
    // ...
}

// Бесконечный цикл
for {
    // ...
}
```

#### break, continue, метки

```go
for i := 0; i < 10; i++ {
    if i == 5 {
        continue   // пропустить итерацию
    }
    if i == 8 {
        break      // выйти из цикла
    }
    fmt.Println(i)
}

// Выход из вложенного цикла по метке
outer:
for i := 0; i < 10; i++ {
    for j := 0; j < 10; j++ {
        if someCondition {
            break outer  // выход из внешнего цикла
        }
    }
}
```

---

### Range

Специальный цикл для итерации по слайсам, мапам и строкам.

#### По слайсу

```go
a := []string{"a", "b", "c"}

for index, value := range a {
    fmt.Printf("index=%d, value=%s\n", index, value)
}

// Только значения
for _, value := range a {
    fmt.Println(value)
}

// Только индексы
for index := range a {
    fmt.Println(index)
}
```

#### По мапе

```go
m := map[int]string{1: "one", 2: "two"}

for key, value := range m {
    fmt.Printf("key=%d, value=%s\n", key, value)
}
```

---

### Switch

```go
// По переменной
switch x {
case 1:
    fmt.Println("one")
case 2, 3:
    fmt.Println("two or three")
default:
    fmt.Println("other")
}

// Без переменной (замена длинных if-else)
switch {
case x > 10:
    fmt.Println("big")
case x > 5:
    fmt.Println("medium")
default:
    fmt.Println("small")
}

// Type switch
var i any = "hello"
switch v := i.(type) {
case int:
    fmt.Println("int:", v)
case string:
    fmt.Println("string:", v)
default:
    fmt.Println("unknown type")
}
```

> **Type switch** — элемент рефлексии. Не рекомендуется для продакшена без необходимости.

---

## Функции

### Объявление функций

```go
// Простая функция
func greet() {
    fmt.Println("Hello!")
}

// С параметрами и возвратом
func sum(a int, b int) int {
    return a + b
}

// Сокращённая запись одинаковых типов
func sum(a, b int) int {
    return a + b
}
```

### Функции как first-class citizens

Функции в Go — полноправные значения: их можно присваивать переменным, передавать как аргументы, хранить в слайсах и мапах.

```go
// Присвоение переменной
a := func() {
    fmt.Println("I'm A")
}
a() // вызов

// Слайс функций
b := func() { fmt.Println("I'm B") }

funcs := []func(){}
funcs = append(funcs, a, b)

for _, f := range funcs {
    f() // вызов каждой функции
}
```

### Возврат нескольких значений

```go
func sum(a, b int) (int, bool) {
    return a + b, true
}

result, ok := sum(2, 3)
```

#### Именованные возвращаемые параметры

```go
func sum(a, b int) (c int, d bool) {
    c = a + b
    d = true
    return // «голый» return — возвращает именованные переменные
}
```

> ⚠️ **Best practice:** Лучше **не использовать** именованные возвращаемые параметры и всегда явно указывать значения в `return`.

---

## Обработка ошибок

### Основы обработки ошибок

В Go нет исключений (`try/catch`). Ошибки — это **значения**, которые возвращаются из функций.

```go
import "errors"

var ErrInvalidAge = errors.New("age must be greater than 18")

func checkAge(age int) error {
    if age < 18 {
        return ErrInvalidAge
    }
    return nil // nil = ошибки нет
}

// Вызов
err := checkAge(16)
if err != nil {
    fmt.Println("error:", err)
}
```

### Best practices обработки ошибок

#### 1. Ошибка — всегда последний возвращаемый параметр

```go
func check(age int) (int, error) {
    if age < 18 {
        return 0, ErrInvalidAge  // zero value + ошибка
    }
    counter++
    return counter, nil          // значение + nil
}
```

#### 2. Всегда проверяйте ошибки

```go
// ✅ Правильно
n, err := strconv.Atoi(s)
if err != nil {
    fmt.Println("conversion failed")
    return
}
fmt.Println(n)

// ❌ Опасно — игнорирование ошибки может привести к панике
n, _ := strconv.Atoi(s)
```

#### 3. Паника при непроверенных ошибках

```go
// Пример цепочки, приводящей к панике
file, _ := os.Open("nonexistent.txt")  // file = nil
stat, _ := file.Stat()                  // ❌ nil pointer dereference!
```

> ⚠️ **Паника (panic)** завершает **всё приложение**. Если паника произошла в веб-сервере — ложится весь сервер.

### Именованные ошибки

```go
var (
    ErrInvalidAge  = errors.New("age must be greater than 18")
    ErrSeriousProblem = errors.New("very serious error")
)

// Сравнение ошибок
err := check(15)
if err != nil {
    if errors.Is(err, ErrSeriousProblem) {
        // обработка критической ошибки
        return
    }
    // обработка обычной ошибки
    fmt.Println("error:", err)
}
```

#### Сокращённая запись проверки

```go
// Инициализация переменной прямо в if
if err := someFunc(); err != nil {
    fmt.Println("error:", err)
    return
}
```

---

## Defer

`defer` откладывает выполнение функции до завершения текущей функции. Гарантирует выполнение **независимо от того, где произошёл return**.

```go
func foo() {
    defer bar()  // bar() вызовется в конце foo()

    if someCondition {
        return   // bar() всё равно выполнится
    }

    fmt.Println("foo")
}
```

### Порядок выполнения — LIFO (стек)

```go
defer fmt.Println("first")
defer fmt.Println("second")
defer fmt.Println("third")

// Вывод:
// third
// second
// first
```

### Типичное использование — освобождение ресурсов

```go
file, err := os.Open("data.txt")
if err != nil {
    return err
}
defer file.Close() // гарантированно закроется
```

---

## Практический пример: работа с файлами

Задача: заменить все вхождения слова в файле.

```go
package main

import (
    "io"
    "os"
    "strings"
)

const (
    oldFileName = "pipeline.go"
    newFileName = "process.go"
)

func renameInFile() error {
    // Открытие файла
    file, err := os.Open(oldFileName)
    if err != nil {
        return err
    }
    defer file.Close()

    // Чтение содержимого
    content, err := io.ReadAll(file)
    if err != nil {
        return err
    }

    // Замена слов
    s := string(content)
    s = strings.ReplaceAll(s, "pipeline", "process")

    // Запись в новый файл
    return os.WriteFile(newFileName, []byte(s), 0644)
}

func main() {
    if err := renameInFile(); err != nil {
        panic(err)
    }
}
```

**Что здесь используется:**

- `os.Open` — открытие файла на чтение
- `defer file.Close()` — гарантированное закрытие
- `io.ReadAll` — чтение всего файла в `[]byte`
- `string(content)` — конвертация `[]byte` → `string`
- `strings.ReplaceAll` — замена подстрок
- `[]byte(s)` — конвертация `string` → `[]byte`
- `os.WriteFile` — запись в файл

### Константы

```go
// Одиночная
const fileName = "data.txt"

// Блок констант
const (
    oldFileName = "pipeline.go"
    newFileName = "process.go"
    maxRetries  = 3
)
```

---

## Указатели (Pointers)

### Основы указателей

```go
a := 1

b := &a            // &  — получить адрес (ссылку)
fmt.Println(b)     // 0xc000012088 (адрес в памяти)

fmt.Println(*b)    // *  — разыменование (получить значение)
// 1
```

**Два оператора:**
| Оператор | Назначение | Пример |
|---|---|---|
| `&` | Получить адрес переменной | `ptr := &x` |
| `*` | Разыменование / объявление типа-указателя | `val := *ptr` / `var p *int` |

### Указатели и функции

**Проблема: передача по значению**

```go
func changeName(name string) {
    name = "Ann" // меняем копию — оригинал не изменится
}

s := "Bob"
changeName(s)
fmt.Println(s) // "Bob" — не изменился!
```

> В Go **всё передаётся копированием**.

**Решение: указатели**

```go
func changeName(name *string) {
    *name = "Ann" // разыменовываем и меняем оригинал
}

s := "Bob"
changeName(&s)
fmt.Println(s) // "Ann" ✅
```

**Со структурами (без явного разыменования):**

```go
type Person struct {
    name string
}

func changeName(p *Person) {
    p.name = "Ann"  // Go автоматически разыменовывает
}

bob := Person{name: "Bob"}
changeName(&bob)
fmt.Println(bob.name) // "Ann" ✅
```

### Zero value указателей

```go
var p *Person  // nil
if p == nil {
    fmt.Println("pointer is nil")
}

// ⚠️ Обращение к nil-указателю вызывает панику!
// p.name  // panic: nil pointer dereference
```

### Варианты использования указателей

| Сценарий | Описание |
|---|---|
| **Мутация объекта** | Изменение оригинала через функцию |
| **Nullable-поля из БД** | `*int` вместо `int` — чтобы отличить `nil` от `0` |
| **Избежание копирования** | Передача большой структуры по ссылке |

### Особенности указателей в Go

- ❌ Нет арифметики указателей (в отличие от C/C++)
- ✅ Runtime сам управляет размещением в памяти
- ✅ Невозможен dangling pointer (ссылка на освобождённую память)
- 📝 В Go «указатель» и «ссылка» — синонимы

---

## ООП в Go

### Типы и структуры

В Go **нет классов**. Вместо них — **типы на основе структур**.

```go
// Go                           // Аналог в языках с классами
type Person struct {            // class Person {
    name string                 //     private string name;
    age  int                    //     private int age;
}                               // }
```

### Методы и ресиверы

Методы объявляются **вне структуры** с указанием **ресивера**:

```go
// Pointer receiver (может мутировать объект)
func (p *Person) ChangeName(newName string) {
    p.name = newName
}

// Value receiver (работает с копией)
func (p *Person) CheckAge(requiredAge int) bool {
    return p.age >= requiredAge
}
```

> **Best practice:** Если хотя бы один метод использует pointer receiver — **все методы** должны использовать pointer receiver.

### Конструктор

```go
func NewPerson(name string, age int) *Person {
    return &Person{
        name: name,
        age:  age,
    }
}

// Использование
bob := NewPerson("Bob", 25)
```

> **Конвенция:** конструктор — функция `New` (или `NewИмяТипа`), возвращающая указатель.

### Экспорт (видимость)

В Go видимость определяется **регистром первой буквы**:

| Регистр | Видимость | Пример |
|---|---|---|
| **Заглавная** буква | Публичное (экспортируемое) | `Name`, `ChangeName()` |
| **Строчная** буква | Приватное (неэкспортируемое) | `name`, `changeName()` |

```go
// person/person.go
package person

type Person struct {
    name string  // приватное — недоступно извне пакета
    Age  int     // публичное — доступно извне
}

func New(name string) *Person {  // публичная функция
    return &Person{name: name}
}
```

```go
// main.go
package main

import "hello-go/person"

func main() {
    bob := person.New("Bob")
    // bob.name   // ❌ ошибка — приватное поле
    // bob.Age    // ✅ публичное поле
    bob.ChangeName("Ann") // ✅ публичный метод
}
```

### Best practices ООП

1. **Выносите типы в отдельные пакеты**
2. **Поля — приватные** (с маленькой буквы), доступ через методы
3. **Используйте конструктор** `New()` для создания объектов
4. **Pointer receiver** — если хотя бы один метод мутирует объект, все методы должны быть с pointer receiver
5. **Возвращайте указатель** из конструктора, если используете pointer receivers

### Чего нет в Go

| Возможность | Статус |
|---|---|
| Наследование | ❌ Отсутствует полностью |
| Перегрузка методов | ❌ Нельзя два метода с одним именем |
| Перегрузка операторов | ❌ Отсутствует |
| Полиморфизм подтипов | ❌ Нет (но есть через интерфейсы) |
| Абстрактные классы | ❌ Нет (используйте интерфейсы) |

> Единственный вид полиморфизма в Go — через **интерфейсы**.

---

## Ссылочные типы

### Слайсы — ссылочный тип

Слайс под капотом — это структура с тремя полями:

```go
// Из исходного кода Go:
type slice struct {
    array unsafe.Pointer  // указатель на массив
    len   int             // длина
    cap   int             // ёмкость
}
```

**Следствие:** присвоение слайса **не копирует данные**, а создаёт ещё одну ссылку на тот же массив.

```go
a := []int{1, 2, 3, 4}
b := a          // b ссылается на тот же массив!

b[0] = 100
fmt.Println(a)  // [100 2 3 4] — a тоже изменился!

c := b
c[3] = 500
fmt.Println(a)  // [100 2 3 500]
fmt.Println(b)  // [100 2 3 500]
fmt.Println(c)  // [100 2 3 500]
// Все три переменные указывают на один массив
```

```
a ──┐
b ──┤──→ [100, 2, 3, 500]  (один массив в памяти)
c ──┘
```

> ⚠️ **Будьте осторожны** при присвоении слайсов — изменение одного повлияет на все остальные ссылки.

### Мапы — тоже ссылочный тип

```go
a := map[int]int{1: 10}
b := a

b[2] = 20
fmt.Println(a) // map[1:10 2:20] — a тоже изменился!
```

> Правило одинаково для **слайсов** и **мап**: присвоение создаёт ещё одну ссылку, а не копию.

---

## Форматированный вывод (fmt.Printf)

```go
fmt.Printf("index=%d, value=%s\n", 0, "hello")
fmt.Printf("type=%T\n", 42)  // type=int
```

| Формат | Описание | Пример |
|---|---|---|
| `%d` | Целое число | `42` |
| `%s` | Строка | `"hello"` |
| `%f` | Float | `3.140000` |
| `%t` | Bool | `true` |
| `%T` | Тип переменной | `int` |
| `%v` | Значение (универсальное) | любое |
| `%+v` | Значение со структурой полей | `{name:Bob age:25}` |
| `%p` | Указатель (адрес) | `0xc000012088` |
| `\n` | Перенос строки | — |

---

## Пакет unsafe

```go
import "unsafe"

var a int
fmt.Println(unsafe.Sizeof(a))  // 8 (байт) — int64

var b int32
fmt.Println(unsafe.Sizeof(b))  // 4 (байта)

var c struct{}
fmt.Println(unsafe.Sizeof(c))  // 0 (пустая структура)
```

> ⚠️ Пакет `unsafe` **не безопасен для продакшена**. Используйте только для локальных исследований.

---

## Краткая шпаргалка

### Объявление переменных

```go
s := "hello"           // короткое объявление (99% случаев)
var s string = "hello" // полное объявление
var s string           // объявление без значения (zero value)
```

### Основные типы

```go
int, int8, int16, int32, int64
uint, uint8, uint16, uint32, uint64
float32, float64
bool
string
byte    // = uint8
rune    // = int32
```

### Составные типы

```go
[]int                   // слайс
[5]int                  // массив
map[string]int          // мапа
struct{ Name string }   // структура
```

### Функции

```go
func name(param Type) ReturnType { }
func name(a, b int) (int, error) { }
```

### Обработка ошибок

```go
result, err := someFunc()
if err != nil {
    return err
}
```

### Циклы

```go
for i := 0; i < n; i++ { }     // классический
for condition { }                // while
for { }                          // бесконечный
for i, v := range slice { }     // итерация
```



# Go для разработчиков — Полное руководство (Часть 2)

> Продолжение руководства. Охватывает ссылочные типы (продолжение), практические задачи, интерфейсы, встраивание, конкурентность, дженерики и дальнейшие шаги.

---

## Оглавление

- [Ссылочные типы (продолжение)](#ссылочные-типы-продолжение)
  - [Мапы — ссылочный тип](#мапы--ссылочный-тип)
  - [Нессылочные типы для сравнения](#нессылочные-типы-для-сравнения)
  - [Полный список ссылочных типов](#полный-список-ссылочных-типов)
- [Практика: структуры данных](#практика-структуры-данных)
  - [Задача 1: Стек](#задача-1-стек)
  - [Задача 2: Связанный список](#задача-2-связанный-список)
- [Интерфейсы](#интерфейсы)
  - [Объявление интерфейса](#объявление-интерфейса)
  - [Duck typing](#duck-typing)
  - [Полиморфизм через интерфейсы](#полиморфизм-через-интерфейсы)
  - [Пустой интерфейс и any](#пустой-интерфейс-и-any)
- [Встраивание (Embedding)](#встраивание-embedding)
  - [Встраивание типов](#встраивание-типов)
  - [Встраивание интерфейсов](#встраивание-интерфейсов)
  - [Предостережения](#предостережения-по-встраиванию)
- [Конкурентность (обзор)](#конкурентность-обзор)
  - [Планировщик Go](#планировщик-go)
  - [Что нельзя делать в Go](#что-нельзя-делать-в-go)
- [Дженерики (Generics)](#дженерики-generics)
  - [Generic-функции](#generic-функции)
  - [Generic-типы](#generic-типы)
  - [Пример: функция Filter](#пример-функция-filter)
- [Итоги и дальнейшие шаги](#итоги-и-дальнейшие-шаги)
  - [Рекомендуемые книги](#рекомендуемые-книги)
  - [Что делать дальше](#что-делать-дальше)

---

## Ссылочные типы (продолжение)

### Мапы — ссылочный тип

Мапы ведут себя точно так же, как слайсы — присвоение создаёт **ещё одну ссылку**, а не копию:

```go
m := map[int]int{1: 10, 2: 20}
n := m          // n ссылается на ту же структуру данных!

n[2] = 777
fmt.Println(m)  // map[1:10 2:777] — m тоже изменилась!
```

#### Почему так происходит?

В исходном коде Go мапа представлена структурой `hmap`, которая содержит **указатель на бакеты** (buckets) — внутренние хранилища данных:

```go
// Из исходного кода Go (упрощённо):
type hmap struct {
    count     int
    buckets   unsafe.Pointer  // ← указатель на данные
    // ...
}
```

Поскольку внутри лежит указатель, при присвоении `n := m` копируется только структура `hmap` с тем же указателем — обе переменные смотрят на одни и те же бакеты.

---

### Нессылочные типы для сравнения

Простые типы (`int`, `string`, `bool`, структуры без указателей) **копируются по значению**:

```go
a := 42
b := a      // b — независимая копия
b = 100

fmt.Println(a) // 42  — a не изменилась
fmt.Println(b) // 100
```

---

### Полный список ссылочных типов

В Go всего **три** встроенных ссылочных типа:

| Тип | Описание |
|---|---|
| `[]T` (слайс) | Содержит указатель на массив |
| `map[K]V` (мапа) | Содержит указатель на бакеты |
| `chan T` (канал) | Используется для конкурентности |

> ⚠️ **Осторожно:** не переприсваивайте слайсы и мапы друг другу напрямую — изменение одной переменной затронет все остальные ссылки на те же данные.

---

## Практика: структуры данных

> В Go встроены только слайсы и мапы. Стеки, очереди, связанные списки и прочие структуры данных приходится реализовывать самостоятельно.

---

### Задача 1: Стек

**Условие:** реализовать стек на основе слайса. Новые элементы добавляются **в начало** (для демонстрации работы с пакетом `slices`). Метод `Pop` забирает элемент тоже **с начала**.

```
Push(10) → [10]
Push(3)  → [3, 10]
Pop()    → 3, стек: [10]
```

#### Решение

```go
package main

import (
    "fmt"
    "slices"
)

type Stack struct {
    arr []int
}

func NewStack(arr []int) *Stack {
    return &Stack{arr: arr}
}

func (s *Stack) Push(n int) {
    s.arr = slices.Insert(s.arr, 0, n)
}

func (s *Stack) Pop() int {
    result := s.arr[0]
    s.arr = s.arr[1:]
    return result
}

func main() {
    s := NewStack([]int{3, 10, 2, 1})
    fmt.Println("Initial:", s.arr) // [3 10 2 1]

    s.Push(777)
    fmt.Println("After Push(777):", s.arr) // [777 3 10 2 1]

    val := s.Pop()
    fmt.Println("Pop:", val)               // 777
    fmt.Println("After Pop:", s.arr)       // [3 10 2 1]

    val = s.Pop()
    fmt.Println("Pop:", val)               // 3
    fmt.Println("After Pop:", s.arr)       // [10 2 1]
}
```

#### Ключевые моменты

**Вставка в начало слайса — три способа:**

```go
// Способ 1: через slices.Insert (рекомендуемый)
s.arr = slices.Insert(s.arr, 0, n)

// Способ 2: через append + spread (без доп. пакетов)
s.arr = append([]int{n}, s.arr...)

// Способ 3: вставка в произвольную позицию через slices.Insert
s.arr = slices.Insert(s.arr, position, element)
```

**Вставка в конец** — тривиальна:

```go
s.arr = append(s.arr, n) // просто append
```

> Pointer receiver (`*Stack`) обязателен в обоих методах, так как они **мутируют** поле `arr`.

---

### Задача 2: Связанный список

**Условие:** реализовать однонаправленный связанный список. Каждый узел (нода) содержит значение и ссылку на следующий узел. Метод `Append` добавляет новый узел в конец списка.

```
New(10) → [10]
Append(20) → [10] → [20]
Append(30) → [10] → [20] → [30]
```

#### Решение

```go
package main

import "fmt"

type Node struct {
    value int
    next  *Node
}

func NewNode(value int) *Node {
    return &Node{value: value}
}

func (n *Node) Append(newValue int) {
    current := n

    for {
        if current.next == nil {
            current.next = NewNode(newValue)
            return
        }
        current = current.next
    }
}

func main() {
    head := NewNode(10)
    head.Append(20)
    head.Append(30)

    // Итерация по списку
    node := head
    for {
        fmt.Println(node.value)
        if node.next == nil {
            break
        }
        node = node.next
    }
    // Вывод:
    // 10
    // 20
    // 30
}
```

#### Разбор алгоритма Append

```
1. Создаём копию ссылки current → head
2. Цикл:
   - Если current.next == nil → мы в конце списка
     → Создаём новый узел, привязываем к current.next
     → Выходим
   - Иначе → current = current.next (идём дальше)
```

#### Что здесь отрабатывается

| Концепция | Где используется |
|---|---|
| ООП (типы, методы) | `type Node struct`, методы с ресиверами |
| Указатели | `*Node`, pointer receiver |
| Конструктор | `NewNode()` |
| Циклы | `for` (бесконечный цикл с `break`) |
| Проверка на nil | `current.next == nil` |
| Ссылочные типы | Вся структура работает на указателях |

---

## Интерфейсы

### Объявление интерфейса

Интерфейс описывает **набор методов**, а не полей:

```go
type Duck interface {
    Quack()
    Fly()
    Swim()
    Walk()
}
```

### Duck typing

В Go нет ключевого слова `implements`. Тип **автоматически** удовлетворяет интерфейсу, если реализует **все его методы**.

> 🦆 «Если что-то крякает как утка, летает как утка, плавает и ходит как утка — значит, это, скорее всего, утка.»

```go
// Тип — конкретная порода утки
type WildDuck struct{}

// Реализуем ВСЕ методы интерфейса Duck
func (d *WildDuck) Quack() { fmt.Println("Wild duck: Quack!") }
func (d *WildDuck) Fly()   { fmt.Println("Wild duck: I'm flying!") }
func (d *WildDuck) Swim()  { fmt.Println("Wild duck: Swimming") }
func (d *WildDuck) Walk()  { fmt.Println("Wild duck: Walking") }

// Другая порода
type Mallard struct{}

func (d *Mallard) Quack() { fmt.Println("Mallard: Quack!") }
func (d *Mallard) Fly()   { fmt.Println("Mallard: I'm flying!") }
func (d *Mallard) Swim()  { fmt.Println("Mallard: Swimming") }
func (d *Mallard) Walk()  { fmt.Println("Mallard: Walking") }
```

> Никакого `implements Duck` — компилятор сам проверяет соответствие набора методов.

### Полиморфизм через интерфейсы

**Единственный вид полиморфизма в Go** — через интерфейсы:

```go
// Функция принимает интерфейс — работает с любым типом, его реализующим
func processDuck(d Duck) {
    d.Fly()
}

func main() {
    wild := &WildDuck{}
    mallard := &Mallard{}

    processDuck(wild)    // Wild duck: I'm flying!
    processDuck(mallard) // Mallard: I'm flying!
}
```

Одна и та же функция обрабатывает **два совершенно разных типа** через единый интерфейс.

### Пустой интерфейс и any

```go
// Пустой интерфейс — ему удовлетворяет ЛЮБОЙ тип
var x interface{}

x = 42
x = "hello"
x = []int{1, 2, 3}
```

`any` — это просто alias для пустого интерфейса:

```go
// Из исходного кода Go:
type any = interface{}
```

```go
var x any
x = 42      // ✅
x = "hello" // ✅
x = true    // ✅
```

**Zero value** интерфейса — `nil`:

```go
var d Duck
fmt.Println(d)        // <nil>
fmt.Println(d == nil) // true
```

---

## Встраивание (Embedding)

### Встраивание типов

Встраивание позволяет **переиспользовать поля и методы** одного типа в другом:

```go
type Person struct {
    name string
    age  int
}

func (p *Person) ChangeName(newName string) {
    fmt.Println("ChangeName called from Person")
    p.name = newName
}

func (p *Person) Name() string {
    return p.name
}

// Employee встраивает Person
type Employee struct {
    Person         // ← встраивание (без имени поля!)
    salary int
}
```

#### Использование

```go
emp := Employee{
    Person: Person{name: "Bob", age: 20},
    salary: 300000,
}

// Методы Person доступны напрямую на Employee
fmt.Println(emp.Name()) // "Bob"
emp.ChangeName("Ann")   // ChangeName called from Person
fmt.Println(emp.Name()) // "Ann"
```

#### Переопределение методов

Если у `Employee` есть свой метод с тем же именем — вызывается **метод Employee**:

```go
func (e *Employee) ChangeName(newName string) {
    fmt.Println("ChangeName called from Employee")
    e.name = newName
}

emp.ChangeName("Ann")         // ChangeName called from Employee
emp.Person.ChangeName("Ann")  // ChangeName called from Person (явный вызов)
```

### Встраивание интерфейсов

Интерфейсы тоже можно встраивать:

```go
type Flyer interface {
    Fly()
}

type Swimmer interface {
    Swim()
}

type Duck interface {
    Flyer       // ← встраивание интерфейса
    Swimmer     // ← встраивание интерфейса
    Quack()
    Walk()
}

// Duck теперь требует: Fly(), Swim(), Quack(), Walk()
```

### Предостережения по встраиванию

> ⚠️ Используйте встраивание **осторожно**:
> - Это **не наследование** в классическом понимании
> - Код становится хрупким — изменения во встроенном типе могут неожиданно сломать внешний
> - В продакшен-коде (особенно в микросервисах) это может создать проблемы при рефакторинге
> - Если нет **жёсткой необходимости** — лучше не использовать

---

## ООП в Go — итоговая сводка

| Принцип ООП | Статус в Go |
|---|---|
| **Инкапсуляция** | ✅ Полная (через регистр первой буквы) |
| **Полиморфизм** | ⚠️ Только через интерфейсы (duck typing) |
| **Наследование** | ❌ Отсутствует (есть встраивание как частичная замена) |
| **Перегрузка методов** | ❌ Невозможна |
| **Абстрактные классы** | ❌ Нет (используйте интерфейсы) |

---

## Конкурентность (обзор)

> По конкурентности в Go есть огромный пласт материала. Здесь — только обзор ключевой концепции.

### Планировщик Go

В Go разработчик **не управляет** распределением кода по ядрам процессора напрямую.

```
В других языках:              В Go:
┌─────────┐                   ┌─────────┐
│ func A ──┼── Thread 1       │ func A  │
│ func B ──┼── Thread 2       │ func B  ├──→ Планировщик ──→ Ядра CPU
│ func C ──┼── Thread 3       │ func C  │    (сам решает)
└─────────┘                   └─────────┘
```

**Как работает планировщик:**

1. Разработчик запускает горутины (конкурентные функции)
2. Все горутины попадают в **планировщик Go**
3. Планировщик **сам** решает:
   - Сколько ядер задействовать
   - Какую горутину на каком ядре выполнять
   - Когда подключить дополнительные ядра
4. Цель планировщика — **максимально эффективно** использовать доступные ресурсы

### Что нельзя делать в Go

| Задача | Возможность |
|---|---|
| Запустить функции конкурентно | ✅ Легко (горутины) |
| Указать, на каком ядре выполнять функцию | ❌ Невозможно |
| Гарантировать параллельное выполнение | ❌ Решает планировщик |
| Микросекундная точность параллелизма | ❌ Не подходит (выбирайте C/C++/Rust) |

> Для задач, требующих сверхточного контроля над параллелизмом (медицина, космос, real-time системы), Go **не подходит**.

---

## Дженерики (Generics)

Дженерики позволяют писать функции и типы, работающие с **любыми типами данных**.

### Пример: функция Filter

**Задача:** написать универсальную функцию фильтрации слайсов.

#### Без дженериков (только для int)

```go
func filterInt(arr []int, f func(int) bool) []int {
    var result []int
    for _, v := range arr {
        if f(v) {
            result = append(result, v)
        }
    }
    return result
}
```

#### С дженериками (универсальная)

```go
func filter[T any](arr []T, f func(T) bool) []T {
    var result []T
    for _, v := range arr {
        if f(v) {
            result = append(result, v)
        }
    }
    return result
}
```

> `[T any]` после имени функции — объявление параметра типа. `any` означает, что `T` может быть любым типом.

#### Использование

```go
func main() {
    // Фильтрация int: оставить числа меньше 10
    nums := []int{1, 2, 3, 4, 5, 6, 100, 200, 300}
    lessThan10 := func(n int) bool { return n < 10 }

    result := filter(nums, lessThan10)
    fmt.Println(result) // [1 2 3 4 5 6]

    // Фильтрация int8
    nums8 := []int8{1, 2, 3, 4, 5, 6, 100}
    lessThan10_8 := func(n int8) bool { return n < 10 }

    result8 := filter(nums8, lessThan10_8)
    fmt.Println(result8) // [1 2 3 4 5 6]

    // Фильтрация string: оставить строки короче 5 символов
    words := []string{"go", "rust", "python", "java", "c"}
    shortWords := func(s string) bool { return len(s) < 5 }

    resultStr := filter(words, shortWords)
    fmt.Println(resultStr) // [go rust java c]
}
```

### Generic-функции

```go
// Синтаксис: func name[T constraint](params) returnType
func filter[T any](arr []T, f func(T) bool) []T { ... }

// С несколькими параметрами типов
func merge[K comparable, V any](m1, m2 map[K]V) map[K]V { ... }
```

### Generic-типы

```go
type Box[T any] struct {
    value T
}

// Использование
intBox := Box[int]{value: 42}
strBox := Box[string]{value: "hello"}

fmt.Println(intBox.value)  // 42
fmt.Println(strBox.value)  // "hello"
```

### Ограничения (Constraints)

```go
// any — любой тип
func print[T any](val T) { ... }

// comparable — типы, которые можно сравнивать (==, !=)
func contains[T comparable](arr []T, target T) bool { ... }

// Числовые ограничения (пакет constraints или свои)
type Number interface {
    int | int8 | int16 | int32 | int64 | float32 | float64
}
func sum[T Number](a, b T) T { return a + b }
```

---

## Итоги и дальнейшие шаги

### Что мы изучили

| Тема | Статус |
|---|---|
| Установка и Hello World | ✅ |
| Переменные и zero values | ✅ |
| Все основные типы данных | ✅ |
| Слайсы, массивы, мапы | ✅ |
| Строки и Unicode (UTF-8) | ✅ |
| Конвертация типов | ✅ |
| Циклы, условия, switch | ✅ |
| Функции (first-class citizens) | ✅ |
| Обработка ошибок | ✅ |
| Defer | ✅ |
| Указатели / ссылки | ✅ |
| ООП (типы, методы, ресиверы) | ✅ |
| Интерфейсы и duck typing | ✅ |
| Встраивание | ✅ |
| Ссылочные типы | ✅ |
| Дженерики | ✅ |
| Конкурентность | 📌 Обзор (отдельная тема) |

### Рекомендуемые книги

Используйте как **справочники** (не обязательно читать от корки до корки):

| Книга | Описание |
|---|---|
| **"The Go Programming Language"** (Donovan & Kernighan) | Классика, местами подустарела, но фундаментально сильная |
| **"Learning Go"** (Jon Bodner) | Более современная, показывает идиоматичный подход к Go |

### Что делать дальше

#### 1. 🧩 Решать алгоритмические задачи (LeetCode)

- Закрепляет механический навык работы с языком
- Полезно для подготовки к собеседованиям
- Можно начинать **сразу** после просмотра основ

#### 2. 🛠 Писать CLI-утилиты

- Go отлично подходит для утилит
- Можно делать клоны существующих инструментов
- Полезно для изучения взаимодействия Go с Linux/OS
- Параллельно осваивается **тулинг Go** (встроенные инструменты)

#### 3. 🌐 Писать бэкенды

- Простой HTTP-сервер на Go пишется за **10 минут**
- В стандартной библиотеке Go **всё есть** для этого из коробки
- Для профессиональной разработки потребуется изучить дополнительные паттерны и инструменты

### Тулинг Go (не покрыт в этом руководстве)

Встроенные инструменты, которые стоит изучить при переходе к практике:

```bash
go build      # компиляция
go test       # тестирование
go vet        # статический анализ
go fmt        # форматирование кода
go mod tidy   # управление зависимостями
go generate   # кодогенерация
go doc        # документация
```

---

## Краткая шпаргалка (полная)

### Типы данных

```go
// Простые
int, int8, int16, int32, int64, uint, uint8, ...
float32, float64
bool
string
byte    // = uint8
rune    // = int32

// Составные
[]T              // слайс
[N]T             // массив
map[K]V          // мапа
struct { ... }   // структура

// Особые
error            // интерфейс ошибки
chan T            // канал
any              // = interface{}
```

### ООП

```go
// Тип
type MyType struct { field int }

// Конструктор
func NewMyType(f int) *MyType { return &MyType{field: f} }

// Pointer receiver (мутация)
func (m *MyType) Set(v int) { m.field = v }

// Value receiver (чтение)
func (m *MyType) Get() int { return m.field }

// Интерфейс
type MyInterface interface { DoSomething() }
```

### Ошибки

```go
var ErrNotFound = errors.New("not found")

func find(id int) (string, error) {
    if id < 0 { return "", ErrNotFound }
    return "result", nil
}

result, err := find(1)
if err != nil {
    if errors.Is(err, ErrNotFound) { ... }
    return
}
```

### Дженерики

```go
func filter[T any](arr []T, f func(T) bool) []T { ... }
type Box[T any] struct { value T }
```

### Ссылочные типы (осторожно!)

```go
// Присвоение создаёт ссылку, НЕ копию
a := []int{1, 2, 3}
b := a     // b указывает на тот же массив
b[0] = 100 // a[0] тоже стало 100!
```
