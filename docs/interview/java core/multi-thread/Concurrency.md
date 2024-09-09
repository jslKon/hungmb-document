# Concurrency

## The problems

## Volatile

The `volatile` keyword is used to indicate that a variable's value will be modified by different threads. It is used to
prevent thread caching. In Java, each thread has its own stack. The volatile keyword will force
the thread to read the value of the variable directly from the main memory.

## Atomic

Atomic
