---
title: C Atomic Instruction Mapping
date: 2021-07-06
tags:
    - system
---


## Note

- Only 64 bits data types
- Compiler: clang 11.0.1
- [https://godbolt.org/z/Te1zeva8M](https://godbolt.org/z/Te1zeva8M)


## Mapping Table


| Operation            | x86-64                                                          | armv8-a                                                                                                                                                                | RISC-V rv64gc                                                                                             |
|----------------------|-----------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------|
| Load (no atomic)     | <code>mov rax, qword ptr [rip + y]</code>                       | <code>adrp x8, y</br>ldr x0, [x8, :lo12:y]</code>                                                                                                                      | <code>lui a0, %hi(y)<br/>ld a0, %lo(y)(a0)                                                        </code> |
| Load (implicit)      | <code>mov rax, qword ptr [rip + x]</code>                       | <code>adrp x8, x</br>add x8, x8, :lo12:x</br>ldar x0, [x8]</code>                                                                                                      | <code>fence rw, rw<br/>lui a0, %hi(x)<br/>ld a0, %lo(x)(a0)<br/>fence r, rw</code>                        |
| Load (relaxed)       | <code>mov rax, qword ptr [rip + x]</code>                       | <code>adrp x8, x</br>ldr x0, [x8, :lo12:x]</code>                                                                                                                      | <code>lui a0, %hi(x)<br/>ld a0, %lo(x)(a0)</code>                                                         |
| Load (consume)       | <code>mov rax, qword ptr [rip + x]</code>                       | <code>adrp x8, x</br>add x8, x8, :lo12:x</br>ldar x0, [x8]</code>                                                                                                      | <code>lui a0, %hi(x)<br/>ld a0, %lo(x)(a0)<br/>fence r, rw</code>                                         |
| Load (acquire)       | <code>mov rax, qword ptr [rip + x]</code>                       | <code>adrp x8, x</br>add x8, x8, :lo12:x</br>ldar x0, [x8]</code>                                                                                                      | <code>lui a0, %hi(x)<br/>ld a0, %lo(x)(a0)<br/>fence r, rw</code>                                         |
| Load (release)       | <code>Watomic-memory-ordering</code>                            | <code>Watomic-memory-ordering</code>                                                                                                                                   | <code>Watomic-memory-ordering</code>                                                                      |
| Load (acq_rel)       | <code>Watomic-memory-ordering</code>                            | <code>Watomic-memory-ordering</code>                                                                                                                                   | <code>Watomic-memory-ordering</code>                                                                      |
| Load (seq_cst)       | <code>mov rax, qword ptr [rip + x]</code>                       | <code>adrp x8, x</br>add x8, x8, :lo12:x</br>ldar x0, [x8]</code>                                                                                                      | <code>fence rw, rw<br/>lui a0, %hi(x)<br/>ld a0, %lo(x)(a0)<br/>fence r, rw</code>                        |
| Store (no atomic)    | <code>mov qword ptr [rip + y], rdi</code>                       | <code>adrp x8, y</br>str x0, [x8, :lo12:y]</code>                                                                                                                      | <code>lui a1, %hi(y)<br/>sd a0, %lo(y)(a1)</code>                                                         |
| Store (implicit)     | <code>xchg qword ptr [rip + x], rdi</code>                      | <code>adrp x8, x</br>add x8, x8, :lo12:x</br>stlr x0, [x8]</code>                                                                                                      | <code>fence rw, w<br/>lui a1, %hi(x)<br/>sd a0, %lo(x)(a1)</code>                                         |
| Store (relaxed)      | <code>mov qword ptr [rip + x], rdi</code>                       | <code>adrp x8, x</br>str x0, [x8, :lo12:x]</code>                                                                                                                      | <code>lui a1, %hi(x)<br/>sd a0, %lo(x)(a1)</code>                                                         |
| Store (consume)      | <code>Watomic-memory-ordering</code>                            | <code>Watomic-memory-ordering</code>                                                                                                                                   | <code>Watomic-memory-ordering</code>                                                                      |
| Store (acquire)      | <code>Watomic-memory-ordering</code>                            | <code>Watomic-memory-ordering</code>                                                                                                                                   | <code>Watomic-memory-ordering</code>                                                                      |
| Store (release)      | <code>mov qword ptr [rip + x], rdi</code>                       | <code>adrp x8, x</br>add x8, x8, :lo12:x</br>stlr x0, [x8]</code>                                                                                                      | <code>fence rw, w<br/>lui a1, %hi(x)<br/>sd a0, %lo(x)(a1)</code>                                         |
| Store (acq_rel)      | <code>Watomic-memory-ordering</code>                            | <code>Watomic-memory-ordering</code>                                                                                                                                   | <code>Watomic-memory-ordering</code>                                                                      |
| Store (seq_cst)      | <code>xchg qword ptr [rip + x], rdi</code>                      | <code>adrp x8, x</br>add x8, x8, :lo12:x</br>stlr x0, [x8]</code>                                                                                                      | <code>fence rw, w<br/>lui a1, %hi(x)<br/>sd a0, %lo(x)(a1)</code>                                         |
| Exchange (implicit)  | <code>mov eax, 42<br/>xchg qword ptr [rip + x], rax</code>      | <code>             adrp x8, x<br/>  add x8, x8, :lo12:x<br/>  mov w9, #42<br/>.LBB16_1:<br/>  ldaxr x0, [x8]<br/>  stlxr w10, x9, [x8]<br/>  cbnz w10, .LBB16_1</code> | <code>lui a0, %hi(x)<br/>addi a0, a0, %lo(x)<br/>addi a1, zero, 42<br/>amoswap.d.aqrl a0, a1, (a0)</code> |
| Exchange (relaxed)   | <code>mov eax, 42<br/>xchg qword ptr [rip + x], rax</code>      | <code>  adrp x8, x<br/>  add x8, x8, :lo12:x<br/>  mov w9, #42<br/>.LBB17_1:<br/>  ldxr x0, [x8]<br/>  stxr w10, x9, [x8]<br/>  cbnz w10, .LBB17_1</code>              | <code>lui a0, %hi(x)<br/>addi a0, a0, %lo(x)<br/>addi a1, zero, 42<br/>amoswap.d a0, a1, (a0)</code>      |
| Exchange (consume)   | <code>mov eax, 42<br/>xchg qword ptr [rip + x], rax</code>      | <code>  adrp x8, x<br/>  add x8, x8, :lo12:x<br/>  mov w9, #42<br/>.LBB18_1:<br/>  ldaxr x0, [x8]<br/>  stxr w10, x9, [x8]<br/>  cbnz w10, .LBB18_1</code>             | <code>lui a0, %hi(x)<br/>addi a0, a0, %lo(x)<br/>addi a1, zero, 42<br/>amoswap.d.aq a0, a1, (a0)</code>   |
| Exchange (acquire)   | <code>mov eax, 42<br/>xchg qword ptr [rip + x], rax</code>      | <code>  adrp x8, x<br/>  add x8, x8, :lo12:x<br/>  mov w9, #42<br/>.LBB19_1:<br/>  ldaxr x0, [x8]<br/>  stxr w10, x9, [x8]<br/>  cbnz w10, .LBB19_1</code>             | <code>lui a0, %hi(x)<br/>addi a0, a0, %lo(x)<br/>addi a1, zero, 42<br/>amoswap.d.aq a0, a1, (a0)</code>   |
| Exchange (release)   | <code>mov eax, 42<br/>xchg qword ptr [rip + x], rax</code>      | <code>  adrp x8, x<br/>  add x8, x8, :lo12:x<br/>  mov w9, #42<br/>.LBB20_1:<br/>  ldxr x0, [x8]<br/>  stlxr w10, x9, [x8]<br/>  cbnz w10, .LBB20_1</code>             | <code>lui a0, %hi(x)<br/>addi a0, a0, %lo(x)<br/>addi a1, zero, 42<br/>amoswap.d.rl a0, a1, (a0)</code>   |
| Exchange (acq_rel)   | <code>mov eax, 42<br/>xchg qword ptr [rip + x], rax</code>      | <code>  adrp x8, x<br/>  add x8, x8, :lo12:x<br/>  mov w9, #42<br/>.LBB21_1:<br/>  ldaxr x0, [x8]<br/>  stlxr w10, x9, [x8]<br/>  cbnz w10, .LBB21_1</code>            | <code>lui a0, %hi(x)<br/>addi a0, a0, %lo(x)<br/>addi a1, zero, 42<br/>amoswap.d.aqrl a0, a1, (a0)</code> |
| Exchange (seq_cst)   | <code>mov eax, 42<br/>xchg qword ptr [rip + x], rax</code>      | <code>  adrp x8, x<br/>  add x8, x8, :lo12:x<br/>  mov w9, #42<br/>.LBB22_1:<br/>  ldaxr x0, [x8]<br/>  stlxr w10, x9, [x8]<br/>  cbnz w10, .LBB22_1</code>            | <code>lui a0, %hi(x)<br/>addi a0, a0, %lo(x)<br/>addi a1, zero, 42<br/>amoswap.d.aqrl a0, a1, (a0)</code> |
| Fetch Add (implicit) | <code>mov eax, 42<br/>lock xadd qword ptr [rip + x], rax</code> | <code>  adrp x8, x<br/>  add x8, x8, :lo12:x<br/>.LBB23_1:<br/>  ldaxr x0, [x8]<br/>  add x9, x0, #42<br/>  stlxr w10, x9, [x8]<br/>  cbnz w10, .LBB23_1</code>        | <code>lui a0, %hi(x)<br/>addi a0, a0, %lo(x)<br/>addi a1, zero, 42<br/>amoadd.d.aqrl a0, a1, (a0)</code>  |
| Fetch Add (relaxed)  | <code>mov eax, 42<br/>lock xadd qword ptr [rip + x], rax</code> | <code>  adrp x8, x<br/>  add x8, x8, :lo12:x<br/>.LBB24_1:<br/>  ldxr x0, [x8]<br/>  add x9, x0, #42<br/>  stxr w10, x9, [x8]<br/>  cbnz w10, .LBB24_1</code>          | <code>lui a0, %hi(x)<br/>addi a0, a0, %lo(x)<br/>addi a1, zero, 42<br/>amoadd.d a0, a1, (a0)</code>       |
| Fetch Add (consume)  | <code>mov eax, 42<br/>lock xadd qword ptr [rip + x], rax</code> | <code>  adrp x8, x<br/>  add x8, x8, :lo12:x<br/>.LBB25_1:<br/>  ldaxr x0, [x8]<br/>  add x9, x0, #42<br/>  stxr w10, x9, [x8]<br/>  cbnz w10, .LBB25_1</code>         | <code>lui a0, %hi(x)<br/>addi a0, a0, %lo(x)<br/>addi a1, zero, 42<br/>amoadd.d.aq a0, a1, (a0)</code>    |
| Fetch Add (acquire)  | <code>mov eax, 42<br/>lock xadd qword ptr [rip + x], rax</code> | <code>  adrp x8, x<br/>  add x8, x8, :lo12:x<br/>.LBB26_1:<br/>  ldaxr x0, [x8]<br/>  add x9, x0, #42<br/>  stxr w10, x9, [x8]<br/>  cbnz w10, .LBB26_1</code>         | <code>lui a0, %hi(x)<br/>addi a0, a0, %lo(x)<br/>addi a1, zero, 42<br/>amoadd.d.aq a0, a1, (a0)</code>    |
| Fetch Add (release)  | <code>mov eax, 42<br/>lock xadd qword ptr [rip + x], rax</code> | <code>  adrp x8, x<br/>  add x8, x8, :lo12:x<br/>.LBB27_1:<br/>  ldxr x0, [x8]<br/>  add x9, x0, #42<br/>  stlxr w10, x9, [x8]<br/>  cbnz w10, .LBB27_1</code>         | <code>lui a0, %hi(x)<br/>addi a0, a0, %lo(x)<br/>addi a1, zero, 42<br/>amoadd.d.rl a0, a1, (a0)</code>    |
| Fetch Add (acq_rel)  | <code>mov eax, 42<br/>lock xadd qword ptr [rip + x], rax</code> | <code>  adrp x8, x<br/>  add x8, x8, :lo12:x<br/>.LBB28_1:<br/>  ldaxr x0, [x8]<br/>  add x9, x0, #42<br/>  stlxr w10, x9, [x8]<br/>  cbnz w10, .LBB28_1</code>        | <code>lui a0, %hi(x)<br/>addi a0, a0, %lo(x)<br/>addi a1, zero, 42<br/>amoadd.d.aqrl a0, a1, (a0)</code>  |
| Fetch Add (seq_cst)  | <code>mov eax, 42<br/>lock xadd qword ptr [rip + x], rax</code> | <code>  adrp x8, x<br/>  add x8, x8, :lo12:x<br/>.LBB29_1:<br/>  ldaxr x0, [x8]<br/>  add x9, x0, #42<br/>  stlxr w10, x9, [x8]<br/>  cbnz w10, .LBB29_1</code>        | <code>lui a0, %hi(x)<br/>addi a0, a0, %lo(x)<br/>addi a1, zero, 42<br/>amoadd.d.aqrl a0, a1, (a0)</code>  |
