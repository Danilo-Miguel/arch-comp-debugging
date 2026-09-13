<h2 align="center">STRACE + GDB - Ponteiro Inválido em Syscall - Assembly x86 (NASM)</h2>

Este exercício combina as duas ferramentas de rastreio de syscalls
que você já usou: **strace** para ver o erro na chamada de sistema, e
**GDB** para confirmar o valor do registrador culpado. O programa lê
um texto digitado e deveria ecoar `Digite algo: <texto digitado>`.

Salve como `bug9.asm` no seu diretório home:

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
    mov ecx, tam_buffer
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

## 1. Monte e ligue (arquitetura x86, 32 bits)

```bash
nasm -f elf32 bug9.asm -o bug9.o
ld -m elf_i386 bug9.o -o bug9
echo "abc123" | ./bug9
```

O texto digitado nunca aparece de volta — só o prompt.

## 2. Investigue com o strace

```bash
echo "abc123" | strace ./bug9
```

Procure a chamada `read(...)`. Se ela retornar um erro do tipo
`EFAULT (Bad address)`, o segundo argumento (o ponteiro do buffer)
não é um endereço de memória válido — ele é só um número pequeno.

## 3. Confirme com o GDB

```bash
gdb ./bug9
(gdb) break *_start
(gdb) run
(gdb) # de "stepi" ate chegar no syscall do read (o segundo int 0x80)
(gdb) info registers ecx
```

Compare o valor de `ecx` (o que deveria ser o **endereço** do
`buffer` na seção `.bss`) com o valor que o registrador realmente
tem antes da chamada de `read`. Um dos dois `mov ecx, ...` no código
está usando a constante numérica `tam_buffer` (que vale `50`) no
lugar do endereço de `buffer`.

## 4. Corrija, remonte e religue

Troque o `mov ecx, tam_buffer` da chamada de `read` para
`mov ecx, buffer` (o endereço do buffer, não o número 50). Depois:

```bash
nasm -f elf32 bug9.asm -o bug9.o
ld -m elf_i386 bug9.o -o bug9
echo "abc123" | ./bug9
```

O `/challenge/check` confere se `bug9.o` e `bug9` existem, se `bug9`
é um executável ELF 32-bit válido, e se, para uma entrada de texto, o
programa devolve `Digite algo: <texto digitado>`.
