<h2 align="center">GDB Avançado - Diagnosticando um SIGSEGV em Assembly x64 (GAS, sintaxe Intel)</h2>

O programa abaixo calcula `10 + 20`, guarda o resultado na memória e
imprime `OK`. Só que ele **quebra** (SIGSEGV / Segmentation fault)
antes de chegar a imprimir qualquer coisa.

Salve como `bug8.s` no seu diretório home:

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

    xor rbx, rbx
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

## 1. Monte e ligue (arquitetura x64)

```bash
as bug8.s -o bug8.o
ld bug8.o -o bug8
./bug8; echo "codigo de saida: $?"
```

Você deve ver `Segmentation fault` e um código de saída `139`
(128 + sinal 11, SIGSEGV) — e nenhum `OK` na tela.

## 2. Investigue com o GDB

```bash
gdb ./bug8
(gdb) run
```

O GDB deve parar exatamente na instrução que causou o crash e
mostrar algo como `SIGSEGV` e um endereço de memória inválido. Depois
do crash, sem sair do GDB:

```bash
(gdb) info registers rax rbx
(gdb) x/i $rip
```

Veja qual instrução falhou e qual endereço de memória ela tentou
acessar (o valor de `rbx` no momento do crash). Esse é o clássico
bug de **ponteiro nulo**: um registrador que deveria apontar para um
espaço de memória válido está zerado.

## 3. Corrija, remonte e religue

Em vez de zerar `rbx` com `xor rbx, rbx`, ele precisa apontar para o
endereço do símbolo `buffer_real`, reservado na seção `.bss` (use
`lea rbx, [rip + buffer_real]`). Depois:

```bash
as bug8.s -o bug8.o
ld bug8.o -o bug8
./bug8; echo "codigo de saida: $?"
```

O `/challenge/check` confere se `bug8.o` e `bug8` existem, se `bug8`
é um executável ELF 64-bit válido, se ele **não trava mais** (código
de saída `0`) e se a saída é `OK`.
