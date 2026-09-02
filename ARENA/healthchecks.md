**HealthCheck** — это специальная проверка, по которой внешняя система понимает:
**“сервис вообще жив?”** и **“сервис реально готов работать?”**

Обычно это HTTP endpoint, например:

* `GET /health`
* `GET /live`
* `GET /ready`

---

# 1. Зачем нужен HealthCheck

### Основные задачи:

* понять, **запущен ли сервис**
* понять, **не завис ли он**
* проверить, **есть ли доступ к БД / Redis / Kafka / внешнему API**
* дать Kubernetes / Docker / load balancer сигнал:

  * можно ли слать трафик
  * надо ли перезапустить контейнер

---

# 2. Какие бывают health checks

Обычно разделяют на **liveness** и **readiness**.

---

## **Liveness probe** — “жив ли процесс?”

Отвечает на вопрос:

> **Приложение вообще работает или зависло?**

Например:

* HTTP сервер поднят
* приложение не в deadlock
* event loop / goroutines не умерли

Если liveness падает — оркестратор может **перезапустить контейнер**.

Пример endpoint:

```http
GET /live
```

Пример ответа:

```json
{
  "status": "ok"
}
```

---

## **Readiness probe** — “готов ли сервис принимать запросы?”

Отвечает на вопрос:

> **Можно ли прямо сейчас отправлять в сервис боевой трафик?**

Тут уже часто проверяют зависимости:

* БД
* Redis
* Kafka / RabbitMQ
* внешний API, если без него сервис не может работать

Если readiness падает:

* контейнер **не обязательно перезапускают**
* но **убирают из балансировки**, чтобы не слать в него запросы

Пример endpoint:

```http
GET /ready
```

---

# 3. Как это обычно выглядит в Go

Чаще всего делают отдельные хендлеры:

* `/live` — просто отвечает 200 OK
* `/ready` — делает проверки зависимостей

---

# 4. Простой пример на Go

## Вариант 1 — базовый healthcheck

```go
package main

import (
	"encoding/json"
	"net/http"
)

type HealthResponse struct {
	Status string `json:"status"`
}

func liveHandler(w http.ResponseWriter, r *http.Request) {
	w.Header().Set("Content-Type", "application/json")
	w.WriteHeader(http.StatusOK)

	resp := HealthResponse{
		Status: "ok",
	}

	_ = json.NewEncoder(w).Encode(resp)
}

func main() {
	http.HandleFunc("/live", liveHandler)

	http.ListenAndServe(":8080", nil)
}
```

### Что делает:

* поднимает endpoint `/live`
* при запросе возвращает `200 OK`
* значит сервис **жив**

---

# 5. Более реальный вариант: readiness с проверкой БД

Допустим у тебя есть `db *sql.DB`.

```go
package main

import (
	"context"
	"database/sql"
	"encoding/json"
	"net/http"
	"time"
)

type App struct {
	DB *sql.DB
}

type HealthResponse struct {
	Status string `json:"status"`
	Error  string `json:"error,omitempty"`
}

func (a *App) liveHandler(w http.ResponseWriter, r *http.Request) {
	writeJSON(w, http.StatusOK, HealthResponse{
		Status: "ok",
	})
}

func (a *App) readyHandler(w http.ResponseWriter, r *http.Request) {
	ctx, cancel := context.WithTimeout(r.Context(), 2*time.Second)
	defer cancel()

	if err := a.DB.PingContext(ctx); err != nil {
		writeJSON(w, http.StatusServiceUnavailable, HealthResponse{
			Status: "fail",
			Error:  "database unavailable",
		})
		return
	}

	writeJSON(w, http.StatusOK, HealthResponse{
		Status: "ok",
	})
}

func writeJSON(w http.ResponseWriter, status int, resp HealthResponse) {
	w.Header().Set("Content-Type", "application/json")
	w.WriteHeader(status)
	_ = json.NewEncoder(w).Encode(resp)
}
```

---

# 6. Как это использовать в `main`

```go
func main() {
	app := &App{
		DB: db, // инициализированная БД
	}

	mux := http.NewServeMux()
	mux.HandleFunc("/live", app.liveHandler)
	mux.HandleFunc("/ready", app.readyHandler)

	http.ListenAndServe(":8080", mux)
}
```

---

# 7. Логика тут такая

## `/live`

Просто говорит:

> “процесс жив, HTTP сервер работает”

Обычно туда **не пихают тяжелые проверки**.

---

## `/ready`

Проверяет:

* БД доступна?
* Redis доступен?
* нужный broker доступен?
* миграции применены?
* сервис прогрет / конфиг загружен?

Если что-то критичное недоступно → вернуть **503 Service Unavailable**

---

# 8. Как это обычно делают на практике в Go-сервисах

Нормальный production-вариант выглядит так:

## `/live`

Минимальная проверка:

* приложение запущено
* не в состоянии graceful shutdown
* основной цикл жив

## `/ready`

Проверка критичных зависимостей:

* PostgreSQL ping
* Redis ping
* Kafka producer/consumer connectivity
* иногда проверка места в пуле коннектов / деградации

---

# 9. Что важно на собесе сказать

Если тебя спросят **“что такое healthcheck и как бы ты сделал в Go?”**, хороший короткий ответ:

> Healthcheck — это endpoint для проверки состояния сервиса. Обычно разделяют **liveness** и **readiness**.
> **Liveness** показывает, жив ли процесс, и обычно отвечает просто 200 OK, без тяжелых зависимостей.
> **Readiness** показывает, готов ли сервис принимать трафик, и там уже проверяют критичные зависимости — например, БД, Redis, брокер сообщений.
> В Go это обычно отдельные HTTP handlers `/live` и `/ready`: `/live` просто возвращает OK, а `/ready` делает `PingContext` к зависимостям с таймаутом и при проблеме возвращает `503 Service Unavailable`.

---

# 10. Как я бы писал это в реальном сервисе

Я бы сделал так:

## Структура

```go
type HealthChecker interface {
	Name() string
	Check(ctx context.Context) error
}
```

Потом отдельные чекеры:

* `PostgresChecker`
* `RedisChecker`
* `KafkaChecker`

И readiness handler проходит по ним циклом.

---

## Пример идеи

```go
type HealthChecker interface {
	Name() string
	Check(ctx context.Context) error
}

type PostgresChecker struct {
	DB *sql.DB
}

func (p *PostgresChecker) Name() string {
	return "postgres"
}

func (p *PostgresChecker) Check(ctx context.Context) error {
	return p.DB.PingContext(ctx)
}
```

А readiness:

```go
func readyHandler(checkers []HealthChecker) http.HandlerFunc {
	return func(w http.ResponseWriter, r *http.Request) {
		ctx, cancel := context.WithTimeout(r.Context(), 2*time.Second)
		defer cancel()

		for _, checker := range checkers {
			if err := checker.Check(ctx); err != nil {
				http.Error(w, checker.Name()+" is unavailable", http.StatusServiceUnavailable)
				return
			}
		}

		w.WriteHeader(http.StatusOK)
		w.Write([]byte("ok"))
	}
}
```

---

# 11. Что не стоит делать

## Не надо в `/live`:

* ходить в БД
* делать тяжелые внешние запросы
* проверять вообще всё подряд

Почему: если временно умерла БД, это не значит, что **процесс надо рестартовать**. Возможно, его просто надо убрать из трафика, а не убить.

---

# 12. Итог в двух строчках

### **HealthCheck** — это endpoint для проверки состояния сервиса.

В Go обычно делают:

* **`/live`** — жив ли процесс
* **`/ready`** — готов ли сервис работать, проверяя БД/Redis/и другие критичные зависимости

Если хочешь, я могу **сразу показать, как healthcheck обычно пишут в Go на собеседовании “по-взрослому”**:
с `chi/gin`, интерфейсом чекеров, красивым JSON-ответом и best practices для production.
