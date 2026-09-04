# LEIA-ME — Restauração do Darwin na máquina de destino

Pacote gerado em 2026-09-04. Branch: `feat/calendario-espelho-microsoft`.
As duas máquinas não se enxergam na rede; a ponte é este repo (`ViniciusCastroo/mobile-teste`),
que já está clonado nas duas.

Conteúdo de `mobile-teste/darwin/`:

| Arquivo | O que é |
|---|---|
| `darwin-feat-calendario-espelho-microsoft.bundle` | Clone completo da branch (histórico inteiro). Não precisa do remote `neotel-br`. |
| `claude-memory/` | Memória do Claude Code deste projeto (`MEMORY.md` + notas). |
| `README.md` | Descrição curta. |
| `LEIA-ME.md` | Este arquivo. |

**O `.env` NÃO está aqui** (segredos). Transportar por pendrive/canal separado — na origem
está em `~/darwin-handoff/env.txt`.

---

## 1. Atualizar a carona na máquina de destino

```bash
cd ~/github/mobile-teste
git pull
```

---

## 2. Recriar o repositório do Darwin a partir do bundle

```bash
mkdir -p ~/github && cd ~/github
git clone ~/github/mobile-teste/darwin/darwin-feat-calendario-espelho-microsoft.bundle darwin
cd darwin
git checkout feat/calendario-espelho-microsoft

# religar ao GitHub real (se esta máquina tiver acesso à org neotel-br):
git remote set-url origin https://github.com/neotel-br/darwin.git
git fetch origin
git branch --set-upstream-to=origin/feat/calendario-espelho-microsoft
```
Se esta máquina **não** tiver acesso à org `neotel-br`: o bundle já tem o histórico
completo; trabalhe local e sincronize na volta gerando um bundle novo (§6).

> O commit `docs: HANDOFF ...` está no bundle mas pode ainda não estar no `origin`
> da org. O `HANDOFF.md` fica na raiz do repo do Darwin depois do clone.

---

## 3. Recriar o `.env`

```bash
cp /caminho/do/env.txt ~/github/darwin/.env
```

Ajustes por máquina (editar o `.env`):

- **`CORS_ALLOWED_ORIGINS`** — contém `http://192.168.130.55:5173` (IP da LAN da máquina
  antiga). Trocar pelo IP desta máquina, ou deixar só `http://localhost:5173,http://localhost:3000`.
- **`AWS_HOST_CONFIG_DIR`** — está `/home/vinicius-silva/.aws`. Ajustar se o usuário/home for outro.
- **`DATABASE_URL` / `REDIS_URL`** — apontam para `postgres:5432` / `redis:6379` (nomes de
  container do compose). **Não mexer.**
- **`BASE_DOMAIN`, `DARWIN_BASE_URL`** — prod/externo. Deixar como estão para dev.
- **Microsoft Graph** (pendências da sessão — ver `HANDOFF.md` §5/§6):
  - `MS_GRAPH_REDIRECT_URI` = `http://localhost:8000/api/integracoes/outlook/callback/`
    (com `/api/` e barra final — o valor no `env.txt` está sem).
  - `MS_GRAPH_SCOPES` deve incluir `Calendars.ReadWrite`.
  - Para testar o espelho: `MS_GRAPH_CALENDAR_ENABLED=True`.

`.env` está no `.gitignore` do repo do Darwin — confirmar com `git check-ignore .env`.

---

## 4. Subir o stack

```bash
cd ~/github/darwin
make dev             # docker compose dev: build + up -d
make migrate         # aplica calendario 0003/0004, core 0008 e demais
make createsuperuser # só se o banco for novo
```

Portas dev padrão: API `8000`, frontend `5173`, Postgres `5432`, Redis `6379`.
Conflito de portas (ex.: projeto `joker` junto):
```bash
HOST_REDIS_PORT=6380 HOST_POSTGRES_PORT=5433 HOST_API_PORT=8009 HOST_FRONTEND_PORT=5174 \
  docker compose -f docker-compose.yml -f docker-compose.override.dev.yml up -d
```

Testar:
```bash
docker compose -f docker-compose.yml -f docker-compose.override.dev.yml exec darwin-backend pytest
# ou bare-metal: cd backend && pytest
```

---

## 5. Restaurar a memória do Claude e retomar

```bash
mkdir -p ~/.claude/projects/-home-<usuario>-github-darwin/memory
cp -a ~/github/mobile-teste/darwin/claude-memory/. \
      ~/.claude/projects/-home-<usuario>-github-darwin/memory/
```
(Na origem o caminho é `~/.claude/projects/-home-vinicius-silva-github-darwin/memory/`.
Ajustar `<usuario>` se o login for diferente.)

Depois, em `~/github/darwin`:
```bash
claude
```
E peça:

> "Leia o HANDOFF.md na raiz e o MEMORY.md em
>  ~/.claude/projects/-home-<usuario>-github-darwin/memory/.
>  Branch feat/calendario-espelho-microsoft, Fase 1 (espelho de entrada) concluída.
>  Continue de onde paramos."

---

## 6. Na volta (sincronizar o trabalho desta máquina)

Com acesso ao `origin`: `git push origin feat/calendario-espelho-microsoft`.
Sem acesso: gere um bundle novo e leve pela mesma carona:
```bash
cd ~/github/darwin
git bundle create ~/github/mobile-teste/darwin/darwin-feat-calendario-espelho-microsoft.bundle \
  feat/calendario-espelho-microsoft
cd ~/github/mobile-teste && git add darwin && git commit -m "chore: atualiza bundle do Darwin" && git push
```
