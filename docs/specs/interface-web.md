# Spec: interface web (beta)

Arquivos: `web-interface/app.js`, `web-interface/views/index.ejs`, `web-interface/views/keys.ejs`.
Regras relacionadas: RN-15 a RN-17.

## Objetivo

Mostrar, no navegador, o saldo em BTC e as chaves encontradas, sem ler o terminal.

## Ativação

- `main.js` chama `iniciarInterfaceWeb(rl)` antes do menu.
- Pergunta "[beta] Deseja iniciar a interface web? (s/n) [n]". Só `s` ou `sim` ativa.
- Ao ativar, pede usuário e senha (HTTP Basic), usados apenas na rota `/keys`.
- Sobe o servidor na porta 3000 e vigia `keys.txt` com `fs.watch` (cria o arquivo vazio se não existir).

## Rotas e eventos

| Item | Comportamento |
| --- | --- |
| `GET /` | Renderiza `index.ejs`, sem autenticação |
| `GET /keys` | Exige Basic Auth quando a interface foi ativada; renderiza as chaves WIF de `keys.txt` |
| socket.io `connection` | Envia `saldoAtualizado` e `carteirasEncontradas` |
| `fs.watch` em `keys.txt` | Reenvia saldo e contagem a todos os clientes |

## Consulta de saldo

- `GET https://blockchain.info/balance?active=<endereço1>,<endereço2>,...` com `axios`.
- Endereços públicos derivados das chaves privadas de `keys.txt`. A chave privada nunca é enviada.
- Soma `final_balance` de cada endereço, divide por 1e8 e arredonda a 2 casas.
- Cache de 60 s, invalidado se a quantidade de chaves mudar. Resultado também gravado em `saldo.txt`.
- Em erro, só registra no console; o saldo não atualiza.

## Páginas

- `index.ejs`: saldo, "Carteiras encontradas: N" e link para `/keys`. Carrega socket.io de `cdn.socket.io`.
- `keys.ejs`: lista de chaves WIF com botão "Copiar". Carrega Tailwind (jsdelivr) e fontes (Google Fonts).

## Comportamento atual que difere do esperado

- Só `/keys` é protegida; a página inicial e o socket.io são públicos na rede local (T-06).
- `express.static` aponta para `web-interface/public`, que não existe (T-08).
- `lerChavesWIF` não trata erro se `keys.txt` for removido com o servidor no ar (T-07).
- O servidor escuta em todas as interfaces de rede (T-06).

## Critérios de aceite para mudanças

- Sem credenciais, `/keys` responde 401 quando a interface foi ativada com senha.
- Nenhuma chave privada ou WIF é enviada a serviços externos.
