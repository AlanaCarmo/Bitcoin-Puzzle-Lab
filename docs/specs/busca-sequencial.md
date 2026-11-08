# Spec: busca sequencial (opções 1, 2 e 3)

Arquivos: `src/bitcoin-find.js`, `src/main.js` (menu), `src/utils/escolherCarteira.js`, `escolherPorcentagem.js`, `escolherMinimo.js`.
Regras relacionadas: RN-01 a RN-09, RN-14.

## Objetivo

Testar chaves em ordem crescente, a partir de um ponto inicial, em um único processo.

## Entradas

| Dado | Origem |
| --- | --- |
| Carteira (1 a 161) | Pergunta no terminal; define `min` e `max` |
| Ponto inicial `key` | Opção 1: `min`. Opção 2: `min + (max - min) x porcentagem`. Opção 3: hexadecimal digitado |

## Fluxo

1. `main.js` chama `encontrarBitcoins(key, min, max)` e fecha a interface de leitura (`rl.close()`).
2. Loop assíncrono: soma 1 a `key`, converte para 64 caracteres hexadecimais, deriva o endereço e procura no `Set` de endereços.
3. A cada 1 s imprime o tempo; a cada 10 s limpa o terminal, mostra resumo e grava `Ultima_chave.txt`.
4. Em acerto: imprime tempo, velocidade, chave e WIF; grava em `keys.txt`; encerra o loop.
5. Cada volta cede o controle com `setImmediate`, para o processo continuar responsivo.

## Saídas

- Console: tempo, resumo, chave e WIF em caso de acerto.
- Arquivos: `keys.txt` (acréscimo), `Ultima_chave.txt` (sobrescrito).

## Comportamento atual que difere do esperado

- A chave inicial nunca é testada (`key++` vem antes do teste). Tarefa T-01.
- O `max` da faixa não é usado: a busca continua além dele. Tarefa T-02.
- A velocidade usa `Number(key)`, imprecisa em faixas grandes (só afeta a exibição).

## Critérios de aceite para mudanças

- Em uma faixa pequena (ex.: carteira 1 a 10, com a chave conhecida), a busca encontra a chave e grava `keys.txt` com o formato da RN-09.
- Nenhuma conversão de chave para `Number` na lógica de busca.
