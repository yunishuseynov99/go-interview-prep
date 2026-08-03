`errgroup` (пакет `golang.org/x/sync/errgroup`) нужен для **запуска нескольких горутин одновременно с удобным ожиданием их завершения и обработкой ошибок**.

Он решает сразу несколько проблем:

* запускает несколько горутин;
* ждет, пока все закончат (`Wait()`);
* если хотя бы одна вернула ошибку — `Wait()` вернет эту ошибку;
* умеет автоматически отменять остальные горутины через `context`.

## Без errgroup

Допустим, нужно одновременно получить пользователя, его заказы и комментарии.

```go
var wg sync.WaitGroup

wg.Add(3)

go func() {
    defer wg.Done()
    // получить пользователя
}()

go func() {
    defer wg.Done()
    // получить заказы
}()

go func() {
    defer wg.Done()
    // получить комментарии
}()

wg.Wait()
```

Проблемы:

* ошибки нужно складывать вручную;
* если одна горутина упала, остальные продолжают работать;
* отменять их неудобно.

---

## С errgroup

```go
g := new(errgroup.Group)

g.Go(func() error {
    // получить пользователя
    return nil
})

g.Go(func() error {
    // получить заказы
    return nil
})

g.Go(func() error {
    // получить комментарии
    return nil
})

if err := g.Wait(); err != nil {
    return err
}
```

Теперь:

* каждая горутина возвращает `error`;
* `Wait()` дождется всех;
* если хоть одна вернула ошибку — она попадет в `Wait()`.

---

# Самая полезная возможность — WithContext

Обычно используют именно его.

```go
g, ctx := errgroup.WithContext(context.Background())
```

Представим, что ты делаешь три HTTP-запроса.

```go
g.Go(func() error {
    return requestUser(ctx)
})

g.Go(func() error {
    return requestOrders(ctx)
})

g.Go(func() error {
    return requestProducts(ctx)
})

if err := g.Wait(); err != nil {
    return err
}
```

Если `requestOrders` вернул ошибку:

```
requestUser      ----->

requestOrders --> ERROR

requestProducts ---------->
```

то `errgroup` автоматически делает

```go
cancel()
```

для созданного контекста.

Если остальные функции используют этот `ctx`, они увидят

```go
ctx.Done()
```

и смогут быстро завершиться.

Например

```go
func requestUser(ctx context.Context) error {
    for {
        select {
        case <-ctx.Done():
            return ctx.Err()

        default:
            // продолжаем работу
        }
    }
}
```

То есть не приходится самостоятельно вызывать `cancel()`.

---

# Что происходит внутри

Когда вызываешь

```go
g.Go(func() error {
    ...
})
```

происходит примерно следующее:

```go
wg.Add(1)

go func() {
    defer wg.Done()

    if err := f(); err != nil {
        сохранить первую ошибку

        если есть context
            cancel()
    }
}()
```

А затем

```go
g.Wait()
```

это почти

```go
wg.Wait()
return firstError
```

---

# Пример из микросервисов

Есть API:

```
GET /profile
```

Чтобы собрать профиль, нужно обратиться в три сервиса:

* User Service
* Orders Service
* Wallet Service

Вместо последовательных запросов

```go
user, _ := getUser()
orders, _ := getOrders()
wallet, _ := getWallet()
```

(допустим, каждый занимает 200 мс → всего около 600 мс)

можно сделать параллельно:

```go
g, ctx := errgroup.WithContext(ctx)

g.Go(func() error {
    user = getUser(ctx)
    return nil
})

g.Go(func() error {
    orders = getOrders(ctx)
    return nil
})

g.Go(func() error {
    wallet = getWallet(ctx)
    return nil
})

if err := g.Wait(); err != nil {
    return err
}
```

Теперь все три запроса выполняются одновременно, и общее время будет примерно **200–220 мс**, а не 600 мс.

---

## Чем `errgroup` отличается от `sync.WaitGroup`

| `sync.WaitGroup`                          | `errgroup.Group`                                                     |
| ----------------------------------------- | -------------------------------------------------------------------- |
| Ждет завершения горутин                   | Ждет завершения горутин                                              |
| Не умеет работать с ошибками              | Собирает первую ошибку                                               |
| Нет отмены остальных задач                | Может отменять через `context`                                       |
| Нужно самостоятельно управлять ошибками   | Ошибки обрабатываются автоматически                                  |
| Используется для любых параллельных задач | Идеален для параллельных операций, которые могут завершиться ошибкой |

На собеседованиях часто ожидают именно такой вывод: **`errgroup` — это по сути `WaitGroup` + обработка ошибок + интеграция с `context` для отмены остальных задач при первой ошибке**. В большинстве современных Go-проектов, особенно в микросервисах, для параллельных операций чаще используют именно `errgroup`, а не голый `sync.WaitGroup`.
