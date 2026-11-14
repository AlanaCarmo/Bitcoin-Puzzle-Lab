# Decisões

Registro das decisões do projeto. Cada item: contexto, decisão e consequência.

## D-01 Ignorar arquivos `.txt` no git

- Data: 2026-11-14
- Contexto: `keys.txt` guarda chaves privadas em texto puro (RN-09, RNF-03). O `CLAUDE.md` determina que o `.gitignore` ignore `*.txt`, mas a regra não existia; `keys.txt`, `Ultima_chave*.txt` e `saldo.txt` podiam ser commitados por acidente.
- Decisão: adicionar `*.txt` ao `.gitignore`, cobrindo todas as saídas da busca e da interface web.
- Consequência: nenhum `.txt` é versionado. Um `.txt` que precise entrar no repositório deve ser liberado explicitamente com `!nome.txt`.
