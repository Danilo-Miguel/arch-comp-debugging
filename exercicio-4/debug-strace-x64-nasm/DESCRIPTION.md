<h2 align="center">Rastreando Syscalls com STRACE - Assembly x64 (NASM)</h2>

O programa abaixo deveria imprimir uma frase inteira, mas ela sai
cortada. Em vez de usar o GDB para olhar registrador por registrador,
desta vez use o **strace**: ele já mostra os argumentos resolvidos de
cada `write(fd, buffer, tamanho)`, inclusive o conteúdo do buffer.

Salve como `bug4.asm` no seu diretório home:

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
    mov rdx, 6
    syscall

    mov rax, 60
    mov rdi, 0
    syscall
```

## 1. Monte e ligue (arquitetura x64)

```bash
nasm -f elf64 bug4.asm -o bug4.o
ld bug4.o -o bug4
./bug4
```

A saída aparece cortada, faltando a maior parte da frase.

## 2. Investigue com o strace

```bash
strace ./bug4
```

Olhe a linha `write(1, "...", N) = N`. O strace mostra literalmente
o pedaço de string que foi enviado e o terceiro argumento (`N`), que
é o tamanho em bytes passado no registrador `rdx`. Compare esse `N`
com o tamanho real da mensagem completa (a constante `tam` já
calcula isso para você: `$ - msg`, ou seja, "aqui" menos o início de
`msg`).

## 3. Corrija, remonte e religue

Troque o valor fixo em `mov rdx, 6` para usar a constante `tam`
(tamanho correto da mensagem), em vez de um número chutado. Depois:

```bash
nasm -f elf64 bug4.asm -o bug4.o
ld bug4.o -o bug4
./bug4
```

O `/challenge/check` confere se `bug4.o` e `bug4` existem, se `bug4`
é um executável ELF 64-bit válido, e se a saída é a mensagem
completa, com a quebra de linha.
