# Code-Exe-Language

# Code Execution Language (CEL) — Official Specification v1.0

## 1 · Preface

Code Execution Language (CEL) is a **compile‑structured, implementation‑oriented scripting language** designed to translate **one‑to‑one into Windows x64 NASM** while remaining easily readable by humans. CEL employs **gated blocks**, **office‑suite punctuation**, **3‑helixical spacing**, and **radial‑off‑grid indentation** to encode structural intent visually as well as syntactically.

---

## 2 · Design Goals

1. **Machine‑Friendly** — Every construct maps deterministically to NASM instructions.
2. **Intuitive‑Intrinsic Semantics** — Statements resemble imperative English mnemonics.
3. **Minimal Surface Area** — A small, orthogonal core of operations.
4. **Visual Semantics** — Layout conveys scope (gates & radial indentation) and flow (3‑helixical spacing).
5. **Zero‑Overhead** — No hidden runtime; generated code is straight NASM.

---

## 3 · Lexical Structure

| Element            | Definition                                                   |
| ------------------ | ------------------------------------------------------------ |
| **Identifier**     | Letter, followed by letters, digits, or `_`                  |
| **Number Literal** | Decimal digits                                               |
| **Gate Marker**    | `[` *identifier* `]`                                         |
| **Block Braces**   | `{` … `}`                                                    |
| **Separator**      | Semicolon `;` between statements                             |
| **Punctuation**    | `:` `,` `[` `]` `{` `}` `;`                                  |
| **Whitespace**     | Space ␠ and Tab ⇥ — line‑endings significant for indentation |

---

## 4 · Complete Grammar (EBNF)

```
program        = { block } ;
block          = gate , "{" , { statement , ";" } , "}" ;
gate           = "[" , identifier , "]" ;
statement      = identifier , ":" , operation , args ;
operation      = "SET"|"ADD"|"SUB"|"MUL"|"DIV"|"DISPLAY"|"COMPARE"|
                 "JUMP_IF"|"LOOP"|"AND"|"OR"|"NOT"|"XOR"|"SHIFT_LEFT"|"SHIFT_RIGHT" ;
args           = identifier | number |
                 identifier , "," , identifier |
                 identifier , "," , number |
                 identifier , "," , identifier , "," , identifier ;
identifier     = letter , { letter | digit | "_" } ;
number         = digit , { digit } ;
```

---

## 5 · Semantics

### 5.1 Data Model

* **Primitive Type** — 64‑bit signed integer (`int64`).
* **Storage** — Each identifier in a block is allocated one quadword in the `.bss` section.
* **Lifetime** — Static for the entire program (single translation unit).

### 5.2 Operations

| Mnemonic      | Args                       | NASM Mapping (conceptual)              | Description                         |
| ------------- | -------------------------- | -------------------------------------- | ----------------------------------- |
| `SET`         | dest, value                | `mov [dest], value`                    | Initialise / overwrite variable     |
| `ADD`         | dest, value                | `add [dest], value`                    | Add value to dest                   |
| `SUB`         | dest, value                | `sub [dest], value`                    | Subtract                            |
| `MUL`         | dest, value                | `imul rax, [dest]; imul rax, value`    | Multiply                            |
| `DIV`         | dest, value                | `xor rdx,rdx; mov rax,[dest]; div val` | Unsigned divide                     |
| `AND` `OR` …  | dest, value                | `and/or/xor/not`                       | Bitwise                             |
| `SHIFT_LEFT`  | dest, amount               | `shl dest, imm`                        | Logical shift left                  |
| `SHIFT_RIGHT` | dest, amount               | `shr dest, imm`                        | Logical shift right                 |
| `COMPARE`     | lhs, rhs                   | `cmp lhs, rhs`                         | Sets FLAGS; result stored by caller |
| `JUMP_IF`     | flagVar, gateName          | Conditional `jne` / `je` to label      | Branch based on last `COMPARE`      |
| `LOOP`        | counter, step, accumulator | Expands to loop gate                   | High‑level counted loop             |
| `DISPLAY`     | value                      | Call `printf` via C ABI                | Console print                       |

> **Note** — `LOOP` expands at compile‑time into a dedicated gate named `LOOP_Block` or a user‑supplied label.

### 5.3 Execution Model

A CEL translation unit is converted into a single NASM file containing:
1. Static data (`section .data`),
2. Uninitialised storage (`section .bss`),
3. Entry point `main` (`section .text`).
Control flow is linear except where altered by `JUMP_IF` or the expansion of `LOOP`.

---

## 6 · Whitespace & Layout Rules

### 6.1 Radial‑Off‑Grid Indentation

Indentation depth equals *gate nesting level*, but each level rotates indentation width by **4 → 1 → 3 spaces** (the “radial” cycle). This keeps sibling blocks visually de‑aligned, forming a spiral‑like margin.

### 6.2 3‑Helixical Spacing

Within a statement the **operation** is aligned to the next multiple of 3 spaces after the longest identifier on that line group, producing a helical ribbon when viewed vertically.

---

## 7 · File Extension & Encoding

* Source files use ASCII/UTF‑8 and carry the extension `.cel`.

---

## 8 · Reference Toolchain (celc)

| Stage         | Responsibility                             |
| ------------- | ------------------------------------------ |
| **Lexer**     | Tokenise identifiers, numbers, punctuation |
| **Parser**    | Build Gate/Block AST validating EBNF       |
| **Semantic**  | Type & arity checks, allocate storage      |
| **Emitter**   | Generate `.asm` with NASM syntax           |
| **Assembler** | Invoke `nasm -f win64`                     |
| **Linker**    | `gcc` / `lld` to produce `.exe`            |

CLI:

```shell
celc hello.cel            # produces hello.exe
celc -S hello.cel         # stop after .asm generation
celc -o build/ hello.cel  # output directory
```

---

## 9 · Error Handling

* **Syntax Error** — Unexpected token; abort compilation.
* **Arity Error** — Wrong number of args for operation.
* **Undefined Identifier** — Use before `SET`.
* **Divide‑by‑Zero** — Compile‑time diagnostic if divisor literal is `0`, else runtime trap left to hardware.

---

## 10 · Sample Program

```
[Main] {
    msg     : SET 68;          ; ASCII 'D'
    shift   : SET 1;
    shifted : SHIFT_LEFT msg, shift;
    result  : DISPLAY shifted;
}
```

Assembled output prints `136` (0x88).

---

## 11 · Versioning & Governance

* Version numbers follow **SemVer** — `MAJOR.MINOR.PATCH`.
* Changes are proposed via **CEL Enhancement Proposals (CEP)** reviewed by the Core Team.
* The canonical spec lives at [https://cel‑lang.dev/spec](https://cel‑lang.dev/spec) (placeholder).

---

## 12 · Roadmap (abridged)

1. v1.1 — User‑defined macros.
2. v1.2 — 32‑bit build target.
3. v2.0 — Modules & symbol visibility.

---

### © 2025 The CEL Project of Joey Soprano — Licensed under Modified Quick-Sample-Reference Long-code (QSRLC) License V2.0
