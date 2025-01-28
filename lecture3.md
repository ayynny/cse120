# Timesharing

- **Definition**: Multiplexing the use of the CPU over time.
- **Assumption**: Only one CPU is available.
  - Consequently, only one program can use that CPU at any instant.
- **Idea**: Slice CPU time among multiple programs (e.g., 3 programs “running simultaneously”).
  - Each program receives some fraction of CPU time.
  - At any single point in time, only one program actually runs.
- **Illusion**: If the time quantum (the time slice) is small enough, it appears as though all programs progress smoothly in parallel.

---

## Process Implementation

- The **kernel** keeps track of each process and its state.
- **Common Process States**:

  1. **Running**: The process is currently using the CPU.
  2. **Ready**: The process can run but is waiting for the CPU.
  3. **Blocked**: The process cannot make progress (it is waiting for some event to occur) and thus cannot use the CPU.

- **Scheduling**:
  - The kernel selects a **Ready** process and lets it run.
  - Eventually, the kernel regains control (via system calls, interrupts, etc.) and selects another process to run.

---

## State Transitions

1. **Dispatch**: Allocate the CPU to a **Ready** process (transition from **Ready** → **Running**).
2. **Preempt**: Take away the CPU from a **Running** process (transition from **Running** → **Ready**).
3. **Sleep**: A process voluntarily gives up the CPU to wait for an event (transition from **Running** → **Blocked**).
4. **Wakeup**: The event has occurred, making the previously blocked process **Ready** again (transition from **Blocked** → **Ready**).

> **Example**: If a process is in the Running state and cannot make further progress (e.g., it’s waiting for I/O), it will go to **Sleep** and become **Blocked**.

---

## Physical vs. Logical Execution

- **Physical Execution**: Whether a process is actually on the CPU (Running) or not (Ready/Blocked).
- **Logical Execution**: From a programmer’s or system’s conceptual standpoint, what the program is allowed or able to do next.
  - Sometimes we reason about a program’s progress or correctness logically, even if physically the CPU might be shared.

---

## Process vs. Kernel

- **Kernel**:
  - Core code supporting processes (manages scheduling, context switching, system calls, etc.).
  - Runs as an extension of the current process or in response to hardware interrupts.
  - Implements **system calls** (e.g., `fork()`, `exit()`, etc.).
- **When the kernel runs**:
  1. Whenever a process makes a **system call** (e.g., a read that might block).
  2. In response to **interrupts** (e.g., timer interrupts, device interrupts).
- **Data structures**:
  - The kernel maintains a list (or table) of processes, tracking:
    - Process ID
    - State (Running, Ready, Blocked)
    - Other metadata (e.g., scheduling priority, memory usage)

---

## How the Kernel Gets Control

1. **Voluntary**:

   - The running process calls a system function that may block (e.g., `read()`).
   - The running process calls a function like `yield()` to relinquish the CPU.
   - In both cases, the kernel can then select a new **Ready** process to run.

2. **Involuntary** (Preemption):
   - A **hardware timer interrupt** occurs after a time slice expires.
   - The interrupt automatically transfers control to the kernel.
   - The kernel then decides which process to run next.

---

## Context Switching

1. A **process** makes a system call or an **interrupt** occurs.
2. The hardware triggers a **trap** or **interrupt**, which switches the CPU mode from **user mode** to **kernel mode**.
3. In **kernel mode**, the kernel:
   - Saves the context (CPU registers, program counter, etc.) of the current process.
   - Selects a **Ready** process to run next.
   - Restores the context of the chosen process.
4. A **return-from-interrupt (RTI)** or **return-from-trap** instruction switches the CPU back to user mode, and execution continues in the newly selected process.

---

## Parallelism and Processes

- A **process** is a program in execution, typically with:
  - Its own address space (memory).
  - Its own registers and state.
- To have multiple paths of execution within a single process **sharing the same address space**, we use **threads**.

---

# Threads

- **Definition**: A thread is a single sequential path of execution within a process.
- **Key Point**: Threads share the process’s memory (address space) but each thread has:
  - Its own program counter (instruction pointer).
  - Its own register set.
  - Its own stack.
- **Motivation**:
  - Multiple threads in one process allow for concurrency/parallelism within the same address space.
  - The kernel schedules threads (not processes) when kernel-level threading is used.

---

## Implementing Threads

### Kernel-Level Threads

- **Thread calls** (create, exit, join) involve the kernel.
- Each thread has:
  - A **user stack**.
  - A **kernel stack**.
- **True parallelism** is possible if the system has multiple CPUs/cores:
  - The kernel can schedule different threads of the same process on different CPUs.
- **Overhead**: Switching between threads requires a kernel-mode context switch.

### User-Level Threads

- Threads are managed by a **user-level library**, entirely in user space.
- **Advantages**:
  - **Portability**: Works on any OS, whether or not the kernel supports threads.
  - **Efficiency**: Switching between threads in user space is fast (no kernel-mode switch).
- **Disadvantages**:
  - The kernel only sees one process; if one user-level thread blocks (e.g., on I/O), the entire process (and thus all user-level threads within it) may also block.
  - No true parallelism on multiprocessor systems unless the kernel is aware of multiple threads.

---

**Note**:

- **Thread Support** refers to how threads are _created and managed_ (in the kernel or in user space).
- **Thread Execution** can happen in user mode or kernel mode (e.g., when a thread makes a system call, it briefly executes in kernel mode).
