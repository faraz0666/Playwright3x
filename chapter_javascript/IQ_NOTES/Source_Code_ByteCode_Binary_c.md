# Source Code vs Byte Code vs Binary Code

**Example file:** `HelloWorld.js`

```javascript
console.log(5 != "6");
console.log(5 !== 5);
```

---

## Comparison Table

| Aspect | Source Code | Byte Code | Binary Code (Machine Code) |
|---|---|---|---|
| **Definition** | Human-readable instructions written in a programming language | Intermediate, platform-independent representation between source and machine code | Native instructions that the CPU executes directly |
| **Who reads it** | Humans & compilers/interpreters | Virtual machines (VMs) and JIT compilers | The physical CPU (processor) |
| **Human-readable?** | ✅ Yes — written in plain text | ❌ No — numeric opcodes and operands | ❌ No — raw binary (1s and 0s) |
| **Our Example** | `console.log(5 != "6");` | `LdaSmi 5`, `EqTestStrict "6"`, `CallRuntime` | `mov eax, 5` → `cmp eax, [addr]` → `jne ...` → `call print` |
| **Language level** | High-level (JavaScript, Python, Java, C++) | Mid-level (closer to hardware but still abstract) | Low-level (architecture-specific) |
| **Portability** | ✅ Platform-independent (write once) | ✅ Usually platform-independent (JVM, V8, .NET IL) | ❌ Tied to specific CPU architecture (x86, ARM, RISC-V) |
| **Execution speed** | Slow (interpreted line-by-line) | Fast (interpreted by VM, can be JIT-compiled) | Fastest (direct CPU execution) |
| **Where it lives** | `.js`, `.py`, `.java`, `.c` files | `.class` (Java), `.pyc` (Python), internal V8 bytecode | `.exe`, `.dll`, `.so`, `.o` files |
| **How it gets there** | Written by a developer | Compiled from source by the language's compiler | Compiled/assembled from byte code or assembly by JIT or assembler |
| **Size** | Smallest (compact text) | Medium (encoded instructions) | Largest (full native instructions with addresses) |
| **Security** | Full visibility — easy to audit | Obfuscated — harder but still reversible | Hardest to reverse-engineer |
| **JavaScript example flow** | You write `console.log(...)` in HelloWorld.js | V8 (Node.js) parses → generates Ignition bytecode | TurboFan (hot paths) compiles to x64/ARM64 machine code |

---

## How It Works for Our `HelloWorld.js`

### 1. Source Code (what you see)

```javascript
console.log(5 != "6");   // logs true (loose equality, "6" coerces to number 6)
console.log(5 !== 5);    // logs false (strict equality, same type and value)
```

- Written by a developer.
- Uses abstractions like `console`, operators (`!=`, `!==`), strings, numbers.
- A human can read, understand, and modify it instantly.

### 2. Byte Code (what V8 generates)

When Node.js runs this file, the **V8 Ignition interpreter** parses the source and generates bytecode. The bytecode is **not plain text** — it's encoded instructions. A rough visualization of what `5 != "6"` might look like in V8 bytecode:

```
LdaSmi [5]          ; Load small integer 5 into accumulator
Star r0             ; Store accumulator in register r0
LdaNamedProperty r1, [console.log]  ; Load the log function
Star r2             ; Store it in r2
LdaSmi [5]          ; Load 5 again
Star r3             ; Store in r3
LdaConstant [1]     ; Load string "6" from constant pool
Star r4             ; Store in r4
CallRuntime [StrictNotEqual, r3, r4]  ; Compare 5 != "6"
CallProperty0 r2, r0  ; Call console.log(result)
```

> This is **not human-readable** without tooling. It's intermediate — the VM can interpret it, and if a function runs often, it gets flagged for JIT compilation.

### 3. Binary / Machine Code (what the CPU runs)

If the code is "hot" (runs many times), V8's **TurboFan JIT compiler** converts the bytecode into **x86-64 machine code** — raw binary bytes that the CPU decodes and executes:

**Human-readable disassembly (approximate x64):**
```assembly
mov     eax, 5              ; Load integer 5 into register EAX
cmp     eax, [rsp+8]       ; Compare with "6" (after coercion to number 6)
setne   al                 ; Set AL to 1 if not equal (true)
movzx   ecx, al
call    [ConsoleLog]       ; Call the logging function
```

**Actual binary (hex representation):**
```
B8 05 00 00 00    48 3B 44 24 08    0F 95 C1    FF 15 [address]
```

- This is **architecture-specific**: x86-64 won't run on ARM.
- This is what physically executes on the silicon.
- Impossible for a human to read or modify without disassembly tools.

---

## The Full Pipeline (Visual)

```
┌──────────────────────────────────────────────────────────────┐
│  SOURCE CODE (.js)                                           │
│  console.log(5 != "6");                                      │
│  └── high-level, human-written text                          │
│                                                              │
│       │  Parser (tokenizer + AST)                            │
│       ▼                                                      │
│  BYTE CODE (V8 Ignition)                                     │
│  LdaSmi [5] → Star r0 → CallRuntime StrictNotEqual           │
│  └── intermediate, VM-executable, platform-independent       │
│                                                              │
│       │  JIT Compiler (TurboFan) — only for "hot" code       │
│       ▼                                                      │
│  BINARY / MACHINE CODE (x86-64 / ARM64)                      │
│  B8 05 00 00 00  48 3B 44 24 08  0F 95 C1                   │
│  └── low-level, CPU-executable, platform-dependent           │
└──────────────────────────────────────────────────────────────┘
```

---

## Key Takeaways

| | Source Code | Byte Code | Binary Code |
|---|---|---|---|
| **Made for** | Humans | Virtual Machines | CPUs |
| **Portable?** | ✅ | ✅ (mostly) | ❌ |
| **Fast?** | ❌ (must be parsed) | ⚡ (interpreted or JIT'd) | ⚡⚡ (direct execution) |
| **Example tool** | VS Code, Notepad | V8 Ignition, JVM, CLR | CPU silicon |

**In one sentence:** You write **source code** so humans can read it; the engine compiles it to **byte code** so the VM can run it efficiently; and the JIT converts hot paths to **binary code** so the CPU can execute it at maximum speed.
