<h2 align="center">GDB Avançado - Laço com Off-by-One em Assembly x86 (GAS, sintaxe Intel)</h2>

O programa abaixo deveria imprimir os dígitos `0123456789` (dez
caracteres, um laço de `0` até `9`). Ele tem um **off-by-one**: o
laço executa uma vez a mais (ou a menos) do que deveria, por causa de
uma comparação de salto condicional errada.

Salve como `bug7.s` no seu diretório home:

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
    jle loop_corpo
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

## 1. Monte e ligue (arquitetura x86, 32 bits)

```bash
as --32 bug7.s -o bug7.o
ld -m elf_i386 bug7.o -o bug7
./bug7
```

Repare que a saída tem um caractere estranho no final, além dos
dígitos `0` a `9`.

## 2. Investigue com o GDB

Desta vez, em vez de olhar só uma vez, vamos **observar o registrador
em cada repetição do laço**:

```bash
gdb ./bug7
(gdb) break loop_corpo
(gdb) run
(gdb) print $ecx
(gdb) continue
(gdb) print $ecx
(gdb) continue
# repita ate o programa terminar, anotando o valor de ecx em cada parada
```

Conte quantas vezes o breakpoint em `loop_corpo` é atingido e quais
valores `ecx` assume. O laço deveria rodar com `ecx` indo de `0` até
`9` (10 vezes). Verifique se é isso que realmente acontece — ou se o
laço roda uma vez a mais, incluindo um valor de `ecx` que não deveria
entrar no corpo do laço.

## 3. Corrija, remonte e religue

O problema está na condição de salto logo após o `cmp ecx, 10`: ela
decide se o laço continua ou não, mas está permitindo uma iteração a
mais. Troque a instrução de salto condicional para que o laço só
entre no corpo quando `ecx` for **estritamente menor** que 10.
Depois:

```bash
as --32 bug7.s -o bug7.o
ld -m elf_i386 bug7.o -o bug7
./bug7
```

O `/challenge/check` confere se `bug7.o` e `bug7` existem, se `bug7`
é um executável ELF 32-bit válido, e se a saída é exatamente
`0123456789` (10 caracteres, sem nada a mais).
