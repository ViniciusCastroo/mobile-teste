# Restaurar o contexto NeoToken S3 em outra máquina (sem GitHub)

Copie esta pasta inteira pra outra máquina (scp / pendrive / rsync).

## 1. Trazer a branch pro repo de lá
No diretório do repo joker (que já tem o histórico até origin/main):

    git fetch /caminho/para/joker-neotoken.bundle wip/neotoken-s3-frontend:wip/neotoken-s3-frontend
    git checkout wip/neotoken-s3-frontend

(se o repo ainda não existir lá:  git clone joker-neotoken.bundle joker  — clona a branch inteira)

## 2. Colocar o .env
    cp env.txt /caminho/para/joker/.env
Depois edite no .env, trocando o IP: ALLOWED_HOSTS, CORS_ALLOWED_ORIGINS,
VITE_API_BASE_URL, S3_VITE_API_BASE_URL, SHARES_PUBLIC_BASE_URL  ->  IP da máquina nova.

## 3. Memória do Claude (opcional)
    cp -r claude-memory/* ~/.claude/projects/-home-<user>-github-joker/memory/
O nome da pasta em ~/.claude/projects/ é derivado do caminho do repo. Se o repo
estiver em /home/<user>/github/joker lá também, é -home-<user>-github-joker.
Se não achar/não bater, tudo bem — o HANDOFF.md no repo cobre o essencial.

## 4. Subir e testar
    docker compose -p darwin down        # se aplicável
    docker compose up -d --build
    make load-names                      # carrega o vocabulário de nomes (ibge_names)

Abra uma sessão do Claude Code no diretório do repo e peça pra ele ler o HANDOFF.md
(está versionado na branch) — tem toda a direção de design, o que foi feito, o que falta.
