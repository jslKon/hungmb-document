# Java Internals

## 1. JVM Architecture

The Java Virtual Machine is the runtime engine that executes Java bytecode. It consists of three
major subsystems.

```
┌───────────────────────────────────────────────────┐
│                   JVM Architecture                 │
├───────────────────────────────────────────────────┤
│                                                    │
│  ┌──────────────┐                                  │
│  │  ClassLoader  │ ← loads .class files            │
│  │   Subsystem   │                                 │
│  └──────┬───────┘                                  │
│         ▼                                          │
│  ┌──────────────────────────────────────────────┐  │
│  │          Runtime Data Areas (Memory)          │  │
│  │  ┌────────┐ ┌───────┐ ┌────────┐ ┌────────┐ │  │
│  │  │ Heap   │ │ Stack │ │Method  │ │  PC    │ │  │
│  │  │        │ │       │ │ Area   │ │Register│ │  │
│  │  └────────┘ └───────┘ └────────┘ └────────┘ │  │
│  └──────────────────────────────────────────────┘  │
│         ▼                                          │
│  ┌──────────────────────────────────────────────┐  │
│  │           Execution Engine                    │  │
│  │  ┌──────────┐ ┌─────┐ ┌────────────────────┐│  │
│  │  │Interpreter│ │ JIT │ │ Garbage Collector  ││  │
│  │  └──────────┘ └─────┘ └────────────────────┘│  │
│  └──────────────────────────────────────────────┘  │
│                                                    │
│  ┌──────────────────────────────────────────────┐  │
│  │     Native Method Interface (JNI)             │  │
│  └──────────────────────────────────────────────┘  │
└───────────────────────────────────────────────────┘
```

> **Reference
**: [JVM Specification - Oracle](https://docs.oracle.com/javase/specs/jvms/se21/html/index.html)

---

## 2. ClassLoader Subsystem

The ClassLoader is responsible for **loading**, **linking**, and **initializing** classes.

### Loading Phase

Three built-in classloaders form a **delegation hierarchy** (parent-first):

```
Bootstrap ClassLoader (C/C++)
    └── Platform ClassLoader (formerly Extension)
            └── Application ClassLoader (classpath)
                    └── Custom ClassLoaders
```

| ClassLoader     | Loads From              | Examples                     |
|-----------------|-------------------------|------------------------------|
| **Bootstrap**   | `$JAVA_HOME/lib` (core) | `java.lang.*`, `java.util.*` |
| **Platform**    | Platform modules        | `java.sql.*`, `javax.*`      |
| **Application** | Classpath (`-cp`)       | Your application classes     |

### Parent Delegation Model

When a class needs to be loaded:

1. **Application CL** → asks parent (Platform CL).
2. **Platform CL** → asks parent (Bootstrap CL).
3. **Bootstrap CL** → tries to load. If found, done. If not, delegates back down.
4. Each loader tries only if its parent failed.

**Why?** Prevents duplicate class loading and ensures core classes (like `java.lang.String`) are
always loaded by the Bootstrap ClassLoader, providing security and consistency.

### Linking Phase

1. **Verify**: Bytecode verification (valid format, no illegal operations).
2. **Prepare**: Allocate memory for static fields, set default values (`0`, `null`, `false`).
3. **Resolve**: Replace symbolic references with direct references.

### Initialization Phase

- Execute static initializers (`static {}` blocks) and static field assignments.
- Happens **lazily** — only when the class is first actively used.

```java
// Custom ClassLoader example
public class MyClassLoader extends ClassLoader {

  @Override
  protected Class<?> findClass(String name) throws ClassNotFoundException {
    byte[] bytes = loadClassBytes(name); // read .class from custom source
    return defineClass(name, bytes, 0, bytes.length);
  }
}
```

**Interview tip**: Custom classloaders are used in app servers (Tomcat — each webapp has its own
classloader), OSGi, hot-reloading frameworks, and plugin systems.

> **Reference
**: [ClassLoader - Java 21 Docs](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/ClassLoader.html) | [Understanding ClassLoaders - Baeldung](https://www.baeldung.com/java-classloaders)

---

## 3. JVM Memory Model

### Runtime Data Areas

```
┌─────────────────── Shared (all threads) ──────────────────┐
│                                                            │
│  ┌──────────────────────────────────┐  ┌────────────────┐ │
│  │            HEAP                   │  │  Method Area   │ │
│  │  ┌───────────┐  ┌──────────────┐ │  │ (Metaspace)    │ │
│  │  │   Young   │  │     Old      │ │  │ - Class info   │ │
│  │  │Generation │  │  Generation  │ │  │ - Method data  │ │
│  │  │┌────┐┌──┐│  │              │ │  │ - Constant pool│ │
│  │  ││Eden││S0││  │              │ │  │ - Static vars  │ │
│  │  │├────┤├──┤│  │              │ │  └────────────────┘ │
│  │  ││    ││S1││  │              │ │                      │
│  │  │└────┘└──┘│  │              │ │                      │
│  │  └───────────┘  └──────────────┘ │                      │
│  └──────────────────────────────────┘                      │
└────────────────────────────────────────────────────────────┘

┌──────────────── Per-Thread ────────────────┐
│                                             │
│  ┌──────────┐  ┌──────────┐  ┌───────────┐│
│  │  Stack   │  │    PC    │  │  Native   ││
│  │ (frames) │  │ Register │  │  Method   ││
│  │          │  │          │  │   Stack   ││
│  └──────────┘  └──────────┘  └───────────┘│
└─────────────────────────────────────────────┘
```

### Heap (Shared)

- Stores **all object instances** and arrays.
- Managed by the **Garbage Collector**.
- Divided into **Young Generation** (Eden + Survivor S0/S1) and **Old Generation**.

### Method Area / Metaspace (Shared)

- Stores **class metadata**, method bytecode, constant pool, static variables.
- **Metaspace** (Java 8+) — uses **native memory**, not heap. Grows automatically (unlike the old
  PermGen which had a fixed size).
- Controlled by `-XX:MaxMetaspaceSize`.

### Stack (Per-Thread)

- Each thread has its own stack.
- Each method invocation creates a **stack frame** containing:
    - **Local variables array** (method params + local vars).
    - **Operand stack** (intermediate computation values).
    - **Frame data** (reference to runtime constant pool, exception table).
- Default stack size: ~512KB–1MB (`-Xss` flag).
- `StackOverflowError` when the stack is full (deep/infinite recursion).

### PC Register (Per-Thread)

- Points to the current bytecode instruction being executed.
- One per thread.

### Native Method Stack (Per-Thread)

- Used for native (JNI) method calls.

### Key JVM Memory Flags

| Flag                   | Purpose             | Example                                         |
|------------------------|---------------------|-------------------------------------------------|
| `-Xms`                 | Initial heap size   | `-Xms512m`                                      |
| `-Xmx`                 | Maximum heap size   | `-Xmx4g`                                        |
| `-Xss`                 | Thread stack size   | `-Xss512k`                                      |
| `-XX:MaxMetaspaceSize` | Max metaspace       | `-XX:MaxMetaspaceSize=256m`                     |
| `-XX:NewRatio`         | Old/Young ratio     | `-XX:NewRatio=2` (Old is 2x Young)              |
| `-XX:SurvivorRatio`    | Eden/Survivor ratio | `-XX:SurvivorRatio=8` (Eden is 8x one Survivor) |

> **Reference
**: [JVM Memory Structure - Oracle](https://docs.oracle.com/javase/specs/jvms/se21/html/jvms-2.html#jvms-2.5) | [JVM Tuning Guide](https://docs.oracle.com/en/java/javase/21/gctuning/introduction-garbage-collection-tuning.html)

---

## 4. Garbage Collection

### How GC Works — Generational Hypothesis

> "Most objects die young."

Objects are allocated in **Eden** (Young Gen). Those that survive multiple GC cycles are promoted to
**Old Gen**.

### GC Process

```
1. Object allocated in Eden
2. Eden fills up → Minor GC (Young Gen only)
   - Live objects copied to Survivor space (S0 ↔ S1)
   - Dead objects reclaimed
   - Objects surviving N cycles → promoted to Old Gen
3. Old Gen fills up → Major GC / Full GC
   - Much slower (larger space, more objects to scan)
   - Often causes application pause (stop-the-world)
```

### GC Roots (Reachability Analysis)

An object is eligible for GC if it is **not reachable** from any GC root.

**GC Roots include:**

- Local variables in active stack frames.
- Active threads.
- Static fields of loaded classes.
- JNI references.

### Garbage Collectors

| Collector                     | Type                            | Pause              | Throughput | Use Case                                     |
|-------------------------------|---------------------------------|--------------------|------------|----------------------------------------------|
| **Serial**                    | Single-threaded, stop-the-world | Long               | Low        | Small apps, client-side                      |
| **Parallel**                  | Multi-threaded, stop-the-world  | Medium             | High       | Batch processing, throughput-focused         |
| **G1** (default since Java 9) | Region-based, concurrent + STW  | Short, predictable | Good       | General purpose, balanced latency/throughput |
| **ZGC** (Java 15+)            | Concurrent, colored pointers    | <1ms (scalable)    | Good       | Ultra-low latency, large heaps (multi-TB)    |
| **Shenandoah**                | Concurrent, brooks pointers     | <10ms              | Good       | Low latency (Red Hat)                        |

### G1 Garbage Collector (Default)

- Divides heap into **equal-sized regions** (~1-32MB each).
- Regions are dynamically assigned as Eden, Survivor, Old, or Humongous.
- **Mixed GC**: collects Young + selected Old regions with most garbage ("garbage first").
- Goal: meet a **pause-time target** (`-XX:MaxGCPauseMillis=200` default).

```
G1 Heap Layout:
┌────┬────┬────┬────┬────┬────┬────┬────┐
│ E  │ E  │ S  │ O  │ O  │ H  │ O  │ E  │
└────┴────┴────┴────┴────┴────┴────┴────┘
E=Eden  S=Survivor  O=Old  H=Humongous
```

**Key G1 phases:**

1. **Young GC**: Evacuate live objects from Eden/Survivor regions.
2. **Concurrent Marking**: Identify garbage in Old regions (runs alongside app).
3. **Mixed GC**: Collect Young + highest-garbage Old regions.
4. **Full GC** (fallback): Stop-the-world, if concurrent marking can't keep up.

### ZGC (Ultra-Low Latency)

- **Sub-millisecond** pause times regardless of heap size (tested up to 16TB).
- Uses **colored pointers** (stores GC metadata in unused pointer bits).
- Uses **load barriers** instead of write barriers.
- Almost entirely **concurrent** — application threads rarely stop.
- Enable with: `-XX:+UseZGC`
- Generational ZGC (Java 21+): `-XX:+UseZGC -XX:+ZGenerational` — even better throughput.

### GC Tuning Basics

```bash
# Monitor GC activity
java -verbose:gc -Xlog:gc* -jar app.jar

# G1 with 200ms pause target
java -XX:+UseG1GC -XX:MaxGCPauseMillis=200 -Xmx4g -jar app.jar

# ZGC for ultra-low latency
java -XX:+UseZGC -Xmx8g -jar app.jar
```

**Common tuning goals:**

- **Throughput**: Maximize time spent on app code (Parallel GC or G1 with higher pause target).
- **Latency**: Minimize GC pauses (ZGC or G1 with lower pause target).
- **Footprint**: Minimize memory usage (Serial GC, smaller heap).

> **Reference
**: [G1 GC - Oracle](https://docs.oracle.com/en/java/javase/21/gctuning/garbage-first-g1-garbage-collector1.html) | [ZGC - Oracle](https://docs.oracle.com/en/java/javase/21/gctuning/z-garbage-collector.html) | [JEP 439: Generational ZGC](https://openjdk.org/jeps/439)

---

## 5. JIT Compilation (Just-In-Time)

### Interpretation vs Compilation

Java uses a **hybrid approach**:

```
.java → javac → .class (bytecode) → JVM
                                      ├── Interpreter (line by line, slow)
                                      └── JIT Compiler (hot code → native, fast)
```

### How JIT Works

1. JVM starts by **interpreting** bytecode.
2. Tracks **method invocation counts** and **loop back-edge counts**.
3. When a method exceeds the threshold → marks as **"hot"**.
4. JIT compiles hot methods to **native machine code** in the background.
5. Future calls use the compiled native code directly.

### Tiered Compilation (Default since Java 8)

| Level | Compiler            | Optimizations             | When                              |
|-------|---------------------|---------------------------|-----------------------------------|
| 0     | Interpreter         | None                      | Initially                         |
| 1     | C1 (Client)         | Basic                     | Simple methods, quick compilation |
| 2     | C1 + profiling      | Basic + counters          | Gathering profile data            |
| 3     | C1 + full profiling | Basic + detailed counters | More profile data                 |
| 4     | C2 (Server)         | Aggressive                | Hot methods after profiling       |

### Key JIT Optimizations

| Optimization               | Description                                                                    |
|----------------------------|--------------------------------------------------------------------------------|
| **Method inlining**        | Replaces method call with the method body (reduces call overhead)              |
| **Loop unrolling**         | Replaces loop iterations with repeated code (reduces branch overhead)          |
| **Escape analysis**        | If an object doesn't escape a method, allocate it on the **stack** (avoids GC) |
| **Dead code elimination**  | Removes unreachable code                                                       |
| **Lock elision**           | Removes locks if object is thread-confined (detected by escape analysis)       |
| **Null check elimination** | Removes redundant null checks                                                  |
| **Intrinsics**             | Replaces known methods (e.g., `Math.sqrt`) with optimized native instructions  |

### Viewing JIT Decisions

```bash
# Print compilation events
java -XX:+PrintCompilation -jar app.jar

# Print inlining decisions
java -XX:+UnlockDiagnosticVMOptions -XX:+PrintInlining -jar app.jar

# JIT log for JITWatch analysis
java -XX:+UnlockDiagnosticVMOptions -XX:+LogCompilation -jar app.jar
```

> **Reference
**: [JIT Compiler - Oracle](https://docs.oracle.com/en/java/javase/21/vm/java-hotspot-virtual-machine-performance-enhancements.html) | [JITWatch Tool](https://github.com/AdoptOpenJDK/jitwatch)

---

## 6. Java Memory Model (JMM)

The JMM defines how threads interact through memory and what behaviors are allowed.

### Key Concepts

#### Happens-Before Relationship

If action A **happens-before** action B, then A's effects are visible to B.

**Happens-before rules:**

1. **Program order**: Each action in a thread happens-before subsequent actions in the same thread.
2. **Monitor lock**: Unlock of a monitor happens-before every subsequent lock of that monitor.
3. **Volatile**: Write to a volatile variable happens-before every subsequent read of that variable.
4. **Thread start**: `thread.start()` happens-before any action in the started thread.
5. **Thread join**: All actions in a thread happen-before `join()` returns.
6. **Transitivity**: If A happens-before B, and B happens-before C, then A happens-before C.

#### Instruction Reordering

Both the **compiler** and **CPU** can reorder instructions for performance, as long as
single-threaded semantics are preserved. But reordering can break multi-threaded code without proper
synchronization.

```java
// Without synchronization, Thread 2 might see x=0, flag=true (reordered)
// Thread 1                    // Thread 2
x =42;if(flag){
flag =true;

print(x); // might print 0!
                               }

// Fix: make flag volatile — establishes happens-before
private volatile boolean flag;
```

> **Reference
**: [JSR 133 (Java Memory Model) FAQ](https://www.cs.umd.edu/~pugh/java/memoryModel/jsr-133-faq.html) | [JMM Spec](https://docs.oracle.com/javase/specs/jls/se21/html/jls-17.html#jls-17.4)

---

## 7. String Pool & Interning

### String Pool (String Constant Pool)

- Stored in the **heap** (moved from PermGen in Java 7).
- String **literals** are automatically interned (stored in the pool).
- `new String("hello")` creates a new object on the heap, NOT in the pool.

```java
String s1 = "hello";           // from string pool
String s2 = "hello";           // same reference from pool
String s3 = new String("hello"); // new object on heap

s1 ==s2;    // true  (same pool reference)
s1 ==s3;    // false (different objects)
s1.

equals(s3); // true (same content)

s3 =s3.

intern(); // put into pool, returns pool reference

s1 ==s3;    // true
```

### Why Strings are Immutable

1. **String pool** would be impossible if strings were mutable (shared references).
2. **Thread safety** — can be shared across threads without synchronization.
3. **Security** — used in class loading, file paths, network URLs.
4. **Hash caching** — `hashCode()` is computed once and cached.

> **Reference
**: [String - Java 21 Docs](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/String.html)

---

## 8. Compilation: javac to Execution

### Full Lifecycle

```
Source (.java)
    │
    ▼  javac (Java Compiler)
Bytecode (.class)
    │
    ▼  ClassLoader
Loaded into JVM
    │
    ▼  Bytecode Verifier
Verified
    │
    ├──▶ Interpreter (all code initially)
    │
    └──▶ JIT Compiler (hot code → native)
            │
            ▼
    Native Machine Code (cached in Code Cache)
```

### Key Diagnostic Tools

| Tool                                      | Purpose                               |
|-------------------------------------------|---------------------------------------|
| `javap -c`                                | Disassemble .class file to bytecode   |
| `jps`                                     | List running JVM processes            |
| `jstack <pid>`                            | Thread dump (detect deadlocks)        |
| `jmap -heap <pid>`                        | Heap summary                          |
| `jmap -dump:format=b,file=heap.bin <pid>` | Heap dump for analysis                |
| `jstat -gc <pid>`                         | GC statistics                         |
| `jconsole` / `VisualVM`                   | GUI monitoring (memory, threads, CPU) |
| `jfr` (Java Flight Recorder)              | Low-overhead profiling for production |
| `async-profiler`                          | CPU/allocation profiling              |

> **Reference
**: [JDK Tools - Oracle](https://docs.oracle.com/en/java/javase/21/docs/specs/man/index.html) | [VisualVM](https://visualvm.github.io/) | [async-profiler](https://github.com/async-profiler/async-profiler)

---

## 9. Class File Format & Bytecode

### .class File Structure

```
ClassFile {
    magic: 0xCAFEBABE
    minor_version, major_version
    constant_pool
    access_flags
    this_class, super_class
    interfaces
    fields
    methods (each contains bytecode in Code attribute)
    attributes
}
```

### Common Bytecode Instructions

| Instruction             | Description                                                   |
|-------------------------|---------------------------------------------------------------|
| `aload_0`               | Load `this` reference onto operand stack                      |
| `invokevirtual`         | Call instance method (with polymorphism)                      |
| `invokeinterface`       | Call interface method                                         |
| `invokespecial`         | Call constructor, super, or private method                    |
| `invokestatic`          | Call static method                                            |
| `invokedynamic`         | Dynamic dispatch (lambdas, string concatenation since Java 9) |
| `new`                   | Create new object                                             |
| `getfield` / `putfield` | Read/write instance field                                     |

```bash
# View bytecode
javap -c -p MyClass.class
```

> **Reference
**: [JVM Instruction Set](https://docs.oracle.com/javase/specs/jvms/se21/html/jvms-6.html)

---

## Interview Questions Cheat Sheet

| Question                        | Key Points                                                                                             |
|---------------------------------|--------------------------------------------------------------------------------------------------------|
| Explain JVM architecture        | ClassLoader → Memory (Heap, Stack, Metaspace, PC) → Execution Engine (Interpreter, JIT, GC)            |
| Heap vs Stack                   | Heap = objects, shared, GC-managed. Stack = local vars/frames, per-thread, auto-freed                  |
| What is Metaspace?              | Replaced PermGen in Java 8. Stores class metadata in native memory. Auto-grows.                        |
| How does G1 GC work?            | Region-based, targets pause time, concurrent marking, mixed GC for high-garbage old regions            |
| G1 vs ZGC                       | G1: general purpose, ~200ms pauses. ZGC: <1ms pauses, concurrent, for large heaps                      |
| What is JIT compilation?        | Compiles hot bytecode to native code at runtime. C1 for quick compile, C2 for aggressive optimization. |
| What is escape analysis?        | JIT optimization: if object doesn't escape method, allocate on stack instead of heap (no GC needed)    |
| ClassLoader delegation          | Child asks parent first. Bootstrap → Platform → Application. Prevents core class tampering.            |
| What causes `OutOfMemoryError`? | Heap full (leak), Metaspace full (too many classes), too many threads (native memory), GC overhead     |
| How to diagnose memory leak?    | Heap dump (`jmap`), analyze in MAT/VisualVM, look for growing collections or unclosed resources        |

---

## Deep Dive Resources (by Section)

### 1. JVM Architecture

- [Understanding JVM Architecture — DZone](https://dzone.com/articles/jvm-architecture-explained)
- [JVM Internals — James Bloom (comprehensive deep dive)](https://blog.jamesdbloom.com/JVMInternals.html)
- [How the JVM Works — Baeldung](https://www.baeldung.com/jvm-vs-jre-vs-jdk)

### 2. ClassLoader Subsystem

- [Java ClassLoaders — Baeldung](https://www.baeldung.com/java-classloaders)
- [Understanding Java ClassLoading — DZone](https://dzone.com/articles/java-classloading-understanding)
- [Tomcat ClassLoader How-To — Apache Tomcat Docs](https://tomcat.apache.org/tomcat-10.1-doc/class-loader-howto.html)

### 3. JVM Memory Model

- [JVM Memory Structure — Baeldung](https://www.baeldung.com/java-stack-heap)
- [Understanding Java Memory Model — Jenkov](https://jenkov.com/tutorials/java-concurrency/java-memory-model.html)
- [Metaspace in Java 8 — DZone](https://dzone.com/articles/java-8-permgen-vs-metaspace)
- [JVM Anatomy Quarks (deep series) — Aleksey Shipilev](https://shipilev.net/jvm/anatomy-quarks/)

### 4. Garbage Collection

- [Java Garbage Collection Handbook — Plumbr (comprehensive guide)](https://plumbr.io/java-garbage-collection-handbook)
- [G1 GC Deep Dive — Oracle](https://docs.oracle.com/en/java/javase/21/gctuning/garbage-first-g1-garbage-collector1.html)
- [Understanding G1 GC Logs — DZone](https://dzone.com/articles/understanding-g1-gc-log-format)
- [ZGC: The Next Generation Low-Latency GC — Inside Java](https://inside.java/2023/11/28/gen-zgc-explainer/)
- [Generational ZGC (JEP 439) — OpenJDK](https://openjdk.org/jeps/439)
- [Shenandoah GC Wiki — OpenJDK](https://wiki.openjdk.org/display/shenandoah/Main)
- [GC Tuning Basics — Baeldung](https://www.baeldung.com/jvm-garbage-collectors)

### 5. JIT Compilation

- [JIT Compilation Overview — Baeldung](https://www.baeldung.com/graal-java-jit-compiler)
- [Understanding JIT Compilation — IBM Developer](https://developer.ibm.com/articles/the-jit-compiler/)
- [Tiered Compilation in JVM — DZone](https://dzone.com/articles/tiered-compilation-in-jvm)
- [Escape Analysis in JVM — Baeldung](https://www.baeldung.com/java-escape-analysis)
- [JITWatch — Visualize JIT compiler decisions](https://github.com/AdoptOpenJDK/jitwatch)

### 6. Java Memory Model (JMM)

- [JSR 133 FAQ — University of Maryland](https://www.cs.umd.edu/~pugh/java/memoryModel/jsr-133-faq.html)
- [Java Memory Model Deep Dive — Jenkov](https://jenkov.com/tutorials/java-concurrency/java-memory-model.html)
- [Happens-Before Explained — Baeldung](https://www.baeldung.com/java-happens-before)
- [Close Encounters of the Java Memory Model Kind (talk) — Aleksey Shipilev](https://shipilev.net/blog/2014/jmm-pragmatics/)

### 7. String Pool & Interning

- [String Pool in Java — Baeldung](https://www.baeldung.com/java-string-pool)
- [Why String is Immutable in Java — Baeldung](https://www.baeldung.com/java-string-immutable)
- [String Concatenation Internals (invokedynamic) — DZone](https://dzone.com/articles/jdk-9jep-280-string-concatenation)

### 8. Compilation & Bytecode

- [Understanding Java Bytecode — DZone](https://dzone.com/articles/introduction-to-java-bytecode)
- [javap: Disassembling Java Bytecode — Baeldung](https://www.baeldung.com/java-class-view-bytecode)
- [Java Flight Recorder Guide — Baeldung](https://www.baeldung.com/java-flight-recorder-monitoring)
- [async-profiler Usage Guide — GitHub](https://github.com/async-profiler/async-profiler)
- [Troubleshooting Memory Leaks with VisualVM — Baeldung](https://www.baeldung.com/java-memory-leaks)
