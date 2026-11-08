# btc-finder

Programa de terminal em Node.js (ES Modules) que testa chaves privadas de Bitcoin, uma a uma, até gerar um dos 161 endereços "puzzle" listados em `src/data/wallets.js`. Tem uma interface web opcional (beta) para ver saldo e chaves encontradas. Projeto de fins educacionais.

Origem: https://github.com/lmajowka/btc-finder

## Comandos

```bash
npm install          # instalar dependências
npm start            # roda node src/main.js (sempre a partir da raiz do projeto)
```

Docker com o código local: `docker build -f Dockerfile.local -t btc-finder-local .` e `docker run -it --name btc-finder-local -p 3000:3000 btc-finder-local`. O `Dockerfile` original clona o GitHub e NÃO usa o código local.

Não existem testes, lint nem build. O programa é interativo (faz perguntas no terminal), então não rode `npm start` em automação sem fornecer as respostas por stdin.

## Mapa do código

- `src/main.js`: entrada. Banner, pergunta da interface web, menu (carteira + 5 modos), cria workers nos modos 4 e 5.
- `src/bitcoin-find.js`: busca sequencial (modos 1 a 3), um único processo.
- `src/worker.js`: busca aleatória dentro de um bloco (modos 4 e 5), roda em worker thread.
- `src/data/wallets.js`: os 161 endereços procurados.
- `src/data/ranges.js`: faixa (min, max) e status (1 encontrada, 0 não) de cada carteira.
- `src/utils/`: perguntas do menu e `generators.js` (`generatePublic`, `generateWIF` via `coinkey`).
- `web-interface/app.js`: Express + socket.io, porta 3000; `views/*.ejs`.

## Documentação (leia antes de mudar o comportamento)

- `docs/requirements.md`: requisitos e regras de negócio (RN-xx). Fonte da verdade do que o programa deve fazer.
- `docs/specs/`: uma spec por área (busca sequencial, busca por blocos, interface web).
- `docs/tasks.md`: backlog de melhorias com critérios de aceite (T-xx), derivado dos pontos de atenção (PA-xx).

## Como trabalhar neste projeto

1. Antes de alterar uma área, leia a spec dela e as RN envolvidas.
2. Mudança de comportamento = atualizar a spec e `docs/requirements.md` no mesmo trabalho. Mudança que corrige um item de `docs/tasks.md` = marcar o item como feito.
3. Preserve as RN existentes, a menos que a tarefa peça mudá-las. Se uma RN parecer um bug, não corrija em silêncio: registre em `docs/tasks.md` ou pergunte.
4. Faça mudanças pequenas e verificáveis. Para validar lógica sem rodar a busca de verdade, use faixas minúsculas (carteira 1 a 10) e as funções puras, nunca faixas grandes.
5. Quando criar testes, use o runner nativo do Node (`node --test`), sem adicionar dependências sem necessidade.

## Convenções do código

- ES Modules (`import`/`export`), sem CommonJS.
- Nomes de funções e mensagens ao usuário em português, como no restante do projeto.
- Indentação de 4 espaços em `src/`.
- Chaves e faixas são `BigInt`. Nunca converta chave para `Number` (perde precisão acima de 2^53).
- Chave privada em texto = 64 caracteres hexadecimais, com zeros à esquerda.

## Cuidados

- `keys.txt` guarda chaves privadas em texto puro. Nunca commitar, imprimir em log de CI nem colar em conversas. O `.gitignore` já ignora `*.txt`; mantenha assim.
- Não rode buscas longas em faixas grandes (carteira 55 em diante é inviável). Não faça benchmark sem avisar a pessoa.
- A única chamada de rede do código Node é `GET https://blockchain.info/balance?active=<endereços>` na interface web, com endereços públicos. Não envie chaves privadas para fora da máquina.
- Não adicione telemetria, nem novas chamadas externas, sem pedido explícito.

## Armadilhas conhecidas (ver docs/tasks.md)

- A busca sequencial não testa a chave inicial e não para no máximo da faixa.
- Os modos 4 e 5 são idênticos (a flag `random` é ignorada no worker) e o worker nunca envia `completed`.
- O sorteio usa `Number(total)` e perde precisão em faixas com mais de 2^53 chaves.
- Vários pacotes do `package.json` não são usados (`node-fetch`, `nodemon`, `postcss`, `tailwindcss`); `coinkey` aparece duas vezes.
