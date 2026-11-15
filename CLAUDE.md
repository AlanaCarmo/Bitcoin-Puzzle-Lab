# btc-finder

Programa de terminal em Node.js (ES Modules) que testa chaves privadas de Bitcoin, uma a uma, até gerar um dos 161 endereços "puzzle" listados em `src/data/wallets.js`. Tem uma interface web opcional (beta) para ver saldo e chaves encontradas. Projeto de fins educacionais.

Origem: https://github.com/lmajowka/btc-finder

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

## Plano de implementação

<!-- - Documente features dos itens implementados em docs/FEATURES.md -->
- Levante as melhorias necessárias, limitada a 10 itens, e registre cada uma como uma task em `docs/tasks.md`, com critérios de aceite claros.
- Apresente as tasks ao usuário uma a uma e pergunte qual deve ser implementada. Não implemente nenhuma sem solicitação explícita.
- Em cada pergunta, inclua a opção "Solicitar nova melhoria", para que o usuário possa propor itens que ainda não estejam no roadmap.
- Ao concluir uma task, faça o commit no git somente após a aprovação explícita do usuário.
- Antes de commitar, mostre a mensagem de commit sugerida e aguarde o ok.

## Como trabalhar neste projeto

- Antes de alterar uma área, leia a spec dela e as RN envolvidas.
- Todas as decisões devem ser documentadas em docs/DECISIONS.md
- Documente eventuais gaps em docs/GAPS.md e sua mitigação se aplicavel.
- Mudança de comportamento = atualizar a spec e `docs/requirements.md` no mesmo trabalho. Mudança que corrige um item de `docs/tasks.md` = marcar o item como feito.
- Preserve as RN existentes, a menos que a tarefa peça mudá-las. Se uma RN parecer um bug, não corrija em silêncio: registre em `docs/tasks.md` ou pergunte.
- Faça mudanças pequenas e verificáveis. Para validar lógica sem rodar a busca de verdade, use faixas minúsculas (carteira 1 a 10) e as funções puras, nunca faixas grandes.
- Quando criar testes, use o runner nativo do Node (`node --test`), sem adicionar dependências sem necessidade.

## Regras obrigatórias 

- Executar exclusivamente operações de leitura em qualquer base de dados.
- Nunca criar, modificar, excluir ou reiniciar recursos.
- Nunca executar deploys ou alterações de infraestrutura.
- Nunca publicar mensagens, executar jobs ou disparar workflows.
- Nunca assumir que uma operação é segura apenas pelo nome.
- Não solicitar permissões adicionais para executar operações de escrita.
- O arquivo deve ser escrito em português, com linguagem técnica, objetiva e sem instruções redundantes.
- Priorize segurança, precisão, baixo custo de consultas.
- Regra: decisões arquiteturais relevantes devem ser questionadas ao usuario em forma de questionario cada uma todas seguindo o seguinte padrao
  - propor multiplas opções e incluir uma opção recomendada
  - para cada opção descrever "que é", "exemplo", "pros", "contras"
  - e uma opção em branco para o usuario escrever outro item se necessario
  - questionario e alternativas em pt-br
- Novas regras de definição devem ser incluidas em CLAUDE.md mediante solicitação.
- Evite puxar informações extensas para a janela de contexto, preze por uso eficiente de tokens.
- Jamais utilizar credenciais reais nos testes de integração.
- Jamais comitar na main.
- Commit só deve ser feito com aprovação explicita do usuario.
- Push exige nova aprovação explícita, solicitada após o commit aprovado. A aprovação do commit não autoriza o push.
- Push via SSH, sem alterar o remoto configurado: `git push git@github.com:AlanaCarmo/Bitcoin-Puzzle-Lab.git <branch>`. O `origin` em HTTPS não tem credencial nesta máquina. Após o push, confirmar com `git ls-remote` que o commit remoto é igual ao `HEAD` local.
- Ambientes de integração todos em docker com os itens versionados.
- Regras de escrita/criação/alteração de código:
    - Sempre usar clean code
    <!-- - regra: use logs bem estruturados -->
    - Sempre construa testes primeiro e depois implementação.
    - Em caso de bugs construa os testes para pegar o bug e entao solucione o bug
    - Sempre escrever código em ingles.
    - Todos os comentarios e documentações em markdown em portugues.
    - Não use separadores textuais, emojis, e evite poluição textual comente apenas o que for relevante com a perspectiva de lógica de código ou RN.
    - Adote sempre uma única forma de fazer cada tarefa, priorizando simplicidade e consistência.
    - Nos testes unitarios jamais utilizar credenciais de ambientes reais.

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
