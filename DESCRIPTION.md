# Debugging Assembly com GDB, STRACE e LTRACE

Este dojo introduz e aprofunda o uso de ferramentas de depuração e
engenharia reversa sobre programas em Assembly (NASM e GAS, x86 e
x64): **GDB** (depurador interativo, registradores e memória),
**strace** (rastreio de chamadas de sistema) e **ltrace** (rastreio
de chamadas de biblioteca).

Cada exercício entrega um programa em Assembly **com um bug
proposital**. Você deve:

1. Salvar o código-fonte fornecido em um arquivo no seu diretório home;
2. **Montar** (`nasm`/`as`) e **ligar** (`ld`) manualmente, prestando
   atenção à arquitetura correta (32 ou 64 bits) e às flags certas
   para cada uma;
3. Usar a ferramenta de depuração indicada (GDB, strace ou ltrace)
   para observar o comportamento real do programa e localizar o bug;
4. Corrigir o código-fonte, remontar, religar e confirmar que o
   `/challenge/check` passa.

Os 10 exercícios são avaliados **individualmente** (cada um é um
"hacking" separado), então cada um aparece separadamente na tabela de
colocação. Faça os 10 para completar o dojo.
