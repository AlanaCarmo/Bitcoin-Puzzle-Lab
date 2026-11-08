# Tarefas (backlog)

Cada tarefa vem dos pontos de atenção (PA) levantados na leitura do código. Estado: `[ ]` aberta, `[x]` feita. Ao concluir, marque aqui e atualize a spec e `requirements.md`.

Prioridade sugerida: correção de busca (T-01 a T-05), depois segurança (T-06), depois higiene (T-07 a T-12).

## Busca

- [ ] **T-01** (PA-01) Testar a chave inicial na busca sequencial.
  Aceite: com início = chave conhecida de uma faixa pequena, ela é encontrada.
- [ ] **T-02** (PA-02) Parar a busca sequencial quando `key` passar do `max` da faixa e avisar "faixa concluída sem achar chave".
  Aceite: em faixa pequena sem a chave, o programa termina sozinho.
- [ ] **T-03** (PA-03) Dar sentido às opções 4 e 5: opção 4 sequencial por bloco (parando no fim e enviando `completed`), opção 5 aleatória. Respeitar a flag `random` no worker.
  Aceite: opção 4 percorre cada bloco em ordem e encerra; opção 5 mantém o sorteio.
- [ ] **T-04** (PA-04) Sortear o deslocamento com `BigInt` (sem `Number(total)`), para cobrir toda a faixa em carteiras acima de 2^53.
  Aceite: nenhuma conversão de `total` para `Number` em `worker.js`.
- [ ] **T-05** (PA-05) Corrigir o progresso percentual do worker (ex.: duas casas decimais, calculado com `BigInt` ou razão).
  Aceite: o progresso aparece diferente de 0% em blocos pequenos.

## Segurança

- [ ] **T-06** (PA-10) Proteger a página inicial e o socket.io com a mesma autenticação de `/keys` e, por padrão, escutar só em `127.0.0.1` (opção para liberar na rede).
  Aceite: sem credenciais, `/` e o socket são recusados.

## Higiene

- [ ] **T-07** (PA-11) Tratar erro de leitura de `keys.txt` em `lerChavesWIF` (retornar lista vazia).
- [ ] **T-08** (PA-08) Remover `express.static` para pasta inexistente ou criar `web-interface/public`.
- [ ] **T-09** (PA-08) Limpar o `package.json`: remover duplicata de `coinkey` e pacotes não usados (`node-fetch`, `nodemon`, `postcss`, `tailwindcss`); remover os arquivos de config de Tailwind e PostCSS se não houver build.
- [ ] **T-10** (PA-07) Gravar `keys.txt`, `Ultima_chave*.txt` e `saldo.txt` em um único diretório definido (raiz do projeto), independentemente de onde `npm start` rodar.
- [ ] **T-11** (PA-06) Atualizar o `Dockerfile` para `node:20` e permitir usar o código local (ou adotar o `Dockerfile.local`).
- [ ] **T-12** (PA-07) Avaliar proteger `keys.txt` (permissão 600 ou aviso ao usuário) já que guarda chaves privadas em texto puro.

## Qualidade

- [ ] **T-13** (PA-09) Criar testes com `node --test` para: conversão chave para hexadecimal de 64 caracteres, derivação de endereço (a chave `0x1` deve gerar `1BgGZ9tcN4rm9KBzDn7KprQz87SZ26SAMH`, o primeiro endereço de `wallets.js`), divisão em blocos e leitura de `ranges.js` (regra 2^(N-1) a 2^N - 1).
- [ ] **T-14** (PA-09) Adicionar ao README uma seção "O que é o projeto" resumindo `CLAUDE.md` e `docs/requirements.md`.

## Observação sobre a viabilidade

A busca só é realista nas faixas pequenas (a faixa da carteira 71 teria cerca de 37 milhões de anos a 1 milhão de chaves por segundo). Melhorias de desempenho só trazem ganho prático nas faixas pequenas ou médias.
