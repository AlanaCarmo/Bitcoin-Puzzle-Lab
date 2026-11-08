# Requisitos e regras de negócio

Descreve o comportamento atual do código. Os itens marcados como "comportamento atual questionável" estão registrados como tarefas em `tasks.md`.

## Visão

Programa de terminal que testa chaves privadas de Bitcoin dentro de uma faixa escolhida e compara o endereço de cada uma com 161 endereços "puzzle" cadastrados. Ao achar correspondência, grava a chave e encerra. Há uma interface web opcional (beta).

## Regras de negócio

### Carteiras e faixas (`src/data/ranges.js`, `src/utils/escolherCarteira.js`)

- **RN-01** Existem 161 carteiras, de 1 a 161. Qualquer outro valor é recusado e a pergunta se repete.
- **RN-02** Nas carteiras 1 a 160, a faixa da carteira N vai de 2^(N-1) até 2^N - 1.
- **RN-03** A carteira 161 tem faixa própria: `0xC0DE00000000000000000000000000000000000000000000327ef00cb3000000` a `0xC0DE00000000000000000000000000000000000000000000327ef00cb3ffffff` (16.777.216 chaves).
- **RN-04** Cada faixa tem status: 1 = já encontrada (78 faixas) e 0 = não encontrada (83 faixas). O programa só informa o status, sem impedir a busca.
- **RN-05** Ao escolher a carteira, são exibidos mínimo, máximo e "número possível de chaves" (máximo - mínimo).

### Geração e comparação de chaves (`src/utils/generators.js`)

- **RN-06** Cada chave candidata é um inteiro escrito em hexadecimal, completado com zeros à esquerda até 64 caracteres.
- **RN-07** O endereço é derivado da chave pública comprimida (endereço que começa com "1"), via `coinkey`.
- **RN-08** A comparação é contra os 161 endereços de `wallets.js`, não só o da carteira escolhida.
- **RN-09** Ao achar: imprime chave privada e WIF, acrescenta `Private key: <hex>, WIF: <wif>` em `keys.txt` (nos blocos, com prefixo `Worker para Bloco N:`) e encerra a busca.

### Modos de busca (`src/main.js`)

| Opção | Nome | Comportamento |
| --- | --- | --- |
| 1 | Comecar do inicio | Sequencial a partir do mínimo da faixa |
| 2 | Escolher uma porcentagem | Número de 0 a 1; sequencial a partir de mínimo + (máximo - mínimo) x número |
| 3 | Escolher minimo | Início digitado em hexadecimal (com ou sem `0x`); sequencial |
| 4 | Dividir em blocos | N blocos, um worker por bloco, chaves sorteadas no bloco |
| 5 | Dividir em blocos (Randomico) | Idêntico à 4 (a flag `random` é ignorada) |

- **RN-10** Nas opções 4 e 5, o número de blocos é perguntado; vazio ou 0 usa o número de CPUs.
- **RN-11** Tamanho do bloco = (máximo - mínimo) / N (divisão inteira). O bloco i começa em mínimo + tamanho x i; o último termina no máximo da faixa. Os blocos são impressos antes da busca.
- **RN-12** Na busca por blocos, cada tentativa sorteia um deslocamento no bloco e pula chaves testadas há pouco. A memória guarda até 1.000.000 chaves; acima disso descarta a metade mais antiga.
- **RN-13** Quando um worker acha uma chave, o processo principal encerra os outros.

### Acompanhamento

- **RN-14** A cada 1 segundo imprime o tempo decorrido (e o progresso, nos workers). A cada 10 segundos limpa o terminal, mostra velocidade, total testado e última chave, e grava `Ultima_chave.txt` (sequencial) ou `Ultima_chave_bloco_N.txt` (blocos). Esses arquivos só são gravados; nada os lê para retomar a busca.

### Interface web (`web-interface/app.js`)

- **RN-15** Ao iniciar, pergunta se deseja abrir a interface web (padrão: não). Se sim, pede usuário e senha, que protegem somente `/keys`.
- **RN-16** A página inicial mostra o saldo total em BTC e a quantidade de carteiras encontradas. O saldo soma `final_balance` dos endereços das chaves de `keys.txt`, consultado em `blockchain.info`, arredondado a 2 casas, com cache de 60 s, atualizado via socket.io quando `keys.txt` muda e gravado em `saldo.txt`.
- **RN-17** `/keys` lista as chaves WIF com botão "Copiar". Porta 3000.

## Requisitos não funcionais atuais

- **RNF-01** Node.js com ES Modules. O `Dockerfile` original usa `node:14`; o `Dockerfile.local` usa `node:20`.
- **RNF-02** A busca roda localmente; a única chamada de rede do Node é a consulta de saldo da interface web.
- **RNF-03** Chaves privadas ficam em texto puro em `keys.txt` (decisão atual, ver tarefa T-12).
- **RNF-04** Não há testes automatizados.

## Comportamento atual questionável

Itens que o código faz hoje e que provavelmente não eram a intenção. Não mude sem uma tarefa que peça: PA-01 a PA-11, detalhados em `tasks.md`.
