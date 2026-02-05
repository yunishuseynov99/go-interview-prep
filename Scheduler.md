✨✨✨✨✨✨✨✨✨✨✨✨✨✨✨✨✨✨✨✨

# Scheduler

⚙️🧠

✨✨✨✨✨✨✨✨✨✨✨✨✨✨✨✨✨✨✨✨

## Goroutines

🧵🚀

"Lightweight threads"
Managed by the Go runtime
Minimal API (go)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## Go follows n:m scheduling

🔀📊

increased flexibility
the number of G's (goroutines**) typically much greater than the number of M's (machines)
the user-space scheduler multiplexes G's (goroutines) over the available M's (machines)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## Run queues

📥➡️⚙️

Go routines follow FIFO in run queue

go scheduler can assign goroutines to OS threads
and OS threads can take goroutines from run queues and run them

access to run queue is synchronized ()

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## Distributed run queues

🌍🧵🧵🧵

-distributed run queues exist

to not run into run queu starvation

but if we give each goroutine a short run time there will be poor locality and excessive context

so every machine has its own run queue and there is the common run queue pool.

each trads run queue length is 256 goroutines

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## why scheduler needs a lock

🔒🤝

gpt says:

Go’s scheduler runs many goroutines on a small number of OS threads. It can:

pause a goroutine in the middle of an operation
run another goroutine at the same time or on another core
run them truly in parallel on multiple CPU cores

So any shared data can be accessed simultaneously.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## GOMAXPROCS

🧮🖥️

Variable that limits the number of operating system threads that can execute on user-level Go code

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## P - Processor

⚡🧠

p is a data structure thst is used to execute Go code
p that executes Go code has an m associated with it
much of states are maintained by P - such as local runqueue
Number of P's = GOMAXPROCS

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## Fairness

⚖️⏳

Presence of resource hogs in a FIFO system leads to something known as the Convoy Effect.

Which is a problem in scheduling.

Convoy effect -

is a situation where one slow or heavy task delays the execution of all other tasks that are waiting behind it in a queue.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## Pre-emption

⏹️⏯️🔥

Go scheduler uses non-cooperative pre-emption - if goroutine is running or consuming a resource for a certain amount of time

it will forcefully going to be stopped and run at a later point of time for remaining duration

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## How we choose which Goroutine to run?

🎯🔍

1. scheduler checks local runqueue where is no lock and if there is no goroutines

2. next we check global runqueue, and processor will take the number of goroutines equivalent
   to the length of run queue divided to the number of precessors that exist

gqlen/GOMAXPROCS + 1

if there is no goroutines in runqueue

(*netpoller* - is responsible for handling async system calls, mainly network I/O)

3. if a particular goroutine is waiting on a network I/O system call to return or get a response back

it is gonna wait on a netpoller and not gonna block an OS thread (So basically we check netpoler for work)

if a NETPOLLER is empty then

4. it picks a random P and steals half of its G's (it retries 4 time if doesn't find work)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## Non Co-operative Pre-emption

👀⏱️

Sysmon
🛰️

1. Daemon running without a P
2. Issues pre-emption requests for long-running Goroutines

✨✨✨✨✨✨✨✨✨✨✨✨✨✨✨✨✨✨✨✨
