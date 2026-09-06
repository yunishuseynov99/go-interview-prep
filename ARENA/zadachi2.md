Вот 10 задач, все с реальными багами, в том же формате.

**Задача 1. Дедлок на двух мьютексах**
```go
// Переводим средства между двумя аккаунтами, блокируем оба аккаунта на время операции.
type Account struct {
	mu      sync.Mutex
	balance int
}

func transfer(from, to *Account, amount int) {
	from.mu.Lock()
	defer from.mu.Unlock()

	to.mu.Lock()
	defer to.mu.Unlock()

	from.balance -= amount
	to.balance += amount
}

func main() {
	a := &Account{balance: 100}
	b := &Account{balance: 100}

	var wg sync.WaitGroup
	wg.Add(2)
	go func() {
		defer wg.Done()
		transfer(a, b, 10)
	}()
	go func() {
		defer wg.Done()
		transfer(b, a, 20)
	}()
	wg.Wait()
}
```

**Задача 2. WaitGroup.Add внутри горутины**
```go
func processAll(items []string) {
	var wg sync.WaitGroup

	for _, item := range items {
		go func(it string) {
			wg.Add(1)
			defer wg.Done()
			fmt.Println("processed:", it)
		}(item)
	}

	wg.Wait()
	fmt.Println("all done")
}
```

**Задача 3. Закрытие канала несколькими горутинами**
```go
func fanIn(chs ...<-chan int) <-chan int {
	out := make(chan int)
	var wg sync.WaitGroup

	for _, ch := range chs {
		wg.Add(1)
		go func(c <-chan int) {
			defer wg.Done()
			for v := range c {
				out <- v
			}
			close(out)
		}(ch)
	}

	go func() {
		wg.Wait()
	}()

	return out
}
```

**Задача 4. Отправка в канал после отмены контекста**
```go
func producer(ctx context.Context, out chan<- int) {
	for i := 0; ; i++ {
		select {
		case <-ctx.Done():
			return
		default:
		}
		out <- i
	}
}

func main() {
	ctx, cancel := context.WithTimeout(context.Background(), time.Second)
	defer cancel()

	out := make(chan int)
	go producer(ctx, out)

	for v := range out {
		fmt.Println(v)
	}
}
```

**Задача 5. Буферизованный канал и "ленивое" чтение**
```go
func main() {
	ch := make(chan int, 3)

	go func() {
		for i := 1; i <= 5; i++ {
			ch <- i
			fmt.Println("sent:", i)
		}
		close(ch)
	}()

	time.Sleep(2 * time.Second)

	for v := range ch {
		fmt.Println("received:", v)
	}
}
```

**Задача 6. Гонка при ленивой инициализации**
```go
type Client struct {
	conn net.Conn
}

var client *Client

func getClient() *Client {
	if client == nil {
		client = &Client{conn: dial()}
	}
	return client
}

func handle(wg *sync.WaitGroup) {
	defer wg.Done()
	c := getClient()
	_ = c
}

func main() {
	var wg sync.WaitGroup
	for i := 0; i < 50; i++ {
		wg.Add(1)
		go handle(&wg)
	}
	wg.Wait()
}
```

**Задача 7. Отмена контекста не пробрасывается**
```go
func fetchAll(ctx context.Context, urls []string) []string {
	results := make([]string, len(urls))
	var wg sync.WaitGroup

	for i, u := range urls {
		wg.Add(1)
		go func(i int, url string) {
			defer wg.Done()
			results[i] = fetchOne(url)
		}(i, u)
	}
	wg.Wait()
	return results
}

func fetchOne(url string) string {
	time.Sleep(3 * time.Second)
	return "data from " + url
}
```

**Задача 8. Livelock через постоянный retry**
```go
type Resource struct {
	mu     sync.Mutex
	locked bool
}

func (r *Resource) tryUse(id int) {
	for {
		r.mu.Lock()
		if r.locked {
			r.mu.Unlock()
			continue // кто-то другой занял, пробуем снова
		}
		r.locked = true
		r.mu.Unlock()
		break
	}

	fmt.Println(id, "using resource")
	time.Sleep(100 * time.Millisecond)

	r.mu.Lock()
	r.locked = false
	r.mu.Unlock()
}
```

**Задача 9. Гонка на срезе при append**
```go
func collectResults(tasks []func() int) []int {
	var results []int
	var wg sync.WaitGroup

	for _, task := range tasks {
		wg.Add(1)
		go func(t func() int) {
			defer wg.Done()
			results = append(results, t())
		}(task)
	}

	wg.Wait()
	return results
}
```

**Задача 10. select с default превращается в busy-loop**
```go
func consume(ch <-chan int, done <-chan struct{}) {
	for {
		select {
		case v, ok := <-ch:
			if !ok {
				return
			}
			fmt.Println(v)
		case <-done:
			return
		default:
		}
	}
}
```

Пробуй находить и чинить сам, скидывай решения — проверю.
