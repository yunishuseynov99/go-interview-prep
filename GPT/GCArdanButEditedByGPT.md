# 🗑️ Garbage Collector in Go

Go’s Garbage Collector (GC) is **non-generational**, **non-compacting**, **concurrent**, **tri-color**, **mark-and-sweep**.

---

## 🎯 GC Priorities

Go GC has **two main priorities**:

1. **Maximize application throughput during collection**
2. **Efficient memory resource usage**

The GC is designed to interfere with the application as little as possible while still keeping memory usage under control.

---

## ⏱️ When Does Garbage Collection Run?

Garbage collection is controlled by the environment variable **`GOGC`**.

* Default value: **`GOGC = 100`**
* The value represents **heap growth percentage** before the next GC cycle starts

Example:

* If `GOGC = 100`, the GC will start when the heap size has **grown by 100%**
* In practice, this means the first GC run starts when the heap reaches **~4 MB of memory in use**

---

## 🔄 What Happens When GC Starts?

A full GC cycle consists of **three phases**:

1. **Mark Start** — *Stop The World (STW)*
2. **Marking** — *Concurrent*
3. **Mark Termination** — *Stop The World (STW)*

⏳ The **total GC time** is the sum of these three phases.

Every GC cycle introduces some **latency cost**, so Go GC aims to keep:

* **STW latency < 100 ms**
* Concurrent phase duration depends on the amount of work being done by the application

---

## ⚙️ Example Scenario

Let’s assume:

* The Go program is running with **4 P’s**
* **4 goroutines** are executing in parallel
* This means **100% CPU = 4G**
* The heap has reached **4 MB**, triggering GC

### 🛑 Stop The World

* All **4 goroutines are stopped**
* This pause allows the GC to safely enable a mechanism called the **Write Barrier**

---

## ✍️ Write Barrier

The **Write Barrier** ensures correctness when application goroutines and GC goroutines operate close to each other in memory.

It guarantees that:

* No data inconsistencies occur
* Heap integrity is preserved
* Data races are avoided

This happens during **context switches**, where goroutines can be safely paused.

---

## 🧮 CPU Usage During GC

The GC does **not** have its own dedicated CPU resources.

* GC work is done using **goroutines**
* By default, the GC uses **25% of total CPU capacity**

In our example:

* Total capacity: **4G**
* GC uses: **25%**
* Application is left with **75% = 3G**

---

## 🖍️ Marking Phase (High-Level Explanation)

You can think of the heap as a **graph of values**.

### 🔍 Marking starts from:

* Goroutine stacks
* Global variables

### The marking process uses **three colors**:

1. **White** — unreachable / not in use
2. **Gray** — reachable but not yet fully scanned
3. **Black** — reachable and fully scanned

### 🔁 Marking Steps

1. Initially, **all values are painted white**
2. Roots are discovered by scanning stacks for pointers into the heap
3. Root values are painted **gray** and added to a queue
4. Gray values are processed:

   * Painted **black**
   * If they reference other heap values, those are painted **gray** and added to the queue
5. This continues until the queue is empty

### ✅ Final State

* **Black** values → still in use
* **White** values → no longer reachable

---

## 🧹 Mark Termination & Sweeping

After marking is complete:

* GC enters the **Mark Termination** phase (STW)
* Once finished, the application returns to **100% CPU capacity**

🧼 **Sweeping does NOT happen inside the GC cycle**

* Sweeping is performed **by the allocator**
* The GC itself is responsible **only for marking**
