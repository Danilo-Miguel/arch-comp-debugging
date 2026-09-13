<h2 align="center">Rastreando Chamadas de Biblioteca com LTRACE - Assembly x64 (NASM)</h2>

O programa abaixo tem duas mensagens na seção `.data`, mas chama
`puts` apontando para a mensagem **errada** — um clássico "bug de
copiar e colar".

Salve como `bug6.asm` no seu diretório home:

```nasm
section .data
    msg_errado db "Mensagem de rascunho, nao deveria aparecer!", 0
    msg_certo  db "Ola, este eh o texto correto!", 0

section .text
    global _start
    extern puts
    extern exit

_start:
    lea rdi, [rel msg_errado]
    call puts

    mov rdi, 0
    call exit
```

## 1. Monte e ligue contra a libc (arquitetura x64)

```bash
nasm -f elf64 bug6.asm -o bug6.o
ld -o bug6 bug6.o /lib/x86_64-linux-gnu/libc.so.6 \
   --dynamic-linker /lib64/ld-linux-x86-64.so.2
./bug6
```

Se o caminho da libc for diferente no seu sistema, descubra o correto
com `find / -name "libc.so.6" 2>/dev/null`.

## 2. Investigue com o ltrace

```bash
ltrace ./bug6
```

O `ltrace` mostra o **conteúdo real da string** passada para `puts`,
já que ela é um argumento de tipo texto. Compare a string que
aparece na chamada `puts(...)` com a mensagem que o programa deveria
mostrar.

## 3. Corrija, remonte e religue

Corrija o `lea` para apontar para o rótulo correto na seção `.data`.
Depois:

```bash
nasm -f elf64 bug6.asm -o bug6.o
ld -o bug6 bug6.o /lib/x86_64-linux-gnu/libc.so.6 \
   --dynamic-linker /lib64/ld-linux-x86-64.so.2
./bug6
```

O `/challenge/check` confere se `bug6.o` e `bug6` existem, se `bug6`
é um executável ELF 64-bit válido, e se a saída é exatamente
`Ola, este eh o texto correto!` (o `puts` adiciona a quebra de linha
sozinho).
