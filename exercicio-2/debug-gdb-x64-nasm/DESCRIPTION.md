<h2 align="center">Depurando com GDB - Registradores em Assembly x64 (NASM)</h2>

Este programa lê um nome digitado pelo usuário e deveria ecoar de
volta `Digite seu nome: <nome digitado>`. Ele tem um bug na
convenção de chamada de syscalls em x64: os registradores usados na
última chamada de `write` estão **trocados**.

Salve como `bug2.asm` no seu diretório home:

```nasm
section .data
    prompt db "Digite seu nome: "
    tam_prompt equ $ - prompt

section .bss
    buffer resb 50

section .text
    global _start

_start:
    ; escreve o prompt
    mov rax, 1
    mov rdi, 1
    mov rsi, prompt
    mov rdx, tam_prompt
    syscall

    ; lê o nome digitado
    mov rax, 0
    mov rdi, 0
    mov rsi, buffer
    mov rdx, 50
    syscall

    ; ecoa o nome de volta
    mov rax, 1
    mov rdi, buffer
    mov rsi, 1
    mov rdx, 50
    syscall

    mov rax, 60
    mov rdi, 0
    syscall
```

## 1. Monte e ligue (arquitetura x64)

```bash
nasm -f elf64 bug2.asm -o bug2.o
ld bug2.o -o bug2
echo "aluno" | ./bug2
```

Note que aqui **não** usamos `-m elf_i386`: em x64 o NASM usa
`-f elf64` e o `ld` já assume o binário de 64 bits por padrão. Rodando
o programa, você deve ver que ele não ecoa seu nome corretamente.

## 2. Investigue com o GDB

```bash
gdb ./bug2
(gdb) break *_start
(gdb) run
(gdb) # dê "stepi" varias vezes ate chegar no ULTIMO syscall (o que ecoa)
(gdb) info registers rax rdi rsi rdx
```

Lembre a convenção de chamada de `write` no Linux x64:
`rax=1 (syscall write)`, `rdi=1 (stdout)`, `rsi=ponteiro do buffer`,
`rdx=tamanho`. Compare isso com os valores reais de `rdi` e `rsi`
logo antes do último `syscall` do programa.

## 3. Corrija, remonte e religue

Troque os registradores para que `rdi` receba o descritor de arquivo
(`1`, stdout) e `rsi` receba o **endereço do buffer**. Depois:

```bash
nasm -f elf64 bug2.asm -o bug2.o
ld bug2.o -o bug2
echo "aluno" | ./bug2
```

O `/challenge/check` confere se `bug2.o` e `bug2` existem, se `bug2`
é um executável ELF 64-bit válido, e se, ao receber um nome de
entrada, o programa devolve `Digite seu nome: <nome>`.
