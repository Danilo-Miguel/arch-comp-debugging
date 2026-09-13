<h2 align="center">LTRACE + GDB - Argumentos Trocados e Registrador Sujo - Assembly x64 (GAS, AT&T)</h2>

Este é o exercício final: um capstone que combina **ltrace** e
**GDB** e junta tudo que você praticou nos exercícios anteriores. O
programa calcula `a + b` (`15 + 27`) e deveria imprimir
`Soma: 15 + 27 = 42`, mas tem **dois bugs**.

Salve como `bug10.s` no seu diretório home:

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
    movq %rdi, %r12          /* r12 (callee-saved) guarda a soma */

    movq $fmt, %rdi
    call strlen               /* chamada intermediaria, so para exemplificar */

    movq $fmt, %rdi
    movq b(%rip), %rsi
    movq a(%rip), %rdx
    movq %rax, %rcx
    xorq %rax, %rax
    call printf

    movq $0, %rdi
    call exit
```

## 1. Monte e ligue contra a libc (arquitetura x64)

```bash
as bug10.s -o bug10.o
ld -o bug10 bug10.o /lib/x86_64-linux-gnu/libc.so.6 \
   --dynamic-linker /lib64/ld-linux-x86-64.so.2
./bug10
```

Se o caminho da libc for diferente no seu sistema, descubra o correto
com `find / -name "libc.so.6" 2>/dev/null`.

## 2. Bug 1 — investigue com o ltrace (ordem dos argumentos)

```bash
ltrace ./bug10
```

Veja a chamada `printf("Soma: %d + %d = %d\n", X, Y, Z)`. Compare os
valores `X`, `Y`, `Z` que realmente chegam na `printf` com a ordem
esperada pelo formato (`a`, depois `b`, depois a soma). A convenção
de chamada x64 passa os argumentos inteiros nesta ordem de
registradores: `%rdi`, `%rsi`, `%rdx`, `%rcx`, `%r8`, `%r9` (o
primeiro, `%rdi`, já está ocupado com o endereço da string de
formato).

## 3. Bug 2 — investigue com o GDB (registrador sujo por uma chamada)

```bash
gdb ./bug10
(gdb) break *_start
(gdb) run
(gdb) # va dando "stepi" ate logo depois do "movq %rdi, %r12"
(gdb) info registers r12
(gdb) # continue dando "stepi" ate logo depois do "call strlen"
(gdb) info registers rax r12
```

Repare que `%rax` muda de valor depois do `call strlen` — toda
função pode alterar `%rax` livremente, pois ele **não é
preservado** entre chamadas (é "caller-saved"). Já `%r12` continua
com a soma correta, porque é um registrador **callee-saved**: por
convenção, qualquer função chamada (`strlen`, `printf`, etc.) é
obrigada a preservar seu valor. O código usa `%rax` (sujo pela
`strlen`) como o terceiro argumento de `printf`, quando deveria usar
`%r12`.

## 4. Corrija, remonte e religue

Corrija os dois bugs:
1. A ordem de `%rsi` e `%rdx` (que hoje carregam `b` e depois `a`,
   invertidos);
2. O registrador movido para `%rcx` (hoje `%rax`, sujo pela
   `strlen`; deveria ser `%r12`, que guarda a soma correta).

Depois:

```bash
as bug10.s -o bug10.o
ld -o bug10 bug10.o /lib/x86_64-linux-gnu/libc.so.6 \
   --dynamic-linker /lib64/ld-linux-x86-64.so.2
./bug10
```

O `/challenge/check` confere se `bug10.o` e `bug10` existem, se
`bug10` é um executável ELF 64-bit válido, e se a saída é exatamente
`Soma: 15 + 27 = 42`.
