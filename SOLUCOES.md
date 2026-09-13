# Gabarito — Debugging com GDB, STRACE e LTRACE (10 exercícios)

Gabarito de referência para os 10 módulos do dojo (`exercicio-1` a
`exercicio-10`). Cada exercício entrega um programa em Assembly com
um bug proposital: o aluno usa GDB, strace e/ou ltrace para localizar
o bug, corrige o código-fonte, monta (`nasm`/`as`) e liga (`ld`) na
arquitetura correta, e roda o `check`. As soluções abaixo passam nos
respectivos scripts `check`.

---

## Exercício 1 — `debug-gdb-x86-nasm` (GDB, NASM, x86 32 bits)

Bug: tamanho fixo (`5`) passado ao `write` em vez do tamanho real da
mensagem. Achado inspecionando `edx` no GDB antes do `int 0x80`.

Arquivo: `bug1.asm`

```nasm
section .data
    msg db "Programa NASM x86 com bug!", 10
    tamanho equ $ - msg

section .text
    global _start

_start:
    mov eax, 4
    mov ebx, 1
    mov ecx, msg
    mov edx, tamanho
    int 0x80

    mov eax, 1
    mov ebx, 0
    int 0x80
```

```bash
nasm -f elf32 bug1.asm -o bug1.o
ld -m elf_i386 bug1.o -o bug1
./bug1
```

Saída esperada: `Programa NASM x86 com bug!`

---

## Exercício 2 — `debug-gdb-x64-nasm` (GDB, NASM, x64)

Bug: `rdi`/`rsi` trocados na syscall final de `write` (o ponteiro do
buffer estava em `rdi` e o descritor de arquivo em `rsi`). Achado
comparando a convenção de chamada x64 com `info registers rdi rsi`
no GDB.

Arquivo: `bug2.asm`

```nasm
section .data
    prompt db "Digite seu nome: "
    tam_prompt equ $ - prompt

section .bss
    buffer resb 50

section .text
    global _start

_start:
    mov rax, 1
    mov rdi, 1
    mov rsi, prompt
    mov rdx, tam_prompt
    syscall

    mov rax, 0
    mov rdi, 0
    mov rsi, buffer
    mov rdx, 50
    syscall

    mov rax, 1
    mov rdi, 1
    mov rsi, buffer
    mov rdx, 50
    syscall

    mov rax, 60
    mov rdi, 0
    syscall
```

```bash
nasm -f elf64 bug2.asm -o bug2.o
ld bug2.o -o bug2
echo "aluno" | ./bug2
```

Saída esperada (para entrada `aluno`): `Digite seu nome: aluno`

---

## Exercício 3 — `debug-strace-x86-nasm` (strace, NASM, x86 32 bits)

Bug: descritor de arquivo errado (`ebx = 2`, stderr) na chamada de
`write`. Achado com `strace ./bug3`, olhando o primeiro argumento de
`write(...)`.

Arquivo: `bug3.asm`

```nasm
section .data
    msg db "Assembly com bug de descritor!", 10
    tam equ $ - msg

section .text
    global _start

_start:
    mov eax, 4
    mov ebx, 1
    mov ecx, msg
    mov edx, tam
    int 0x80

    mov eax, 1
    mov ebx, 0
    int 0x80
```

```bash
nasm -f elf32 bug3.asm -o bug3.o
ld -m elf_i386 bug3.o -o bug3
./bug3
```

Saída esperada em stdout: `Assembly com bug de descritor!`

---

## Exercício 4 — `debug-strace-x64-nasm` (strace, NASM, x64)

Bug: tamanho fixo (`6`) passado ao `write` em vez da constante `tam`.
Achado com `strace ./bug4`, comparando o terceiro argumento de
`write(1, "...", N)` com o tamanho real da mensagem.

Arquivo: `bug4.asm`

```nasm
section .data
    msg db "Debugando syscalls em x64!", 10
    tam equ $ - msg

section .text
    global _start

_start:
    mov rax, 1
    mov rdi, 1
    mov rsi, msg
    mov rdx, tam
    syscall

    mov rax, 60
    mov rdi, 0
    syscall
```

```bash
nasm -f elf64 bug4.asm -o bug4.o
ld bug4.o -o bug4
./bug4
```

Saída esperada: `Debugando syscalls em x64!`

---

## Exercício 5 — `debug-ltrace-x64-att` (ltrace, GAS AT&T, x64 + libc)

Bug: argumento errado (`7`) passado para `printf` em vez de `42`.
Achado com `ltrace ./bug5`, olhando o segundo argumento da chamada de
`printf`.

Arquivo: `bug5.s`

```gas
.section .data
formato:
    .string "O valor calculado eh: %d\n"

.section .text
.global _start
.extern printf
.extern exit

_start:
    movq $formato, %rdi
    movq $42, %rsi
    xorq %rax, %rax
    call printf

    movq $0, %rdi
    call exit
```

```bash
as bug5.s -o bug5.o
ld -o bug5 bug5.o /lib/x86_64-linux-gnu/libc.so.6 \
   --dynamic-linker /lib64/ld-linux-x86-64.so.2
./bug5
```

Saída esperada: `O valor calculado eh: 42`

---

## Exercício 6 — `debug-ltrace-x64-nasm` (ltrace, NASM, x64 + libc)

Bug: `lea` apontando para o rótulo errado (`msg_errado` em vez de
`msg_certo`). Achado com `ltrace ./bug6`, olhando o texto real
passado para `puts(...)`.

Arquivo: `bug6.asm`

```nasm
section .data
    msg_errado db "Mensagem de rascunho, nao deveria aparecer!", 0
    msg_certo  db "Ola, este eh o texto correto!", 0

section .text
    global _start
    extern puts
    extern exit

_start:
    lea rdi, [rel msg_certo]
    call puts

    mov rdi, 0
    call exit
```

```bash
nasm -f elf64 bug6.asm -o bug6.o
ld -o bug6 bug6.o /lib/x86_64-linux-gnu/libc.so.6 \
   --dynamic-linker /lib64/ld-linux-x86-64.so.2
./bug6
```

Saída esperada: `Ola, este eh o texto correto!`

---

## Exercício 7 — `debug-gdb-loop-x86-intel` (GDB avançado, GAS Intel, x86 32 bits)

Bug: `jle` em vez de `jl` na comparação do laço, fazendo `ecx = 10`
entrar no corpo do laço (uma iteração a mais). Achado observando o
valor de `ecx` a cada parada em `break loop_corpo` no GDB.

Arquivo: `bug7.s`

```gas
.intel_syntax noprefix
.global _start

.section .bss
digito: .skip 1

.section .text
_start:
    mov ecx, 0

loop_digitos:
    cmp ecx, 10
    jl loop_corpo
    jmp fim

loop_corpo:
    mov eax, ecx
    add eax, '0'
    mov [digito], al

    push ecx
    mov eax, 4
    mov ebx, 1
    lea ecx, [digito]
    mov edx, 1
    int 0x80
    pop ecx

    inc ecx
    jmp loop_digitos

fim:
    mov eax, 1
    mov ebx, 0
    int 0x80
```

```bash
as --32 bug7.s -o bug7.o
ld -m elf_i386 bug7.o -o bug7
./bug7
```

Saída esperada: `0123456789`

---

## Exercício 8 — `debug-gdb-crash-x64-intel` (GDB avançado, GAS Intel, x64)

Bug: `rbx` zerado (`xor rbx, rbx`) usado como ponteiro, causando
SIGSEGV ao escrever em `[rbx]` (endereço `0`). Achado com `run` no
GDB (o crash aponta a instrução exata) e `info registers rbx`.

Arquivo: `bug8.s`

```gas
.intel_syntax noprefix
.global _start

.section .data
ok_msg: .ascii "OK\n"

.section .bss
    .lcomm buffer_real, 8

.section .text
_start:
    mov rax, 10
    add rax, 20

    lea rbx, [rip + buffer_real]
    mov [rbx], rax

    lea rsi, [rip + ok_msg]
    mov rdi, 1
    mov rdx, 3
    mov rax, 1
    syscall

    mov rax, 60
    mov rdi, 0
    syscall
```

```bash
as bug8.s -o bug8.o
ld bug8.o -o bug8
./bug8
```

Saída esperada: `OK` (código de saída `0`, sem SIGSEGV)

---

## Exercício 9 — `debug-combo-strace-gdb-x86-nasm` (strace + GDB, NASM, x86 32 bits)

Bug: `mov ecx, tam_buffer` usado na chamada de `read` (carrega o
número `50` em vez do endereço de `buffer`), causando `EFAULT` na
syscall. Achado com `strace` (mostra o `EFAULT`) e confirmado no GDB
inspecionando `ecx` antes do `read`.

Arquivo: `bug9.asm`

```nasm
section .data
    prompt db "Digite algo: "
    tam_prompt equ $ - prompt
    tam_buffer equ 50

section .bss
    buffer resb 50

section .text
    global _start

_start:
    mov eax, 4
    mov ebx, 1
    mov ecx, prompt
    mov edx, tam_prompt
    int 0x80

    mov eax, 3
    mov ebx, 0
    mov ecx, buffer
    mov edx, tam_buffer
    int 0x80

    mov eax, 4
    mov ebx, 1
    mov ecx, buffer
    mov edx, tam_buffer
    int 0x80

    mov eax, 1
    mov ebx, 0
    int 0x80
```

```bash
nasm -f elf32 bug9.asm -o bug9.o
ld -m elf_i386 bug9.o -o bug9
echo "abc123" | ./bug9
```

Saída esperada (para entrada `abc123`): `Digite algo: abc123`

---

## Exercício 10 — `debug-combo-ltrace-gdb-x64-att` (ltrace + GDB, GAS AT&T, x64 + libc)

Dois bugs combinados:

1. Argumentos de `printf` fora de ordem (`b` antes de `a` em
   `%rsi`/`%rdx`). Achado com `ltrace`, comparando os valores reais
   passados com a ordem esperada pelo formato.
2. Terceiro argumento inteiro (`%rcx`) carregado de `%rax` — que foi
   sujado pelo `call strlen` (registrador caller-saved) — em vez de
   `%r12` (registrador callee-saved, que preserva a soma). Achado no
   GDB comparando `rax` e `r12` antes e depois do `call strlen`.

Arquivo: `bug10.s`

```gas
.section .data
fmt: .string "Soma: %d + %d = %d\n"
a: .quad 15
b: .quad 27

.section .text
.global _start
.extern printf
.extern strlen
.extern exit

_start:
    movq a(%rip), %rdi
    addq b(%rip), %rdi
    movq %rdi, %r12

    movq $fmt, %rdi
    call strlen

    movq $fmt, %rdi
    movq a(%rip), %rsi
    movq b(%rip), %rdx
    movq %r12, %rcx
    xorq %rax, %rax
    call printf

    movq $0, %rdi
    call exit
```

```bash
as bug10.s -o bug10.o
ld -o bug10 bug10.o /lib/x86_64-linux-gnu/libc.so.6 \
   --dynamic-linker /lib64/ld-linux-x86-64.so.2
./bug10
```

Saída esperada: `Soma: 15 + 27 = 42`

---

## Resumo (ferramenta, arquitetura e saída esperada por exercício)

| Exercício | Ferramenta principal | Arquivo | Arquitetura | Saída esperada |
|---|---|---|---|---|
| 1 | GDB | `bug1.asm` | x86 (NASM) | `Programa NASM x86 com bug!` |
| 2 | GDB | `bug2.asm` | x64 (NASM) | `Digite seu nome: aluno` |
| 3 | strace | `bug3.asm` | x86 (NASM) | `Assembly com bug de descritor!` |
| 4 | strace | `bug4.asm` | x64 (NASM) | `Debugando syscalls em x64!` |
| 5 | ltrace | `bug5.s` | x64 (GAS AT&T + libc) | `O valor calculado eh: 42` |
| 6 | ltrace | `bug6.asm` | x64 (NASM + libc) | `Ola, este eh o texto correto!` |
| 7 | GDB avançado | `bug7.s` | x86 (GAS Intel) | `0123456789` |
| 8 | GDB avançado | `bug8.s` | x64 (GAS Intel) | `OK` |
| 9 | strace + GDB | `bug9.asm` | x86 (NASM) | `Digite algo: abc123` |
| 10 | ltrace + GDB | `bug10.s` | x64 (GAS AT&T + libc) | `Soma: 15 + 27 = 42` |
