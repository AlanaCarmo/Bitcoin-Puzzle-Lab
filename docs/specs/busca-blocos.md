# Spec: busca por blocos com workers (opções 4 e 5)

Arquivos: `src/main.js`, `src/worker.js`, `src/utils/escolherPorcentagemBlocos.js`.
Regras relacionadas: RN-06 a RN-14.

## Objetivo

Dividir a faixa da carteira em N blocos e testar cada bloco em paralelo, um worker thread por bloco.

## Entradas

- Carteira (1 a 161) e número de blocos (vazio ou 0 = número de CPUs).

## Divisão em blocos

- Tamanho = (max - min) / N, divisão inteira.
- Bloco i: início = min + tamanho x i; fim = início + tamanho - 1; o último bloco termina em `max`.
- Antes de iniciar, imprime início, fim e intervalo de cada bloco.

## Fluxo

1. `main.js` cria um `Worker('./src/worker.js')` por bloco (caminho relativo ao diretório de execução).
2. Envia ao worker `{ key, min, max, blocoId, found, random? }`.
3. O worker sorteia deslocamentos aleatórios dentro do bloco, evita repetir chaves recentes (até 1.000.000 em memória; ao passar disso descarta a metade mais antiga) e testa cada chave como na busca sequencial.
4. A cada 1 s imprime progresso; a cada 10 s grava `Ultima_chave_bloco_N.txt`.
5. Em acerto: grava `keys.txt` (com prefixo do bloco), envia `{ found: true }` ao processo principal, que encerra todos os workers.

## Diferença entre as opções 4 e 5

Nenhuma hoje: a opção 5 envia `random: true`, mas o worker ignora a flag e sempre sorteia.

## Comportamento atual que difere do esperado

- Opção 4 deveria ser sequencial por bloco (T-03).
- O worker nunca envia `completed`; a mensagem `completed` esperada em `main.js` não existe (T-03).
- `Math.random() * Number(total)` perde precisão acima de 2^53 chaves (T-04).
- O progresso percentual é divisão inteira de `BigInt` e quase sempre mostra 0% (T-05).
- O flag `found` é copiado na criação do worker; os workers só param pelo `terminate()` do processo principal.

## Critérios de aceite para mudanças

- Em carteira pequena (1 a 10) com 2 a 4 blocos, a chave é encontrada e todos os workers encerram.
- Se um modo sequencial por bloco for criado, o worker deve parar no fim do bloco e enviar `completed`.
