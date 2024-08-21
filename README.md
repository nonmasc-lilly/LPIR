# LillyPad Intermediate Representation

[//]: # (TODO: create formal tex file)

This is the IR for LillyPad and is roughly based
on my knowledge of x86-64 assembly.

It has the following registers:

#### General Purpose Registers
a8  b8  c8  d8
a16 b16 c16 d16
a32 b32 c32 d32
a64 b64 c64 d64
#### Special Registers (all 64 bits)
IP : instruction pointer
SP : stack pointer
BP : base pointer
#### Flags register (8 bit)
FL : flags register, has layout of (padding,carry,zero) where padding is 6 bits

in which the naming convention is simple:

```bnf
<register> ::= <name> <size> | "IP" | "SP" | "BP" | "FL"
<name>     ::= "a" |  "b" |  "c" |  "d"
<size>     ::= "8" | "16" | "32" | "64"
```

There are also the following instructions:

```
-------------------------------------------------------------------------------------
| add  r r   | a0 =   a0 + a1                  |                              | c z |
| sub  r r   | a0 =   a0 - a1                  |                              | c z |
| mul  r r   | a0 =   a0 * a1                  |                              | c z |
| div  r r   | a0 =   a0 / a1                  |                              | c z |
| imul r r   | a0 =   a0 * a1                  | where a0 and a1 are signed   | c z |
| idiv r r   | a0 =   a0 / a1                  | where a0 and a1 are signed   | c z |
| fadd r r   | a0 =   a0 + a1                  | where a0 and a1 are floating |   z |
| fsub r r   | a0 =   a0 - a1                  | where a0 and a1 are floating |   z |
| or   r r   | a0 =   a0 | a1                  |                              |   z |
| and  r r   | a0 =   a0 & a1                  |                              |   z |
| not    r   | a0 =   ~a0                      |                              |   z |
| xor  r r   | a0 =   a0 ^ a1                  |                              |   z |
| csub r r   | fl = ?(a0 - a1)                 |                              | c z |
| cor  r r   | fl = ?(a0 | a1)                 |                              |   z |
| cand r r   | fl = ?(a0 | a1)                 |                              |   z |
| jun  m     | IP = a0                         |                              |     |
| jzr  m     | ?z -> IP = a0                   |                              |     |
| jcy  m     | ?c -> IP = a0                   |                              |     |
| push r64   | [SP] = a0, SP = SP + 8          |                              |     |
| push i64   | [SP] = a0, SP = SP + 8          |                              |     |
| pop  r64   | SP = SP - 8, a0 = [SP]          |                              |     |
| stb  m r8  | [a0] = a1                       |                              |     |
| stx  m r16 | [a0] = a1                       |                              |     |
| ste  m r32 | [a0] = a1                       |                              |     |
| str  m r64 | [a0] = a1                       |                              |     |
| ldr  r i64 | a0 = a1                         |                              |   z |
| mvr  r r   | a0 = a1                         |                              |   z |
| ldm  r m   | a0   = [a1]                     |                              |   z |
| call m     | [SP] = IP, SP = SP + 8, IP = a0 |                              |     |
| ret        | SP = SP - 8, IP = [SP]          |                              |     |
-------------------------------------------------------------------------------------
```

labels may be defined as so:

```
<label> ::= "lbl" <iden> ":"
<iden>  ::= <alpha> | <alpha> <iden> | <alpha> <digit> <iden>
```

where alpha is any alphabetical character uppercase or lowercase, and digit is
any decimal arabic numeral

We may also allocate memory to the file with:

```
<reserve> ::= "res" <WSP> <int>
<WSP>     ::= " " | 9 | <WSP> <WSP> | <nothing>
<nothing> ::= ""
```

We may set those reserved bytes at compile time with:

```
<cset>   ::= "cset" <WSP> <iden> <WSP> <param>
<param>  ::= <int> | <string> | <param> <WSP> "," <WSP> <param>
```

Comments are written as so:

```
<comment> ::= ";" <mchar>
<mchar> ::= <WSP> | <vchar> | <mchar> <mchar>
```

where vchar is any vsisible character (0x20->0x7F)

We may also declare an area as: executable, readable, or writeable

We do this with the "segment" keyword formatted as so:

```
<segment> ::= "segment" <WSP> <kindofseg>
<kindofseg> ::= "executable" | "readable" | "writeable"
```