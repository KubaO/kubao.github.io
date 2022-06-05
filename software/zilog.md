---
layout: default
title: eZ8 and Z80 Stuff
parent: Software
permalink: /software/zilog
---
{% include switcher.html %}

# Zilog Z8/Z80 Stuff
{: .no_toc }

I know entirely too much about eZ8. 'twas two decades of fun!
{: .fs-6 .fw-300 }

{% include toc.html %}

---
## Fast CRC-16 on Zilog Z8 and Z80
*February 19, 2012*

I wondered how many instructions does it really take to update the value of a 16 bit CRC. That's given an input character, and our taget are Zilog's Z8/eZ8 and also Z80 architectures. In spite of lots of *Z*'s and *8*'s, the Z8 and Z80 have nothing really in common, other than the Zilog brand.

- eZ8 is a pipelined speed-up of Z8 Harvard architecture. It has a fairly orthogonal 2-operand instruction set that works directly on RAM operands (they call them "registers" but it's just banked RAM). It's not hard to learn those opcodes - there's just a few, and most of them support one of three sets of addressing modes.

- Z80 is a CISC with an instruction set designed by Intel for their 8080/MCS-80 product line, with Zilog's extensions that improve performance - sometimes by quite a bit vs. 8080.

But, back to CRC-16. It turns out, it's only 6 instructions on Z8, and 8 fast instructions on Z80. If you want to preserve the input character, there's an extra load involved (not shown).

```nasm
; Z8/eZ8/Encore! architecture
; _crc_table - crc table, 512 bytes long, see text
; RR12 (in/out) - crc to be updated
; R1 (in/out) - input character, clobbered
; R0 (out) - clobbered

    ld   r0, #high(_crc_table) ; table is aligned to 256 bytes
    xor  r1, r13      ; RR0 = crc_table + (data ^ crc.l)

    ldc  r13, @rr0    ; crc.l = crc_table[].l
    xor  r13, r12     ; crc.l = crc.h ^ crc_table[].l
    inc  r0
    ldc  r12, @rr0    ; crc.h = crc_table[].h
```

```nasm
; Z80 architecture
; _crc_table - crc table, 512 bytes long, see text
; BC (in/out) - crc to be updated
; A (in/out) - input character, clobbered
; HL (out) - clobbered

    ld  H, high(_crc_table)
    xor C
    ld  L, A        ; HL = crc_table + (data ^ crc.l)

    ld  A, B        ; A = crc.h
    xor (HL)        ;     crc.h ^ crc_table[].l
    ld  C, A        ; crc.l = crc.h ^ crc_table[].l
    inc H
    ld  B, (HL)     ; crc.h = crc_table[].h
```

The precomputed CRC table is generated using Rocksoft™ Model CRC Algorithm Table Generation, with 2 byte width, and reverse set to true. The table must be aligned at the 256 byte boundary -- the LSB of its first byte must be equal to 0. On Z8, the table is stored in the code memory. The table as used by the assembly routine must be reordered first.

The bytes in the table are reordered, such that each word element of the table is split, and LSBs are grouped in the first 256 bytes of the table, followed by MSBs in the following 256 bytes of the table. This allows the entire calculation of the address for table lookup to take just a couple instructions - this trick seems to speed things up on both Z8 and Z80.

A C program reordering the table as described above is shown below.

```c
// big endian version (Z8/eZ8/Encore!)
void reorder_table_Z8(const unsigned char input_table[512],
                      unsigned char output_table[512])
{
  for (int i = 0; i < 255; ++i) {
    output_table[i] = input_table[i*2 + 1]; // LSB
    output_table[256+i] = input_table[i*2 + 0]; // MSB
  }
}

// little endian version (Z80)
void reorder_table_Z80(const unsigned char input_table[512],
                      unsigned char output_table[512])
{
  for (int i = 0; i < 255; ++i) {
    output_table[i] = input_table[i*2 + 0]; // LSB
    output_table[256+i] = input_table[i*2 + 1]; // MSB
  }
}
```

---
## Pascal-Style Local Functions in C (for eZ8 Encore!)
*circa 2009*

When I was a kid, I used Turbo Pascal. It had one feature that I sorely miss in every embedded C compiler: local functions. Local, as in local to a block. Those become a necessity to maintain decent code performance and avoid the penalty of passing duplicate data via arguments. They also help maintain readable, decently factorized code.

A local function, were C to have it, would look like this:

```c
void fun1(void) {
 int a;
 void fun2(void) {
   a = 0;
 }
}
```

I hope you see that `fun2()` is local to the main block of `fun1()`, and that the variable a is in scope within `fun2()`.

A somewhat less clean, but equally well performing way of accomplishing this would be:

```c
extern int a;
void fun2(void) {
  a = 0;
}
void fun1(void) {
...
  fun2();
}
```

Now the question remains: where do we actually define the variable a, which is supposed to be local to fun1?

It becomes easy if your compiler supports static frames. Static frames are, as their name implies, allocated statically by the linker, and are overlaid according to the call tree. With usual dynamic frames, automatic variables end up on the stack. It should not be any harder to do with dynamic frames, but I haven't checked it out yet.

ZDS II, the C IDE for Zilog's Z8 Encore! and ZNEO products, supports static frames at the assembler and linker level. A frame ends up defining a near and far segment; for a function called fun1 those segments are called ?_n_fun1 and ?_f_fun1, respectively.

Assuming our code is in a file named `file.c`, and we're compiling for large model, we get:

```c
// file.c
extern int a;

#pragma asm segment ?_f_fun1
#pragma asm _a ds 2
#pragma asm segment file_TEXT

void fun2(void) {
  a = 0;
}

void fun1(void) {
...
  fun2();
}
```

Here, we manually allocate storage for a in `fun1`'s far frame (due to model being large). This method can be used to bring back the nifty (albeit buggy) feature of ZDS II 4.9.x, which got abandoned and is no more present in ZDS II 4.11.0: arbitrary far/near automatic variables.

The syntax used to look like this:

```c
void fun(void) {
  near int a;
  far int b;
  ...
}
```

The near/far storage specification was ignored when using dynamic frames, but for static frames it allowed you selecting whether given automatic variable would be stored in near or far memory space. Accesses to far variables take an extra clock cycle per byte, and can bring an extra load penalty in some cases, where the data has to be transferred from far memory to a register using LDX, before being useable by the target opcode.

We implement this functionality as follows:

```c
// file.c
#pragma asm segment ?_n_fun
#pragma asm _a ds 2
#pragma asm segment ?_f_fun
#pragma asm _b ds 2
#pragma asm segment file_TEXT

void fun(void) {
  extern near int a;
  extern far int b;
  ...
}
```

The main drawback of this method, besides it being prone to typos and suffering from relegating part of the compiler to the wrong side of the keyboard, is that the assembly-level symbols _a and _b really have file scope due to the fact that assembler sees it so. Thus, the canonical way of using this trick is to have a complete tree of nested functions all in one C source file, and having unique names for all the automatic variables that have to be accessed by the local functions.

We shoot two birds with one stone here: we regain the foreign-model automatic variables of ZDS II 4.9.x vintage, and we Pascal-style local functions, which can access the automatic variables present in the scope of the caller's call site. Naturally, this is a hack which should only be used where performance or storage limitations demand it. It could be implemented with an extra C preprocessor.

