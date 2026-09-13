<h2 align="center">Rastreando Syscalls com STRACE - Assembly x86 (32 bits, NASM)</h2>

Agora você vai usar o **strace**, que mostra todas as chamadas de
sistema (syscalls) feitas por um programa, com seus argumentos
reais. O programa abaixo deveria imprimir uma mensagem na saída
padrão (stdout), mas algo está errado.

Salve como `bug3.asm` no seu diretório home:

```nasm
section .data
    msg db "Assembly com bug de descritor!", 10
    tam equ $ - msg

section .text
    global _start

_start:
    mov eax, 4
    mov ebx, 2
    mov ecx, msg
    mov edx, tam
    int 0x80

    mov eax, 1
    mov ebx, 0
    int 0x80
```

## 1. Monte e ligue (arquitetura x86, 32 bits)

```bash
nasm -f elf32 bug3.asm -o bug3.o
ld -m elf_i386 bug3.o -o bug3
./bug3
```

Rodando direto no terminal a mensagem ainda aparece na tela (stdout e
stderr compartilham o mesmo terminal), então o bug não é óbvio só de
olhar a saída. Tente isolar o stdout:

```bash
./bug3 1>/tmp/saida.txt
cat /tmp/saida.txt
```

O arquivo fica vazio! A mensagem não foi escrita em stdout.

## 2. Investigue com o strace

```bash
strace ./bug3
```

Procure a linha da chamada `write(...)` na saída do strace. Ela
mostra exatamente qual **descritor de arquivo** (primeiro argumento)
foi usado. Descritores padrão no Linux: `0` = stdin, `1` = stdout,
`2` = stderr. Compare o valor que aparece no strace com o que era
esperado para imprimir em stdout.

## 3. Corrija, remonte e religue

Corrija o registrador `ebx` na chamada de `write` para usar o
descritor correto de stdout. Depois:

```bash
nasm -f elf32 bug3.asm -o bug3.o
ld -m elf_i386 bug3.o -o bug3
./bug3 1>/tmp/saida.txt
cat /tmp/saida.txt
```

O `/challenge/check` confere se `bug3.o` e `bug3` existem, se `bug3`
é um executável ELF 32-bit válido, e se a mensagem completa aparece
na **saída padrão** (stdout) — e não em stderr.
