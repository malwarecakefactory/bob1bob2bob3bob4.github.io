---
title: "Lumma Stealer: Deobfuscating Control Flow Flattening"
date: 2026-02-06
categories:
  - malware-analysis
tags:
  - lumma
  - obfuscation
  - reverse-engineering
  - ida-pro
excerpt: "Breaking down Lumma Stealer's indirect control flow obfuscation and building an automated deobfuscation tool."
header:
  overlay_color: "#000"
  overlay_filter: "0.7"
---

## Overview

Lumma Stealer is an information stealer sold as malware-as-a-service (MaaS). This post covers the control flow obfuscation techniques used in a recent sample and how to defeat them.

**Sample**: `lumma1_api_hashing_patched.exe` (32-bit PE)

## Control Flow Obfuscation

Lumma uses indirect jumps through lookup tables to flatten control flow. Instead of direct conditional branches, the binary computes a table index and dispatches through it.

### Pattern 1: Jump Table Dispatch

The most common pattern uses `setcc` to produce a 0 or 1 index:

```nasm
xor     eax, eax
cmp     ecx, edx
setz    al              ; eax = 0 or 1
jmp     dword ptr [eax*4 + 0x458D8C]  ; dispatch through table
```

The jump table at `0x458D8C` contains two entries -- the "true" and "false" branch targets.

### Pattern 2: Complex OR Conditions

Multiple conditions are OR'd together to produce the dispatch index:

```nasm
xor     eax, eax
cmp     ecx, 0x10
setz    al
xor     edx, edx
test    ebx, ebx
setz    dl
or      eax, edx        ; eax = (cond1 || cond2) ? 1 : 0
jmp     dword ptr [eax*4 + 0x458D90]
```

## Deobfuscation Approach

<!-- TODO: Fill in details about your deobfuscation script -->

The deobfuscation tool:
1. Parses the PE and disassembles with Capstone
2. Identifies indirect jump patterns
3. Reads jump table entries from `.rdata`
4. Patches indirect jumps into direct conditional branches
5. Runs multiple passes to resolve chains

## Results

<!-- TODO: Add before/after screenshots or IDA pseudocode -->

After deobfuscation, IDA's decompiler can produce clean pseudocode, making the malware's logic significantly easier to analyze.

## Tools

- [IDA Pro](https://hex-rays.com/ida-pro/) for disassembly and decompilation
- [Capstone](https://www.capstone-engine.org/) for programmatic disassembly
- [pefile](https://github.com/erocarrera/pefile) for PE parsing

---

*This is an ongoing analysis. More posts will cover API hashing resolution and C2 communication.*
