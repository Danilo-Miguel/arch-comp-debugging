<h2 align="center">Rastreando Chamadas de Biblioteca com LTRACE - Assembly x64 (GAS, AT&T)</h2>

Diferente do strace (que rastreia **syscalls**), o **ltrace** rastreia
**chamadas de funções de biblioteca** (como `printf`, `puts`,
`strlen`). Aqui vamos chamar a `printf` da libc diretamente do
Assembly, sem usar `gcc` nem `_start` de C.

Salve como `bug5.s` no seu diretório home:

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
    movq $7, %rsi
    xorq %rax, %rax
    call printf

    movq $0, %rdi
    call exit
```

O valor correto que deveria aparecer é `42`, mas o programa imprime
outra coisa.

## 1. Monte e ligue contra a libc (arquitetura x64)

Como não estamos usando `gcc`, ligamos manualmente contra a `libc.so.6`
do sistema e o interpretador dinâmico:

```bash
as bug5.s -o bug5.o
ld -o bug5 bug5.o /lib/x86_64-linux-gnu/libc.so.6 \
   --dynamic-linker /lib64/ld-linux-x86-64.so.2
./bug5
```

Se o caminho da libc for diferente no seu sistema, descubra o correto
com `find / -name "libc.so.6" 2>/dev/null` (ou `ldd --version`).

## 2. Investigue com o ltrace

```bash
ltrace ./bug5
```

Procure a linha `printf("O valor calculado eh: %d\n", N)`. O `ltrace`
mostra o valor **real** de `N` que está sendo passado como argumento
para a `printf` — compare com o valor `42` esperado.

## 3. Corrija, remonte e religue

Ajuste o valor movido para `%rsi` (o argumento inteiro passado à
`printf`, seguindo a convenção de chamada do x64: `%rdi` = 1º
argumento, `%rsi` = 2º argumento) para que o programa imprima `42`.
Depois:

```bash
as bug5.s -o bug5.o
ld -o bug5 bug5.o /lib/x86_64-linux-gnu/libc.so.6 \
   --dynamic-linker /lib64/ld-linux-x86-64.so.2
./bug5
```

O `/challenge/check` confere se `bug5.o` e `bug5` existem, se `bug5`
é um executável ELF 64-bit válido, e se a saída é exatamente
`O valor calculado eh: 42`.
