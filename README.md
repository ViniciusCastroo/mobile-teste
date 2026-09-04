# Transporte do contexto NeoToken S3 (joker)

Repo temporário só pra levar a branch pra outra máquina (as duas não se
enxergam na rede, mas ambas chegam no GitHub).

- `joker-neotoken.bundle` — a branch `wip/neotoken-s3-frontend` do repo `joker`,
  histórico completo, self-contained.
- `claude-memory/` — notas de memória do Claude Code desta sessão.
- `LEIA-ME.md` — passo a passo de restauração.

## Restaurar na outra máquina

    git clone https://github.com/ViniciusCastroo/mobile-teste.git
    cd mobile-teste

    # repo joker JÁ existe lá:
    cd ~/github/joker   # ajuste o caminho
    git fetch ~/github/mobile-teste/joker-neotoken.bundle \
        wip/neotoken-s3-frontend:wip/neotoken-s3-frontend
    git checkout wip/neotoken-s3-frontend

    # repo joker NÃO existe lá:
    cd ~/github
    git clone ~/github/mobile-teste/joker-neotoken.bundle joker
    cd joker && git checkout wip/neotoken-s3-frontend

Depois: recriar o `.env` (NÃO está aqui — segredos), `docker compose up -d --build`,
`make load-names`, e pedir pro Claude ler o `HANDOFF.md` (está na branch).
