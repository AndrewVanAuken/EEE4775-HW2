# EEE4775 HW2
**Name**: Andrew VanAuken  
  
## Part A – ESP32 memory map
### 1. Internal SRAM
**Address Range:**
- Instruction Bus (IRAM): 0x40370000 – 0x403DFFFF
- Data Bus (DRAM): 0x3FC88000 – 0x3FCFFFFF
  
**Typical Use:** Used for program execution, stack memory, heap memory, global variables, FreeRTOS data structures, and time-critical code placed in IRAM.

### 2. External flash (mapped via cache; data and instruction caches are separate!)
**Address Range:**
- Instruction Bus (IRAM): 0x42000000 – 0x43FFFFFF
- Data Bus (DRAM): 0x3C000000 – 0x3DFFFFFF
  
**Typical Use:** Stores application code, constants, read-only data, and other information kept in external flash memory. Access occurs through the ESP32-S3 cache system.

### 3. ROM (there's a 0 and 1; why?)
### ROM0
**Address Range:** 0x40000000 – 0x4003FFFF

**Typical Use:** Contains built-in bootloader code and ROM functions provided by Espressif.

### ROM1
**Address Range:** 0x3FF00000 – 0x3FF1FFFF

**Typical Use:** Contains additional ROM-resident data and support functions that are accessed through the data bus.

### 4. Interrupt / ISR (if you can't find an address, state where these exist)
**Address Range:** No address range. ISRs are placed in memory by the linker and may reside in flash or IRAM depending on the configuration.

**Typical Use:** Interrupt Service Routines handle hardware events such as timers, GPIO interrupts, communication peripherals, and other asynchronous events.

**Why does putting an ISR "close to the processor" matter for worst-case latency?**
- Placing an ISR in IRAM improves worst-case interrupt latency because the processor can execute the code directly from internal memory without waiting for external flash access through the cache. If an ISR is stored in flash, cache misses or cache-disabled situations can introduce delays. Keeping critical ISRs in IRAM makes interrupt response more predictable, which is important for real-time systems.

---

## Part B – C concept exercises
## Wokwi Project
https://wokwi.com/projects/465571531454799873

---

## Part C - `volatile` and the compiler
### 1. If two FreeRTOS tasks share an `int`, is `volatile` enough? Why or why not?
- No, volatile is not enough. It only tells the compiler not to cache the variable in a CPU register and to always read/write directly to memory. It does not guarantee atomicity or memory ordering.

### 2. Minimum guarantee `volatile` actually provides on Xtensa?
- The minimum guarantee provided by `volatile` is that every read and write written in the source code will result in an actual memory access. The compiler cannot cache the value in a register or remove accesses that appear unnecessary.

### 3. Name a primitive that gives the guarantee `volatile` does not.
- An `atomic` type (like stdatomic.h in C11) or a critical section in FreeRTOS guarantees atomicity and memory ordering. `volatile` only ensures the compiler doesn't optimize memory access. It does not guarantee that a multi-instruction operation won't be interrupted or reordered by the CPU.

### Citation
https://www.freertos.org/FreeRTOS_Support_Forum_Archive/March_2016/freertos_Atomicity_and_volatile_6c001145j.html

---

## Part D - Industry anchor
### Rule 1. 
- Keep control flow simple and avoid constructs such as recursion and goto.

### Rule 2.
- Every loop should have a known upper bound.

### Rule 3.
- Do not use dynamic memory allocation after initialization.

### Rule 4.
- Keep functions small and focused.

### Rule 5.
- Use assertions to verify assumptions and detect errors.

### Rule 6.
- Declare variables in the smallest scope possible.

### Rule 7.
- Always check return values and validate inputs.

### Rule 8.
- Use the preprocessor sparingly and keep macros simple.

### Rule 9.
- Limit pointer complexity and avoid unnecessary indirection.

### Rule 10.
- Compile with strong warnings enabled and use static analysis tools.

### Hardest Rule to follow
The rule I would find hardest to follow is Rule 2. In previous classes, many of the programs on the MSP430 used loops that ran until a condition changed or used infinite loops as part of the main structure. Because of that, I did not spend much time thinking about the maximum time a loop could run. Working with the ESP32 and learning more about real-time systems has allowed me to see how a long-running loop could affect system timing. To follow this rule better, I would need to design loops with clear limits and exit conditions instead of assuming they’ll eventually finish. I would also spend more time considering whether breaking a large task into smaller pieces would make the system easier to maintain.

### Citation
https://en.wikipedia.org/wiki/The_Power_of_10:_Rules_for_Developing_Safety-Critical_Code
