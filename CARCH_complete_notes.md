# Computer Architecture: Complete Notes (Sem 7)

My handwritten class notes (113 pages), typed up lecture by lecture, with the gaps filled from **Patterson & Hennessy, *Computer Organization and Design*, 4th ed.** The book uses MIPS; every example here is rewritten in **RISC-V**.

**Legend:**
- **➕ Added / ➕ Completion:** content that wasn't in my notes, filled in from the book.
- **⚠️ Correction:** something in my handwritten notes that was wrong.
- **[illegible]:** couldn't be read from the scan.

**Exam 2:** 15 Nov, 10:00–1:00. Units 3–4 are the post-Exam-1 material.

---
## Contents

- Lecture 02 — Basics (30 Jul 2026) · page 1
- Lecture 03 — ISA intro: add / sub / addi (31 Jul 2026) · pages 2–3
- Lecture 04 — Memory operands: ld / sd, endianness (04 Aug 2026) · pages 4–7
- Lecture 05 — Signed/unsigned numbers, logical ops (06 Aug 2026) · pages 8–9
- Lecture 06 — Logical & conditional operations (07 Aug 2026) · pages 10–11
- Lecture 07 — Signed comparisons, jumps, special registers (11 Aug 2026) · pages 12–13
- Lecture 08 — Large immediates, pseudo-instructions, RISC-V extensions (13 Aug 2026) · pages 14–15
- Lecture 09 — Representing instructions: R and I formats (14 Aug 2026) · pages 16–17
- Lecture 10 — S format, PC, branch addressing, U format (18 Aug 2026) · pages 18–19
- Lecture 11 — Jump addressing, decoding (20 Aug 2026) · pages 20–21
- Lecture 12 — Procedure Calling in RISC-V (21 Aug 2026) · pages 22–27
- Lecture 13 — Leaf Procedures & Register Usage (25 Aug 2026) · pages 28–29
- Lecture 14 — Non-Leaf Procedures (27 Aug 2026) · pages 30–31
- Lecture 15 — CPU Execution Time (22 Sep 2026) · page 32
- Lecture 16 — Arithmetic for Computers (01 Sep 2026) · pages 33–34
- Lecture 17 — Multiplication & Division (03 Sep 2026) · pages 35–36
- Lecture 18 — Fixed Point & Floating Point Representation (04 Sep 2026) · pages 37–39
- Lecture 19 — Arithmetic on Floating Points (08 Sep 2026) · page 40
- Lecture 20 — Floating-Point Instructions in RISC-V (10 Sep 2026) · pages 41–43
- Lecture 21 — Other FP Standards, Conclusion & Custom Multiplication (11 Sep 2026) · pages 44–46
- Lecture 22 — Intro to Processor Implementation, Logic Design Basics, Combinational Elements (15 Sep 2026) · pages 47–50
- Lecture 23 — ALU, Sequential Elements, Register File, Memory (17 Sep 2026) · pages 51–56
- Lecture 24 — Clocking, CPU Overview, Datapath & Control Basics (18 Sep 2026) · pages 57–59
- Lecture 25 — Full Datapath & Control Path (22 Sep 2026) · pages 60–61
- Lecture 26 — Multicycle Datapath & Pipelining Intro (29 Sep 2026) · pages 62–65
- Lecture 27 — Pipelining and ISA Design; Hazards intro (01 Oct 2026) · pages 66–68
- Lecture 28 — Data Hazards: NOPs, Code Scheduling, Forwarding (06 Oct 2026) · pages 69–71
- Lecture 29 — Forwarding recap, Stalls, Control Hazards, Branch Prediction intro (08 Oct 2026) · pages 72–75
- Lecture 30 — Branch Prediction (1-bit, 2-bit), Branch Data Hazards, Exceptions (09 Oct 2026) · pages 76–79
- Lecture 31 — Problems (13 Oct 2026) · pages 80–81
- Lecture 32 — Improving Performance Further: ILP / Multi-issue (15 Oct 2026) · pages 82–84
- Lecture 33 — (16 Oct 2026) · page 85
- Lecture 34: Memory hierarchy (27 Oct 2026) · pages 86–89
- Lecture 35: Cache memory, direct-mapped cache (29 Oct 2026) · pages 90–93
- Lecture 36: Direct-mapped cache (cont.), main memory supporting caches (30 Oct 2026) · pages 94–96
- Lecture 37: Associativity, CAM, writing (03 Nov 2026) · pages 97–98
- Lecture 38: Caches, write policies (05 Nov 2026, Thursday) · pages 99–100
- Lecture 39: Important terms, sources of misses, design trade-offs, unified vs split, cache performance (06 Nov 2026, Friday) · pages 101–104
- Lecture 40: Multilevel caches, overhead of caches, SPM, prefetching (10 Nov 2026, Tuesday) · pages 105–108
- Lecture 41: I/O, MMIO, interrupts, DMA (12 Nov 2026, Thursday) · pages 109–113
- Formula sheet (end)

---




---

# UNIT 1: Instruction Set Architecture (RISC-V)

---

## Lecture 02 — Basics (30 Jul 2026) · page 1

- No notes for this lecture: "I did not have a pen to write notes." (Page is otherwise blank.)


---

### ➕ Completion (from P&H): L2: CARCH basics (p.1: "did not have a pen")

**Levels of abstraction:** application → algorithm → high-level language → (compiler) → assembly → (assembler) → machine code → **ISA** → microarchitecture/processor → circuits/gates → devices/transistors. Same picture as my p.86.

**ISA vs microarchitecture:**
- The **ISA** is the contract between hardware and software: instructions, registers, memory model, encodings (e.g. RV64I).
- The **microarchitecture** is one particular implementation of it (single-cycle, pipelined, OoO). Many microarchitectures can implement the same ISA.

**Five classic components:** input, output, memory, **datapath**, **control**. Datapath + control = the processor.

**Eight great ideas (P&H ch.1):**
1. Design for Moore's law
2. Use abstraction
3. Make the common case fast
4. Performance via parallelism
5. Performance via pipelining
6. Performance via prediction
7. Hierarchy of memories
8. Dependability via redundancy

**RISC-V word size** (from my typed note): word = 32 bits always. The register width is called **XLEN** (32 for RV32, 64 for RV64). A doubleword is 64 bits.


---

## Lecture 03 — ISA intro: add / sub / addi (31 Jul 2026) · pages 2–3

> My note: complete detailed notes weren't written because I didn't know slides wouldn't be provided.

- Chapter 1 wrapped up with a slide on **RISC vs CISC**, then we moved on to RISC-V.

### Chapter 02: Instruction Set Architecture

#### Add / subtract
- Arithmetic instructions have **3 operands**:
  - 2 source operands (the latter two)
  - 1 destination operand (the first one)

```asm
add a, b, c     # a <- b + c
sub a, b, c     # a <- b - c
```

**Example:** compile `f = (g + h) - (i + j);`

```c
f = (g + h) - (i + j);
```

```asm
add t0, g, h
add t1, i, j
sub f, t0, t1
```

> ⚠️ Correction: the notes write the last line as `add f, t0, t1`; for `(g+h) - (i+j)` it must be `sub f, t0, t1`.

#### Registers
- **NOTE:** RISC-V has **32 registers**. Their *width* depends on the architecture (RV32 → 32-bit, RV64 → 64-bit). **We use RV64.**
- Typed note (word size):
  - People often say word size = architecture size (32-bit arch → 32-bit word, 64-bit arch → 64-bit word).
  - But in **RISC-V documentation, a "word" is fixed at 32 bits**, whether RV32 or RV64. The architecture's natural width is called **XLEN** (32 or 64 bits).
  - (So in RV64: word = 32 bits, doubleword = 64 bits.)
- Registers are named **x0 to x31**.

**Example:** `f = (g + h) + (i + j)` with f→x19, g→x20, h→x21, i→x22, j→x23

```asm
add x5,  x20, x21
add x6,  x22, x23
add x19, x5,  x6
```

- Common mistake: forgetting to put the final result in the register that is needed (here **x19**).
- There is no restriction on a register being a source *and* a destination register.

**Think yourself — Q:** Can you multiply using just add & sub?
**A:** No (not in general) — you need a compare/branch to know where to stop the repeated addition. Similarly, exponents can't be done either.

### Handling constants in an expression
- Use **immediate operands**:

```asm
addi x22, x21, 2    # x22 <- x21 + 2 ; avoids a load instruction to fetch the constant
```

**Q:** Do we need a `subi`?
**A:** No — `addi` with a negative constant does the same job.

- Operands can be:
  - a register: x0, x1, …, x31
  - an immediate operand: a constant value

**Doubt — Q:** Is it true that only the third operand is allowed to be a constant value?
**A:** Yes — only the third operand can be an immediate value.


---

## Lecture 04 — Memory operands: ld / sd, endianness (04 Aug 2026) · pages 4–7

So far we had `addi rd, rs1, imm` (rd = destination register, rs = source register — the order matters: destination first).

**Q:** Do we need `addi rd, imm1, imm2`?
**A:** No — operations on two constants (like 5 + 8) can be done by the compiler.

**Q:** How do we handle composite data structures (arrays, structs) that don't fit in 32 registers?
**A:** **Memory operands.**

Diagram (CPU ↔ main memory):

```
 +-----+------+   LOAD  (memory -> register)    +-------------+
 | CPU | regs | <------------------------------ | main memory |
 |     | ALU  | ------------------------------> |             |
 +-----+------+   STORE (register -> memory)    +-------------+
```

### Storing arrays in memory
- Example: 10 elements, each 64-bit → 80 bytes, stored in **contiguous locations**.
- Every address location in memory holds **1 byte** → **byte-addressable memory**.

**Q:** You want to store the bytes 1 to 8 in memory. How do you choose the order?
**A:** It depends on what the processor designer wants to implement, but it must be documented for the user.
- Choice 1: **little endian**
- Choice 2: **big endian**

Side diagram (two columns of 8 memory cells): choice 1 stores 1,2,…,8 top-to-bottom; choice 2 stores 8,7,…,1.

- Either works if you read all 8 bytes together, but to read smaller portions you need to know the offset. Example: read the 3rd byte of an integer.
- **Most processors follow little endian:** MSB at the higher address, LSB at the lower address.

Byte numbering of a 64-bit value: B7 = MSB … B0 = LSB (B7 B6 B5 B4 B3 B2 B1 B0).

| Address | Little endian (LE) | Big endian (BE) |
|---|---|---|
| 7 | B7 | B0 |
| 6 | B6 | B1 |
| 5 | B5 | B2 |
| 4 | B4 | B3 |
| 3 | B3 | B4 |
| 2 | B2 | B5 |
| 1 | B1 | B6 |
| 0 | B0 | B7 |

Typed note (endianness check):

```c
int x = 0x12345678;
char *p = (char *)&x;
printf("%x\n", p[0]);
```

- On a **little-endian** machine: prints `78`
- On a **big-endian** machine: prints `12`
- Because `p` points to the **lowest address**. ("Think once again.")

Example: two 64-bit data P and R stored one after the other, little endian:
`P0 P1 P2 … P7 R0 R1 … R7` (increasing addresses).

### Memory operand example

```c
A[12] = h + A[8];   // each element is 8 bytes; h -> x21, base address of A -> x22
```

- Address of A[8] = x22 + 8×8 = x22 + 64; address of A[12] = x22 + 8×12 = x22 + 96.

```asm
addi x23, x22, 64     # address computation (done by compiler)
ld   x9, 0(x23)       # load A[8]
add  x9, x21, x9      # small optimization: reuse x9
addi x23, x22, 96
sd   x9, 0(x23)       # store into A[12]
```

- If the value is in a register → use it directly in `add`; if you only have a pointer (address known) → use `ld` and `sd`.
- `ld` = load doubleword; `sd` = store doubleword.
- `ld x9, n(x23)` ⇒ x9 ← Mem[x23 + n]; similarly for `sd`. The compiler takes care of the offset.
- We **can't** add memory operands directly in arithmetic ops — arithmetic works only on registers (unlike CISC; RISC-V is a **load–store architecture**).

**Q:** Can we do some optimisation here?
**A:** Yes — fold the offset into the load/store:

```asm
ld  x9, 64(x22)
add x9, x21, x9
sd  x9, 96(x22)
```

> ⚠️ Correction: the notes write `add x23, x21, x9` followed by `sd x9, 96(x22)` — that would store the old A[8]. Either use `add x9, x21, x9` (as above) or store x23.

- Is this always useful? Not during iterations (where the address changes) — there we have to write it manually.

**Example (loop vs. unrolling):**

```c
for (i = 0; i < 500; i++)
    A[i] = 2;      // x21 holds 2, x22 = base of A
```

This way (loop body):

```asm
# loop {
sd   x21, 0(x22)
addi x22, x22, 8
# }
```

Other way (**unrolled**):

```asm
sd x21, 0(x22)
sd x21, 8(x22)
...
sd x21, 3992(x22)
```

| | Loop | Unrolled |
|---|---|---|
| # instructions executed | 1000 | 500 |
| Manual writing | easy | 500 lines to be written |
| Scalability | scalable | not scalable (OK for small arrays) |

- The compiler can be told which kind of code to generate. **Unrolling** = unroll the loop.
- **Doubt:** Do we have loops in assembly? Yes, maybe taught later (via branches — see L6).

Format reminder (boxed):

```asm
sd rs2, offset(rs1)   # rs2 = source register, rs1 = base address register
ld rd,  offset(rs1)   # rd  = destination,     rs1 = base address register
```

**Example:**

```c
A[12] = h + A[8];
A[10] = k + A[8];      // h -> x21, k -> x20, A (base) -> x22
```

Option 1:

```asm
ld  x9, 64(x22)
add x10, x21, x9
add x11, x20, x9
sd  x10, 96(x22)
sd  x11, 80(x22)
```

Option 2 (OR):

```asm
ld  x9, 64(x22)
add x10, x21, x9
sd  x10, 96(x22)
add x10, x20, x9
sd  x10, 80(x22)
```

- Optimisation: the load was **reused** (A[8] wasn't modified in between). To choose between options, analyse which one costs less.
- Registers are faster to access than memory, and better energy-wise; more instructions (due to `ld`/`sd`) cost more — a load is costlier than an add.
- Compilers use registers as much as possible → **register allocation**.


---

## Lecture 05 — Signed/unsigned numbers, logical ops (06 Aug 2026) · pages 8–9

### Until now
- Overview of the computer
- Basic RISC-V ISA: add, sub, addi
- Data layout in memory: byte-addressable, little endian
- Data transfer: load and store operations

### Mini quiz
- `mov` is an instruction in other ISAs. RISC-V doesn't need it: `addi x, y, 0` copies y into x (also `add x, y, x0`, since **x0 is always 0**).
- A way to initialise a register to 0: `sub x5, x6, x6` ⇒ x5 becomes 0.

### Unsigned and signed (2's complement) integers
- Negation → 2's complement = complement, then +1.
- Why is 2's complement the negative of the number?
  - x + x̄ = 111…1₂
  - x + x̄ + 1 = 1000…0₂ (the carry out is overflow/discarded) = 0
  - ⇒ **−x = x̄ + 1**
- 2's complement of (2's complement of x) = x.
- Value of a 2's complement number: MSB is the sign (weight −2ⁿ⁻¹), everything else the same. Range reduces to **−2ⁿ⁻¹ to 2ⁿ⁻¹ − 1**.

> ⚠️ Correction: notes write the upper limit as 2ⁿ − 1; for signed n-bit it is 2ⁿ⁻¹ − 1 (2ⁿ − 1 is the unsigned max).

- **Sign extension** (signed only):
  - positive binary number: add 0s
  - negative binary number: add 1s
  - For unsigned, always add 0s (zero extension).

**Q:** How does the CPU know which to do?
**A:** There are 2 instructions:
- `lb` → load byte → sign-extend
- `lbu` → load byte unsigned → zero-extend
- We don't need `ldu`, because the max we can load is 64 bits — no extension needed. Extension is needed only for byte, halfword and word loads.
- C language has signed/unsigned types; the compiler decides whether to use signed/unsigned instructions.
- **What is the default? → signed.**

### Multi-word addition
- For more-bit addition, make sure the carry isn't lost: use a pair of registers to store each number (P:Q + R:S).
- How do you know there was a carry? → (unsigned) if the result is less than one of the operands.

### Logical operations
- Shift left, shift right, bit-by-bit AND, OR, XOR, NOT → bitwise manipulation.

| Operation | Instructions |
|---|---|
| Shift left | `sll`, `slli` |
| Shift right | `srl`, `srli`, `sra`, `srai` |
| AND | `and`, `andi` |
| OR | `or`, `ori` |
| XOR | `xor`, `xori` |


---

## Lecture 06 — Logical & conditional operations (07 Aug 2026) · pages 10–11

- We don't need an unsigned store (e.g. `sbu`/`sdu`) because memory is byte-addressable — we just store the bits as they are. (Student: "didn't click much".)

> ➕ Added: a store writes exactly the low n bits of the register to memory, so there's nothing to extend. Signed vs unsigned only matters when *loading* a narrow value into a wider register.

### Shift logical operations

```asm
slli rd, rs1, imm   # rd <- rs1 << imm   (MSBs fall off, 0s added at the right)
srli rd, rs1, imm   # rd <- rs1 >> imm   (LSBs fall off, 0s added at the left)
```

- Max imm value for it to be meaningful? **63** (6 bits in RV64). If more → mod is taken.
- Shift right:
  - `srli` → logical shift: always 0s added.
  - `srai` → arithmetic shift: sign bit copied into the MSBs.
- Register variants:

```asm
sll rd, rs1, rs2
srl rd, rs1, rs2
sra rd, rs1, rs2
```

**Doubt — Q:** If the register value ≥ 64, does it take mod?
> ➕ Added: Yes — in RV64 only the low 6 bits of rs2 are used (shift amount = rs2 mod 64).

- Shifts can be used for multiplication/division (compiler optimisation):
  - × 2ⁿ → left shift by n bits
  - ÷ 2ⁿ → right shift by n bits (`srai` for signed)
- Shifting an odd number right → ÷2 then floor.
- Multiply by 7: ×8 − 1 → left shift by 3, subtract.

> ⚠️ Correction: x × 7 = (x << 3) − **x**, not (x << 3) − 1.

### AND / OR / XOR
- **AND**: used to clear some bits and leave the rest unchanged.
- **OR**: used to set some bits.
- **XOR**: used to invert some bits.
- All have immediate versions too.
- **NOT does not exist** — XOR with all 1s is used: `xori rd, rs1, -1` (−1 = all 1s).

### Conditional operations
- Branch to a labelled instruction if the condition is true:

```asm
beq rs1, rs2, L1   # if (rs1 == rs2) branch to instruction labelled L1
bne rs1, rs2, L1   # if (rs1 != rs2) branch to L1
```

- Now using this, you are capable of dividing exactly divisible numbers (repeated subtraction until zero).

**Example:** f→x19, g→x20, h→x21, i→x22, j→x23

```c
if (i == j) f = g + h;
else        f = g - h;
```

```asm
      bne x22, x23, Else
      add x19, x20, x21
      beq x0, x0, Exit     # always true -> unconditional jump
Else: sub x19, x20, x21
Exit:
```

- No registers are affected by branches. Either memory changes → happens in `sd`; or code behaviour (control flow) changes → happens here.

**Example:** i→x23, k→x24 (notes label i as x22 but the code uses x23)

```c
while (i != k) i += 1;
```

- "i not equal to k" → test the opposite polarity to exit:

```asm
Loop: beq  x23, x24, Exit
      addi x23, x23, 1
      beq  x0, x0, Loop
Exit:
```

**Example:** i→x22, k→x24, A base address @ x25

```c
while (A[i] == k) i += 1;
```

```asm
Loop: ld   x9, 0(x25)
      bne  x9, x24, Exit
      addi x22, x22, 1
      addi x25, x25, 8     # move to next element (8 bytes)
      beq  x0, x0, Loop
Exit:
```

- (Side note: x25 here is the address of A[i], i.e. it assumes i starts at 0. "Test to practice loop.")

### More conditional operations

```asm
blt rs1, rs2, L1   # branch if rs1 <  rs2
bge rs1, rs2, L1   # branch if rs1 >= rs2
```

- Now any kind of division can be done.
- We don't need `ble` or `bgt` (swap operands: `ble a,b` = `bge b,a`; `bgt a,b` = `blt b,a`). ✔ done.

**Example:** a→x22, b→x23

```c
if (a >= b) a += 1;
```

As written in notes:

```asm
      bge  x23, x22, Exit
      addi x22, x22, 1
Exit:
```

> ⚠️ Correction: `bge x23, x22` skips when b ≥ a, which wrongly skips the a == b case. Skip when a < b: `blt x22, x23, Exit`.


---

## Lecture 07 — Signed comparisons, jumps, special registers (11 Aug 2026) · pages 12–13

### Signed conditional operations
- Signed comparison: `blt`, `bge`
- Unsigned comparison: `bltu`, `bgeu`
- Example: x22 = 0xFFFF…F, x23 = 0x0001
  - Signed: x22 = −1, x23 = 1 ⇒ x22 < x23
  - Unsigned: x22 = very large number, x23 = 1 ⇒ x22 > x23

### High-level language features
- Arithmetic expressions: +, −, *, /, %, ^, logical, bitwise
- Conditionals: if-else, ==, !=, <, <=, >, >= ; nested — can be done too.
- Loops: while, do-while, for, …
- Jumps: goto, procedure/function call
  - You can go inside a function, but how do you come back?
  - We could add a label, but to which label do we return?
  - We have to store the return address in a register (which you are not supposed to corrupt inside the function).

### Jump operations

**Jump and link:** `jal rd, L1`
- rd ← PC + 4 (return address)
- Jump to label L1
- If rd is x0, the address is not saved (`jal x0, f`) → use when you don't want to come back.
- Used for function call or goto.
- Don't corrupt this register inside the function.

**Jump and link register:** `jalr rd, n(rs1)`
- rd ← PC + 4
- Jump to address (rs1 + n)
- Used to return from a function.

**Doubt — Q:** Can two locations have the same label?
**A:** No.

**Q:** Can we get the address of a label into a register using these instructions?
**A:** Yes, indirectly: `jal x3, L` with `L:` as the very next instruction puts the address of L (PC+4) into x3 and simply continues.

> ➕ Added: in practice `auipc` (or the `la` pseudo-instruction) is used to get a PC-relative address into a register.

**Q:** Can recursion be implemented using these instructions?
**A:** Yes — use a **stack**: a mechanism to save things on the stack.
- `sd` to save, `ld` to restore. Must restore into the same register.
- Push order x3, x4, x5 → restore order x5, x4, x3 (LIFO).

**Q:** Passing arguments?
**A:** Store them on the stack.

> ➕ Added: RISC-V convention passes the first 8 arguments in registers a0–a7 (x10–x17) and only extra ones on the stack; ra = x1 holds the return address, sp = x2 the stack pointer.

**Q:** Indirect reading of PC?
**A:** `jal x3, L` / `L: …` (PC is not normally accessible to you).

### Byte / halfword / word operations

| Size | Load (sign-ext) | Load unsigned | Store |
|---|---|---|---|
| Byte (8) | `lb` | `lbu` | `sb` |
| Halfword (16) | `lh` | `lhu` | `sh` |
| Word (32) | `lw` | `lwu` | `sw` |

### Special registers
- **x0** is a special register:
  - always returns the value 0
  - also called the **zero register**
  - writes to x0 are ignored
  - can be used to initialise any other register
  - used to compare with 0


---

## Lecture 08 — Large immediates, pseudo-instructions, RISC-V extensions (13 Aug 2026) · pages 14–15

- **Note:** `addi rd, rs1, imm` → imm range **−2048 to +2047** (12-bit signed).

### Handling large immediates
How to load a large (32-bit) immediate, e.g. `0x2011234D`, into a register?

- **Option 1:** store the value in memory and load it using `ld`.
- **Option 2:** use logical operations:

```asm
addi x6, x0, 0x201     # x6 = 0x201
slli x6, x6, 12        # x6 = 0x201000
addi x6, x6, 0x123     # x6 = 0x201123
slli x6, x6, 8         # x6 = 0x20112300
addi x6, x6, 0x4D      # x6 = 0x2011234D
```

> ⚠️ Correction: the notes write the last line as `addi x6, x9, 0x4d`; the source must be x6.

- Too many instructions → a new instruction: **`lui`** (load upper immediate)

```asm
lui rd, immediate
```

- Copies the 20-bit immediate into bits [31:12] of rd
- Extends bit 31 into bits [63:32] (sign-extends — answers the "signed/unsigned?" side question: signed)
- Clears bits [11:0] of rd to 0

Example: load `0x2011234D`

```asm
lui  x19, 0x20112      # x19 = 0x20112000
addi x19, x19, 0x34D   # x19 = 0x2011234D
```

> ➕ Added: if the low 12 bits have bit 11 set (≥ 0x800), `addi` sign-extends them as negative, so the `lui` value must be incremented by 1 to compensate.

### Practice problems (to do later)
- Multiply two numbers by repeated addition
- Divide two numbers by repeated subtraction (quotient & remainder)
- Find the largest among n numbers
- Check if even/odd
- Element-wise addition of two arrays
- GCD of two numbers (check for zeros/negatives)
- Transpose a given 2D array/matrix
- Generate bit patterns using logical operations
- Numerical series:
  - 1, 2, 3, 4, …
  - Fibonacci series
  - Triangular series: F(n) = F(n−1) + n
  - Square series: F(n) = F(n−1) + (2n − 1)

> ⚠️ Correction: notes write the square series as F(n−1) + F(2n−1); it should be F(n−1) + (2n − 1) (since n² − (n−1)² = 2n − 1).

### Pseudo-instructions
Enable simpler assembly, but are not implemented in machine code (the assembler translates them).

| Pseudo | Actual instruction |
|---|---|
| `mv rd, rs` | `addi rd, rs, 0` (or `add rd, rs, x0`) — copies; does **not** remove the contents of rs |
| `neg rd, rs` | `sub rd, x0, rs` |
| `j offset` | `jal x0, offset` |
| `li rd, imm` | load immediate — no 2048 restriction (expands to `lui` + `addi` if needed) |
| `nop` | no operation — used to create delay |

- **Doubt:** then what is li's restriction? → the `lui` restriction (32-bit constants via lui+addi).
- NOP in notes: `add x0, x0, x0`.

> ➕ Added: the canonical encoding of `nop` is `addi x0, x0, 0` (any write to x0 has no effect, so `add x0,x0,x0` also does nothing).

### RISC-V extensions
- **I:** base integer class — already discussed
- **M:** integer multiplication, division, remainder
- **A:** atomic memory operations — e.g. `i++` as one read-modify-write. With 2 CPUs there can be synchronisation issues without this operation.
- **F:** single-precision floating point → 32-bit
- **D:** double-precision floating point → 64-bit (e.g. `fadd`)
- **C:** compressed instructions — only 2 operands; achieve smaller (16-bit) instructions
- **G:** IMAFD together
- e.g. **RV32IMA**

### Computation flow (diagram: stack of boxes, top to bottom)

```
Specification
Algorithm
High-level code
Assembly / machine code
ISA
Processor
Circuit / gates
Devices / transistors
```


---

## Lecture 09 — Representing instructions: R and I formats (14 Aug 2026) · pages 16–17

### Stored program computers
- (Student: "did not listen anything.")

> ➕ Added: stored-program concept — instructions are represented as numbers and stored in memory just like data; the CPU fetches and executes them.

### Representing instructions
- Encoded in binary.
- RISC-V → **32-bit instructions** → uniform-length instructions.
- Split into fields.

### RISC-V R-format instructions

| funct7 [31:25] | rs2 [24:20] | rs1 [19:15] | funct3 [14:12] | rd [11:7] | opcode [6:0] |
|---|---|---|---|---|---|
| 7 bits | 5 bits | 5 bits | 3 bits | 5 bits | 7 bits |

- **opcode:** operation code
- **rd:** destination register
- **rs1:** first source register
- **rs2:** second source register
- **funct3, funct7:** functions of 3 or 7 bits — additional opcode

**Example:** `add x9, x20, x21`

| Field | funct7 | rs2 (x21) | rs1 (x20) | funct3 | rd (x9) | opcode |
|---|---|---|---|---|---|---|
| Decimal | 0 | 21 | 20 | 0 | 9 | 51 |
| Binary | 0000000 | 10101 | 10100 | 000 | 01001 | 0110011 |

- Hex: `0000 0001 0101 1010 0000 0100 1011 0011` = **0x015A04B3** ✔ (verified)
- All instructions that follow this format have 3 registers (2 sources, 1 destination); they differ in opcode, funct3, funct7.
- The opcode helps the CPU decide the format.

### RISC-V I-format instructions
- funct7 + rs2 → replaced by a 12-bit immediate: −2048 to +2047 (−2¹¹ to 2¹¹ − 1).

| immediate [31:20] | rs1 [19:15] | funct3 [14:12] | rd [11:7] | opcode [6:0] |
|---|---|---|---|---|
| 12 bits | 5 bits | 3 bits | 5 bits | 7 bits |

- Used by: load instructions `ld rd, imm(rs1)`; `addi`, etc.
- `slli` ⇒ the immediate field is split:

| funct6 [31:26] | shamt [25:20] | rs1 | funct3 | rd | opcode |
|---|---|---|---|---|---|
| 6 bits | 6 bits | 5 | 3 | 5 | 7 |

  - shamt only up to 64 (0–63).

**Doubt — Q:** `slli rd, rs, imm` — is the immediate signed/unsigned?
**A (notes):** Signed only, but just use positive numbers.

> ➕ Added: more precisely, the shift amount is a 6-bit **unsigned** field (0–63); the upper 6 bits hold funct6 (000000 for `slli`/`srli`, 010000 for `srai`).

**Example 1:** `addi` with immediate −51 (handwritten registers hard to read; the encoded fields give rd = x10, rs1 = x5, i.e. `addi x10, x5, -51`)

- 51 = 0000 0011 0011 → −51 = **1111 1100 1101** (invert + 1) ✔

| imm [31:20] | rs1 [19:15] | funct3 | rd [11:7] | opcode |
|---|---|---|---|---|
| 111111001101 | 00101 | 000 | 01010 | 0010011 |

> ➕ Added: full word = `1111 1100 1101 0010 1000 0101 0001 0011` = **0xFCD28513**.

**Example 2:** `ld x29, 100(x1)`

- rd = x29, rs1 = x1 (00001), imm = 100 = 000001100100
- Field order: imm | rs1 | funct3 | rd | opcode

> ⚠️ Correction: x29 = **11101** (the notes appear to write 11100, which is x28).

> ➕ Added (completion):
>
> | imm [31:20] | rs1 | funct3 | rd | opcode |
> |---|---|---|---|---|
> | 000001100100 | 00001 | 011 | 11101 | 0000011 |
>
> = `0000 0110 0100 0000 1011 1110 1000 0011` = **0x0640BE83**


---

## Lecture 10 — S format, PC, branch addressing, U format (18 Aug 2026) · pages 18–19

- Right now 32 bits are used for a complete instruction. Can you reduce it to 16?
  - C-I: reduce the number of registers
  - C-II: reduce the number of operands
  - Example: RISC-V **compressed instructions**.
- We are done with R-format and I-format.

### RISC-V S-format instructions
- **Note:** the store instruction doesn't come under I-format. Why? It uses **two sources**: `sd x9, 0(x22)` → x9 and x22 are both sources; there is no rd → "S-format".

Diagram comparing the three formats (the immediate takes the rd slot in S):

| Format | [31:25] | [24:20] | [19:15] | [14:12] | [11:7] | [6:0] |
|---|---|---|---|---|---|---|
| R | funct7 (7) | rs2 (5) | rs1 (5) | funct3 (3) | rd (5) | opcode (7) |
| I | immediate[11:0] (12) | ← | rs1 (5) | funct3 (3) | rd (5) | opcode (7) |
| S | imm[11:5] (7) | rs2 (5) | rs1 (5) | funct3 (3) | imm[4:0] (5) | opcode (7) |

- S is specifically for stores: `sd rs2, imm(rs1)`.
- Example: (student: "need to see from someone to write")

> ➕ Added example: `sd x9, 64(x22)` → imm = 64 = 0000010 00000
>
> | imm[11:5] | rs2 (x9) | rs1 (x22) | funct3 | imm[4:0] | opcode |
> |---|---|---|---|---|---|
> | 0000010 | 01001 | 10110 | 011 | 00000 | 0100011 |
>
> = **0x049B3023**

Margin summary of formats (my orange notes):

| Format | Fields (MSB → LSB) |
|---|---|
| R | funct7(7) · rs2(5) · rs1(5) · funct3(3) · rd(5) · opcode(7) |
| I | imm[11:0](12) · rs1(5) · funct3(3) · rd(5) · opcode(7) |
| S | imm[11:5] · rs2 · rs1 · funct3 · imm[4:0] · opcode |
| B | imm[12\|10:5] · rs2 · rs1 · funct3 · imm[4:1\|11] · opcode |
| U | imm[31:12] (20 bits; student wrote [19:0]) · rd[4:0] · opcode[6:0] |

### Program counter (PC)
- Holds the address of the currently executing instruction.
- Basically updated by:
  - **+4** — normal code flow (next instruction)
  - **target location** — jump/branch
- For the compressed extension → +2 bytes.
- Not any random value: a multiple of 4 (more precisely, it is **never an odd number** — multiple of 2 with C).
- It is also a register, but not accessible by us (not one of x0–x31).
- Can be accessed indirectly by:

```asm
    jal x5, L1
L1: ...
```

### Branch addressing
- Branch instructions: opcode, two registers, target address.
- Branches can be forward or backward.
- They are mostly **near** branches → optimisation → **PC-relative addressing**:
  - **Target address = PC + immediate**
  - Advantage: fewer bits needed.
  - Notes: "some complication: code portability decreases; 'relocatable code'".

> ⚠️ Correction: PC-relative addressing makes code **relocatable/position-independent** — it *improves* portability (the code works wherever it's loaded).

**B-format** (13-bit offset ± from PC):

| imm[12] [31] | imm[10:5] [30:25] | rs2 [24:20] | rs1 [19:15] | funct3 [14:12] | imm[4:1] [11:8] | imm[11] [7] | opcode [6:0] |
|---|---|---|---|---|---|---|---|

- imm[12] is the sign bit, placed as the 1st (MSB) bit.
- imm[0] is not stored because it is always 0 (addresses are always even).

**Example:** `beq x28, x?, -96` (second register [illegible]) — 13-bit immediate

- −96 = 1 1111 1010 0000 (13 bits) ✔ (verified)

| imm[12] | imm[11] | imm[10:5] | imm[4:1] | imm[0] (not stored) |
|---|---|---|---|---|
| 1 | 1 | 111101 | 0000 | 0 |

- The −96 calculation is done by the compiler/assembler.
- Range: **−4096 to 4094** (always even; notes write "4095 → 4094").

### U-format: `lui` and `auipc`

| immediate [31:12] | rd [11:7] | opcode [6:0] |
|---|---|---|
| 20 bits | 5 bits | 7 bits |

- `lui x9, imm`: 7-bit opcode + 5-bit rd + 20-bit imm = 32 bits.
- **`auipc`** (add upper immediate to PC): value loaded = PC + (imm << 12).

**Doubt — Q:** `auipc x5, imm` — does x5 store the entire PC + imm, or do only bits [31:12] of x5 store PC + imm?
**A:** x5 stores the **entire** PC + (imm << 12), not just bits [31:12].


---

## Lecture 11 — Jump addressing, decoding (20 Aug 2026) · pages 20–21

### Jump addressing
`jal rd, Label` — inside the computer, the label is just an **offset from PC** (the immediate value).
- opcode: 7 bits, rd: 5 bits, immediate: 20 bits

**J-format:**

| imm[20] [31] | imm[10:1] [30:21] | imm[11] [20] | imm[19:12] [19:12] | rd [11:7] | opcode [6:0] |
|---|---|---|---|---|---|

- `beq x0, x0, L` → unconditional branching.
- `jal x0, L` → gives a larger range to jump (21-bit offset ≈ ±1 MiB vs ±4 KiB for branches). ("OK got it")
- `jalr rd, n(rs1)` → follows **I-format** (like load).
  - It lets us do a very long jump: rs1 is a full 64-bit register (± 12-bit offset) → very large jump.

> ➕ Added: `jal` range is −2²⁰ to 2²⁰ − 2 bytes (±1 MiB); for anywhere in a 32-bit range use `auipc` + `jalr`.

### Core instruction formats (slide)

| Type | 31–25 | 24–20 | 19–15 | 14–12 | 11–7 | 6–0 |
|---|---|---|---|---|---|---|
| R | funct7 | rs2 | rs1 | funct3 | rd | opcode |
| I | imm[11:0] | ← | rs1 | funct3 | rd | opcode |
| S | imm[11:5] | rs2 | rs1 | funct3 | imm[4:0] | opcode |
| B | imm[12\|10:5] | rs2 | rs1 | funct3 | imm[4:1\|11] | opcode |
| U | imm[31:12] | ← | ← | ← | rd | opcode |
| J | imm[20\|10:1\|11\|19:12] | ← | ← | ← | rd | opcode |

### RISC-V addressing summary
- Immediate addressing ✓ (operand in instruction, e.g. `addi`)
- Register addressing ✓ (operands in registers, e.g. `add`)
- Base addressing ✓ (register + offset, e.g. `ld`/`sd`)
- PC-relative addressing ✓ (branches, `jal`, `auipc`)

### Decoding machine code
1. Look at the **opcode** to find the format.
2. Parse the fields.

> ➕ Added — worked decode: `0x015A04B3` → binary `0000000 10101 10100 000 01001 0110011`; opcode 0110011 = R-type; funct3 000 & funct7 0000000 = add; rd = 9, rs1 = 20, rs2 = 21 → `add x9, x20, x21`.

### Overview (diagram: what happens in the CPU)

```
Fetch --> Decode --+--> regs --> [Reg file] --> [ ALU ] --> rd
                   +--> imm value ------------> [ ALU ]
                   +--> [Branch/jump unit]
                   +--> [Mem unit]   (if not an ALU operation)
```

- Page 21: blank (no content).

---

## Lecture 12 — Procedure Calling in RISC-V (21 Aug 2026) · pages 22–27

*Friday, class @ 9 am.*

**Recap so far:**
- We have covered the instruction set ✓
- Decoding / encoding of those instructions ✓

### Procedure Calling in RISC-V

**Q:** What is the difference between a procedure and a function?
**A:**
- A **procedure** is supposed to *do something* — **no return** value.
- A **function** *returns* a value.

**Q:** Why do we need functions?
**A:**
- Reduce redundancy (reuse code).
- Team development (split work into modules).

**Q:** Are `jal` and `jalr` enough for at least a `void f(void)` function?
**A:** **No** — because typically a function is **not allowed to corrupt registers** (the caller's values must survive the call), so we also need a convention for saving/restoring registers (the stack).

### Procedure / Function Calls

- **Transfer of control**, which involves:
  1. Pass arguments
  2. Jump to the function
  3. Pass back the result
  4. Return to the original point
- **Variables:**
  - `auto` — the default
  - `static`
- **Types of procedures:**
  - Leaf
  - Nested
  - Recursive

### Memory Layout

- `.text` → code section
- `.data` → data section, which holds:
  1. **Static data**
  2. **Dynamic data**
  3. **Stack**

Diagram (high address at top):

```
+-------------------+  high address
|      Stack        |   grows ↓
|        ↓          |
|                   |
|        ↑          |
|  Dynamic (heap)   |   grows ↑
+-------------------+
|      .bss         |
|      .data        |   static data
+-------------------+
|   code / .text    |
+-------------------+  low address
```

- **Static data:** scope is **global**; lifetime is **throughout the program**.
- **Dynamic data:** whatever you allocate using `malloc` (or similar) → grows/shrinks via `malloc` and `free`.
- **Stack:** automatic storage (`auto`) → grows or shrinks with function calls/returns.
- To **retain the value of a variable** across calls, use **`static`**.

**Q:** Why does dynamic data grow upward and the stack grow downward?

Diagram: two memory maps side by side. Left: `Stack / Dynamic / Static data / Text / Reserved` as fixed blocks. Right: `Stack ↓ … ↑ Dynamic`, then `Static data / Text / Reserved` — the stack and dynamic regions share one free gap and grow toward each other → **"DOWNWARD GROWING STACK"**.

> ➕ Added: **A:** Placing stack (top, growing down) and heap (above static data, growing up) at opposite ends of one free region lets either use all of the leftover space; neither needs a fixed size limit, and memory runs out only when they actually meet.

### Local Data on the Stack

- The stack (`sp`) is initialised to a **very high address**.
- Create additional space on the stack (by decrementing `sp`) for each procedure.

**Stack frame (activation record)** — three snapshots: before the call, during the call, after the call:

```
 Before call        During call                 After call
                 fp → +--------------------+     fp →
 fp →                 | 1 saved arg regs   |
 sp → ----            | 2 saved ra (x1)    |     sp → ----
                      | 3 saved regs       |
                      | 4 local arrays/    |
                 sp → |   structs          |  ← top of the stack
                      +--------------------+
```

1. Saved **argument registers** (if any) — `x10, x11, …`
2. Saved **return address** — `x1`
3. Saved **registers** (if any)
4. **Local arrays and structs** (if any)

- `fp` → **frame pointer** (points to the start of the current frame; `sp` points to the top of the stack).

- **Caller:** the function inside which another function is called.
- **Callee:** the function being called.

**Q:** What is a garbage collector? *(my side question)*
> ➕ Added: **A:** A runtime component (Java, Python, Go, …) that automatically frees heap memory that is no longer reachable by the program. C has none, so memory from `malloc` must be released with `free`.

*Note to self: read about type qualifiers (`const`, `volatile`) in the reference book.*

### Storage Classes in C

| Class / specifier | Memory allocated in | Initial value | Lifetime |
|---|---|---|---|
| `auto int` | stack | garbage | block |
| `static int` | data | 0 | global (entire program) |
| `register int` | CPU register | garbage | block |
| `extern int` | data | 0 | global |

> ⚠️ Correction: A `static` local has whole-program **lifetime** but only **block scope** (a `static` global has file scope). "Global" in this table means lifetime, not visibility.

### Example Procedure Call

```c
int main() {          // CALLER
    int a, b;
    int res;
    res = mul(a, b);
    return 0;
}

int mul(int x, int y) {   // CALLEE
    int prod = 0;
    while (x) {
        prod += y;
        x--;
    }
    return prod;
}
```

Steps for a procedure call:
1. Put arguments in a **predefined location**: registers **`x10` to `x17`** → up to **8 arguments** (`x10` = 1st, `x17` = 8th). This is a convention, not a hardware rule.
2. Transfer control to the procedure.
3. Acquire storage for the procedure (stack — per function).
4. Perform the task.
5. Place the return value in a predefined location: **`x10`** (reused).
6. Return to the place of the call.

- Argument passing and return values happen **through registers**.

### Procedure Call Instructions

```asm
jal  x1, ProcedureLabel   # calling: x1 (ra) = PC+4, jump to label
jalr x0, 0(x1)            # returning: jump to address in x1, discard link
```


---

## Lecture 13 — Leaf Procedures & Register Usage (25 Aug 2026) · pages 28–29

### Leaf Procedure Example

**C code:**

```c
int64_t leaf_example(int64_t g, int64_t h, int64_t i, int64_t j) {
    int64_t f;
    f = (g + h) - (i + j);
    return f;
}
```

- Arguments `g` to `j` → `x10` to `x13` (convention).
- `x20` → holds `f`; the return value goes in `x10`.
- `x5`, `x6` → temporary registers.
- My note: "These registers may be in use by the caller, so we need to save them first."

**RISC-V assembly:**

```asm
leaf_example:
    addi sp, sp, -24     # 3 x 8 bytes → save 3 registers: x5, x6, x20   (change SP)
    sd   x5, 16(sp)      # store
    sd   x6, 8(sp)
    sd   x20, 0(sp)
    add  x5, x10, x11    # x5 = g + h                                     (operate)
    add  x6, x12, x13    # x6 = i + j
    sub  x20, x5, x6     # f = x5 - x6
    addi x10, x20, 0     # final return value must be in x10              (return value)
    ld   x20, 0(sp)      # load back
    ld   x6, 8(sp)
    ld   x5, 16(sp)
    addi sp, sp, 24      # change SP back
    jalr x0, 0(x1)       # return to caller
```

> ⚠️ Correction: The notes have the third operation as `add x20, x5, x6`. It must be **`sub x20, x5, x6`** because f = (g+h) − (i+j).

> ⚠️ Correction: By the RISC-V convention, `x5`/`x6` are **temporaries (caller-saved)**, so a callee does *not* have to save them. Only `x20` (s4, callee-saved) must be saved. The textbook saves all three just to illustrate. The minimal version saves only `x20`, or uses temporaries for `f` and saves nothing.

**Stack during the call:**

```
 Before          During                     After
 sp → ----      +------------------+       sp → ----
                | contents of x5   |  ← sp+16
                | contents of x6   |  ← sp+8
          sp →  | contents of x20  |  ← sp+0
                +------------------+
```
- *No need to clear* the old values after popping. That memory is now outside the stack and will be overwritten later.

### Side Note: How to Use Assembly in C?
1. Inline assembly: `__asm__( "..." );`
2. A separate assembly file (`.s`), compiled/linked together with the C file.

**Q:** What if the caller wasn't using the registers we planned to save?
**A:** Then saving them was wasted work. This is solved by a **convention** in which the **caller saves the registers it uses** (for some registers) and the callee saves others → caller-saved vs callee-saved registers.

### Caller vs Callee Saving
**Caller-saving:**
- Can be optimised for back-to-back calls.
- But the callee may not modify all the registers (so the saving may be unnecessary).
- Not all registers can be safely saved by the caller (e.g. the return address).

**Callee-saving:**
- But the temporary registers may not all be in use by the caller (so the saving may be unnecessary).

### Register Usage
Some guarantees are given about registers:
- **`x5–x7`, `x28–x31`: temporary registers**
  - *Not* preserved by the callee (procedure).
  - The caller saves them if it needs them after the call. → The callee may use them freely.
- **`x8–x9`, `x18–x27`: saved registers**
  - If the callee uses them, the callee saves and restores them. → The caller can rely on them surviving a call.

**RISC-V register convention table** (pasted from slides):

| Register | ABI Name | Description | Saver |
|---|---|---|---|
| x0 | zero | Hard-wired zero | — |
| x1 | ra | Return address | Caller |
| x2 | sp | Stack pointer | Callee |
| x3 | gp | Global pointer | — |
| x4 | tp | Thread pointer | — |
| x5–7 | t0–2 | Temporaries | Caller |
| x8 | s0/fp | Saved register / frame pointer | Callee |
| x9 | s1 | Saved register | Callee |
| x10–11 | a0–1 | Function arguments / return values | Caller |
| x12–17 | a2–7 | Function arguments | Caller |
| x18–27 | s2–11 | Saved registers | Callee |
| x28–31 | t3–6 | Temporaries | Caller |

Student annotation on the `x12–17` row (struck through and split): "`x12, x13, x14, x15`: up to here the number of register bits can be reduced; `x16–x17` beyond. Attempt to keep the minimal essentials within `x0–x15`; extra registers, if needed, come from `x16–x31`."

> ➕ Added: This is the idea behind the **RV32E / RV64E** (embedded) base ISA, which has only 16 registers `x0–x15`. It also explains why compressed (RVC) instructions mostly use 3-bit fields that reach only the most-used registers `x8–x15` (s0–s1, a0–a5). The ABI is laid out so that the essential registers fall in the low 16.


---

## Lecture 14 — Non-Leaf Procedures (27 Aug 2026) · pages 30–31

**Non-leaf procedure:** the callee itself starts behaving as a caller (it calls another function).

### Example 1

```c
int64_t f1(int64_t a, int64_t b, int64_t c) {
    int64_t x, y;
    x = a - f2(b);
    y = x + c;
    return y;
}

int64_t f2(int64_t m) {
    return m * 8 + 5;
}
```

- `a, b, c` → `x10, x11, x12`; result in `x10`.
- `m` (the argument of `f2`) has to come in `x10`, so we need to save `x10` (an argument register, caller-saved).
- The return address `ra = x1` also has to be saved, because `jal` to `f2` will overwrite it.

```asm
f1:
    addi sp, sp, -32     # save 3 registers (arguments) + 1 register (ra)
    sd   x1, 24(sp)      # store these regs because they are function arguments / ra
    sd   x10, 16(sp)
    sd   x11, 8(sp)
    sd   x12, 0(sp)
    addi x10, x11, 0     # argument of f2 → x10 (b was in x11)
    jal  x1, f2          # call function
    addi x5, x10, 0      # move result to another register (x5 = f2(b))
    ld   x12, 0(sp)      # restore registers
    ld   x11, 8(sp)
    ld   x10, 16(sp)
    ld   x1, 24(sp)
    addi sp, sp, 32      # restore stack pointer
    sub  x6, x10, x5     # x = a - f2(b)      } computation
    add  x7, x6, x12     # y = x + c          }
    addi x10, x7, 0      # return value in x10
    jalr x0, 0(x1)       # return from function

f2:
    slli x6, x10, 3      # x6 = m * 8   (x6 → temporary register)
    addi x10, x6, 5      # return m*8 + 5
    jalr x0, 0(x1)
```

> ➕ Added: Saving `x11` (b) is not actually needed, since `b` is not used after the call. Only `ra`, `a` (x10) and `c` (x12) have to survive. A tighter frame would be 24 bytes. The code is still correct as written.

### Example 2 — Factorial
*"See from slides."* (Slides: Google Drive link in the notes.)

> ➕ Added: the standard recursive factorial (P&H RISC-V):
> ```c
> long long fact(long long n) {
>     if (n < 1) return 1;
>     else return n * fact(n - 1);
> }
> ```
> ```asm
> fact:
>     addi sp, sp, -16
>     sd   x1, 8(sp)        # save return address
>     sd   x10, 0(sp)       # save n
>     addi x5, x10, -1      # x5 = n - 1
>     bge  x5, x0, L1       # if (n - 1) >= 0, i.e. n >= 1, go to L1
>     addi x10, x0, 1       # return 1
>     addi sp, sp, 16       # pop (nothing restored)
>     jalr x0, 0(x1)
> L1: addi x10, x10, -1     # n = n - 1
>     jal  x1, fact         # fact(n - 1)
>     addi x6, x10, 0       # x6 = fact(n - 1)
>     ld   x10, 0(sp)       # restore n
>     ld   x1, 8(sp)        # restore ra
>     addi sp, sp, 16
>     mul  x10, x10, x6     # return n * fact(n - 1)
>     jalr x0, 0(x1)
> ```

### Example 3 — String Copy
*"See slides again."*

> ➕ Added: `strcpy` is a leaf procedure that uses byte loads/stores (P&H):
> ```c
> void strcpy(char x[], char y[]) {
>     size_t i = 0;
>     while ((x[i] = y[i]) != '\0') i += 1;
> }
> ```
> ```asm
> strcpy:                    # x10 = x[], x11 = y[]
>     addi sp, sp, -8
>     sd   x19, 0(sp)        # save s3 (i)
>     add  x19, x0, x0       # i = 0
> L1: add  x5, x19, x11      # &y[i]
>     lbu  x6, 0(x5)         # x6 = y[i]
>     add  x7, x19, x10      # &x[i]
>     sb   x6, 0(x7)         # x[i] = y[i]
>     beq  x6, x0, L2        # stop at '\0'
>     addi x19, x19, 1       # i++
>     jal  x0, L1
> L2: ld   x19, 0(sp)
>     addi sp, sp, 8
>     jalr x0, 0(x1)
> ```
> (Using a temporary instead of `x19` for `i` would avoid the save/restore entirely.)

*Note: revised signed and unsigned numbers.*


---

## Lecture 15 — CPU Execution Time (22 Sep 2026) · page 32

- Only heading written: **"CPU execution time & stuff."** The rest of the page is blank.

> ➕ See completion section.


---

### ➕ Completion (from P&H): L15: CPU time and performance (p.32: "CPU execution time & stuff")

```
Performance = 1 / Execution time
"X is n times faster than Y":  n = Time_Y / Time_X

CPU time = CPU clock cycles × Clock cycle time = CPU clock cycles / Clock rate
CPU clock cycles = Instruction count × CPI
⇒ CPU time = IC × CPI × T = (IC × CPI) / f            ← the "iron law"

With instruction classes:  CPU cycles = Σ (CPI_i × IC_i),   avg CPI = Σ CPI_i × (IC_i / IC)
```

| Factor | Affected by |
|---|---|
| Instruction count | algorithm, language, compiler, ISA |
| CPI | ISA, microarchitecture (pipelining, caches), compiler |
| Clock period | microarchitecture, technology |

**Book example 1:** A runs a program in 10 s at 2 GHz. Design B should take 6 s but needs 1.2× the clock cycles. What clock rate does B need?
- Cycles_A = 10 s × 2 GHz = 20×10⁹
- Cycles_B = 1.2 × 20×10⁹ = 24×10⁹
- f_B = 24×10⁹ / 6 s = **4 GHz**

**Book example 2:** same ISA. A: cycle time 250 ps, CPI 2.0. B: cycle time 500 ps, CPI 1.2.
- Time per instruction: A = 2.0 × 250 = 500 ps; B = 1.2 × 500 = 600 ps
- **A is 1.2× faster**

**Amdahl's law:**

```
T_improved = T_affected / speedup + T_unaffected
```

You can't speed up the whole program by more than 1 / (unaffected fraction).

**Pitfall:** MIPS (million instructions per second) = f / (CPI × 10⁶). It ignores instruction count, so it can't compare different ISAs.

**Power:** Dynamic power ∝ ½ × C × V² × f. This is the "power wall" behind §2's deeper-pipeline discussion.


---

# UNIT 2: Arithmetic for Computers

---

## Lecture 16 — Arithmetic for Computers (01 Sep 2026) · pages 33–34

### Overflow Check
- **Addition:**
  - both numbers (+) → result (−) → overflow occurred
  - both numbers (−) → result (+) → overflow occurred
  - (Operands of different signs can never overflow.)
- **Subtraction:**
  - (+) − (−) → result (−) → overflow
  - (−) − (+) → result (+) → overflow
- **RISC-V does not have an overflow bit / overflow flag.**

> ➕ Added: In RISC-V, overflow is detected in software with branches. Unsigned: `add t0, t1, t2; bltu t0, t1, overflow`. Signed (when one operand's sign is known): `add t0, t1, t2; slti t3, t2, 0; slt t4, t0, t1; bne t3, t4, overflow`.

### Adder Hardware — Ripple Carry Adder

**Half adder** (inputs a, b → outputs sum, carry):
- `S = a XOR b`
- `C = a AND b`

**Full adder** (inputs a, b, c_in → outputs sum, c_out):
- `S = a XOR b XOR c_in`
- `C_out = (a AND b) OR (a AND c_in) OR (b AND c_in)`

### 4-bit Adder
`A = A3 A2 A1 A0`, `B = B3 B2 B1 B0`, `S = S3 S2 S1 S0 = A + B`, `C` → carry.

```
  A3 B3     A2 B2     A1 B1     A0 B0
   ↓ ↓       ↓ ↓       ↓ ↓       ↓ ↓
C ←[FA3]←C2─[FA2]←C1─[FA1]←C0─[HA]
     ↓         ↓         ↓        ↓
    S3        S2        S1       S0
```
≡ a block "4-bit Adder" with inputs `cin, B, A` and outputs `Cout, S`.

> ⚠️ Correction: If the 4-bit block is to have a `cin` input (as the block symbol shows, and as needed for chaining), bit 0 must be a **full adder**, not a half adder. A half adder at bit 0 works only for a standalone adder with no carry-in.

- **16-bit adder:** similarly connect four 4-bit adders.
- **64-bit adder:** similarly connect four 16-bit adders.

### Arithmetic for Multimedia
- **SIMD** → Single Instruction, Multiple Data.
- One 64-bit adder can be used as:
  - two 32-bit adders,
  - four 16-bit adders,
  - eight 8-bit adders,
  - sixteen 4-bit adders,
  
  in **parallel**, by **blocking the carry** between the sub-words.
- Normally we have **wrap-around** behaviour on overflow, but multimedia needs:
  - **saturation**
  - **clipping**
  
  Both are done by hardware (e.g. audio/pixel values clamp at the max/min instead of wrapping).

### Multiplication — Long-Multiplication Approach
- n-bit **+** n-bit → at most **n+1** bits.
- n-bit **×** m-bit → at most **n+m** bits.
- *Try this approach for the old lab assignment.*

Slide photo, "Multiplication approach":
```
 multiplicand      1000
 multiplier      × 1001
                 ------
                   1000
                  0000
                 0000
                1000
 product        1001000      (8 × 9 = 72 ✓)
```
- "Length of product is the sum of operand lengths."
- Hardware (first version): **Multiplicand** register (128 bits, shift left) → **128-bit ALU** → **Product** register (128 bits, write). **Multiplier** register (64 bits, shift right). A **Control test** block checks the multiplier's LSB and drives the write and the shifts.
- Takes **64 steps** (one per multiplier bit). *Analyse this once.*

> ➕ Added: Algorithm per step: if `Multiplier[0] == 1` then `Product += Multiplicand`; then shift Multiplicand left 1 and Multiplier right 1. Repeat 64 times.


---

## Lecture 17 — Multiplication & Division (03 Sep 2026) · pages 35–36

### Optimized Multiplier
Worked example: `1001 × 1011` (9 × 11). The product register initially holds `0000 | 1011` (upper half | multiplier). The boxed bit is the LSB being tested.

```
→ 0000 1011    LSB=1: 0000 + 1001
→ 1001 1011    then right shift
→ 0100 1101    LSB=1: 0100 + 1001, then right shift   (notes show "0100 1011")
→ 0110 1110    LSB=0
  ⋮  continue
```
> ➕ See completion section.

### Booth Multiplier
On-page fragments only: `010 ⇒ 000 + 010`; `0 1 0` with arrows (scanning bit pairs); `110`; `1011011`; partial rows `0000000`, `1000000`, `1100 0`, `1 1`. Margin note: "Write the intuition here."
> ➕ See completion section.

### Fast Multiplier
- Uses multiple adders (side doodle: `101001`).
- Cost / performance trade-off.
- Can be pipelined → several multiplications performed in parallel.

### RISC-V Multiplication
- `mul` instructions (slides p. 18): `mul`, `mulh`, `mulhu`, `mulhsu`.
- *Understand how `mul` works in detail.*

### Division
- "Refer to slides."

### RISC-V Division (M-extension)
- Four instructions (R-format):
  - `div`, `rem` → **signed** divide, remainder *(margin: "what is this?")*
  - `divu`, `remu` → **unsigned** divide, remainder
- **Division overflow:** `−2^(n−1) / −1 → 2^(n−1)`, which cannot be represented in signed n bits.

> ➕ See completion section.


---

### ➕ Completion (from P&H): L17: Multiplication and division (p.35–36, partly blank)

#### Multiplication

**Basic shift-and-add (n-bit):** for each multiplier bit from the LSB:
- if the bit is 1, add the multiplicand to the product;
- shift the multiplicand left and the multiplier right.

This takes n iterations; a 64-bit multiply takes 64 steps.

**Optimized version (on my page):**
- The multiplier starts in the **right half of the product register**.
- Add to the **left half**, then shift the whole product register right.
- Only a 64-bit adder is needed, and one register is saved.

**Booth's algorithm** (signed, two's complement). Look at the pair (Q₀, Q₋₁) each step:

| Q₀ Q₋₁ | Action |
|---|---|
| 0 0 | nothing (middle of a run of 0s) |
| 1 1 | nothing (middle of a run of 1s) |
| 1 0 | A ← A − M (start of a run of 1s) |
| 0 1 | A ← A + M (end of a run of 1s) |

Then **arithmetic** right shift A, Q, Q₋₁.

**Worked example:** 2 × (−3) in 4 bits. M = 0010, −M = 1110, Q = 1101, A = 0000, Q₋₁ = 0.

| Step | Q₀Q₋₁ | Operation | A | Q | Q₋₁ |
|---|---|---|---|---|---|
| init | | | 0000 | 1101 | 0 |
| 1 | 10 | A−M → 1110; shift | 1111 | 0110 | 1 |
| 2 | 01 | A+M → 0001; shift | 0000 | 1011 | 0 |
| 3 | 10 | A−M → 1110; shift | 1111 | 0101 | 1 |
| 4 | 11 | –; shift | 1111 | 1010 | 1 |

Result A:Q = 1111 1010 = **−6** ✓

**Fast multiplier:**
- Use one adder per multiplier bit and arrange them as a **tree**.
- 64 adders in a tree are log₂64 = **6 adder delays** instead of 64, at the cost of much more hardware.
- The tree can also be pipelined.

**RISC-V M extension:**
- `mul`: low 64 bits of the product
- `mulh`: high 64 bits, signed × signed
- `mulhu`: unsigned × unsigned
- `mulhsu`: signed × unsigned

For a full 128-bit product, use `mulh` followed by `mul` with the same operands; hardware can fuse the pair. There is **no overflow trap**; software checks `mulh` if needed.

#### Division ("refer to slides" on my page)

**Restoring division (n-bit, A:Q registers):** repeat n times:
1. Shift A:Q left by 1.
2. A ← A − M.
3. If A < 0: **restore** (A ← A + M) and set Q₀ = 0. Otherwise set Q₀ = 1.

At the end, Q = quotient and A = remainder.

**Example:** 7 ÷ 2 (4-bit). Q = 0111, M = 0010, A = 0000.

| Step | After shift A / Q | A − M | Result A / Q |
|---|---|---|---|
| 1 | 0000 / 111_ | 1110 (<0) restore | 0000 / 1110 |
| 2 | 0001 / 110_ | 1111 (<0) restore | 0001 / 1100 |
| 3 | 0011 / 100_ | 0001 (≥0) | 0001 / 1001 |
| 4 | 0011 / 001_ | 0001 (≥0) | 0001 / 0011 |

**Quotient 0011 = 3, remainder 0001 = 1** ✓

**Signed division:**
- Divide the magnitudes.
- Quotient is negative if the signs differ.
- **The remainder takes the sign of the dividend:** −7 ÷ 2 = −3 remainder −1.

**RISC-V:** `div`, `divu`, `rem`, `remu` (all R-format). **No traps:**
- Divide by zero → quotient = all 1s (−1), remainder = the dividend.
- Overflow (−2⁶³ ÷ −1, my note: "+2⁶³ not representable") → quotient = −2⁶³, remainder = 0.

**Faster dividers:** you can't parallelize like multiplication, because each step depends on the sign of the previous subtraction. SRT division guesses several quotient bits per step using a lookup table.


---

## Lecture 18 — Fixed Point & Floating Point Representation (04 Sep 2026) · pages 37–39

### Fixed Point
- An **integer with an imaginary binary point at a fixed position**.

**Example:** `11011.1010` (point assumed after the 5th bit):

`1×2⁴ + 1×2³ + 0×2² + 1×2¹ + 1×2⁰ + 1×2⁻¹ + 0×2⁻² + 1×2⁻³ + 0×2⁻⁴`
= 16 + 8 + 0 + 2 + 1 + 0.5 + 0 + 0.125 + 0 (student wrote **26.625**)

> ⚠️ Correction: 16 + 8 + 2 + 1 = 27, so the value is **27.625**, not 26.625.

**Same bits as 2's complement:**

`−1×2⁴ + 1×2³ + 0×2² + 1×2¹ + 1×2⁰ + 1×2⁻¹ + 0×2⁻² + 1×2⁻³ + 0×2⁻⁴`
(student wrote = −6 + 0.625 = **−5.375**)

> ⚠️ Correction: The integer part is −16 + 8 + 2 + 1 = **−5**, so the value is −5 + 0.625 = **−4.375**. Check: the bit pattern `110111010` as a 9-bit integer is −70, and −70 / 2⁴ = −4.375 ✓.

- **Notation:** `fixed<w, b>` where `w` = total bits and `b` = fractional bits.
  - Here: `fixed<9, 4>`.
- **Precision** = difference between two consecutive representable numbers = **2⁻ᵇ**.
- Fixed-point values can be operated on just like integers.

### Floating Point
- **FP32** → single precision; **FP64** → double precision.

```
+---+-------------+------------------------+
| S |  exponent   |   fraction (mantissa)  |
+---+-------------+------------------------+
 1     single: 8      single: 23 bits
       double: 11     double: 52 bits
```
- Student wrote: single: `fp<8, 32>`, double: `fp<11, 52>`.

> ⚠️ Correction: To be consistent with the field sizes (exponent bits, fraction bits), single should be **`fp<8, 23>`** (32 bits in total) and double **`fp<11, 52>`** (64 bits in total).

**Value:** `x = (−1)^S × (1 + Fraction) × 2^(Exponent − Bias)`

- `S` → sign bit.
- **Normalised significand:** `1.0 ≤ |significand| < 2.0` (the leading 1 is implicit / hidden).
- **Exponent:** stored with a bias. **127 = bias** for single; values below the bias are negative exponents and values above are positive exponents. (**Bias for double = 1023.**)

**Q:** Why not store the exponent as a signed number instead of using a bias?
> ➕ Added: **A:** With a biased exponent, the most negative exponent is `00…0` and the most positive is `11…1`. Floats with the same sign then order the same way as their bit patterns read as unsigned integers, so comparison and sorting can use simple integer hardware. 2's complement would put negative exponents "above" positive ones.

- Exponents **all zeros** and **all ones** are **reserved**.

**Minimum (smallest normalised) representable number (single):**
- exponent = `0000…01` → 1 − 127 = **−126**
- fraction = `1.000…0`
- `1.000 × 2⁻¹²⁶ ≈ ±1.2 × 10⁻³⁸` ✓

**Highest representable number (single):**
- exponent = `111…10` → 254 − 127 = **+127**
- fraction = `1.111…11`
- `1.11…11 × 2¹²⁷` (student wrote ±3.5 × 10⁻³⁸)

> ⚠️ Correction: The largest value is (2 − 2⁻²³) × 2¹²⁷ ≈ **±3.4 × 10⁺³⁸**. The exponent is positive, not −38.

- Side note: `a * a / a ≠ a` in general for floating point (rounding, overflow).

**Relative precision:**
- Single: 2⁻²³ → 23 × log₁₀2 ≈ 23 × 0.3 ≈ **6–7 decimal digits**.
- Double: 2⁻⁵² → 52 × log₁₀2 ≈ 52 × 0.3 ≈ **15–16 decimal digits**.

### Example — Represent −0.75
Repeated ×2 on the fraction:
- 0.75 × 2 = **1**.50
- 0.50 × 2 = **1**.00

→ 0.75 = (0.11)₂ = **1.1 × 2⁻¹**

- Sign = 1
- Exponent = −1 + 127 = **126** (single); = −1 + 1023 = **1022** (double)
- Mantissa (fraction) = `100…00`

> ➕ Added: Full single-precision encoding: `1 | 01111110 | 1000…0` = **0xBF400000**.

### Denormal Numbers
- Exponent = `000…0`:
  - `x = (−1)^S × (0 + Fraction) × 2^(−bias)` (as written)

> ⚠️ Correction: For denormals the exponent is **1 − bias**, so `x = (−1)^S × (0.Fraction) × 2⁻¹²⁶` for single. It is not 2⁻¹²⁷. This keeps denormals contiguous with the smallest normal number.

- Exponent = **all 1s**, fraction = **all 0s** → ±∞ (if out of range). Student wrote: S = 1 → +∞, S = 0 → −∞.

> ⚠️ Correction: It is the other way round. **S = 0 → +∞**, **S = 1 → −∞**.

- Exponent = **all 1s**, fraction **≠ 0** → **NaN** (Not a Number).


---

## Lecture 19 — Arithmetic on Floating Points (08 Sep 2026) · page 40

- Page reads: **"KK DID NOT WRITE !!!!!"**
- Topics covered in class (not written):
  - FP arithmetic & hardware
  - Examples of conversion between FP and decimal
- "Seen from slides" (Google Drive link in the notes).

> ➕ See completion section.


---

### ➕ Completion (from P&H): L19: Floating-point arithmetic (p.40: "KK did not write")

#### IEEE 754 recap

```
value = (−1)^S × (1 + Fraction) × 2^(Exponent − Bias)
Single (FP32): 1 | 8  | 23   bias 127      Double (FP64): 1 | 11 | 52   bias 1023
```

| Exponent field | Fraction = 0 | Fraction ≠ 0 |
|---|---|---|
| 0 | ±0 | denormal: (−1)^S × 0.F × 2^(1−bias) |
| all 1s | ±∞ | NaN |

#### Converting decimal → FP32: −0.75
- −0.75 = −0.11₂ = −1.1₂ × 2⁻¹
- S = 1. Exponent = −1 + 127 = 126 = 0111 1110. Fraction = 1000…0
- Bits: `1 01111110 10000000000000000000000` = **0xBF400000**

#### Converting FP32 → decimal: `1 10000001 01000…0`
- S = 1. Exponent = 129 − 127 = 2. Significand = 1.01₂ = 1.25
- Value = −1.25 × 2² = **−5.0**

#### FP addition steps (example: 0.5 + (−0.4375), keeping 4 bits of precision)
- 0.5 = 1.000₂ × 2⁻¹ and −0.4375 = −1.110₂ × 2⁻²

1. **Align:** shift the smaller number right until the exponents match. −1.110 × 2⁻² → −0.111 × 2⁻¹
2. **Add the significands:** 1.000 − 0.111 = 0.001 × 2⁻¹
3. **Normalize:** 1.000 × 2⁻⁴, and check for overflow/underflow of the exponent.
4. **Round** to the available bits; renormalize if rounding carries out.
- Result = 2⁻⁴ = **0.0625** ✓

#### FP multiplication steps
1. **Add the exponents** and subtract the bias once: (E₁ + E₂ − bias).
2. **Multiply the significands.**
3. **Normalize** and check for overflow/underflow.
4. **Round.**
5. **Sign** = S₁ XOR S₂.

#### Accurate arithmetic
- Hardware keeps 2 extra bits during operations: **guard** and **round**. A **sticky** bit records whether any 1s were shifted out beyond them.
- This makes "round to nearest even" correct.
- Rounding modes: nearest-even (default), toward 0, toward +∞, toward −∞.
- **Accuracy is measured in ulps** (units in the last place). IEEE guarantees results within ½ ulp.

#### FP is not associative
- Example: (−1.5×10³⁸ + 1.5×10³⁸) + 1.0 = **1.0**
- But −1.5×10³⁸ + (1.5×10³⁸ + 1.0) = **0.0**, because 1.0 is lost when rounding against a huge number.
- So a compiler can't freely reorder FP sums (this matters for parallel reductions).

#### RISC-V F/D instructions (L20 recap)
- **Separate register file:** f0–f31.
- Loads/stores: `flw`/`fsw`, `fld`/`fsd`.
- Arithmetic: `fadd.s/d`, `fsub`, `fmul`, `fdiv`, `fsqrt`, and fused multiply-add `fmadd`.
- Compares: `feq`, `flt`, `fle` write **0/1 to an integer register**; then use `beq`/`bne`.
- Conversions: `fcvt.*` between int and FP; `fmv.x.d` moves raw bits.


---

## Lecture 20 — Floating-Point Instructions in RISC-V (10 Sep 2026) · pages 41–43

### F and D Extensions
- **F** → single-precision (`float`)
- **D** → double-precision (`double`)

- Adds **32 new registers** for floating point: **`f0`–`f31`**, **shared between F and D**.
- Integer registers `x0`–`x31` and FP registers `f0`–`f31` are separate. **The opcode determines** whether `f` or `x` registers are used.
- `f0`–`f31` are **64-bit registers**:
  - Supporting **F** → 32-bit values (upper 32 bits are all 1s)
  - Supporting **D** → 64-bit values → normal use

**Register file by configuration (RV64):**

| Extensions | Integer regs | FP regs |
|---|---|---|
| neither F nor D | `x0–x31`, 64 bits | — |
| **F** only | `x0–x31`, 64 bits | `f0–f31`, 32 bits |
| **D** (no F) | `x0–x31`, 64 bits | `f0–f31`, 64 bits, all used |
| **F + D** | `x0–x31`, 64 bits | `f0–f31`, 64 bits: D uses all 64 normally; F values stored as `1111…1 | 32-bit float` |

> ⚠️ Correction: The notes write "x0 to x32" and "f0 to f32". It should be **x0–x31** and **f0–f31** (32 registers each).
> ➕ Added: Filling the upper 32 bits with 1s is called **NaN-boxing**. A single-precision value inside a 64-bit register looks like a double NaN, so it cannot be mistaken for a valid double. (In practice D requires F, so "D without F" does not occur.)

### New Instructions
- **Arithmetic**
- **Data transfer** → **Q:** Why different ones? Because the result must be stored either in an `x` register or an `f` register.
- **FP instructions operate only on FP registers.**

```asm
flw f5, 0(x2)      # load word into f5; base address still comes from integer reg x2
```
- The machine word `| 5 | 2 | … |`: the register fields are interpreted as `f`-register or `x`-register numbers **because the opcode was `flw`**.

```asm
fadd.s f2, f4, f6
```
- *"You will never have an overflow"* (no trap: FP overflow gives ±∞ per IEEE 754).
- Fahrenheit-to-Celsius example (from slides).

Side box: "Assume these floats are stored in memory. `faddi` is meaningless. How would you store an immediate in that format?"
> ➕ Added: That is why RISC-V has **no FP immediate instructions**. FP constants are kept in memory and loaded with `flw`/`fld` (e.g. via `gp`), or built in an `x` register and moved with `fmv.w.x`.

> ➕ Added: the Fahrenheit→Celsius example (P&H):
> ```c
> float f2c(float fahr) { return ((5.0f/9.0f) * (fahr - 32.0f)); }
> ```
> ```asm
> f2c:                     # fahr in f10, result in f10
>     flw    f0, const5(x3)    # f0 = 5.0
>     flw    f1, const9(x3)    # f1 = 9.0
>     fdiv.s f0, f0, f1        # f0 = 5.0/9.0
>     flw    f1, const32(x3)   # f1 = 32.0
>     fsub.s f10, f10, f1      # fahr - 32.0
>     fmul.s f10, f0, f10      # (5/9) * (fahr - 32)
>     jalr   x0, 0(x1)
> ```

**Q:** What about `beq a, b, <address>` for FP registers?
> ➕ Added: **A:** There is **no FP branch**. Compare with `feq.s`/`flt.s`/`fle.s`, which write 0/1 into an **integer** register, then branch with `beq`/`bne` on that `x` register.

### Instructions
- Arithmetic: `fadd.s`, `fsub.s`, `fmul.s`, `fdiv.s`, `fsqrt.s`, plus the `.d` versions of each: `fadd.d`, `fsub.d`, `fmul.d`, `fdiv.d`, `fsqrt.d`.
- Comparison: `feq.s`, `flt.s`, `fle.s` (and `.d`)
  - e.g. `feq.s x?, f?, f?`: the **destination is an `x` register**, the **sources are `f` registers**.
  - It writes **0 or 1** → use this in a further branch.

### Other Instructions
**① Conversion instructions** (one `x` register and one `f` register, except (e)):
- (a) `fcvt.s.w` / `fcvt.w.s` — int32 (`w`, 32-bit) ↔ single
- (b) `fcvt.s.l` / `fcvt.l.s` — int64 (`l`, 64-bit) ↔ single
- (c) `fcvt.d.w` / `fcvt.w.d` — int32 ↔ double
- (d) `fcvt.d.l` / `fcvt.l.d` — int64 ↔ double
- (e) `fcvt.d.s` / `fcvt.s.d` — single ↔ double (both `f` registers)

(Naming: `fcvt.<dest>.<src>`.)

Example:
```c
float a;
double b;
c = a + b;
```
- If `c` is `float` → convert `double b` → `float` (`fcvt.s.d`).
- If `c` is `double` → convert `float a` → `double` (`fcvt.d.s`).

> ➕ Added: In C, `a + b` is always computed in **double** (`a` is promoted). If `c` is `float`, the double *result* is then narrowed with `fcvt.s.d`. Converting `b` to float first is what the notes say, but C does not do it that way.

**② Move between `x` and `f` registers** (raw bit copy, no conversion):
- `fmv.x.w`, `fmv.w.x` (single, 32-bit)
- `fmv.x.d`, `fmv.d.x` (double, 64-bit)
- My question: "Should it be `s` instead of `w`?"
> ➕ Added: **A:** `w` is correct in the current spec. The instruction copies a raw 32-bit **word** of bits; it does not interpret it as a single. Older spec versions called these `fmv.x.s` / `fmv.s.x`, which were renamed to stress that no conversion happens.


---

## Lecture 21 — Other FP Standards, Conclusion & Custom Multiplication (11 Sep 2026) · pages 44–46

### Other and Recent Standards
**Q (margin):** Find the precision & range of all these.

1. **FP128** — 128-bit floating point numbers, *quadruple-precision*
   - Sign: 1 bit
   - Exponent: 15 bits
   - Mantissa: 112 bits
2. **FP16** — 16-bit floating point numbers, *half-precision*
   - Sign: 1 bit
   - Exponent: 5 bits
   - Mantissa: 10 bits
   - Problems with FP16 vs FP32:
     - Range is significantly smaller.
     - Degradation in accuracy.
     - (ML) doesn't care too much about precision ⇓
3. **BFloat16** — 16-bit **Brain Floating Point**, by Google
   - Used in Google's Tensor Processing Unit (TPU).
   - Sign: 1 bit
   - Exponent: 8 bits
   - Mantissa: 7 bits
   - Precision comes down, although the range is the same as FP32.
4. **Posit Number System**

> ➕ See completion section.

### Posit Number System
Number-line diagram: −∞ … 0 … +∞, with tick marks very dense near 0 and spreading out farther away (tapered precision).
- We also don't want a direct jump to ∞ from the largest number.
- `1.M × 2^E` → split as `1.M × 2^E1 × 2^E2`:
  - changing `E2` → larger jumps
  - changing `E1` → slightly smaller jumps
  - The `2^E2` part is called the **"regime"**.
- "POSIT NUMBER SYSTEM" *(open to explore)*.

### Conclusion
- Bits have no inherent meaning.
- Computer representations of numbers have:
  - finite range and precision,
  - which we need to account for in programs.
- ISAs support arithmetic over:
  - signed & unsigned integers,
  - SIMD and saturating operations,
  - FP approximations to reals.

*(Rough work in the margin: `a₁ + b₁x + k₁x²`, `a₂ + b₂x`.)*

### Projects — Custom Instructions
- Example: **MAC** (multiply-accumulate) instruction
  ```asm
  mac res, ai, bi      # res = res + ai * bi
  ```
- LLMs are dominated by MAC operations (the notes say "quantised by").

### Problem
- Multiplication consumes time, power and resources.

### "Approximate Multipliers"
- Accuracy doesn't matter much in ML applications.
  - Approximate the operand to the nearest power-of-2 number.
  - **Mitchell's algorithm** (logarithmic multiplication).
- Example: `322 × 121` → `log(322 × 121) = log 322 + log 121` → computing logs is complex.
- For a binary number like `00100100`: `log₂(·) ≈ position of the 1st set bit` → the **leading 1 bit**.

### "ROBA Algorithm" (Rounding-Based Approximate multiplier)
`A × B = (Ar − A)(Br − B) + Ar×B + A×Br − Ar×Br`
- `Ar`, `Br` → nearest powers of two.
- Remove `(Ar − A)(Br − B)` as part of the approximation.
- The remaining terms can be done by **shifting** (multiplying by a power of 2 = shift).
- Add one instruction: **"LOD"** (leading-one detector)
  ```asm
  LOD rd, rs
  ```

> ➕ See completion section.

---

### ➕ Completion (from P&H): L21: The red question "find the precision and range of all these", plus custom multipliers worked out

#### Formats

| Format | S / E / M bits | Bias | Max normal | Min normal | Decimal digits ≈ |
|---|---|---|---|---|---|
| FP16 (half) | 1 / 5 / 10 | 15 | (2 − 2⁻¹⁰) × 2¹⁵ = **65504** | 2⁻¹⁴ ≈ 6.1×10⁻⁵ | ~3.3 |
| **BF16** | 1 / 8 / **7** | 127 | ≈ **3.4×10³⁸** (same as FP32) | 2⁻¹²⁶ ≈ 1.2×10⁻³⁸ | ~2.4 |
| FP32 (single) | 1 / 8 / 23 | 127 | ≈ 3.4×10³⁸ | ≈ 1.2×10⁻³⁸ | ~7.2 |
| FP64 (double) | 1 / 11 / 52 | 1023 | ≈ 1.8×10³⁰⁸ | ≈ 2.2×10⁻³⁰⁸ | ~15.9 |
| FP128 (quad) | 1 / 15 / 112 | 16383 | ≈ 1.19×10⁴⁹³² | ≈ 3.4×10⁻⁴⁹³² | ~34 |

- **Range** is set by the exponent bits; **precision** by the mantissa bits.
- Decimal digits ≈ log₁₀(2^(M+1)).
- **BF16 is 1/8/7.** If my notes say anything other than 7 mantissa bits, correct it.
- **BF16 = FP32 with the low 16 bits cut off:** the same range, much less precision. Converting is trivial, which is why it's used in TPUs.

#### Mitchell's algorithm (approximate multiplication via logs)
- Write A = 2^k × (1 + m), with 0 ≤ m < 1. Then log₂A ≈ **k + m**, using the approximation log₂(1+m) ≈ m.
- **k comes from the LOD** (leading-one detector: the position of the first 1 bit). m is just the bits after the leading 1, so no table is needed.
- Add the two approximate logs, then take the antilog the same way (2^(k+m) ≈ 2^k × (1+m)).

**Example 322 × 121 (from my notes):**
- 322 = 2⁸ × 1.2578 → log ≈ 8.2578
- 121 = 2⁶ × 1.8906 → log ≈ 6.8906
- Sum = 15.1484 → 2¹⁵ × 1.1484 ≈ **37,631**
- Exact = 38,962 → **−3.4% error**
- Mitchell always **underestimates**; its worst case is about −11%.

#### ROBA (Rounding-Based Approximate) multiplier

```
A×B = Ar×B + A×Br − Ar×Br + (A−Ar)(B−Br)
```

Ar and Br are A and B rounded to the **nearest power of 2**. Drop the last term:

```
A×B ≈ Ar×B + A×Br − Ar×Br
```

Every term is a multiplication by a power of 2, so each is **just a shift**, and the result needs only adders.

**Example 322 × 121:**
- Ar = 256, Br = 128
- 256×121 + 322×128 − 256×128 = 30,976 + 41,216 − 32,768 = **39,424**
- Exact = 38,962 → **+1.2% error**
- The dropped term is (322−256)(121−128) = −462 ✓

The **custom instruction** idea from my notes: add an `LOD rd, rs` instruction (and/or a MAC instruction, `mac rd, rs1, rs2`: rd += rs1 × rs2) so these approximate multipliers run fast. This works because **ML is error-tolerant**, so small approximation errors don't hurt.


*End of completion notes. Corrections are marked with ⚠️. One remaining gap: the 11/13 lecture, which I didn't attend and can't reconstruct from these files.*


---

# UNIT 3: Hardware & Processor Design, Pipelining

---

## Lecture 22 — Intro to Processor Implementation, Logic Design Basics, Combinational Elements (15 Sep 2026) · pages 47–50

### Generating and Executing a Program

Levels of abstraction, from software (top) down to hardware (bottom):

```
 SOFTWARE   Specification
    |          ↓
    |       Algorithm
    |          ↓
    |       High-level code
    |          ↓
    |       Assembly / machine code
    |          ↓
 ---------- ISA ------------------  e.g. x86, ARM, (RISC-V)
    |          ↓
 HARDWARE   Processor ............  e.g. i3, i5, i7
    |          ↓
    |       Circuits & gates
    |          ↓
    |       Devices / transistors
```

### Introduction

- **CPU performance factors**
  - **Instruction count** — determined by the ISA and the compiler.
  - **CPI and cycle time** — determined by the CPU hardware.
- **Two implementations of RISC-V** (processor implementation; side note: "400 MHz board"):
  1. Simplified implementation (single-cycle)
  2. Pipelined version
- **Instructions supported by the simplified implementation:**
  - Memory: `ld`, `sd`
  - Arithmetic/logical: `add`, `sub`, `and`, `or`
  - Comparison / control transfer: `beq`
- Goal: design a CPU that supports all these instructions → then pipelining / caching on top of it.

### Logic Design Basics

```
            ┌─→ wire : single bit
  [RF] ═══/═══→ [ALU]
            └─→ bus  : group of bits (e.g. 64-bit bus)
```

- **Wire** = carries a single bit. **Bus** = a group of wires carrying a multi-bit value (drawn with a slash, e.g. 64-bit bus between register file and ALU).

**Digital blocks:**

| Type | What it does | Example |
|---|---|---|
| **Combinational** | Operates on data; output is a function of the *current* inputs only | `c = a + b` |
| **Sequential** (a.k.a. **state**) | Stores information | `c = a + c` (new `c` depends on old `c`) → different output every time it's done |

### Combinational Elements

- **Gates:** AND, OR, NOT (the basic gates), XOR, XNOR, NAND, NOR, etc.
- **Decoders, multiplexers**, etc.
- **Adders, comparators**, etc.
- Drew the gate symbols for AND, OR, NOT, and XOR (XOR = OR symbol with an extra curved line at the input).

### Decoder — "pattern matching"

- **n inputs → 2ⁿ outputs**; exactly one output is 1 (the one whose index matches the input pattern).
- 2×4 decoder: inputs I₁, I₀ → outputs Q₃ Q₂ Q₁ Q₀.

| I₁ I₀ | Q₃ Q₂ Q₁ Q₀ |
|---|---|
| 0 0 | 0 0 0 1 |
| 0 1 | 0 0 1 0 |
| 1 0 | 0 1 0 0 |
| 1 1 | 1 0 0 0 |

Gate-level implementation (drawn as four 2-input AND gates fed by I₁, I₀ and their inverses):

```
Q0 = I1' · I0'
Q1 = I1' · I0
Q2 = I1  · I0'
Q3 = I1  · I0
```

**Example:** glow an LED according to switches — switches S₁, S₂ → 2×4 decoder → LED0, LED1, LED2, LED3 (exactly one LED glows for each switch combination).

**Q:** Where can a decoder be used in a CPU implementation?
**A:** To find (identify) the instruction — e.g. the 7-bit opcode goes into a decoder → 2⁷ = 128 outputs, one per instruction type.

> ⚠️ Correction: the notes say "128 **bit** o/p" — it's 128 separate one-bit output lines (one-hot), not a 128-bit value.

### Multiplexer (Mux) — "selector"

- Select line S chooses which input is passed to output Q.

| S | Q |
|---|---|
| 0 | I₀ |
| 1 | I₁ |

Gate-level 2:1 mux (as drawn): S ANDed with I₁, NOT S ANDed with I₀, both into an OR gate:

```
Q = S·I1 + S'·I0
```

- **Symbol:** a trapezoid, or a box labelled "mux".
- **Note:** inputs of a mux need not be 1 bit. E.g. a 64-bit 2:1 mux with a 1-bit select:
  - when S = 0: `Q[63:0] = I0[63:0]`
  - when S = 1: `Q[63:0] = I1[63:0]`

> ⚠️ Correction: in my mux symbol sketch, I₀ is drawn on the input labelled "1" and I₁ on the input labelled "0" — it should be I₀ on the "0" input and I₁ on the "1" input, matching the truth table.

**Q:** Where are multiplexers used?
**A:**
1. **Registers** — selecting one of the 32 64-bit register outputs, using a 5-bit select line (register number).
2. **Addition** — choosing whether the second operand is an immediate or a register value (continued in Lecture 23).


---

## Lecture 23 — ALU, Sequential Elements, Register File, Memory (17 Sep 2026) · pages 51–56

### Another place where a mux is used

```
  rs1 ─────────────────→ A ┐
                            [ + ] ──→ sum
  rs2 ──→┌───┐
         │MUX│──────────→ B ┘
  immed →└───┘
           ↑
        I-type?   (select line)
```

- Second adder input = `rs2` (R-type) or the immediate (I-type) — chosen by a mux whose select is "is it I-type?".

### ALU: Arithmetic & Logic Unit

- **Data inputs:** two inputs A and B, each of register width.
- **Output Q:** generates a result of the same bit width.
- **Control input F** ("func? — what to do?") — selects the operation.
- **Number of control bits:** need enough bits to encode all operations.

> ⚠️ Correction: the notes write `#bits(F) ≤ log₂(no. of ops)`. It should be **#bits(F) ≥ ⌈log₂(number of operations)⌉** (e.g. 4 ops need at least 2 bits; RISC-V's P&H ALU uses a 4-bit ALU control).

- **ALUs also have flags** — give you an extra hint about the result.
  - e.g. result = 0 → **Z (zero)** flag
  - Other examples: **N** → negative, **V** → overflow

### Sequential Elements

**Q:** What functionality cannot be achieved by combinational elements?
**A:** Storage.

**Flip-flop** — stores 1 bit of data in a circuit.
- Uses a clock.
- Edge-triggered (rising edge, shown by the ▷ on the clk input).
- Diagram: D-flip-flop box with inputs D and clk, output Q; timing diagram showing clk, D changing at arbitrary times, and Q only changing (taking D's value) at rising clock edges.

### Register

- **Register** — multiple flip-flops sharing the same clock.
- **FF/register with write control:**
  - Only updates on the clock edge when the write-control input is 1.
  - Used when the stored value is required later.
- **Application of registers:** 32 such registers needed, to store 64 bits each — for x0–x31 storage.

### Register File

- **Register file** = array of registers.
- Single-port version (inputs Addr, D, write, clk; output Q) accesses one register at a time.

**Q:** How to handle reading 2 inputs at a time (e.g. `rs1` and `rs2`)?
**A:** A single port is not sufficient → use a **multi-ported** register file.

- First drawing: 3 full ports, each with (Addr, D, write) → outputs Q0, Q1, Q2.
- Refined drawing (what RISC-V needs): 2 read ports + 1 write port:
  - Port 0: Addr0 = `rs1`, write = 0 → Q0 (read data 1)
  - Port 1: Addr1 = `rs2`, write = 0 → Q1 (read data 2)
  - Port 2: Addr2 = `rd`, D2 = write data, write enabled → writes
- **Outputs → multiplexed** (a 32:1 mux on each read port, select = 5-bit register number).
- **Write → decoder:** Addr2 (5 bits) goes into a 5→32 decoder; its 32 outputs are the register-enable signals, so only the selected register is written (on the clock edge, when RegWrite = 1).

(Student note: "Why are we doing this? didn't listen in class.")

> ➕ Added: Why: an R-type instruction like `add rd, rs1, rs2` must read two registers and write one in the same cycle, so the register file needs 2 read ports + 1 write port. Reads are combinational (mux); writes are clocked (decoder + write enable).

### Memory

**Q:** What is the difference between a register file and memory?
**A:** We don't build memory using flip-flops (too expensive for large sizes; memory uses SRAM/DRAM arrays). Memory is much larger and slower, and is accessed by address rather than register number.

- Memory interface: inputs **Addr**, **write data**, **write-en**, **clk** → output **read data**.
- "Memory cannot be multi-ported."

> ⚠️ Correction: memory *can* be multi-ported (e.g. dual-port SRAM), but extra ports are costly, so main memory is typically single-ported — one access per cycle. That's why the single-cycle datapath uses separate instruction and data memories.

### Lab 4 notes

- `x3 → 0x10000000` (built with `lui`):
  - `lui` loads a 20-bit immediate into bits [31:12]. Immediate bits written out: `0001 0000 0000 0000 0000` (bit 19 … bit 0), i.e. `lui x3, 0x10000` → x3 = 0x1000_0000. ("8 × 4 = 32 bit" — 8 hex digits × 4 bits.)

```asm
lui x3, 0x10000      # x3 = 0x10000000
```

**RISC-V register conventions:**

| Registers | Use |
|---|---|
| x1 (ra) | Stores return address |
| x2 (sp) | Stack pointer |
| x3, x4 | "anything" |
| x10–x17 (a0–a7) | Function arguments; x10 (and x11) also hold return values |
| x5–x7, x10–x17, x28–x31 | **Caller-saved** |
| x8–x9, x18–x27 | **Callee-saved** |

> ⚠️ Correction: x3 = **gp** (global pointer) and x4 = **tp** (thread pointer) — they are reserved, not free for "anything" in standard code (though in a bare-metal lab you may use them as scratch).


---

## Lecture 24 — Clocking, CPU Overview, Datapath & Control Basics (18 Sep 2026) · pages 57–59

### Clocking Methodology

(Student took a photo of the slide.)

```
   ┌─────────┐      ┌───────────────┐
┌─→│  State  │ ───→ │ Combinational │ ──┐
│  │ element │      │     logic     │   │
│  └─────────┘      └───────────────┘   │
└───────────────────────────────────────┘
```

- e.g. Program Counter (PC → +4 adder → back into PC).
- The CPU avoids **combinational loops** — every loop must pass through a state element (clocked), so values update once per clock edge.

> ➕ Added: Edge-triggered clocking: combinational logic computes between edges; state elements are updated on the clock edge. Clock period must be ≥ longest combinational delay (critical path).

### Instruction Execution

1. **PC → instruction memory: Fetch** the instruction.
2. **Decode** the instruction to identify the operation; register numbers → register file → **read** registers.
3. **Execute**, depending on instruction class:
   - Use the ALU to calculate:
     - arithmetic result
     - memory address for load/store
     - branch comparison
   - Access data memory for load/store.
4. **Write** the result to the destination register.
5. **PC ← PC + 4** (or branch target address).

Side notes:
- *Fetch* = reading an **instruction** from memory; *load* = reading **data** from memory.
- This is how a CPU works: do PC → PC + 4, loop.

### CPU Overview (diagram)

Description of the drawn overview datapath:
- **PC** → **Instruction memory** (Addr → Instruction).
- Adder computing **PC + 4** (circled: "PC → PC + 4").
- Second adder computing the **branch/jump** target (PC + immediate) (circled: "Branch/Jump"); both feed back to the PC.
- Instruction fields → **Registers** (Reg#, Reg#, Reg# for rs1, rs2, rd; Data = write data).
- Register outputs → **ALU**; a **mux for immediate** in front of the ALU's second input (circled) chooses register vs immediate.
- ALU output → **Data memory** Address; register data → data memory Data (for stores).
- Memory read data → back to register write data (circled: "ld/load").

### Datapath vs Control

- **Datapath:** tells you the different paths along which data can move.
- **Control:** which path is to be chosen?
  - Multiplexer → select lines
  - ALU → function (operation)
  - Register file → write (RegWrite)
  - Memory → read / write (MemRead / MemWrite)

### Datapath and Control — instructions covered

- Arithmetic/logical: `add`, `sub`, `and`, `or`
- Memory reference: `ld`, `sd`
- Control transfer: `beq`

**Instruction fetch** diagram: PC → IM (Read Addr → Instr); an Add unit adds 4 to PC and feeds back into PC.

Side note — **Q:** Which instructions read the PC? **A:** `auipc` or `jal` (also branches use it for the target).

### R-format Instructions (e.g. `add`)

```
           ┌────────────────────┐        ALU operation
 rs1 ───→  │ Read reg 1  Read   │──────→ ┌─────┐
 rs2 ───→  │ Read reg 2  data 1 │        │ ALU │──┐
 rd  ───→  │ Write reg   Read   │──────→ └─────┘  │
     ┌──→  │ Write data  data 2 │                 │
     │     └────────────────────┘                 │
     │              ↑ RegWrite                    │
     └────────────────────────────────────────────┘
```

- Read two registers, ALU performs the operation (set by ALU operation), result written back to `rd` (RegWrite = 1).

### Load/Store Instructions

```asm
ld x5, 0(x3)     # x5 = Mem[x3 + 0]
sd x5, 0(x3)     # Mem[x3 + 0] = x5
```

- Read register operands.
- Calculate address using the 12-bit offset: `rs1 + offset` — rs1 is 64-bit, offset is 12-bit ??
  → the 12-bit offset is **sign-extended** to a 64-bit offset.
- **Imm Gen** unit: 32-bit instruction in → 64-bit sign-extended immediate out.
- **Data memory** unit: inputs Address, Write data; output Read data; controls **MemWrite** and **MemRead**.
- Load: read memory, write result into register. Store: write register value into memory.

"See slides — branching also covered."

> ➕ Added: **Branch (`beq rs1, rs2, offset`)**: read rs1, rs2; ALU subtracts them and the **Zero** flag tells if they're equal. Target = PC + (sign-extended imm, shifted left 1) — the immediate is in half-word units, hence "shift left 1". If Zero = 1 (and Branch = 1), PC ← target; else PC ← PC + 4.


---

## Lecture 25 — Full Datapath & Control Path (22 Sep 2026) · pages 60–61

### Full Datapath — the steps

1. **Fetch:** instruction read.
2. **Decode:**
   - opcode
   - register number → register value
   - imm/offset
3. **Execute:** invoke the ALU for
   - arithmetic
   - logical operation
   - generate address for load/store
   - check condition (branch)
4. **Memory access:** data memory (not instruction memory) — `ld`/`sd`.
5. **Write back:** writes the generated value to a register — either the output of the ALU or the output of memory.

### Full Datapath slide (photo, annotated)

| Part of slide | My annotation |
|---|---|
| PC + Read address + Instruction memory | Fetch |
| Top-left Add with 4 | Update PC |
| Shift left 1 + Add (Sum) + PCSrc mux | Branch and jump instructions |
| Registers (read reg 1/2, write reg, write data) | Decode |
| Mux before ALU (ALUSrc) | Choosing I or R format |
| Imm Gen (32 → 64) | For branch (address) or immediate value |
| ALU (Zero, ALU result) | Get address of memory |
| Data memory | Accessing data memory |
| Mux after data memory | "to store the return address" |

> ⚠️ Correction: the mux after data memory in this slide is the **MemtoReg** mux — it chooses whether the **ALU result** or the **memory read data** is written back to the register file. (Storing a return address for `jal` needs an extra PC+4 input, not shown in this basic datapath.)

### Control Path

Side list of the 7 instructions: `add`, `sub`, `or`, `and`, `ld`, `sd`, `beq`.

- **RegWrite:** 1 → all the rest (R-format, `ld`); 0 → branch, store.
- **MemRead:** only for `ld`.
- **MemWrite:** only for `sd`.
- **ALUSrc:** 1 → immediate (I-type, `sd`); 0 → register.
- **MemtoReg:** 1 → memory data goes to register (only `ld`); 0 → address/ALU result goes to register.
- **ALU operation:**
  - `ld` → add
  - `sd` → add
  - `beq` → sub
  - R-type → add / sub / or / and (based on funct fields)
- **PCSrc:** "KK will tell."

> ➕ Added: **PCSrc = Branch AND Zero.** PCSrc = 0 → PC ← PC + 4; PCSrc = 1 → PC ← branch target (PC + imm·2).

My question: **Q:** How to compare to 0?
> ➕ Added: **A:** For `beq`, the ALU computes rs1 − rs2; the ALU's **Zero** output is 1 when the result is 0 (i.e. rs1 == rs2). Zero is essentially a NOR of all result bits.

> ➕ Added: Summary of main control signals (P&H):
>
> | Instr | ALUSrc | MemtoReg | RegWrite | MemRead | MemWrite | Branch | ALUOp |
> |---|---|---|---|---|---|---|---|
> | R-type | 0 | 0 | 1 | 0 | 0 | 0 | 10 |
> | `ld` | 1 | 1 | 1 | 1 | 0 | 0 | 00 |
> | `sd` | 1 | X | 0 | 0 | 1 | 0 | 00 |
> | `beq` | 0 | X | 0 | 0 | 0 | 1 | 01 |


---

## Lecture 26 — Multicycle Datapath & Pipelining Intro (29 Sep 2026) · pages 62–65

### Multicycle Datapath

- Theme: in which instruction type, which paths/modules are active/inactive.
- Slide photo (P&H Chapter 4, slide 57): the full datapath divided by vertical red lines into 5 parts — **Fetch | Decode + Reg read | Execute | Memory | Reg write**.
- We see 5 parts here: we can either do everything together in a single step (single-cycle), **or** break it into 5 steps.
- Background: not every instruction needs all 5 steps → save time.

| Instruction | Steps needed |
|---|---|
| R-type | IF, ID, EX, WB (4) |
| Branch | IF, ID, EX (3) |
| `ld` | IF, ID, EX, MEM, WB (5) |
| `sd` | IF, ID, EX, MEM (4) |

**The steps:**
1. **IF** — instruction fetch
2. **ID** — instruction decode (+ register read)
3. **EX** — execute
4. **MEM** — memory access
5. **WB** — write back

- So there seems to be an **advantage** (each instruction takes only the steps it needs).
- **Drawback?** Complexity — need to add clocking / sequentiality → control gets more complex (control becomes a state machine).

### Latency vs Throughput

- **Latency** = "How long does one thing take?" (time for each instruction)
- **Throughput** = "How many things can we do per second?"

### Pipeline Performance

Stage delays: IF 200 ps, ID 100 ps, EX 200 ps, MEM 200 ps, WB 100 ps.

| Instr | IF | ID | EX | MEM | WB | Total |
|---|---|---|---|---|---|---|
| R-type | 200 | 100 | 200 | — | 100 | 600 ps |
| Branch | 200 | 100 | 200 | — | — | 500 ps |
| `ld` | 200 | 100 | 200 | 200 | 100 | 800 ps |
| `sd` | 200 | 100 | 200 | 200 | — | 700 ps |

- Stages here are **not balanced**.
- Time between instructions (pipelined) vs. time between instructions (non-pipelined) / #stages — equality comes with stages being balanced.

> ⚠️ Correction: the relation is **Time between instr.(pipelined) ≥ Time between instr.(non-pipelined) / #stages** (the notes show "≤"). With unbalanced stages the pipelined clock is set by the slowest stage, so the speedup is less than #stages; equality only if balanced.

> ➕ Added: In this example the pipelined clock = 200 ps (slowest stage), vs. 800 ps single-cycle (set by `ld`) → ~4× throughput, not 5×.

- Pipelining is **not reducing latency** — just that **throughput has improved**.

### Pipeline Speedup

- p stages, N instructions.
- Tₛ → single-cycle clock period.
- Tₚ → pipelined clock period = longest stage delay = Tₛ/p (if balanced).
- **Single cycle:** N × Tₛ
- **Pipeline:** (p + N − 1) × Tₚ
- If p ≪ N: (p + N − 1) × Tₚ ≈ N × Tₛ/p → **speedup ≈ p**.

**Note:** if some instruction doesn't use a stage, it still needs to wait → pipelining doesn't skip stages.

My doubt: "Then why multi-stage?"
> ➕ Added: Because even though each instruction passes through all p stages (latency doesn't improve, may even get slightly worse), a new instruction **starts every cycle**, so throughput goes up ~p×. Fixed stage usage also keeps the pipeline simple (no structural conflicts on e.g. the WB port).

### Example

Stage delays: IF 250 ps, ID 350 ps, EX 150 ps, MEM 300 ps, WB 200 ps; **1000 instructions**.

- **Single cycle:** clock = sum = 1250 ps → 1000 × 1250 = **1,250,000 ps**.
- **Pipeline:** clock = max = 350 ps → (1000 − 1 + 5) × 350 = 1004 × 350 = **351,400 ps**.

> ➕ Added: Speedup = 1,250,000 / 351,400 ≈ **3.56×** (less than 5 because stages are unbalanced).

**Q:** Can you make it better?
**A:** Break ID again (split it).
> ➕ Added: Splitting ID (350) into two 175 ps stages → 6 stages, max = 300 ps (MEM) → (1000 − 1 + 6) × 300 = 301,500 ps ≈ 4.15× speedup.

### Why the RISC-V ISA is pipeline-friendly

- Every instruction is 32 bits → fetch & decode are easy (fetch in one cycle).
- Register read & decode are in the same stage because we have few & regular instruction formats (source/dest register fields are always in the same place).
- No memory pointers as operands (only `ld`/`sd` access memory) → things go in sequence (address computed in EX, memory accessed in MEM).

> ➕ Added: Also, memory operands are aligned, so a memory access takes a single cycle.

---

## Lecture 27 — Pipelining and ISA Design; Hazards intro (01 Oct 2026) · pages 66–68

### Pipelining and ISA Design

**Till now in this chapter:**
- Basic logic blocks, and how to connect these blocks together
- Single-cycle CPU
- Multi-cycle datapath
- So we suggest **pipelining**.

**Today:** changes in hardware for pipelining; how the pipeline works.

**Slide photo — "Multi-cycle Datapath" (P&H Ch. 4, slide 57):** the single-cycle RISC-V datapath, with my red outlines splitting it into the 5 stages:

```
| Fetch           | Decode + Reg read     | Execute              | Memory        | Reg write |
| PC, Instr. mem, | Register file (rs1,   | ALU (+ALUSrc mux),   | Data memory   | MemtoReg  |
| PC+4 adder      | rs2, rd, write data), | Shift-left-1 + Add   | (MemRead/     | mux back  |
|                 | Imm Gen (32 -> 64)    | for branch target,   |  MemWrite)    | to reg    |
|                 |                       | PCSrc mux            |               | file      |
```

### Pipeline Registers

- Need **registers between stages** — to hold the information produced in the previous cycle.
- **IF/ID** → stores the instruction (32-bit)
- **ID/EX** → register values (2 × 64-bit data)
- **EX/MEM** → result (64-bit)

> ➕ Added: Each pipeline register also carries everything later stages need: PC (for branch target), sign-extended immediate, the register numbers (rs1, rs2, rd), and the **control signals** for the remaining stages (EX, MEM, WB fields). MEM/WB holds the memory read data / ALU result + rd for write-back.

**IF for load, store** — drawing of the shading convention: a block shaded on its **right half = read from** the block; shaded on its **left half = written to** the block. (E.g., register file is written in the first half of a cycle and read in the second half.)

### Hardware changes

- Need pipeline registers; for write-back, the register-write path (rd + data) must be carried along and shared across stages.
- The **control path** also needs to be stored in those pipeline registers.

### Hazards

1. **Structural hazard** — a required resource is busy.
2. **Data hazard** — need to wait for a previous instruction to complete its data read/write.
   ```asm
   add x5, x6, x7
   add x9, x5, x8      # needs x5 from previous add
   ```
3. **Control hazard** — deciding on a control action depends on a previous instruction.
   - To fetch the next instruction just after a branch, the target is known only later → need to wait for clarity on what the next target is.

**Effect of a hazard:** you have to wait.

### Structural hazard

- **Where can it occur?** Suppose I have only one memory for both instruction memory and data memory.
- **Handling:** you cannot handle it during runtime — it has to be handled at **design time** (e.g., separate instruction and data memories/caches).

### Data hazard

- Not known at CPU-making (design) time — caused by the sequence of instructions.
- Handling: either in design (hardware) or at execution.

```asm
add x19, x0, x1
sub x2,  x19, x3     # needs x19
```

**Method I — add bubbles (stall for two cycles):**

| Instr | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 |
|---|---|---|---|---|---|---|---|---|
| add x19,x0,x1 | IF | ID | EX | MEM | WB | | | |
| sub x2,x19,x3 | | IF | stall | stall | ID | EX | MEM | WB |

(sub's ID reads x19 in the same cycle add writes it — write in first half, read in second half.)

**Method II — bypassing / forwarding:** feed add's ALU output (sitting in EX/MEM) directly into sub's ALU input — no stall.

| Instr | 1 | 2 | 3 | 4 | 5 | 6 |
|---|---|---|---|---|---|---|
| add x19,x0,x1 | IF | ID | EX | MEM | WB | |
| sub x2,x19,x3 | | IF | ID | EX ← fwd | MEM | WB |

- Forwarding/bypassing → **add a multiplexer** at the ALU input.
- **Select line** → whether a hazard is detected or not.
- **Problem:** a load instruction's output is available only **after the MEM block** → forwarding alone can't fix it (see load-use, Lecture 28).


---

## Lecture 28 — Data Hazards: NOPs, Code Scheduling, Forwarding (06 Oct 2026) · pages 69–71

### Data Hazard — revision

- Revision: bubbles, forwarding/bypassing.
- For a load followed by a use → **adding a bubble + bypassing**:

```asm
ld  x1, 0(x2)
sub x4, x1, x5
```

| Instr | 1 | 2 | 3 | 4 | 5 | 6 | 7 |
|---|---|---|---|---|---|---|---|
| ld x1,0(x2) | IF | ID | EX | MEM | WB | | |
| sub x4,x1,x5 | | IF | ID | stall | EX ← fwd (MEM/WB) | MEM | WB |

### Solving the Hazard Problem

- **Case I:** only a simple pipelined CPU, **no hazard support**
  - Sol: add **NOPs** between data-hazard instructions (between write-back and where the value is needed).
  - This can be done by the compiler / user (if ASM is being written by hand).
- **Case II:** pipelined CPU **+ forwarding**
  - **0 NOPs** for ALU instructions
  - **1 NOP** needed for `ld` instruction (load-use)

### Code Scheduling to Avoid Stalls

With forwarding (ld still needs a NOP). Original code:

```asm
ld  x1, 0(x0)
ld  x2, 8(x0)
add x3, x1, x2      # load-use hazard on x2
sd  x3, 24(x0)
ld  x4, 16(x0)
add x5, x1, x4      # load-use hazard on x4
sd  x5, 32(x0)
```

- Cycles = N + P − 1 = 7 + 5 − 1 = 11, **+ 2 NOPs = 13 cycles**.

**After reordering:**

```asm
ld  x1, 0(x0)
ld  x2, 8(x0)       # no NOP needed
ld  x4, 16(x0)      # no NOP needed
add x3, x1, x2
sd  x3, 24(x0)
add x5, x1, x4
sd  x5, 32(x0)
```

- Reordering of `ld x4` can be done because x4 is not used by the instructions it was moved above.
- Cycles = N + P − 1 = 7 + 5 − 1 = **11 cycles**.

> ⚠️ Correction: the last store offset reads like "37(x0)" in the notes; it should be `sd x5, 32(x0)` (doubleword offsets are multiples of 8; this is the standard P&H example).

### Data Hazard in ALU Instructions

- Forward to the n-th instruction if the destination of the (n−1)-th or (n−2)-th instruction = its source.
- ★ Need to keep passing the **register numbers** down the pipeline (x2 here) → **ID/EX.RegisterRs1**, **ID/EX.RegisterRs2**.

```asm
sub x2,  x1, x3     # writes x2
add x12, x2, x5     # (n-1): forward from EX/MEM
or  x13, x6, x2     # (n-2): forward from MEM/WB
add x14, x2, x2     # reads x2 in ID same cycle sub writes it (reg file: write 1st half, read 2nd half)
sd  x15, 100(x2)    # no hazard
```

| Instr | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 |
|---|---|---|---|---|---|---|---|---|---|
| sub x2,x1,x3 | IF | ID | EX | MEM | WB | | | | |
| add x12,x2,x5 | | IF | ID | EX ← EX/MEM | MEM | WB | | | |
| or x13,x6,x2 | | | IF | ID | EX ← MEM/WB | MEM | WB | | |
| add x14,x2,x2 | | | | IF | ID | EX | MEM | WB | |
| sd x15,100(x2) | | | | | IF | ID | EX | MEM | WB |

**→ Detect if there is a need of forwarding:**
- **Case I** (immediately consecutive instructions): `EX/MEM.RegisterRd = ID/EX.RegisterRs1` (or Rs2) → forward from **EX/MEM** to ALU.
- **Case II:** `MEM/WB.RegisterRd = ID/EX.RegisterRs1` (or Rs2) → forward from **MEM/WB** to ALU.
- Only to be done when:
  - the instruction actually writes rd (has rd);
  - x0 is not the rd register.
- **NOTE:** all these checks go to the **select lines of the new multiplexer**.

> ➕ Added — full forwarding conditions (RISC-V):
> - EX hazard: `if (EX/MEM.RegWrite && EX/MEM.RegisterRd != 0 && EX/MEM.RegisterRd == ID/EX.RegisterRs1) ForwardA = 10` (same for Rs2 → ForwardB = 10)
> - MEM hazard: `if (MEM/WB.RegWrite && MEM/WB.RegisterRd != 0 && MEM/WB.RegisterRd == ID/EX.RegisterRs1) ForwardA = 01` (same for Rs2 → ForwardB = 01)
> - Otherwise ForwardA/B = 00 (value from register file).

### Double Data Hazard

```asm
add x1, x1, x2
add x1, x1, x3
add x1, x1, x4
```

- **Problem:** 2 forwarding paths + 1 normal path → **both conditions are true** for the third add.
- → make one of them **higher priority** (the most recent result, i.e. EX/MEM).

**Slide photo — "Forwarding Paths":** ID/EX → two muxes in front of the ALU, each with **2-bit select lines** (ForwardA, ForwardB). Mux inputs: register file value, EX/MEM ALU result, MEM/WB write-back value. A **Forwarding unit** receives Rs1, Rs2 (from ID/EX), EX/MEM.RegisterRd and MEM/WB.RegisterRd and drives the selects.

> ➕ Added — revised MEM-hazard rule for the double data hazard: forward from MEM/WB only if the EX/MEM condition is **not** true:
> `if (MEM/WB.RegWrite && MEM/WB.RegisterRd != 0 && !(EX/MEM.RegWrite && EX/MEM.RegisterRd != 0 && EX/MEM.RegisterRd == ID/EX.RegisterRs1) && MEM/WB.RegisterRd == ID/EX.RegisterRs1) ForwardA = 01` (same for Rs2/ForwardB).

### Load-use

- ★ For load-use you **still need to add a stall** → make hardware do it → **"load-use hazard detection"**.

> ➕ Added — load-use hazard detection (in ID stage):
> `if (ID/EX.MemRead && (ID/EX.RegisterRd == IF/ID.RegisterRs1 || ID/EX.RegisterRd == IF/ID.RegisterRs2))` → stall one cycle.


---

## Lecture 29 — Forwarding recap, Stalls, Control Hazards, Branch Prediction intro (08 Oct 2026) · pages 72–75

### Revision

- **Forwarding** — condition check + priority of conditions
  - Not just passing the value; we need to know the **index of the register**.
  - Does the instruction have rd?
  - Is rd / rs = x0?
- **Reordering**
- **Stalls**

```asm
add x0, x0, x0
add x5, x0, x2      # forwarding x0 here: no issue (value is 0 anyway)
```
but if
```asm
add x0, x10, x11
add x5, x0, x2      # forwarding here would be WRONG
```
- Because we are forwarding the **output of the ALU**, which might not be 0 → x0 should **not** be forwarded.
- **Double data hazard** → priority given to the **consecutive (most recent) instruction's** condition.

**Slide photo — "Datapath with Forwarding" (P&H Ch. 4, slide 106):** full pipelined datapath with Control, Hazard-free forwarding: IF/ID.RegisterRs1, IF/ID.RegisterRs2, IF/ID.RegisterRd passed into ID/EX (Rs1, Rs2, Rd); forwarding unit compares them with EX/MEM.RegisterRd and MEM/WB.RegisterRd and drives the two ALU-input muxes.

### Inserting a stall

- Whatever instruction you have → **no change**:
  - don't change state (registers, memory);
  - **stop the PC from updating**.

**Example:**
```asm
ld  x5, 0(x2)
add x9, x5, x2
```
- **Software approach:** insert a NOP in between.
- **Hardware:** emulate the NOP — duplicate this instruction (`add` is decoded again next cycle) + avoid updating any reg/mem (no PC increment).

> ➕ Added: In hardware the bubble is made by forcing the control signals in ID/EX to 0 (so EX/MEM/WB do nothing), and by preventing PC and IF/ID from being written, so the `add` and the instruction behind it are re-fetched/re-decoded.

- Forwarding HW + hazard detection + stall insertion → **data hazard has been taken care of**.
- ★ **Code re-ordering** saves extra cycles (due to stalls) → mostly used when there are more hazards [this is optimisation].

### Control Hazard

```asm
        beq x1, x7, LABEL
        add ...
LABEL:  sub ...
```

- Wrong instructions entered the pipeline → need to discard these instructions → not really viable (as default).
- ★ The **AND gate** that checks Zero and the branch condition is at the **memory stage** (original design).
- The target instruction should come right after beq, but it needs the output of MEM → **3-cycle latency → 3 bubbles**.

| Instr | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 |
|---|---|---|---|---|---|---|---|---|
| beq x1,x7,LABEL | IF | ID | EX | MEM | WB | | | |
| (bubble) | | stall | | | | | | |
| (bubble) | | | stall | | | | | |
| (bubble) | | | | stall | | | | |
| target (sub/add) | | | | | IF | ID | EX | MEM |

- ★ Latency (total penalty) depends on how many branches return true (taken).

**Solution:** don't use the ALU to compare — do the comparison **without reusing the ALU**, as early as possible:
- As soon as fetched → **no** (don't know register values yet).
- As soon as decoded → **yes**.
- Diagram: register file outputs (x1, x3) feed an **equality comparator "="** placed before the ID/EX register.
  1. Add this block.
  2. Move the shift-left-by-1 and the branch-target adder.
- → improves latency → now you need only a **one-cycle NOP** (unlike the default 3 cycles).

> ⚠️ Correction: the notes say "move shift-left-1 & add to the **execute** stage" — they are moved **into the ID (decode) stage**, together with the comparator; that is what reduces the penalty to 1 cycle.

**Slide photo — "Example: Branch Taken" (Clock 4):** pipeline holds, from IF to WB: `ld x4, 50(x7)` (being fetched from target), **bubble (nop)** in ID (the flushed instruction), `beq x1, x3, 16` [offset partly illegible] in EX, `sub x10, …` in MEM, `before<1>` in WB. The hazard detection unit / IF.Flush zeroes the wrongly fetched instruction.

### Branch Prediction

- Comes into the picture when there are **many stages** (would need many stalls) → to avoid them.
- Building the theory — a for loop:
  ```asm
  loop:  ...
         ...
         beq x0, x0, <-ve offset>   # backward branch
  ```
- In RISC-V:
  - can predict branches **not taken**;
  - fetch the instruction after the branch with no delay. *(student: "doesn't make sense to me")*

> ➕ Added: "Predict not taken" just means keep fetching PC+4 as usual (no delay). If the branch turns out not taken, nothing is lost; if it is taken, the fetched instruction(s) are flushed and fetching restarts at the target — paying the penalty only when taken.

- **Static branch prediction:**
  - backward branch → predict **taken**
  - forward branch → predict **not taken**
- **Dynamic branch prediction:** the previous occurrence determines the present prediction (taken / not taken).

**Q (doubt):** After a prediction is false, some instructions still got executed — how do you undo it?
> ➕ Added — **A:** They haven't changed any state yet: the branch resolves in ID, before the wrongly fetched instruction reaches MEM/WB. The hardware **flushes** them (turns them into NOPs by zeroing their control signals / IF/ID), so nothing is written to registers or memory.


---

## Lecture 30 — Branch Prediction (1-bit, 2-bit), Branch Data Hazards, Exceptions (09 Oct 2026) · pages 76–79

*(Attached PDF "9thoctcarch"; content is phone photos of the paper notebook, dated 9th Oct, Friday. A struck-out "Quiz-2" heading at the top.)*

### Branch Prediction Recap

- Maintain a table: **PC → decision (taken / not taken)**.
- **Dynamic prediction:**
  - check the table, expect the same outcome;
  - start fetching from fall-through / target;
  - if wrong, **flush the pipeline and flip the prediction**.

### 1-Bit Predictor: Shortcoming

- 0 → taken, 1 → not taken
- Nested loops:
  ```
  for ...
      for ...
          ...
  ```
- **Inner-loop branches are mispredicted twice** → on the 1st and last iteration.

> ➕ Added: On the last iteration the branch falls through (mispredict, bit flips to not-taken); on re-entering the inner loop next time, the first iteration is taken but predicted not taken (mispredict again).

### 2-Bit Predictor

- Instead use a 2-bit predictor: [partly illegible: "T=0, t=1"]
  - TT → T
  - NN → N
  - TN → T
  - NT → N   (prioritise the latest one)

**State diagram (photo):**

```
 (Taken, self-loop)
 Predict taken (TT) --Not taken--> Predict taken (TN)
 Predict taken (TT) <---Taken----- Predict taken (TN)
                                      |   ^
                            Not taken |   | Taken
                                      v   |
 Predict not taken (NN) <--Not taken-- Predict not taken (NT)
 Predict not taken (NN) ----Taken----> Predict not taken (NT)
 (Not taken, self-loop)
```

| State | Prediction | On Taken → | On Not taken → |
|---|---|---|---|
| TT (strong taken) | Taken | TT | TN |
| TN (weak taken) | Taken | TT | NT |
| NT (weak not taken) | Not taken | TN | NN |
| NN (strong not taken) | Not taken | NT | NN |

> ➕ Added: Prediction flips only after **two consecutive mispredictions** → the inner-loop exit costs only 1 misprediction per loop instead of 2.

- **NOTE:** You write T or NT against the PC value of the **label**, and not the beq instruction? → **No — against the beq instruction's PC only.**
- **Doubt:** if the prediction is written against the PC of the label, then how could we predict in the IF stage? → **[Cleared]** (it's indexed by the branch's own PC, known at IF).

```
beq rs1, rs2, imm
   ├── not taken → PC + 4
   └── taken     → PC + (imm << 1)     → 1-cycle penalty
                                        Solution: add a Branch Target Buffer
```

> ➕ Added: A **Branch Target Buffer (BTB)** is a cache indexed by the branch's PC that stores the predicted target address, so in IF you already know both the prediction and where to fetch next → no 1-cycle penalty for a correctly predicted taken branch.

### Data Hazard for Branches

```asm
add rd, rs1, rs2
beq rs1, rs2, imm      # rs1 here = rd of the add (squiggle)
```

- Comparison was moved to the **ID stage** → the old forwarding wouldn't be good enough. *Why?*
- Opt 1: add more hardware.
- Opt 2: accept the problem.
  - Sol: **stall it for 1 cycle for `add`, 2 cycles for `ld`.**
  - (sketch: `ld` / `add` followed by the branch, with the stall slots marked)

> ➕ Added — **Why:** the beq now needs its operands at the start of ID, but an ALU result is only ready at the end of the producer's EX (and a load's at the end of MEM), and the old forwarding paths only feed the ALU in EX. So: ALU op immediately before beq → 1 stall; `ld` immediately before beq → 2 stalls; ALU op 2 instructions before → forward to ID comparator, no stall.

### Exceptions & Interrupts

- **Interrupts** → from outside (external).
- **Exceptions** → in(side) the system.

**Handling exceptions** — as an example of a control hazard:
- Jump to a **handler** to process the exception.
- Save the PC of the interrupted (offending) instruction → **SEPC** (Supervisor Exception Program Counter).
- Save an indication of the problem → **SCAUSE** (Supervisor Exception Cause Register).

**Exceptions in a pipeline:**
- A new entry to the PC (handler address).
- Flush.

> ➕ Added: Treat it like a mispredicted branch — flush the offending instruction and those after it (IF/ID/EX), let earlier instructions complete, set SEPC and SCAUSE, then load the handler address into the PC.


---

## Lecture 31 — Problems (13 Oct 2026) · pages 80–81

### Problem 1 — `swap rs1, rs2`

Examine the difficulty of adding a proposed `swap rs1, rs2` instruction to RISC-V.
- Reg[rs1] = Reg[rs2]

> ➕ Added: and simultaneously Reg[rs2] = Reg[rs1] (both old values) — that's why it needs two writes.

- **Option 1:** add **two write ports** in the register file (sketch: register file with 2 read (R) + 2 write (W) ports).
- **Option 2:** keep 2 read + 1 write port and use a **temp** register → **2-cycle latency** → pipelining (hazards/stall) will have to be handled.

### Problem 2a

*(Heading only — nothing written.)*

### Problem 4 — CPI due to mispredicted branches

| Instruction | Frequency |
|---|---|
| R-type | 40% |
| beq / bne | 25% |
| jal | 5% |
| ld | 25% |
| sd | 5% |

**Q:** What is the CPI due to mispredicted branches?
Assume: no data hazards; no delay slots; branch predictor accuracy = 45%.

**A:**
- Ideal CPI = 1
- With hazards: 1 + Δ
- CPI = 1 + 0.55 × 0.25 (0.25 = branch instructions, 0.55 = fraction wrongly predicted)

> ➕ Added: General form CPI = 1 + (branch freq) × (mispredict rate) × (penalty). With a 1-cycle penalty (branch resolved in ID): 1 + 0.25 × 0.55 × 1 = **1.1375**. If the branch resolves later (e.g., 3-cycle penalty in MEM) → 1 + 0.1375 × 3 = 1.4125. Check which penalty the question states.

*(Side scribbles on the page: "Microprogrammed", "function inlining" — terms mentioned in class.)*


---

## Lecture 32 — Improving Performance Further: ILP / Multi-issue (15 Oct 2026) · pages 82–84

*(Attached PDF "15thoctcarch"; photo of notebook dated 15/10, Thursday. Page 84 is a blurry photo of the reverse of the sheet — only bleed-through, no new content.)*

### Improving Performance Further

Where does the scope lie to further improve performance? Look at the following code:

```asm
ld  x10, 0(x2)      # what happens if this access gets a cache miss?
add x7,  x8, x9
sra x12, x19, x22
ld  x28, 0(x17)
```

- Subsequent instructions don't have any dependence — will reordering help? **No.**
- What if we had **two independent pipelines**?

### Instruction-Level Parallelism

Multiple pipelines (**multi-issue pipeline**):

| Pipeline 1 | Pipeline 2 |
|---|---|
| `ld x10, 0(x2)` | `add x7, x8, x9` |
| | `sra x12, x19, x22` |
| | `ld x28, 0(x17)` |

*(scribble "nvaugd" next to Pipeline 2 — [illegible])*

- Case I: stalls → handled in one pipe
- Case II: independent instructions run in the other
- → **higher throughput, CPI < 1**.
- CPI ↔ **IPC** (instructions per cycle) → here IPC = 2 if balanced.

### Dual-Issue Pipeline

- **Case I:** (i+1)-th instruction depends on the i-th → P-I gets all the instructions, P-II gets all NOPs.
- **Case II:** otherwise → split across both.
- How to identify dependences and divide instructions into the two pipelines?
- **Q:** Who does it?

> ➕ Added — **A:** Either the **compiler** (static multiple issue — packets/VLIW) or the **hardware** at run time (dynamic multiple issue — superscalar).


---

### ➕ Completion (from P&H): L32–33: Instruction-level parallelism (p.82–85; L33 page is blank)

#### Answer: two ways to do multiple issue (P&H 4.10)

| | **Static multiple issue** | **Dynamic multiple issue** |
|---|---|---|
| Who decides what issues together | **Compiler** | **Hardware**, at run time |
| Also called | VLIW (Very Long Instruction Word) | **Superscalar** |
| Unit of issue | "Issue packet": one long instruction with N slots | Hardware picks 0..N instructions per cycle |
| Hazard handling | Compiler removes most hazards; NOPs fill empty slots | Hardware detects; often executes **out of order** |
| Pros | Simple, low-power hardware | Works on unmodified binaries; adapts to cache misses at run time |
| Cons | Code tied to the machine; code bloat from NOPs; can't adapt to unpredictable cache misses | Complex, power-hungry hardware |

#### Static dual-issue RISC-V (book's design)

- **Slot 1:** ALU or branch instruction. **Slot 2:** load or store.
- Packets are 64-bit aligned: ALU/branch first, then ld/sd.
- Extra hardware: 2 more register-file read ports and 1 more write port, plus a separate adder for ld/sd address calculation.
- **Extra hazards:**
  - An ALU result can't be used by the ld/sd in the **same** packet.
  - Load-use latency is still 1 cycle, but that is now **2 instructions** (a whole packet) that can't use the loaded value.

#### Worked example: scheduling a loop for dual issue

The loop adds scalar `x21` to every element of an array, walking downward:

```asm
Loop: ld   x31, 0(x20)      # x31 = array element
      add  x31, x31, x21    # add scalar
      sd   x31, 0(x20)      # store result
      addi x20, x20, -8     # decrement pointer
      blt  x22, x20, Loop   # loop while x20 > x22
```

**Scheduled for dual issue (no unrolling):**

| Cycle | ALU/branch slot | ld/sd slot |
|---|---|---|
| 1 | nop | `ld x31, 0(x20)` |
| 2 | `addi x20, x20, -8` | nop |
| 3 | `add x31, x31, x21` | nop |
| 4 | `blt x22, x20, Loop` | `sd x31, 8(x20)` ← offset becomes **8** because x20 was already decremented |

5 instructions in 4 cycles → **IPC = 1.25** (CPI 0.8), against an ideal of 2.

**Loop unrolling (×4) + register renaming:**
- Make 4 copies of the body.
- Use different temporary registers (x28, x29, x30, x31) so the copies don't falsely depend on each other.
- Keep a single `addi x20,x20,-32` and a single branch.
- Result: 14 instructions in 8 cycles → **IPC = 1.75**. The cost is more registers and bigger code.

**Register renaming** removes **name dependences** (anti-dependences / WAR and output dependences / WAW). These exist only because the same register name is reused; no data actually flows between the instructions. True data dependences (RAW) cannot be renamed away.

#### Dynamic pipelining (superscalar, out-of-order)

The pipeline is split into three parts:

1. **In-order issue unit:** fetches, decodes and sends instructions to the functional units.
2. **Reservation stations** sit in front of each functional unit. They hold the operands, wait until all of them are ready, then execute. Execution is **out of order**.
3. **Commit unit / reorder buffer:** results are written back to registers and memory **in order**. This keeps exceptions precise and lets wrong speculation be thrown away.

- **Speculation:** guess the outcome (e.g. a branch, or that a load doesn't alias an earlier store) and execute ahead. If the guess was wrong, flush. The commit unit makes this safe.
- **Link to L40 notes** ("interactions with advanced CPUs"): during a cache miss, the pending load/store waits in the load/store unit while independent instructions keep executing. Dependent instructions wait in reservation stations.

#### Deeper pipelines vs broader pipelines (lecture 32 title)

**Deeper pipelines (superpipelining):** more stages, each stage doing less work, so the clock can be faster.

| Pros | Cons |
|---|---|
| Higher clock frequency | Larger **branch-misprediction penalty**, since more stages are flushed |
| | More data-hazard stall cycles (load-use gap grows) |
| | Pipeline-register overhead (setup time, clock skew) becomes a bigger fraction of the shrinking stage |
| | More power (Power ∝ C·V²·f): the "power wall" ended the GHz race around 2005 |

**Broader (wider) pipelines:** multiple issue (2-, 4-, 6-wide).

| Pros | Cons |
|---|---|
| CPI < 1 | Limited by the program's true dependences (actual ILP is limited) |
| | More register ports, more forwarding paths |
| | Hazard checks grow roughly as N² across the N instructions in flight |
| | Branch every ~5 instructions means you must predict well |

**Takeaway:** both approaches hit diminishing returns (power, dependences), which is why industry moved to **multicore**.


---

## Lecture 33 — (16 Oct 2026) · page 85

*(Page is blank — only the "Lecture - 33" header and date; no notes taken.)*

---

### ➕ Completion (from P&H): Interrupts and exceptions in the pipeline (from the topics overview)

| | Exception (trap) | Interrupt |
|---|---|---|
| Cause | Inside the CPU: illegal instruction, misaligned access, page fault, `ecall`, divide trap | Outside the CPU: I/O device, timer |
| Timing | **Synchronous**: tied to a specific instruction | **Asynchronous**: not tied to any instruction |

**What the hardware does (RISC-V naming):**
1. Save the address of the offending/interrupted instruction in **SEPC** (supervisor exception PC; MEPC in machine mode).
2. Record the reason in **SCAUSE** (MSB = 1 → interrupt, MSB = 0 → exception; the low bits give the code).
3. Jump to the handler address held in **STVEC** (the trap vector).
4. The handler finishes with `sret`/`mret`, which returns to SEPC.

**In the 5-stage pipeline (P&H 4.9):** an exception is handled like a **control hazard**.
- Example: an `add` overflows in EX (the book's case) or an instruction is illegal in ID.
- **Flush** the faulting instruction and all instructions after it, i.e. those in earlier stages: turn them into NOPs and zero their control signals.
- Let the instructions **before** it finish.
- Write SEPC and SCAUSE, and load the handler address into PC. That handler address is the **extra input to the PC mux** I noted on p.111 ("interrupt input to mux before PC").

**Precise exception:** the state is exactly as if all instructions before the faulting one completed and none after it did. RISC-V requires precise exceptions. That is why out-of-order CPUs commit in order.

**Multiple exceptions in the same cycle:** service the one from the **earliest** instruction, i.e. the one deepest in the pipeline.


---

# UNIT 4: Memory Hierarchy (Caches) and I/O

---

## Lecture 34: Memory hierarchy (27 Oct 2026) · pages 86–89

### Where we are
Once caches are covered, we have the complete picture of how the CPU works. The levels of a computer system:

```
Specification → Algorithm → High-level code
   ┌───────────────────────────────┐
   │ Assembly / machine code       │
   │ ISA                           │   ← "this is our stuff" (this course)
   │ Processor                     │
   └───────────────────────────────┘
→ Circuits and gates → Devices & transistors
```

### Memory hierarchy: motivation
- Ways we've already improved CPU efficiency:
  - **Pipelining** speeds up CPU execution time by about 5× (for 5 stages).
  - Hazards are handled.
- **But memory access is slow.** Fetching data from RAM costs extra cycles, roughly **50 cycles**. Can we do something to improve this?

### Locality
- **Example: loops.** Inside a loop, data is reused (e.g. `i`) and instructions are reused.
- So bring that data/instruction close to the CPU: if you used something, you are likely to use it again. This is **temporal locality** (time-dependent). It is useful because programs use loops very heavily.
- **Book analogy:** bring the whole book series (the ones you're most likely to read) to your desk together.
- **In the CPU, the "series" is arrays.** For `a[i] = b[i] + x`:
  - Accesses go `a[0] = b[0] + x`, then `a[1] = b[1] + x`, and so on.
  - Instead of bringing `a[0]`, `a[1]`, … one by one, bring all of them (or several) together, because (i) almost all of them will be used, and (ii) the cost of getting r of them together is less than r × the cost of getting one.
- That covers data. **What about instructions?** They execute in sequence (mostly).
- **Spatial locality:** if you use `a[0]`, you will likely use `a[1]`, `a[2]`, …
- **Two kinds of locality:** temporal and spatial.

### Processor–memory interface (slide photo)
- Memory (RAM) ↔ **CACHE** ↔ processor.
- Inside the processor: Fetch unit → Decode unit → ALU → Register File (RF), all coordinated by the **Control Unit**.
- The **Load and Store Unit (LSU)** also talks to the cache and to the RF.

*(Slide: Chapter 5, "Large and Fast: Exploiting Memory Hierarchy")*

### Principle of locality
- Programs access a small proportion of their address space at any time.
- Temporal locality and spatial locality.
- This gives the **memory hierarchy**: CPU (registers/flip-flops) → cache (**SRAM**) → main memory (**DRAM**) → disk. Cache and main memory are the part this architecture course focuses on.

### Memory technology

| Technology | Access time | Cost |
|---|---|---|
| Static RAM (SRAM) | 0.5–2.5 ns | $500–$1000 per GB |
| Dynamic RAM (DRAM) | 50–70 ns | $3–$6 per GB |
| Magnetic disk | 5–20 ms | $0.01–$0.02 per GB |

**Ideal memory:** the access time of SRAM, with the capacity and cost/GB of disk.


---

## Lecture 35: Cache memory, direct-mapped cache (29 Oct 2026) · pages 90–93

### "Memory wall"
- **Temporal locality:** bring in the data/instructions that are used very often.
- **Spatial locality:** bring in the neighbouring elements along with the ones being brought in.

### Cache memory
- The level of the memory hierarchy **closest to the CPU**.
- **Picture:** a cache holding X₁, X₄, Xₙ₋₁, …, X₃, Xₙ₋₂, X₅, "stored randomly"; a request comes for address Xₙ.
  - **Q:** How do we know if the data is present?
  - **Q:** Where do we look?
  - → This is what the cache organization answers.

### Direct-mapped cache (slide photo)
- 8-entry cache (indices 000–111). Many memory blocks map onto it; each memory block goes to the cache entry given by the **last few bits of its address**.
  - For example, memory addresses 00001, 01001, 10001, 11001 all map to cache index 001.
- The cache stores data by the **last few bits**, and the **upper bits are stored as metadata**. This metadata is called the **"tag"**.

### Cache example: 8 blocks, 1 word/block, direct-mapped

**Initial state:** indices 000–111, all V = N (invalid).

**① Word address 22 = 10110 → miss → cache block 110**
- Entry 110 becomes V = Y, Tag = 10, Data = Mem[10110]. ← inserted ("cache warm-up")

**② Word address 22 = 10110 → hit → block 110**
- Entry 110 is used.

**③ Word address 30 = 11110 → miss → block 110**
- Entry 110 becomes Y, Tag = **11**, Data = Mem[11110]. ← changed ("**eviction**"; the removed entry is the **victim**)

> ⚠️ **Correction:** I wrote tag "10" here. 30 = `11 110`, so the tag is **11**.

### Hit and miss rates

```
Hit rate  = Hits / Total accesses     (ideally 1)
Miss rate = Misses / Total accesses
```

**Q:** All of this is only for reads. What about writes? *(→ Lecture 38)*

### Address subdivision (slide photo)
- 64-bit address: **Tag** = bits 63–12 (52 bits), **Index** = bits 11–2 (10 bits → 1024 entries), **byte offset** = bits 1–0.
- Each entry holds Valid, Tag (52 bits) and Data (32 bits).
- **Hit** = Valid AND (stored tag == address tag); the data goes out.
- The slide raises the question of whether blocks should hold a doubleword or a word.

**What is bad about this?**
- If accesses keep alternating 30, 22, 30, 22, … they always fight over the same entry (110), while other cache entries might be empty.
- *(→ this is a **conflict miss**; fixed by associativity in Lecture 37)*

*(Admin: project PDFs stage I 29 Oct, II 9 Nov, III 18 Nov, IV 24 Nov; presentations 20/21/22 Nov; lab exam (coding) 11 Nov.)*


---

## Lecture 36: Direct-mapped cache (cont.), main memory supporting caches (30 Oct 2026) · pages 94–96

### Direct-mapped cache
- The location is determined by the address.
- Direct-mapped means there is only one choice of location:

```
Index = Block address mod (number of blocks in cache)
```

- The number of blocks must be a power of 2.

> ⚠️ **Correction:** I wrote "mod block size". It is **mod number of blocks**.

### Example 1
Given cache access time = 1 cycle, memory access time = 100 cycles, cache miss rate = 10%.

**(i) Find the average memory access time.**
- AMAT = 1 (cache) + 0.1 (missed) × 100 (memory access) = **11 cycles**

**(ii) What should the miss rate be to get an average of 1.5 cycles?**
- 1 + M × 100 = 1.5 → **M = 0.5%**

### Example 2
Address size = 40 bits; number of entries (8 B each) in the cache = 128. Find the number of bits in each field.

| Tag | Index | Offset |
|---|---|---|
| 30 bits | 7 bits | 3 bits |

**Overhead:** storing 30 extra tag bits per 8-byte entry. **Solution:** increase the block size.

### Handling spatial locality
- Fetch data from the next few addresses too; this unit is called a **"block"**.
- A cache block can be larger than a standard word or doubleword.
- Fetch the entire block together. This enables hits for sequential accesses.

**Q:** Is fetching one larger block faster than multiple smaller blocks?
**A:** Yes. *(Why: see the completion below.)*

### Main memory supporting caches
Preparing data in memory takes time, so we want a wider effective bus. Three organizations:

| (a) One-word-wide memory | (b) Wider memory | (c) Interleaved memory |
|---|---|---|
| Processor ↔ Cache ↔ Bus ↔ Memory | Processor ↔ Multiplexor ↔ wide Cache ↔ wide Bus ↔ wide Memory | Processor ↔ Cache ↔ Bus ↔ Memory banks 0, 1, 2, 3 |
| | Too much hardware cost | Words 0,1,2,3 in banks 0–3; 4,5,6,7 in banks 0–3; … |

**Q:** What will be a problem with interleaved memory organization?
**A:** If one chip goes bad, all data becomes kind of useless (every block has a word in every bank).

**Note:** the compiler makes small padding changes so that block addressing is useful (aligns data to blocks).


---

### ➕ Completion (from P&H): L34–36: Memory hierarchy and direct-mapped caches (p.86–96)

#### Terms the slides skipped (P&H 5.1)
- **Block / line:** the minimum unit copied between two levels.
- **Hit:** the data is found in the upper level.
  - **Hit rate** = hits / accesses.
  - **Hit time** = time to access the upper level, including the time to decide hit or miss.
- **Miss:** the data must be fetched from the lower level.
  - **Miss rate** = 1 − hit rate.
  - **Miss penalty** = time to fetch the block from the lower level, place it in the cache and deliver the word.

#### Direct-mapped placement

```
Cache index = (Block address) mod (Number of blocks in cache)
```

- Because the number of blocks is a power of 2, the mod is free: the index is just the **low log₂(#blocks) bits** of the block address.
- ⚠️ p.94 says "Block add (mod) Block size". It should be **mod number of blocks**. Block *size* is what the offset bits come from.

#### Address breakdown

```
| ----- Tag ----- | --- Index --- | --- Block offset --- | Byte offset |
                    log2(#sets)     log2(words/block)     log2(bytes/word)
Tag bits = address bits − index bits − offset bits
```

- **Valid bit:** whether the entry holds real data. All entries start invalid.
- **Tag:** the high address bits, stored so we can tell **which** memory block sits in this entry. This is the "metadata" I wrote about.
- **Slide photo (p.92):** 64-bit address, 1024 entries, one 4-byte word per block. Byte offset = bits 1:0 (2 bits), index = bits 11:2 (10 bits), **tag = bits 63:12 (52 bits)**. Hit = valid AND (stored tag == address tag).

#### Completing the direct-mapped trace (p.90–92)

8 blocks, 1 word per block, word addresses. Index = low 3 bits, tag = remaining bits. The book's full sequence (P&H Fig 5.9):

| # | Word addr | Binary | Hit/Miss | Index | What happens |
|---|---|---|---|---|---|
| 1 | 22 | 10 110 | miss (cold) | 110 | load Mem[10110], tag 10 |
| 2 | 26 | 11 010 | miss (cold) | 010 | load Mem[11010], tag 11 |
| 3 | 22 | 10 110 | **hit** | 110 | |
| 4 | 26 | 11 010 | **hit** | 010 | |
| 5 | 16 | 10 000 | miss | 000 | load Mem[10000], tag 10 |
| 6 | 3 | 00 011 | miss | 011 | load Mem[00011], tag 00 |
| 7 | 16 | 10 000 | **hit** | 000 | |
| 8 | 18 | 10 010 | miss | 010 | **replaces 26** (tag 11 → 10) |
| 9 | 16 | 10 000 | **hit** | 000 | |

Hit rate = 4/9, miss rate = 5/9.

⚠️ **Correction to my p.92 trace (address 30):** 30 = `11 110`, so index 110 and **tag = 11, not 10**. The entry 110 changes from (tag 10, Mem[10110]) to **(tag 11, Mem[11110])**. The old block is the **victim** of the eviction. My note "if you keep getting 30, 22 it always fills the same slot" is exactly a **conflict miss**, and associativity (L37) fixes it.

#### Field-size problems (completing p.94 Q2)

**Q2 (mine):** 40-bit address, 128 entries of 8 B each, direct-mapped.
- Offset = log₂8 = **3**
- Index = log₂128 = **7**
- Tag = 40 − 7 − 3 = **30**
- Overhead per entry = 30 tag + 1 valid = 31 bits for 64 bits of data, which is a lot. My note's fix is right: bigger blocks spread the tag over more data.

**Book example (P&H 5.2): total bits in a cache.** 16 KiB of data, 4-word (16 B) blocks, 32-bit addresses.
- Blocks = 16 KiB / 16 B = 2¹⁰ → index 10 bits. Offset = 4 bits. Tag = 32 − 10 − 4 = 18 bits.
- Per block: 128 data + 18 tag + 1 valid = 147 bits.
- Total = 2¹⁰ × 147 = **147 Kbits**, about 1.15× the data alone.

**Q1 (mine, p.94):** hit = 1 cycle, memory = 100 cycles, miss rate 10%.
- AMAT = 1 + 0.1 × 100 = **11 cycles** (the final number is missing from my page).
- For AMAT = 1.5: 1 + M × 100 = 1.5 → **M = 0.5%** ✓.

#### Handling spatial locality: multi-word blocks
- Block address = ⌊byte address / bytes per block⌋.
- **Example:** 64 blocks of 16 B. Byte address 1200 → block 1200/16 = 75 → index 75 mod 64 = **11**.

**Answer to "Is fetching one larger block faster than several smaller blocks? Yes, why?"** All three costs are paid once for the whole block instead of once per word:
- the address is sent once,
- the DRAM row is opened once (the access latency), and
- the words then stream out back-to-back (burst).

You also store one tag instead of several.

#### Main memory supporting caches (completing p.95)

Assumptions (book): 1 bus cycle to send the address, **15** bus cycles per DRAM access, 1 bus cycle to send one word, block = 4 words.

| Organization | Miss penalty | Bandwidth (bytes/cycle) |
|---|---|---|
| (a) One-word-wide memory | 1 + 4×15 + 4×1 = **65** cycles | 16/65 ≈ 0.25 |
| (b) Wide memory (4 words) | 1 + 15 + 1 = **17** cycles | 16/17 ≈ 0.94 |
| (c) Interleaved, 4 banks, 1-word bus | 1 + 15 + 4×1 = **20** cycles | 16/20 = 0.80 |

- **Interleaving** gets most of the benefit of a wide memory: all banks access in parallel, then send one word each over a narrow bus. It avoids the cost of a wide bus and mux.
- **Downsides of interleaving:**
  - Needs at least as many banks (chips) as words per block.
  - Consecutive words are spread across chips, so one failing chip corrupts part of **every** block. That is the point on my p.95.
- Modern DRAM gets the same effect with internal banks plus burst/DDR transfers.

**Reducing the miss penalty of large blocks:**
- **Early restart:** resume execution as soon as the needed word arrives, without waiting for the whole block.
- **Critical word first:** fetch the requested word first, then the rest of the block.


---

## Lecture 37: Associativity, CAM, writing (03 Nov 2026) · pages 97–98

### Spectrum of associativity: for a cache of 8 entries
- **One-way set associative (direct-mapped):** 8 blocks (0–7), each holding Tag | Data.
- **Two-way set associative:** 4 sets (0–3), each holding (Tag | Data) × 2.
- **Four-way set associative:** 2 sets (0–1), each holding (Tag | Data) × 4.
- **Eight-way set associative (fully associative):** 1 set holding (Tag | Data) × 8.

**More associativity → more searches**
- The number of tag bits ↑ → needs more storage.
- More comparators are needed (huger hardware).

### Set-associative cache organization (slide photo)
- 4-way set associative, 256 sets.
- Address: **Tag = 22 bits** (31–10), **Index = 8 bits** (9–2), byte offset (1–0).
- The index selects one set (row 0–255) in each of the 4 ways (each way holds V, Tag, Data).
- 4 comparators check the 4 tags in parallel; their outputs, ANDed with V, drive a **4-to-1 multiplexor** that selects the Data. The OR of the 4 matches gives **Hit**.

**Q:** Do we go and search sequentially?
**A:** No. We extract the tags using the index and compare all the tags (in parallel).

> **Note to self:** I missed the associative cache example and did not listen to the class from here.

Headings written but not filled: **CAM**, **writing**. *(→ completed below and in Lecture 38)*

*(Admin: CARCH project stages 9th/18th/24th; lab exam 11th 3:5[?]; presentations 20/21/22; **final exam 15th, 10–1**.)*


---

### ➕ Completion (from P&H): L37: Associativity (p.97–98: "associative cache example missed, did not listen from here")

#### Placement rule

```
Set index = (Block address) mod (Number of sets)
#sets = #blocks / associativity
```

- The block can go in **any way** within its set.
- On lookup, compare the tags of **all ways of that set in parallel** (one comparator per way). The matching way drives a mux.
- That is the photo on p.97: a 4-way cache with 256 sets → 8 index bits, 22-bit tag, 4 comparators and a 4-to-1 mux.
- **Answer to "Do we search sequentially?" (p.98):** No. The index selects the set, then all ways' tags are compared **simultaneously**.

#### The missed example (P&H 5.3): 4 one-word blocks, block addresses 0, 8, 0, 6, 8

**Direct-mapped** (index = addr mod 4: 0→0, 6→2, 8→0):

| Addr | Hit/Miss | Block 0 | Block 1 | Block 2 | Block 3 |
|---|---|---|---|---|---|
| 0 | miss | M[0] | | | |
| 8 | miss | **M[8]** | | | |
| 0 | miss | **M[0]** | | | |
| 6 | miss | M[0] | | M[6] | |
| 8 | miss | **M[8]** | | M[6] | |

→ **5 misses**. 0 and 8 keep evicting each other: a conflict.

**2-way set associative** (2 sets; set = addr mod 2, so all three addresses map to set 0; **LRU** replacement):

| Addr | Hit/Miss | Set 0, way 0 | Set 0, way 1 |
|---|---|---|---|
| 0 | miss | M[0] | |
| 8 | miss | M[0] | M[8] |
| 0 | **hit** | M[0] | M[8] |
| 6 | miss | M[0] | **M[6]** ← replaces LRU = 8 |
| 8 | miss | **M[8]** ← replaces LRU = 0 | M[6] |

→ **4 misses**

**Fully associative** (any block anywhere):

| Addr | Hit/Miss | Contents |
|---|---|---|
| 0 | miss | 0 |
| 8 | miss | 0, 8 |
| 0 | **hit** | 0, 8 |
| 6 | miss | 0, 8, 6 |
| 8 | **hit** | 0, 8, 6 |

→ **3 misses**: only the compulsory ones.

**Lesson:** higher associativity → fewer conflict misses. The costs are a longer hit time (comparators + mux), more hardware and power, and more tag bits.

#### Tag-size cost of associativity (P&H)

4096 blocks, 4-word blocks (offset 4 bits), 32-bit address:

| Associativity | Sets | Index bits | Tag bits | Total tag bits |
|---|---|---|---|---|
| Direct-mapped | 4096 | 12 | 16 | 16 × 4096 = 64 K |
| 2-way | 2048 | 11 | 17 | 17 × 4096 = 68 K |
| 4-way | 1024 | 10 | 18 | 18 × 4096 = 72 K |
| Fully assoc. | 1 | 0 | 28 | 28 × 4096 = 112 K |

Each **doubling of associativity halves the number of sets**, which removes 1 index bit and adds 1 tag bit.

#### Replacement policies (which block in the set to evict)
- **LRU (Least Recently Used):** evict the block unused for the longest time.
  - Exploits temporal locality.
  - 2-way needs just 1 bit per set.
  - True LRU gets expensive beyond 4 ways, so real CPUs use **pseudo-LRU**.
- **Random:** simple. At high associativity it is nearly as good as LRU.
- **FIFO:** evict the oldest-inserted block.
- Direct-mapped caches need no policy: there is only one choice.

#### CAM: Content Addressable Memory (heading on p.98)
- A normal RAM takes an **address** and returns data.
- A **CAM** takes **data (a tag)** and returns *where* it matched (or hit/miss). The comparator is built into every entry, so all entries are searched in one cycle.
- **Used for:** highly associative caches (the book notes 8-way and above often use CAM for tags), fully associative structures like the **TLB**, and network routing tables.
- **Cost:** more area and power per bit than SRAM.


---

## Lecture 38: Caches, write policies (05 Nov 2026, Thursday) · pages 99–100

### Write-through
- **Data hit:** if we update **only** the cache, memory becomes inconsistent.
- **Write-through:** update memory as well. This slows down the write process.
- **Write buffer:** CPU → write → cache, and also CPU → write buffer → main memory.
  - The buffer is added so the CPU can proceed to the next instruction without waiting for memory.
  - It is not helpful when we have continuous writes (the buffer fills up).

### Write-back
- Dirty block. *(Notes stop here; completed below.)*


---

### ➕ Completion (from P&H): L38: Write policies (completing p.99, which stops at "write back: dirty block")

#### Write-back (copy-back)
- On a store **hit**, write **only the cache** block and set its **dirty bit = 1**.
- Memory is updated **only when the dirty block is evicted** (replaced).
- **Clean** blocks (dirty = 0) can simply be overwritten on eviction, with no write to memory.
- **Pros:** many writes to the same block cost one memory write. Uses much less memory bandwidth, which is good when the CPU generates writes faster than memory can absorb them.
- **Cons:**
  - Memory is temporarily stale, which causes the I/O coherence problem (see L41).
  - A miss that evicts a dirty block costs **two** transfers: write the old block back, then read the new one. A **write-back buffer** hides this: copy the dirty block to the buffer, read the new block, then write the buffer to memory.
  - More complex to implement.
- Stores need care: you can't write before you know it's a hit, or you'd overwrite another block's data. Either check the tag first (two cycles), or use a **store buffer**.

#### Write misses (store to a block not in cache)

| Policy | What happens | Usually paired with |
|---|---|---|
| **Write-allocate** (fetch on write) | Fetch the block into the cache, then write it (as a hit) | **Write-back** |
| **No-write-allocate** (write-around) | Write straight to memory; don't bring the block into cache | **Write-through** |

**Motivation for no-write-allocate:** programs sometimes write whole blocks they won't read soon. A classic case is the OS zeroing a page; caching those blocks would just pollute the cache.

#### Summary table

| | Write-through | Write-back |
|---|---|---|
| Memory up to date? | Always | Only after eviction |
| Extra state bit | None (valid only) | **Dirty bit** |
| Memory traffic | One memory write per store | One write per dirty eviction |
| Read miss cost | Read only | May be write-back **+** read |
| Needs | Write buffer | Write-back buffer (optional) |
| Typical level | Some L1s, simple embedded caches | L2/L3, most modern L1 D-caches |


---

## Lecture 39: Important terms, sources of misses, design trade-offs, unified vs split, cache performance (06 Nov 2026, Friday) · pages 101–104

### Important terms
- Block/line, tag, valid bit, dirty bit, associativity, capacity
- Write-back, write-through
- Hit rate, miss rate, hit time, miss penalty
- Victim block/line

### Sources of misses (the 3 C's)
**Q:** When does each happen?
- **Compulsory misses** (cold-start misses): the first access to a block.
- **Capacity misses:** due to the finite cache size; a replaced block is later accessed again.
- **Conflict misses** (collision misses):
  - Happen in a non-fully-associative cache, due to competition for entries in a set.
  - Would not occur in a fully associative cache of the same total size.

**Q:** How to reduce each?
- (i) Compulsory: increase the block size.
- (ii) Capacity: a bigger cache.
- (iii) Conflict: increase associativity.

**Q:** Are they independent?
**A:** No. Example: with the total size kept the same, increasing the block size decreases the number of blocks (and so effectively the associativity/sets) [check once].

### Cache design trade-offs

| Design change | Effect on miss rate | Negative performance effect |
|---|---|---|
| Increase cache size | Decreases capacity misses | May increase access time |
| Increase associativity | Decreases conflict misses | May increase access time |
| Increase block size | Decreases compulsory misses | Increases miss penalty; for very large blocks, may increase miss rate due to pollution |

### Block size considerations
- A larger block size should reduce the miss rate, due to spatial locality.
- But in a fixed-sized cache … *(incomplete; completed below)*

### Unified cache vs split cache
- **Unified cache:** instruction memory (IM) and data memory (DM) are stored together. One can even evict the other.
- **Split cache:**
  - Separate caches for IM and DM (I-cache and D-cache).
  - Allows a separate cache configuration for each; they can be of different sizes.

**Q:** How to choose?
- Advantage of choosing **unified**: the sizes can be adjusted dynamically.
- Reason for choosing **split**: data and instructions are of different natures. Instructions come in sequential order; data comes in random order.
- **Split cache is mostly used in processors at level 1 (L1).**

### Measuring cache performance

```
D-cache stalls = (Memory accesses / Program) × Miss rate × Miss penalty
I-cache stalls = (Instructions / Program) × (Misses / Instruction) × Miss penalty
CPU time ≈ #inst × CPI × CP + (#ld/sd) × miss rate × miss penalty × CP   (my scribble)
```

> ⚠️ **Correction:** my scribble mixes cycles and time. The clean form is in the completion below.


---

### ➕ Completion (from P&H): L39: Sources of misses, trade-offs, block size, unified/split, performance (completing p.101–104)

#### Completing "Block size considerations: But in a fixed-sized cache ..." (p.103)
- **Larger blocks reduce the miss rate** at first, thanks to spatial locality.
- **But in a fixed-size cache**, larger blocks mean **fewer blocks**. That causes:
  1. More competition for the fewer slots → more conflict/capacity misses. The miss rate **eventually goes back up**.
  2. A block may be evicted before most of its words are used (**cache pollution**).
  3. The **miss penalty grows**, since more words must be transferred. The latency part stays the same; the transfer part grows with block size.
- So miss rate vs. block size is **U-shaped**: the best block size depends on the cache size, and for a small cache, very large blocks hurt.
- Fixes for the penalty: early restart and critical word first (see §4).

#### Unified vs split (recap and additions)
- **Split** (separate I-cache and D-cache) is used at **L1** because:
  - The pipeline fetches an instruction **and** accesses data **in the same cycle** (IF and MEM stages). One unified cache would be a **structural hazard**; split caches double the bandwidth.
  - Each can be tuned separately: different size, associativity, block size, and read-only I-cache.
- **Unified** is used at **L2/L3**: one pool, so the space is shared dynamically and the overall hit rate is slightly higher for the same total size.

#### Measuring cache performance (cleaning up p.104)

⚠️ The scribbled formula on p.104 mixes terms. The clean form (P&H 5.3):

```
CPU time = (CPU execution clock cycles + Memory-stall clock cycles) × Clock cycle time

Memory-stall cycles = (Memory accesses / Program) × Miss rate × Miss penalty
                    = (Instructions / Program) × (Misses / Instruction) × Miss penalty

Split into:
  I-cache stalls = IC × I-miss rate × Miss penalty                    (every instruction is fetched)
  D-cache stalls = IC × (fraction of ld/sd) × D-miss rate × Miss penalty

CPI_total = CPI_base + I-stalls/instr + D-stalls/instr
AMAT      = Hit time + Miss rate × Miss penalty
```

**Write-through stall detail:** Write-stall cycles = (Writes/Program × Write miss rate × Write miss penalty) + write-buffer stalls. The buffer stalls are ignored when the buffer is deep enough.

#### Book example: calculating cache performance
I-cache miss rate 2%, D-cache miss rate 4%, base CPI = 2, miss penalty = 100 cycles, loads and stores = 36% of instructions.
- I-miss cycles = I × 0.02 × 100 = **2.00 I**
- D-miss cycles = I × 0.36 × 0.04 × 100 = **1.44 I**
- CPI with stalls = 2 + 2 + 1.44 = **5.44**
- A perfect cache would be 5.44 / 2 = **2.72× faster**.

**Follow-up (Amdahl's law):** improve the pipeline so base CPI = 1 with the same memory.
- CPI = 1 + 3.44 = 4.44, so the perfect cache is now **4.44×** faster.
- The fraction of time spent stalling rises from 3.44/5.44 = **63%** to 3.44/4.44 = **77%**.
- A faster CPU makes memory **more** important. The same happens if you raise the clock rate: the miss penalty in cycles grows.

#### Book example: AMAT
Clock 1 ns, miss penalty 20 cycles, miss rate 0.05, hit time 1 cycle.
- AMAT = 1 + 0.05 × 20 = **2 cycles = 2 ns**


---

## Lecture 40: Multilevel caches, overhead of caches, SPM, prefetching (10 Nov 2026, Tuesday) · pages 105–108

- Multilevel cache comparison between ARM and Intel across L1, L2, L3.
- **LLC:** last-level cache.

### Overhead of caches
- More area.
- Consumes power.
- Different behaviour of the program: **"unpredictability"**. The same program takes a different time to run on each run.
  - **How do we budget the time?** You will want to budget according to the worst case (all misses).
  - But then why would we use a cache at all?
  - → **"Real-time systems"**: you would not want a cache, because caches are unpredictable.
- **Alternative to caches: "scratchpad memory" (SPM)**, a "software-managed cache".
  - With a cache, your program doesn't have to know about the cache; the system takes care of it.
  - With SPM, you instead let the programmer configure that small memory.

```
CPU ──── SPM          (data moved between SPM and main memory, done by the programmer)
 └────────────────── main memory
```

- **Overheads:** no (cache-style) overheads.
- A mix of caches and SPM can also be used.
- SPM management can also be done by compilers.

### Caches in the RED-V board
- Data cache: **16 KB SPM**
- Instruction cache: **2-way** instruction cache, **16 KB**

### Interactions with advanced CPUs
Out-of-order CPUs can execute instructions during a cache miss:
- The pending store stays in the load/store unit.
- Dependent instructions wait in reservation stations.

### Spatial locality exists in data: PREFETCHING
**Example:** `a[1000]`, with a block holding a[0], a[1], a[2], a[3].
- Access pattern: miss, hit, hit, hit → hit rate = 3/4 = 75%.

**How to make it better?**
- Increase the block size. But that has other side effects.
- Use a small piece of hardware that:
  - fetches a block even before it is requested by the CPU,
  - reduces the miss penalty,
  - is useful for regular access patterns.


---

### ➕ Completion (from P&H): L40: Multilevel caches, overhead, SPM, prefetching (completing p.105–108)

#### Multilevel caches
- **L1:** small and fast, split I/D. Designed to **minimize hit time** (so it fits in the cycle).
- **L2:** bigger, unified. Designed to **minimize the miss rate to memory**, so it uses higher associativity and larger blocks. Its hit time matters less.
- **L3 / LLC (Last-Level Cache):** largest, usually **shared** among all cores.
- Because L2 exists, L1 can be **smaller** than it would be in a single-level design, and L2 can be larger and slower.
- My note mentions an ARM vs Intel comparison across L1/L2/L3. The typical pattern is the same on both: per-core L1 I/D of ~32–64 KB, L2 of ~256 KB–2 MB (per core or per cluster), and a shared LLC of several MB.

**Local vs global miss rate:**
- **Local miss rate** of L2 = L2 misses / **L2 accesses** (only the L1 misses reach L2). It is usually high, because L1 has already taken the easy hits.
- **Global miss rate** = misses / **all CPU accesses** = L1 miss rate × L2 local miss rate.

#### Book example: performance of multilevel caches
- Base CPI = 1, clock = 4 GHz (0.25 ns), main memory = 100 ns, L1 miss rate = 2% per instruction.
- Add an L2 with 5 ns access that cuts the global miss rate to memory to 0.5%.

Steps:
- Memory miss penalty = 100 / 0.25 = **400 cycles**.
- L1 only: CPI = 1 + 0.02 × 400 = **9**.
- L2 penalty = 5 / 0.25 = **20 cycles**.
- With L2: CPI = 1 + 0.02 × 20 + 0.005 × 400 = 1 + 0.4 + 2.0 = **3.4**.
- Speedup = 9 / 3.4 = **2.6×**

Cross-check: (2% − 0.5%) × 20 = 0.3 for the hits in L2, plus 0.5% × (20 + 400) = 2.1 for the misses that go to memory → 1 + 0.3 + 2.1 = 3.4 ✓

**Inclusive vs exclusive** (brief):
- **Inclusive:** everything in L1 is also in L2. This makes coherence checks easy.
- **Exclusive:** a block is in only one level, which gives more effective capacity.

#### Overhead of caches and scratchpad memory (recap; my notes cover this well)
- **Costs of caches:** area, power, and unpredictable timing (the same program takes a different time on each run).
- **Real-time systems** budget for the **worst-case execution time (WCET)**, where every access is assumed to miss. If you budget for the worst case anyway, the cache's average-case benefit is wasted.
- **Scratchpad memory (SPM):** a small on-chip SRAM mapped at fixed addresses and managed by software (programmer or compiler, which copies data in and out, often via DMA).
  - Deterministic timing.
  - No tags or comparators, so less area and power.
  - ARM Cortex-R parts call this **TCM (Tightly Coupled Memory)**.
- **RED-V board:** data side = 16 KB SPM (a DTIM); instruction side = 2-way 16 KB I-cache.

#### Interaction with advanced (out-of-order) CPUs
- **Nonblocking (lockup-free) caches** keep serving the CPU during a miss:
  - **Hit under miss:** later hits are served while a miss is outstanding.
  - **Miss under miss:** several misses are outstanding at once, overlapping their penalties.
- Together with OoO execution (pending store in the LSU, dependents waiting in reservation stations), much of the miss latency is **hidden**. This makes simple stall formulas pessimistic for OoO CPUs.

#### Prefetching (completing p.107)
**My example:** `a[1000]` of words, 4-word blocks. Each block gives miss, hit, hit, hit → hit rate 3/4 = 75%. Spatial locality alone can't beat this, because the **first** access to each block always misses (compulsory).

**Prefetching:** fetch a block **before** the CPU asks for it, so even that first access hits.
- **Hardware prefetchers:**
  - **Next-line (sequential):** on a miss (or access) to block B, also fetch B+1.
  - **Stride prefetcher:** detects a constant stride per load instruction (e.g. `a[i]` stepping by 8 B, or `a[i*k]`) and fetches `address + stride` ahead of time.
  - **Stream buffer:** places prefetched blocks in a small separate buffer, not the cache, to avoid **pollution**.
- **Software prefetching:** the compiler or programmer inserts prefetch instructions a few iterations ahead (e.g. RISC-V Zicbop `prefetch.r`, or `__builtin_prefetch` in C). These are hints: they never fault and change no architectural state.

**What makes prefetching good or bad:**
- **Accuracy:** useless prefetches waste bandwidth and evict useful blocks (pollution).
- **Timeliness:** too late and the latency isn't hidden; too early and the block is evicted before use.
- **Coverage:** the fraction of misses removed.
- Works best for **regular access patterns** (arrays, streaming) and poorly for pointer-chasing (linked lists, trees).


---

## Lecture 41: I/O, MMIO, interrupts, DMA (12 Nov 2026, Thursday) · pages 109–113

### I/O organization

```
CPU ── cache ── mem
 │
Bridge
 │
I/O
```

**Q:** Are our instructions enough to have I/O configuration?
- Devices have a software interface: **registers**.
- We use `ld`/`sd`: the I/O registers can be mapped to addresses (**memory-mapped I/O**).
- Kinds of device registers:
  - **Control register:** settings of the device (configuration).
  - **Status register:** tells the status of the device.
  - **Data register:** the content of the packet, e.g. the key pressed (keystroke).

### Problem with MMIO
- A **coherence problem** in I/O, due to the cache:
  - Data remains in the cache for a long period.
  - The data in the cache is not updated when the I/O device updates it.
- So I/O regions are made **non-cacheable**.
- **Q:** Is our system's I/O cacheable?

### When should software check the I/O registers?
I/O can be extremely slow. Two options:
- **Polling:**
  - Wastes CPU time in `ld`/`sd`.
  - More bus traffic.
- **Interrupts**

### Interrupts
- A **wire** connects the I/O device to the CPU.
- A **handler** in software acts on the interrupt: the **ISR** (Interrupt Service Routine).
- It must identify the source among multiple I/O devices (several interrupt lines come into the CPU).

### Vectored interrupt
- You know the source of the interrupt directly. The lines are combined with **different interrupt handlers for each line**.
- This saves searching for which interrupt it is; every source has a unique index.
- **Architecture difference:** the interrupt is an extra input to the **mux before the PC**, plus the **SEPC** and **SCAUSE** registers.

### Interrupt service flow
1. Complete the ongoing instruction (from the EX stage).
2. Save all registers (not just caller/callee-saved).
3. Check the cause of the interrupt.
4. Act as per the interrupting device's request.
5. Restore all registers.
6. Jump back to the interrupted address (**SEPC**).

**Other points:**
- **Interrupts within an ISR:** in general, interrupts are disabled in an ISR.
- **NMI (non-maskable interrupts) / traps:** should not be disabled.
- **Priority** among devices.
- **SCAUSE, SEPC**: handled by the OS.

### Direct Memory Access (DMA)
- An **accelerator for data transfer**.
- **Configuration:** source address, destination address, length.
- Works without CPU intervention.
- **Transfer types:** I/O → memory, memory → I/O, memory → memory.
- **Operating modes:**
  - **Burst transfer:** a bulk transfer; the CPU waits.
  - **Cycle stealing:** works only when the CPU is idle.

> ⚠️ **Correction:** "only when the CPU is idle" describes *transparent* mode. See the completion below.

**Processor classes:**
- **A:** application class
- **R:** real-time class, e.g. control systems
- **M:** microcontroller

*(Admin: grading breakdown: labs 10–12, quizzes 9–11, HW 1.5×4, lab exam 10%, exams 10–20 / 20–24, project 16–18, CP 2–3.)*


---

### ➕ Completion (from P&H): L41: I/O, MMIO, polling, interrupts, DMA (completing p.109–113)

#### Two ways to reach device registers
1. **Memory-mapped I/O (MMIO):** device registers get ordinary physical addresses, so plain `ld`/`sd` (`lw`/`sw`) access them. **RISC-V uses this.** It needs no new instructions, and the OS protects devices with ordinary memory protection.
2. **Dedicated I/O instructions** (x86 `in`/`out`): a separate I/O address space, with instructions that are privileged in kernel mode.

**Answer to "Are our instructions enough for I/O configuration?" (p.109):** Yes. With MMIO, `ld`/`sd` are enough.

**Example: polling a UART in RISC-V.** The base address and bit positions are illustrative.

```asm
        li   t0, 0x10013000     # UART base address
wait:   lw   t1, 4(t0)          # read STATUS register
        andi t1, t1, 1          # bit 0 = "data ready"
        beqz t1, wait           # not ready → keep polling
        lw   a0, 0(t0)          # read DATA register (the received byte)
```

#### The "problem with MMIO" note (p.109–110): caching
- If an MMIO address were cached:
  - the CPU would keep reading the **stale cached copy** of the status register and never see the device change;
  - writes might sit in a write-back cache and never reach the device.
- **Fix:** MMIO regions are marked **non-cacheable** (and strongly ordered). On RISC-V this is set by the platform's **PMAs (Physical Memory Attributes)**. In C, use `volatile` so the compiler doesn't optimize the loads away.
- **Answer to "Is our system's I/O cacheable?":** No. Device regions must be uncached. Normal RAM is cached.

#### Polling vs interrupts
**Polling:** the CPU repeatedly reads the status register.
- **Pros:** simple, with predictable timing, which is why hard real-time embedded systems use it.
- **Cons:** wastes CPU time on ld/sd and adds bus traffic, especially because I/O is slow.

**Polling overhead (illustrative, in the style of the book's example):** 1 GHz CPU, 400 cycles per poll.
- **Mouse** polled 30×/s → 12,000 cycles/s ≈ **0.001%** of the CPU. Fine.
- **Disk** at 4 MB/s, read in 16-B chunks → 250,000 polls/s × 400 = 10⁸ cycles/s = **10%** of the CPU. And you must poll all the time, even when the disk is idle. Bad, so use interrupts or DMA.

**Interrupt-driven I/O:** the device raises a signal on an **interrupt line** and the CPU runs an **ISR** (Interrupt Service Routine) only when needed.
- Differences from an exception: an interrupt is **asynchronous** (not tied to any instruction), so the CPU only checks for it between instructions. It must also identify **which device** raised it, and devices have **priorities**.

#### Identifying the source
- **Non-vectored (polled) interrupts:** one shared line. The handler reads a cause register (or polls each device) to find the source.
- **Vectored interrupts:** each source has its own number/vector, and hardware jumps **directly** to that source's handler. This saves the search (my note: "everyone has a unique index").
- **On RISC-V:** `stvec`/`mtvec` hold the trap base address.
  - MODE = 0 (direct): all traps go to BASE.
  - MODE = 1 (**vectored**): interrupts jump to **BASE + 4 × cause**.
  - Exceptions always go to BASE.

#### Interrupt service flow (cleaned up from p.111)
1. **Finish the current instruction** (the one in EX) and let earlier instructions complete; flush later ones. Hardware saves the PC → **SEPC** and the reason → **SCAUSE**, disables interrupts, and jumps to the trap vector.
2. The ISR **saves all registers** it uses. This means **all** of them, not just caller/callee-saved, because the interrupted code didn't "call" anything and expects every register intact.
3. **Check the cause** (SCAUSE, or the vector number), plus the device's status register.
4. **Service the device**, e.g. read the data register, which usually also clears the device's interrupt request.
5. **Restore** all registers.
6. **`sret`** → jump back to SEPC and re-enable interrupts.

**Other points:**
- **Nested interrupts:** interrupts are normally **disabled inside an ISR**. To allow a higher-priority interrupt to preempt, the ISR first saves SEPC/SCAUSE, then re-enables interrupts.
- **NMI (Non-Maskable Interrupt):** cannot be disabled. Used for critical events such as hardware errors, power failure or the watchdog.
- **Priority:** when several devices interrupt at once, the highest priority is served first. Internal exceptions generally outrank I/O interrupts, and fast devices get higher priority. On RISC-V, the **PLIC** (Platform-Level Interrupt Controller) does this priority arbitration for external interrupts, and the **CLINT** handles timer and software interrupts.
- **CSRs involved:**
  - `sstatus.SIE`: global enable
  - `sie`: per-source enable
  - `sip`: pending
  - `sepc`, `scause`, `stval` (faulting address or instruction), `stvec`
  - The OS owns all of these, as my note says ("handled by OS").

#### DMA: Direct Memory Access (completing p.112)
- **Why:** for bulk transfers (disk, network, audio), even interrupts are too costly, since that would be one interrupt per word. A **DMA controller** moves the data between device and memory **without the CPU**.

**Three steps (P&H 6.6):**
1. **CPU sets up the DMA:** source address, destination address, length (byte count), direction/device.
2. **DMA becomes bus master:** it arbitrates for the bus and transfers the words, generating each next address itself. The CPU keeps running.
3. **DMA interrupts the CPU once at the end** (or on error). That is one interrupt per **block**, not per word.

**Transfer types:** I/O→memory, memory→I/O, memory→memory.

**Operating modes:**
- **Burst (block) mode:** the DMA holds the bus for the whole block. Fastest transfer, but the CPU **can't access memory/bus** until it finishes, so it waits if it misses in the cache.
- **Cycle stealing:** the DMA takes the bus for **one word at a time**, then gives it back. The CPU is slowed slightly, not stopped.
- **Transparent (hidden) mode:** the DMA uses the bus **only when the CPU isn't using it**. Zero CPU slowdown, but the slowest transfer.
- ⚠️ My p.112 says cycle stealing "works only when the CPU is idle". That describes **transparent mode**. Cycle stealing *takes* cycles from the CPU, hence the name.

**DMA + caches: the coherence (stale data) problem.** DMA talks to **memory directly**, bypassing the cache, so the two copies can disagree:
- **Device → memory (input):** if the CPU has those addresses cached, it reads the **old** cached copy.
- **Memory → device (output) with a write-back cache:** the newest data may be dirty in the cache and not yet in memory, so the DMA sends **stale** data.

**Solutions:**
- Put DMA buffers in **non-cacheable** memory.
- Software **flushes** (writes back) dirty lines before an output DMA and **invalidates** lines before reading input DMA data.
- Hardware **coherent DMA**: the DMA traffic is snooped by the cache.

**Also:** DMA contends with the CPU for memory bandwidth. Caches help, because most CPU accesses hit in the cache and leave memory free for the DMA.

**Spectrum (book):** polling → interrupt-driven → DMA → I/O processor. Each step moves more work from the CPU to an increasingly smart controller: fewer CPU cycles, but more hardware cost.

#### A / R / M classes (end of p.112)
ARM's processor profiles (Cortex-A, -R, -M); RISC-V cores are marketed the same way:
- **A, Application:** runs a full OS (Linux/Android), has an MMU, virtual memory and big caches. Phones, laptops.
- **R, Real-time:** deterministic timing; often uses **TCM/scratchpad** instead of relying on caches (links to L40). Disk controllers, automotive, modems.
- **M, Microcontroller:** tiny, low power, no MMU, often no cache, fast deterministic interrupts. Sensors, IoT. The **RED-V board is in this class.**

#### Lecture on 11/13 (not attended)
I can't tell what was covered from the files here: the tracker only says "didn't attend", and it is past the end of the topics overview. It most likely continued I/O or started virtual memory. **Ask a classmate or check the slides.**


---

## 13 Nov 2026: not attended
No notes. The topic is unknown; check the slides or ask a classmate.

---

# Formula sheet

```
#sets            = #blocks / associativity
offset bits      = log2(block size in bytes)
index bits       = log2(#sets)
tag bits         = address bits − index − offset
total cache bits = #blocks × (data bits + tag + valid [+ dirty])

AMAT             = Hit time + Miss rate × Miss penalty
AMAT (2-level)   = HitL1 + MRL1 × (HitL2 + MRL2_local × PenaltyMem)
Memory stalls/instr = I-miss rate × Penalty + (ld/sd fraction) × D-miss rate × Penalty
CPI_total        = CPI_base + memory stalls per instruction
CPU time         = IC × CPI_total × clock period
Global MR (L2)   = MR_L1 × local MR_L2
Penalty (cycles) = penalty (ns) / clock period (ns)

Memory organization, 4-word block (1 addr + 15 access + 1/word):
  1-word wide: 1 + 4·15 + 4·1 = 65   |   4-word wide: 1+15+1 = 17   |   4-bank interleaved: 1+15+4 = 20

Write-through without buffer: CPI = base + (store fraction × memory write time)
Speedup from perfect cache = CPI_stall / CPI_perfect
Multi-issue: IPC = 1 / CPI;  N-issue ideal CPI = 1/N
```
