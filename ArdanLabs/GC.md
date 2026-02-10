# 🗑️ Garbage Collector in GO

Go GC is a non-generational, non-compacting, concurrent, tri-color, mark and sweep

---

## 🎯 Go's GC priorities

Go's GC has 2 priorities

1 - makes shure that the app has the highest level-amount of throughput during collection
2 - memory resource usage

---

## ⏱️ When should garbage collection take place?

there is a environmental variable - **GOGC**

GOGC by default = **100%**

the value of it means when the first garbage collection take place

if it is 100% it will start when the heap will reach **4meg of memory in use**

---

## 🔄 What happens when GC first starts?

GC runs in 3 different phases

1 - Mark Start - STW (Stop The World phase)
2 - Marking (Concurrent phase)
3 - Mark Termination ( also Stop The World phase)

all these 3 phases combined will be a total amount of time it took for Garbage collectin

when a gc takes place there is going to be latency cost

GC wnats to make sure STW phase latency is < **100ms**

concurrent phase time depends on the work we are doing

---

## ⚙️ Example

for example:

we are runnig our go program with **4 P's**

and **4 goroutines** are working in parallel

so **100% of CPU = 4G**

imagine we've used **4 megs of memory on heap**

and GC starts

STW runs and all **4 G's** will be stopped

we have stop goroutines so we can turn on a flag called **Right Barrier**

---

## ✍️ Right Barrier flag

Right Barrier flag - it will allow if app gorutine and gc gorutine get too close in memory based on boundaries of heap

we can make sure they dont make incosistencies, integrity issues in data

we avoid data races

it happens on context switch

we can cause them to stop on context switch

GC doesnt have its own cpu capacity (it also uses gorutines to do garbage collection)

---

## 🧮 CPU capacity usage

so how much GC should use of this capacity?

GC will take **25%**

so in this situation we left with **75% = 3G**

---

## 🖍️ Marking phase

after that the marking phase starts

marking high level explanation

you can think of heap as a **qraph of values**

it starts at stacks

there is **3 phases to marking**

---

## 🔍 Marking process

we find root values on the heap (there could be any amount of stacks)

we scan stacks looking for pointers out into the heap

first every value is painted **white**

when we find root values we will paint them **grey** putting them in queue

once we have all of the values we can directly access from stack (even global variables)

we will pull them out of the queue and painting them **black**

and look if that black value points into the heap

and if it does paint it **grey** and put them into the queue and so on

at the end we will have only **black and white values**

white - no longer in use
black - still in use

---

## 🧹 Mark termination & sweeping

and the we go to **Mark termination phase**

after that we are back to **100% capacity**

and the sweeping actually happens outside of GC block

it happens in allocator

all GC does is marking
