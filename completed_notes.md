# CARCH — Completion Notes for the Gaps in My Handwritten Notes

**Source:** Patterson & Hennessy, *Computer Organization and Design*, 4th ed. (revised). That edition uses MIPS; I've rewritten every example in **RISC-V** to match the course. Section numbers like (P&H 5.3) point to the book.

**How to use this:** each section says which lecture/page of `handwrittnen class notes.pdf` it completes. Recaps of what I already wrote are kept short; the new material is the long part.

**Order:** post-Exam-1 material (Exam 2) comes first, then the older ISA-side gaps in Part B.

> ⚠️ = a correction to something in my handwritten notes.

---

## Contents

**Part A: Exam-2 material**
1. [L25: Control path, PCSrc (p.60–61)](#l25)
2. [L32–33: ILP, multi-issue, deeper and broader pipelines (p.82–85)](#l32)
3. [Pipeline: interrupts and exceptions](#pipe-exc)
4. [L34–36: Memory hierarchy and direct-mapped caches (p.86–96)](#l34)
5. [L37: Associativity, replacement, CAM (p.97–98)](#l37)
6. [L38: Write policies (p.99–100)](#l38)
7. [L39: 3 C's, trade-offs, block size, split/unified, cache performance (p.101–104)](#l39)
8. [L40: Multilevel caches, cache overhead, SPM, prefetching (p.105–108)](#l40)
9. [L41: I/O, MMIO, polling, interrupts, DMA (p.109–113)](#l41)
10. [Formula sheet](#formulas)

**Part B: Older gaps (pre-Exam-1)**

11. [L2: CARCH basics](#l2)
12. [L15: CPU time and performance](#l15)
13. [L17: Multiplication and division](#l17)
14. [L19: Floating-point arithmetic](#l19)
15. [L21: Precision/range question, custom multipliers worked out](#l21)

---

# PART A: Exam-2 material

<a id="l25"></a>
## 1. L25: Control path, finishing PCSrc (p.61, "PCSrc: KK will tell")

### Main control signals for the single-cycle RISC-V datapath (P&H 4.4)

| Instr | ALUSrc | MemtoReg | RegWrite | MemRead | MemWrite | Branch | ALUOp |
|---|---|---|---|---|---|---|---|
| R-type (add/sub/and/or) | 0 (reg) | 0 (ALU result) | 1 | 0 | 0 | 0 | 10 |
| `ld` | 1 (imm) | 1 (memory) | 1 | 1 | 0 | 0 | 00 |
| `sd` | 1 (imm) | X | 0 | 0 | 1 | 0 | 00 |
| `beq` | 0 (reg) | X | 0 | 0 | 0 | 1 | 01 |

X = don't care: nothing is written to a register, so the MemtoReg mux output doesn't matter.

**ALU control** (2-bit ALUOp from main control + funct7/funct3 → 4-bit ALU control):
- ALUOp `00` → **add** (address calculation for ld/sd)
- ALUOp `01` → **subtract** (beq compares by subtracting)
- ALUOp `10` → look at funct fields: add `0010`, sub `0110`, and `0000`, or `0001`

### PCSrc

```
PCSrc = Branch AND Zero
PCSrc = 0  →  PC ← PC + 4                       (next sequential instruction)
PCSrc = 1  →  PC ← PC + (sign-extended imm << 1) (branch target)
```

- The branch target is computed by a **separate adder**, not the main ALU, because the ALU is busy comparing the two registers.
- The immediate is shifted left by 1 because RISC-V branch offsets count half-words. That bit is implicit, which is why the B-format immediate has no bit 0.
- PCSrc is **not** set directly by the main control unit, since it depends on the comparison result. Control produces `Branch`, the ALU produces `Zero`, and an AND gate combines them.

**Answer to the yellow note "how to compare to 0?":** for `beq rs1, rs2`, the ALU computes `rs1 − rs2`. The ALU has a **Zero output**, which is 1 when every result bit is 0 (a big NOR of the result bits). Zero = 1 means rs1 == rs2.

In the pipelined CPU (L29/30), this same PCSrc mux is what gets moved earlier, into ID, to reduce the branch penalty (comparison shifting).

---

<a id="l32"></a>
## 2. L32–33: Instruction-level parallelism (p.82–85; L33 page is blank)

### Recap of what I wrote
- If `ld x10,0(x2)` misses in the cache, independent instructions behind it still wait in a simple pipeline.
- Idea: have **two pipelines** (multi-issue) so independent instructions go in parallel → **CPI < 1**. We then talk about **IPC** (instructions per cycle) = 1/CPI. Ideal IPC = 2 for 2-issue.
- Dual issue: if instruction i+1 depends on i, the second slot gets a NOP.
- The open question at the end: **"How to identify dependence and divide instructions into the pipes? Who does it?"**

### Answer: two ways to do multiple issue (P&H 4.10)

| | **Static multiple issue** | **Dynamic multiple issue** |
|---|---|---|
| Who decides what issues together | **Compiler** | **Hardware**, at run time |
| Also called | VLIW (Very Long Instruction Word) | **Superscalar** |
| Unit of issue | "Issue packet": one long instruction with N slots | Hardware picks 0..N instructions per cycle |
| Hazard handling | Compiler removes most hazards; NOPs fill empty slots | Hardware detects; often executes **out of order** |
| Pros | Simple, low-power hardware | Works on unmodified binaries; adapts to cache misses at run time |
| Cons | Code tied to the machine; code bloat from NOPs; can't adapt to unpredictable cache misses | Complex, power-hungry hardware |

### Static dual-issue RISC-V (book's design)

- **Slot 1:** ALU or branch instruction. **Slot 2:** load or store.
- Packets are 64-bit aligned: ALU/branch first, then ld/sd.
- Extra hardware: 2 more register-file read ports and 1 more write port, plus a separate adder for ld/sd address calculation.
- **Extra hazards:**
  - An ALU result can't be used by the ld/sd in the **same** packet.
  - Load-use latency is still 1 cycle, but that is now **2 instructions** (a whole packet) that can't use the loaded value.

### Worked example: scheduling a loop for dual issue

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

### Dynamic pipelining (superscalar, out-of-order)

The pipeline is split into three parts:

1. **In-order issue unit:** fetches, decodes and sends instructions to the functional units.
2. **Reservation stations** sit in front of each functional unit. They hold the operands, wait until all of them are ready, then execute. Execution is **out of order**.
3. **Commit unit / reorder buffer:** results are written back to registers and memory **in order**. This keeps exceptions precise and lets wrong speculation be thrown away.

- **Speculation:** guess the outcome (e.g. a branch, or that a load doesn't alias an earlier store) and execute ahead. If the guess was wrong, flush. The commit unit makes this safe.
- **Link to L40 notes** ("interactions with advanced CPUs"): during a cache miss, the pending load/store waits in the load/store unit while independent instructions keep executing. Dependent instructions wait in reservation stations.

### Deeper pipelines vs broader pipelines (lecture 32 title)

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

<a id="pipe-exc"></a>
## 3. Interrupts and exceptions in the pipeline (from the topics overview)

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

<a id="l34"></a>
## 4. L34–36: Memory hierarchy and direct-mapped caches (p.86–96)

### Recap of what I wrote
- Memory wall: memory takes ~50+ cycles.
- Locality: **temporal** (same item again soon, e.g. loops) and **spatial** (nearby items soon, e.g. arrays and sequential instructions).
- Hierarchy: registers → cache (SRAM) → main memory (DRAM) → disk.
- Technology table: SRAM 0.5–2.5 ns, $500–1000/GB; DRAM 50–70 ns, $3–6/GB; disk 5–20 ms, $0.01–0.02/GB. Ideal memory = SRAM speed with disk capacity and cost.

### Terms the slides skipped (P&H 5.1)
- **Block / line:** the minimum unit copied between two levels.
- **Hit:** the data is found in the upper level.
  - **Hit rate** = hits / accesses.
  - **Hit time** = time to access the upper level, including the time to decide hit or miss.
- **Miss:** the data must be fetched from the lower level.
  - **Miss rate** = 1 − hit rate.
  - **Miss penalty** = time to fetch the block from the lower level, place it in the cache and deliver the word.

### Direct-mapped placement

```
Cache index = (Block address) mod (Number of blocks in cache)
```

- Because the number of blocks is a power of 2, the mod is free: the index is just the **low log₂(#blocks) bits** of the block address.
- ⚠️ p.94 says "Block add (mod) Block size". It should be **mod number of blocks**. Block *size* is what the offset bits come from.

### Address breakdown

```
| ----- Tag ----- | --- Index --- | --- Block offset --- | Byte offset |
                    log2(#sets)     log2(words/block)     log2(bytes/word)
Tag bits = address bits − index bits − offset bits
```

- **Valid bit:** whether the entry holds real data. All entries start invalid.
- **Tag:** the high address bits, stored so we can tell **which** memory block sits in this entry. This is the "metadata" I wrote about.
- **Slide photo (p.92):** 64-bit address, 1024 entries, one 4-byte word per block. Byte offset = bits 1:0 (2 bits), index = bits 11:2 (10 bits), **tag = bits 63:12 (52 bits)**. Hit = valid AND (stored tag == address tag).

### Completing the direct-mapped trace (p.90–92)

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

### Field-size problems (completing p.94 Q2)

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

### Handling spatial locality: multi-word blocks
- Block address = ⌊byte address / bytes per block⌋.
- **Example:** 64 blocks of 16 B. Byte address 1200 → block 1200/16 = 75 → index 75 mod 64 = **11**.

**Answer to "Is fetching one larger block faster than several smaller blocks? Yes, why?"** All three costs are paid once for the whole block instead of once per word:
- the address is sent once,
- the DRAM row is opened once (the access latency), and
- the words then stream out back-to-back (burst).

You also store one tag instead of several.

### Main memory supporting caches (completing p.95)

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

<a id="l37"></a>
## 5. L37: Associativity (p.97–98: "associative cache example missed, did not listen from here")

### Recap
For 8 entries: 1-way (direct-mapped, 8 sets), 2-way (4 sets), 4-way (2 sets), 8-way (fully associative, 1 set). More ways means more tags compared in parallel, more tag bits and bigger comparators.

### Placement rule

```
Set index = (Block address) mod (Number of sets)
#sets = #blocks / associativity
```

- The block can go in **any way** within its set.
- On lookup, compare the tags of **all ways of that set in parallel** (one comparator per way). The matching way drives a mux.
- That is the photo on p.97: a 4-way cache with 256 sets → 8 index bits, 22-bit tag, 4 comparators and a 4-to-1 mux.
- **Answer to "Do we search sequentially?" (p.98):** No. The index selects the set, then all ways' tags are compared **simultaneously**.

### The missed example (P&H 5.3): 4 one-word blocks, block addresses 0, 8, 0, 6, 8

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

### Tag-size cost of associativity (P&H)

4096 blocks, 4-word blocks (offset 4 bits), 32-bit address:

| Associativity | Sets | Index bits | Tag bits | Total tag bits |
|---|---|---|---|---|
| Direct-mapped | 4096 | 12 | 16 | 16 × 4096 = 64 K |
| 2-way | 2048 | 11 | 17 | 17 × 4096 = 68 K |
| 4-way | 1024 | 10 | 18 | 18 × 4096 = 72 K |
| Fully assoc. | 1 | 0 | 28 | 28 × 4096 = 112 K |

Each **doubling of associativity halves the number of sets**, which removes 1 index bit and adds 1 tag bit.

### Replacement policies (which block in the set to evict)
- **LRU (Least Recently Used):** evict the block unused for the longest time.
  - Exploits temporal locality.
  - 2-way needs just 1 bit per set.
  - True LRU gets expensive beyond 4 ways, so real CPUs use **pseudo-LRU**.
- **Random:** simple. At high associativity it is nearly as good as LRU.
- **FIFO:** evict the oldest-inserted block.
- Direct-mapped caches need no policy: there is only one choice.

### CAM: Content Addressable Memory (heading on p.98)
- A normal RAM takes an **address** and returns data.
- A **CAM** takes **data (a tag)** and returns *where* it matched (or hit/miss). The comparator is built into every entry, so all entries are searched in one cycle.
- **Used for:** highly associative caches (the book notes 8-way and above often use CAM for tags), fully associative structures like the **TLB**, and network routing tables.
- **Cost:** more area and power per bit than SRAM.

---

<a id="l38"></a>
## 6. L38: Write policies (completing p.99, which stops at "write back: dirty block")

### Recap
- **Write-through:** on a store, update both the cache **and** memory.
  - Always consistent, but every write goes to slow memory.
  - A **write buffer** lets the CPU continue.
  - It doesn't help when writes come continuously (the buffer fills → stall).

**Why write-through alone is bad (book number):** 10% of instructions are stores, base CPI = 1 and each memory write takes 100 cycles. Then CPI = 1 + 0.1 × 100 = **11**, more than 10× slower. Hence the write buffer.
- If the buffer is full, the CPU stalls.
- Bursts of writes cause stalls even when the average rate is fine, so real buffers have **several entries**.

### Write-back (copy-back)
- On a store **hit**, write **only the cache** block and set its **dirty bit = 1**.
- Memory is updated **only when the dirty block is evicted** (replaced).
- **Clean** blocks (dirty = 0) can simply be overwritten on eviction, with no write to memory.
- **Pros:** many writes to the same block cost one memory write. Uses much less memory bandwidth, which is good when the CPU generates writes faster than memory can absorb them.
- **Cons:**
  - Memory is temporarily stale, which causes the I/O coherence problem (see L41).
  - A miss that evicts a dirty block costs **two** transfers: write the old block back, then read the new one. A **write-back buffer** hides this: copy the dirty block to the buffer, read the new block, then write the buffer to memory.
  - More complex to implement.
- Stores need care: you can't write before you know it's a hit, or you'd overwrite another block's data. Either check the tag first (two cycles), or use a **store buffer**.

### Write misses (store to a block not in cache)

| Policy | What happens | Usually paired with |
|---|---|---|
| **Write-allocate** (fetch on write) | Fetch the block into the cache, then write it (as a hit) | **Write-back** |
| **No-write-allocate** (write-around) | Write straight to memory; don't bring the block into cache | **Write-through** |

**Motivation for no-write-allocate:** programs sometimes write whole blocks they won't read soon. A classic case is the OS zeroing a page; caching those blocks would just pollute the cache.

### Summary table

| | Write-through | Write-back |
|---|---|---|
| Memory up to date? | Always | Only after eviction |
| Extra state bit | None (valid only) | **Dirty bit** |
| Memory traffic | One memory write per store | One write per dirty eviction |
| Read miss cost | Read only | May be write-back **+** read |
| Needs | Write buffer | Write-back buffer (optional) |
| Typical level | Some L1s, simple embedded caches | L2/L3, most modern L1 D-caches |

---

<a id="l39"></a>
## 7. L39: Sources of misses, trade-offs, block size, unified/split, performance (completing p.101–104)

### Recap (already well covered in my notes)
- Important terms list.
- **3 C's:**
  - **compulsory** (cold start): fix with bigger blocks or prefetching
  - **capacity**: fix with a bigger cache
  - **conflict** (collision): fix with more associativity
- They are not independent.
- The trade-off table:
  - a bigger cache may increase access time;
  - more associativity may increase access time;
  - bigger blocks increase the miss penalty, and very large blocks increase the miss rate (pollution).

### Completing "Block size considerations: But in a fixed-sized cache ..." (p.103)
- **Larger blocks reduce the miss rate** at first, thanks to spatial locality.
- **But in a fixed-size cache**, larger blocks mean **fewer blocks**. That causes:
  1. More competition for the fewer slots → more conflict/capacity misses. The miss rate **eventually goes back up**.
  2. A block may be evicted before most of its words are used (**cache pollution**).
  3. The **miss penalty grows**, since more words must be transferred. The latency part stays the same; the transfer part grows with block size.
- So miss rate vs. block size is **U-shaped**: the best block size depends on the cache size, and for a small cache, very large blocks hurt.
- Fixes for the penalty: early restart and critical word first (see §4).

### Unified vs split (recap and additions)
- **Split** (separate I-cache and D-cache) is used at **L1** because:
  - The pipeline fetches an instruction **and** accesses data **in the same cycle** (IF and MEM stages). One unified cache would be a **structural hazard**; split caches double the bandwidth.
  - Each can be tuned separately: different size, associativity, block size, and read-only I-cache.
- **Unified** is used at **L2/L3**: one pool, so the space is shared dynamically and the overall hit rate is slightly higher for the same total size.

### Measuring cache performance (cleaning up p.104)

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

### Book example: calculating cache performance
I-cache miss rate 2%, D-cache miss rate 4%, base CPI = 2, miss penalty = 100 cycles, loads and stores = 36% of instructions.
- I-miss cycles = I × 0.02 × 100 = **2.00 I**
- D-miss cycles = I × 0.36 × 0.04 × 100 = **1.44 I**
- CPI with stalls = 2 + 2 + 1.44 = **5.44**
- A perfect cache would be 5.44 / 2 = **2.72× faster**.

**Follow-up (Amdahl's law):** improve the pipeline so base CPI = 1 with the same memory.
- CPI = 1 + 3.44 = 4.44, so the perfect cache is now **4.44×** faster.
- The fraction of time spent stalling rises from 3.44/5.44 = **63%** to 3.44/4.44 = **77%**.
- A faster CPU makes memory **more** important. The same happens if you raise the clock rate: the miss penalty in cycles grows.

### Book example: AMAT
Clock 1 ns, miss penalty 20 cycles, miss rate 0.05, hit time 1 cycle.
- AMAT = 1 + 0.05 × 20 = **2 cycles = 2 ns**

---

<a id="l40"></a>
## 8. L40: Multilevel caches, overhead, SPM, prefetching (completing p.105–108)

### Multilevel caches
- **L1:** small and fast, split I/D. Designed to **minimize hit time** (so it fits in the cycle).
- **L2:** bigger, unified. Designed to **minimize the miss rate to memory**, so it uses higher associativity and larger blocks. Its hit time matters less.
- **L3 / LLC (Last-Level Cache):** largest, usually **shared** among all cores.
- Because L2 exists, L1 can be **smaller** than it would be in a single-level design, and L2 can be larger and slower.
- My note mentions an ARM vs Intel comparison across L1/L2/L3. The typical pattern is the same on both: per-core L1 I/D of ~32–64 KB, L2 of ~256 KB–2 MB (per core or per cluster), and a shared LLC of several MB.

**Local vs global miss rate:**
- **Local miss rate** of L2 = L2 misses / **L2 accesses** (only the L1 misses reach L2). It is usually high, because L1 has already taken the easy hits.
- **Global miss rate** = misses / **all CPU accesses** = L1 miss rate × L2 local miss rate.

### Book example: performance of multilevel caches
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

### Overhead of caches and scratchpad memory (recap; my notes cover this well)
- **Costs of caches:** area, power, and unpredictable timing (the same program takes a different time on each run).
- **Real-time systems** budget for the **worst-case execution time (WCET)**, where every access is assumed to miss. If you budget for the worst case anyway, the cache's average-case benefit is wasted.
- **Scratchpad memory (SPM):** a small on-chip SRAM mapped at fixed addresses and managed by software (programmer or compiler, which copies data in and out, often via DMA).
  - Deterministic timing.
  - No tags or comparators, so less area and power.
  - ARM Cortex-R parts call this **TCM (Tightly Coupled Memory)**.
- **RED-V board:** data side = 16 KB SPM (a DTIM); instruction side = 2-way 16 KB I-cache.

### Interaction with advanced (out-of-order) CPUs
- **Nonblocking (lockup-free) caches** keep serving the CPU during a miss:
  - **Hit under miss:** later hits are served while a miss is outstanding.
  - **Miss under miss:** several misses are outstanding at once, overlapping their penalties.
- Together with OoO execution (pending store in the LSU, dependents waiting in reservation stations), much of the miss latency is **hidden**. This makes simple stall formulas pessimistic for OoO CPUs.

### Prefetching (completing p.107)
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

<a id="l41"></a>
## 9. L41: I/O, MMIO, polling, interrupts, DMA (completing p.109–113)

### Recap
- CPU ↔ memory, with I/O hanging off a **bridge**.
- Devices are controlled through **device registers**:
  - **control** (configuration),
  - **status** (ready/busy/error),
  - **data** (the actual byte, e.g. a keystroke).

### Two ways to reach device registers
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

### The "problem with MMIO" note (p.109–110): caching
- If an MMIO address were cached:
  - the CPU would keep reading the **stale cached copy** of the status register and never see the device change;
  - writes might sit in a write-back cache and never reach the device.
- **Fix:** MMIO regions are marked **non-cacheable** (and strongly ordered). On RISC-V this is set by the platform's **PMAs (Physical Memory Attributes)**. In C, use `volatile` so the compiler doesn't optimize the loads away.
- **Answer to "Is our system's I/O cacheable?":** No. Device regions must be uncached. Normal RAM is cached.

### Polling vs interrupts
**Polling:** the CPU repeatedly reads the status register.
- **Pros:** simple, with predictable timing, which is why hard real-time embedded systems use it.
- **Cons:** wastes CPU time on ld/sd and adds bus traffic, especially because I/O is slow.

**Polling overhead (illustrative, in the style of the book's example):** 1 GHz CPU, 400 cycles per poll.
- **Mouse** polled 30×/s → 12,000 cycles/s ≈ **0.001%** of the CPU. Fine.
- **Disk** at 4 MB/s, read in 16-B chunks → 250,000 polls/s × 400 = 10⁸ cycles/s = **10%** of the CPU. And you must poll all the time, even when the disk is idle. Bad, so use interrupts or DMA.

**Interrupt-driven I/O:** the device raises a signal on an **interrupt line** and the CPU runs an **ISR** (Interrupt Service Routine) only when needed.
- Differences from an exception: an interrupt is **asynchronous** (not tied to any instruction), so the CPU only checks for it between instructions. It must also identify **which device** raised it, and devices have **priorities**.

### Identifying the source
- **Non-vectored (polled) interrupts:** one shared line. The handler reads a cause register (or polls each device) to find the source.
- **Vectored interrupts:** each source has its own number/vector, and hardware jumps **directly** to that source's handler. This saves the search (my note: "everyone has a unique index").
- **On RISC-V:** `stvec`/`mtvec` hold the trap base address.
  - MODE = 0 (direct): all traps go to BASE.
  - MODE = 1 (**vectored**): interrupts jump to **BASE + 4 × cause**.
  - Exceptions always go to BASE.

### Interrupt service flow (cleaned up from p.111)
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

### DMA: Direct Memory Access (completing p.112)
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

### A / R / M classes (end of p.112)
ARM's processor profiles (Cortex-A, -R, -M); RISC-V cores are marketed the same way:
- **A, Application:** runs a full OS (Linux/Android), has an MMU, virtual memory and big caches. Phones, laptops.
- **R, Real-time:** deterministic timing; often uses **TCM/scratchpad** instead of relying on caches (links to L40). Disk controllers, automotive, modems.
- **M, Microcontroller:** tiny, low power, no MMU, often no cache, fast deterministic interrupts. Sensors, IoT. The **RED-V board is in this class.**

### Lecture on 11/13 (not attended)
I can't tell what was covered from the files here: the tracker only says "didn't attend", and it is past the end of the topics overview. It most likely continued I/O or started virtual memory. **Ask a classmate or check the slides.**

---

<a id="formulas"></a>
## 10. Formula sheet (Part A)

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

---

# PART B: Older gaps (pre-Exam-1)

<a id="l2"></a>
## 11. L2: CARCH basics (p.1: "did not have a pen")

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

<a id="l15"></a>
## 12. L15: CPU time and performance (p.32: "CPU execution time & stuff")

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

<a id="l17"></a>
## 13. L17: Multiplication and division (p.35–36, partly blank)

### Multiplication

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

### Division ("refer to slides" on my page)

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

<a id="l19"></a>
## 14. L19: Floating-point arithmetic (p.40: "KK did not write")

### IEEE 754 recap

```
value = (−1)^S × (1 + Fraction) × 2^(Exponent − Bias)
Single (FP32): 1 | 8  | 23   bias 127      Double (FP64): 1 | 11 | 52   bias 1023
```

| Exponent field | Fraction = 0 | Fraction ≠ 0 |
|---|---|---|
| 0 | ±0 | denormal: (−1)^S × 0.F × 2^(1−bias) |
| all 1s | ±∞ | NaN |

### Converting decimal → FP32: −0.75
- −0.75 = −0.11₂ = −1.1₂ × 2⁻¹
- S = 1. Exponent = −1 + 127 = 126 = 0111 1110. Fraction = 1000…0
- Bits: `1 01111110 10000000000000000000000` = **0xBF400000**

### Converting FP32 → decimal: `1 10000001 01000…0`
- S = 1. Exponent = 129 − 127 = 2. Significand = 1.01₂ = 1.25
- Value = −1.25 × 2² = **−5.0**

### FP addition steps (example: 0.5 + (−0.4375), keeping 4 bits of precision)
- 0.5 = 1.000₂ × 2⁻¹ and −0.4375 = −1.110₂ × 2⁻²

1. **Align:** shift the smaller number right until the exponents match. −1.110 × 2⁻² → −0.111 × 2⁻¹
2. **Add the significands:** 1.000 − 0.111 = 0.001 × 2⁻¹
3. **Normalize:** 1.000 × 2⁻⁴, and check for overflow/underflow of the exponent.
4. **Round** to the available bits; renormalize if rounding carries out.
- Result = 2⁻⁴ = **0.0625** ✓

### FP multiplication steps
1. **Add the exponents** and subtract the bias once: (E₁ + E₂ − bias).
2. **Multiply the significands.**
3. **Normalize** and check for overflow/underflow.
4. **Round.**
5. **Sign** = S₁ XOR S₂.

### Accurate arithmetic
- Hardware keeps 2 extra bits during operations: **guard** and **round**. A **sticky** bit records whether any 1s were shifted out beyond them.
- This makes "round to nearest even" correct.
- Rounding modes: nearest-even (default), toward 0, toward +∞, toward −∞.
- **Accuracy is measured in ulps** (units in the last place). IEEE guarantees results within ½ ulp.

### FP is not associative
- Example: (−1.5×10³⁸ + 1.5×10³⁸) + 1.0 = **1.0**
- But −1.5×10³⁸ + (1.5×10³⁸ + 1.0) = **0.0**, because 1.0 is lost when rounding against a huge number.
- So a compiler can't freely reorder FP sums (this matters for parallel reductions).

### RISC-V F/D instructions (L20 recap)
- **Separate register file:** f0–f31.
- Loads/stores: `flw`/`fsw`, `fld`/`fsd`.
- Arithmetic: `fadd.s/d`, `fsub`, `fmul`, `fdiv`, `fsqrt`, and fused multiply-add `fmadd`.
- Compares: `feq`, `flt`, `fle` write **0/1 to an integer register**; then use `beq`/`bne`.
- Conversions: `fcvt.*` between int and FP; `fmv.x.d` moves raw bits.

<a id="l21"></a>
## 15. L21: The red question "find the precision and range of all these", plus custom multipliers worked out

### Formats

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

### Mitchell's algorithm (approximate multiplication via logs)
- Write A = 2^k × (1 + m), with 0 ≤ m < 1. Then log₂A ≈ **k + m**, using the approximation log₂(1+m) ≈ m.
- **k comes from the LOD** (leading-one detector: the position of the first 1 bit). m is just the bits after the leading 1, so no table is needed.
- Add the two approximate logs, then take the antilog the same way (2^(k+m) ≈ 2^k × (1+m)).

**Example 322 × 121 (from my notes):**
- 322 = 2⁸ × 1.2578 → log ≈ 8.2578
- 121 = 2⁶ × 1.8906 → log ≈ 6.8906
- Sum = 15.1484 → 2¹⁵ × 1.1484 ≈ **37,631**
- Exact = 38,962 → **−3.4% error**
- Mitchell always **underestimates**; its worst case is about −11%.

### ROBA (Rounding-Based Approximate) multiplier

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

---
*End of completion notes. Corrections are marked with ⚠️. One remaining gap: the 11/13 lecture, which I didn't attend and can't reconstruct from these files.*
