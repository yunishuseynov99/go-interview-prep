The `sync` package in Go provides **low-level concurrency primitives** for safely sharing data between goroutines.

It’s part of the standard library: `import "sync"`

Below is a complete, structured overview of everything important you need to know.

---

# 1️⃣ `sync.Mutex`

The most basic lock.

### What it does

Ensures **only one goroutine** can access a critical section at a time.

### Example

```go
var mu sync.Mutex
var counter int

func increment() {
    mu.Lock()
    counter++
    mu.Unlock()
}
```

### Best Practice

Always unlock with `defer` when possible:

```go
mu.Lock()
defer mu.Unlock()
```

### When to use

* Protect shared variables
* Protect maps
* Protect slices
* Any shared mutable state

---

# 2️⃣ `sync.RWMutex`

Read-write mutex.

### What it does

* Multiple readers allowed simultaneously
* Only one writer
* Writers block readers

### Example

```go
var mu sync.RWMutex
var data = make(map[string]int)

func read(key string) int {
    mu.RLock()
    defer mu.RUnlock()
    return data[key]
}

func write(key string, value int) {
    mu.Lock()
    defer mu.Unlock()
    data[key] = value
}
```

### When to use

* Read-heavy workloads
* Caches
* Configuration storage

⚠️ Don’t use it if writes are frequent — it can hurt performance.

---

# 3️⃣ `sync.WaitGroup`

Waits for a group of goroutines to finish.

### Example

```go
var wg sync.WaitGroup

for i := 0; i < 3; i++ {
    wg.Add(1)

    go func() {
        defer wg.Done()
        fmt.Println("working")
    }()
}

wg.Wait()
```

### Rules

* Call `Add()` before starting goroutine
* Always call `Done()`
* Never copy a WaitGroup

### When to use

* Parallel processing
* Worker pools
* Waiting for async jobs

---

# 4️⃣ `sync.Once`

Ensures something runs exactly once.

### Example

```go
var once sync.Once

func initialize() {
    once.Do(func() {
        fmt.Println("Initialized once")
    })
}
```

### When to use

* Singleton initialization
* Lazy loading
* Global setup

---

# 5️⃣ `sync.Cond`

Condition variable.

Used for signaling between goroutines.

### Example

```go
var mu sync.Mutex
cond := sync.NewCond(&mu)

go func() {
    mu.Lock()
    cond.Wait() // waits for signal
    mu.Unlock()
}()

mu.Lock()
cond.Signal()
mu.Unlock()
```

### When to use

* Advanced coordination
* Producer-consumer patterns

⚠️ Rarely needed in modern Go — channels are usually better.

---

# 6️⃣ `sync.Map`

Concurrent map (thread-safe without manual locks).

### Example

```go
var m sync.Map

m.Store("a", 1)
value, ok := m.Load("a")
m.Delete("a")
```

### When to use

* Highly concurrent access
* Read-heavy scenarios
* Unknown keys
* Long-lived shared cache

⚠️ Regular `map + Mutex` is usually faster if contention is low.

---

# 7️⃣ `sync.Pool`

Object reuse pool.

Reduces garbage collector pressure.

### Example

```go
var pool = sync.Pool{
    New: func() any {
        return make([]byte, 1024)
    },
}

buf := pool.Get().([]byte)
pool.Put(buf)
```

### When to use

* Frequently allocated temporary objects
* High-performance systems
* Buffers
* Parsers
* JSON encoding

⚠️ Not guaranteed to keep objects (GC may clear it).

---

# 8️⃣ Memory Model + Happens-Before

`sync` primitives establish **memory synchronization**.

For example:

* `Unlock()` happens-before another `Lock()`
* `WaitGroup.Done()` happens-before `Wait()`
* `Once.Do()` ensures memory visibility

This prevents stale reads and race conditions.

---

# 9️⃣ Zero Values Are Valid

All sync types work without initialization:

```go
var mu sync.Mutex
var wg sync.WaitGroup
var once sync.Once
```

Except:

* `sync.Cond` → must use `sync.NewCond`
* `sync.Map` → zero value works

---

# 🔟 Common Mistakes

### ❌ Forgetting to unlock

Deadlock.

### ❌ Copying sync types

Never do:

```go
func foo(mu sync.Mutex) // BAD
```

Always pass by pointer.

### ❌ Using Mutex inside struct and copying struct

Also bad.

### ❌ Calling WaitGroup.Add inside goroutine

Race condition risk.

---

# 1️⃣1️⃣ Mutex vs Channels

Go philosophy:

> "Do not communicate by sharing memory; share memory by communicating."

But in practice:

Use `sync` when:

* Protecting in-memory state
* You need very low overhead
* You're building infrastructure code

Use channels when:

* You model communication
* You build pipelines
* You build workers

---

# 1️⃣2️⃣ Performance Insights

Fastest → slowest (rough idea):

* Mutex
* RWMutex
* sync.Map
* Channels
* sync.Cond (rarely used)

Always benchmark in real workload.

---

# 1️⃣3️⃣ Advanced Pattern: Double-Checked Locking

```go
if v == nil {
    mu.Lock()
    if v == nil {
        v = newValue()
    }
    mu.Unlock()
}
```

Used for lazy initialization.

---

# 1️⃣4️⃣ Atomic vs sync

For simple counters, prefer `sync/atomic`:

```go
import "sync/atomic"

atomic.AddInt64(&counter, 1)
```

Faster than Mutex for simple numeric operations.

---

# 🧠 Summary Table

| Type      | Purpose                         |
| --------- | ------------------------------- |
| Mutex     | Exclusive lock                  |
| RWMutex   | Multiple readers, single writer |
| WaitGroup | Wait for goroutines             |
| Once      | Run once                        |
| Cond      | Signaling                       |
| Map       | Concurrent map                  |
| Pool      | Object reuse                    |

---

# 🎯 Practical Rule of Thumb

* Protect shared state → Mutex
* Read-heavy → RWMutex
* Run N goroutines → WaitGroup
* Init once → Once
* High perf object reuse → Pool
* Highly concurrent cache → Map
