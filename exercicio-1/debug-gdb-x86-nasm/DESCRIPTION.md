<h2 align="center">Depurando com GDB - Registradores em Assembly x86 (32 bits, NASM)</h2>

Este é o seu primeiro contato com o **GDB** neste dojo. O programa
abaixo deveria imprimir a mensagem completa, mas foi escrito com um
bug: o tamanho de bytes passado para a syscall `write` está errado
(um valor fixo, "chutado", em vez do tamanho real da string).

Salve o código como `bug1.asm` no seu diretório home:

```nasm
section .data
    msg db "Programa NASM x86 com bug!", 10

section .text
    global _start

_start:
    mov eax, 4
    mov ebx, 1
    mov ecx, msg
    mov edx, 5        ; tamanho passado para o write
    int 0x80

    mov eax, 1
    mov ebx, 0
    int 0x80
```

## 1. Monte e ligue (arquitetura x86, 32 bits)

```bash
nasm -f elf32 bug1.asm -o bug1.o
ld -m elf_i386 bug1.o -o bug1
./bug1
```

Repare que a arquitetura importa: `-f elf32` no NASM e `-m elf_i386`
no `ld` são o que garantem um executável **ELF 32-bit**. Rodando
você já vai notar que a saída está cortada — não aparece a mensagem
inteira nem a quebra de linha.

## 2. Investigue com o GDB

```bash
gdb ./bug1
(gdb) break *_start
(gdb) run
(gdb) stepi        # repita ate chegar perto do "int 0x80"
(gdb) info registers eax ebx ecx edx
```

Observe o valor de `edx` logo antes do `int 0x80` (a chamada de
`write`). Compare com o tamanho real da string `msg` (conte os bytes,
incluindo o `10` do final, que é a quebra de linha). Você vai ver que
o valor em `edx` não bate com o tamanho real da mensagem.

## 3. Corrija, remonte e religue

Corrija `bug1.asm` para que `edx` receba o tamanho **correto** da
mensagem (dica: use `tamanho equ $ - msg` na seção `.data` e troque
`mov edx, 5` por `mov edx, tamanho`, em vez de "chutar" um número).
Depois:

```bash
nasm -f elf32 bug1.asm -o bug1.o
ld -m elf_i386 bug1.o -o bug1
./bug1
```

O `/challenge/check` confere se `bug1.o` e `bug1` existem no seu
diretório home, se `bug1` é um executável ELF 32-bit válido, e se a
saída do programa é a mensagem completa, com a quebra de linha.
