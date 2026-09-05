Ок, вот 10 задач в похожем формате — комментарий с описанием, код с багом. Постарайся сначала найти баг сам, потом сверим. Уровень примерно как в твоих двух примерах.

**Задача 1. Утечка горутины через таймер**
```go
// Хотим с таймаутом дождаться результата, иначе вернуть ошибку.
func fetchWithTimeout(fetch func() int) (int, error) {
	ch := make(chan int)
	go func() {
		ch <- fetch()
	}()

	select {
	case res := <-ch:
		return res, nil
	case <-time.After(2 * time.Second):
		return 0, fmt.Errorf("timeout")
	}
}
```

**Задача 2. Копирование мьютекса**
```go
type Counter struct {
	mu    sync.Mutex
	value int
}

func (c Counter) Inc() {
	c.mu.Lock()
	defer c.mu.Unlock()
	c.value++
}

func main() {
	c := Counter{}
	var wg sync.WaitGroup
	for i := 0; i < 100; i++ {
		wg.Add(1)
		go func() {
			defer wg.Done()
			c.Inc()
		}()
	}
	wg.Wait()
	fmt.Println(c.value)
}
```

**Задача 3. Паника при закрытии канала**
```go
// Несколько воркеров пишут результат, диспетчер закрывает канал по готовности.
func worker(id int, jobs <-chan int, results chan<- int, wg *sync.WaitGroup) {
	defer wg.Done()
	for j := range jobs {
		results <- j * 2
	}
}

func run(jobs []int) []int {
	jobsCh := make(chan int, len(jobs))
	results := make(chan int, len(jobs))
	var wg sync.WaitGroup

	for _, j := range jobs {
		jobsCh <- j
	}
	close(jobsCh)

	for i := 0; i < 3; i++ {
		wg.Add(1)
		go worker(i, jobsCh, results, &wg)
	}

	go func() {
		close(results)
		wg.Wait()
	}()

	var out []int
	for r := range results {
		out = append(out, r)
	}
	return out
}
```

**Задача 4. sync.Once не спасает**
```go
type Config struct {
	once sync.Once
	data map[string]string
}

func (c *Config) Get(key string) string {
	c.once.Do(func() {
		c.data = loadConfig()
	})
	return c.data[key]
}

func loadConfig() map[string]string {
	time.Sleep(100 * time.Millisecond)
	return map[string]string{"env": "prod"}
}

func main() {
	c := &Config{}
	var wg sync.WaitGroup
	for i := 0; i < 10; i++ {
		wg.Add(1)
		go func() {
			defer wg.Done()
			fmt.Println(c.Get("env"))
		}()
	}
	wg.Wait()
}
```

**Задача 5. RWMutex и запись внутри "чтения"**
```go
type Cache struct {
	mu   sync.RWMutex
	data map[string]int
	hits int
}

func (c *Cache) Get(key string) int {
	c.mu.RLock()
	defer c.mu.RUnlock()
	c.hits++ // считаем количество обращений
	return c.data[key]
}
```

**Задача 6. Ограничение конкурентности семафором**
```go
// Нужно обработать 1000 задач, но не более 10 одновременно.
func processAll(tasks []func() error) []error {
	sem := make(chan struct{}, 10)
	errs := make([]error, len(tasks))
	var wg sync.WaitGroup

	for i, task := range tasks {
		wg.Add(1)
		sem <- struct{}{}
		go func(i int, task func() error) {
			defer wg.Done()
			errs[i] = task()
			<-sem
		}(i, task)
	}
	wg.Wait()
	return errs
}
```

**Задача 7. Двойная отправка в unbuffered канал**
```go
// Первая горутина, которая успеет посчитать, "побеждает".
func firstResult(fns []func() int) int {
	ch := make(chan int)
	for _, fn := range fns {
		go func(fn func() int) {
			ch <- fn()
		}(fn)
	}
	return <-ch
}
```

**Задача 8. atomic и обычная переменная вперемешку**
```go
type Stats struct {
	count int64
	total int64
}

func (s *Stats) Add(v int64) {
	atomic.AddInt64(&s.count, 1)
	s.total += v
}

func main() {
	s := &Stats{}
	var wg sync.WaitGroup
	for i := 0; i < 1000; i++ {
		wg.Add(1)
		go func(v int64) {
			defer wg.Done()
			s.Add(v)
		}(int64(i))
	}
	wg.Wait()
	fmt.Println(s.count, s.total)
}
```

**Задача 9. Контекст без отмены**
```go
func handleRequest(w http.ResponseWriter, r *http.Request) {
	ctx, _ := context.WithTimeout(r.Context(), 5*time.Second)

	result := make(chan string)
	go func() {
		result <- longOperation(ctx)
	}()

	select {
	case res := <-result:
		fmt.Fprint(w, res)
	case <-ctx.Done():
		http.Error(w, "timeout", http.StatusGatewayTimeout)
	}
}
```

**Задача 10. Map как поле структуры без защиты**
```go
type EventBus struct {
	subscribers map[string][]chan string
}

func NewEventBus() *EventBus {
	return &EventBus{subscribers: make(map[string][]chan string)}
}

func (b *EventBus) Subscribe(topic string) chan string {
	ch := make(chan string, 1)
	b.subscribers[topic] = append(b.subscribers[topic], ch)
	return ch
}

func (b *EventBus) Publish(topic, msg string) {
	for _, ch := range b.subscribers[topic] {
		ch <- msg
	}
}
```

Пробуй находить баги и чинить, а я проверю решения — можешь скидывать по одной или сразу пачкой.
