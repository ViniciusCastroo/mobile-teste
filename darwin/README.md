# darwin-handoff (carona de contexto)

Repositório **privado, pessoal** — só serve de ponte para retomar o trabalho do
Darwin (branch `feat/calendario-espelho-microsoft`, espelho de calendário Microsoft 365)
em outra máquina. Não é o repo do produto (esse é `neotel-br/darwin`).

## Conteúdo

| Item | Descrição |
|---|---|
| `darwin-feat-calendario-espelho-microsoft.bundle` | Clone completo da branch (histórico inteiro). |
| `claude-memory/` | Memória do Claude Code deste projeto. |
| `LEIA-ME.md` | Passo a passo de restauração. |

## NÃO está aqui (de propósito)

- `env.txt` / qualquer `.env` — segredos. Transportar por canal separado (pendrive, etc.).

## Uso

Ver `LEIA-ME.md`. Resumo: clonar este repo → `git clone` a partir do `.bundle` →
recriar `.env` → `make dev` / `make migrate` → abrir `claude` no repo do Darwin e pedir
pra ler `HANDOFF.md` + `MEMORY.md`.
