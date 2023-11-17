---
title: "Synchronization"
categories:
  - Blog
tags:
  - OS
  - Synchronize
---

# synchronize

## Coke machine

```cpp
// Initially, no coke
Semaphore fullSlots = 0;
// Initially, num empty slots
Semaphore emptySlots = bufSize
// No one using machine
Semaphore mutex = 1

Producer(item) {
  // wait until space
  semaP(&emptySlots);
  // wait until machine free
  semaP(&mutex);
  Enqueue(item);
  semaV(&mutex);
  // tell consumer there is more coke
  semaV(&fullSlots);
}

Consumer() {
  // check if there's a coke
  semaP(&fullSlots);
  // wait until machine free
  semaP(&mutex);
  item = Dequeue();
  semaV(&mutex);
  // tell producer need more
  semaV(&emptySlots);
}
```

## Monitor

- Mesa Monitor Structure

```cpp
lock
while (need to wait) { // Mesa style
  cv.wait();
}
unlock

do something so no need to wait

lock

cv.signal();

unlock
```

## Mesa vs. Hoare monitors

```cpp
while (isEmpty(&queue)) {
  cond_wait(&buf_CV, &buf_lock);
} // Mesa style, when signaled, waiter wait in the ready queue

if (is Empty(&queue)) {
  cond_wait(&buf_CV, &buf_lock);
} // Hoare style, when signaled, waiter runs immediately
```


## Does notify_one need to be in the lock?

```cpp
thread t;

void foo() {
    std::mutex m;
    std::condition_variable cv;
    bool done = false;

    t = std::thread([&]() {
        {
            std::lock_guard<std::mutex> l(m);  // (1)
            done = true;  // (2)
        }  // (3)
        cv.notify_one();  // (4)
    });  // (5)

    std::unique_lock<std::mutex> lock(m);  // (6)
    cv.wait(lock, [&done]() { return done; });  // (7)
}

void main() {
    foo();  // (8)
    t.join();  // (9)
}
// this notify_one need to be in the lock
// critical section
// the main thread exit when the lock is free, the cv is deleted before t thread call it.
```

## Semaphore to Monitor

```cpp
Wait(Lock *the_lock, Semaphore *the_sema) {
  //keep a conditional variable
  release(the_lock);
  semaP(the_sema);
  acquire(the_lock);
}

Signal(Semaphore *the_sema) {
  if (semaphore queue is not empty) // not legal to look at contents of semaphore queue
    semaV(the_sema)
}

// signaler can slip in after lock release and before waiter executes semaphore.P()
// cause dead lock, the queue is empty, but the waiter just about to come in the queue
```