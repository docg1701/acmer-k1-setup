# AGENTS.md

## Contexto do projeto

Repositório de documentação da gravadora **ACMER K1 7W** e do servidor de
impressão **printbox** (Debian + cncjs). Idioma: **pt-BR**. Sem código —
apenas especificações de hardware, parâmetros de firmware, tabela de materiais
e configuração do servidor.

## Stack (fatos — não questionar)

- **Máquina**: ACMER K1 7W, GRBL 1.1h fork ACMER, USB serial `/dev/ttyACM0` 115200, sem WiFi
- **Servidor**: Debian 13 (console) + cncjs 1.11.2 (Node 20), porta 8000, hostname `printbox`
- **Webcam**: ustreamer 5.4, porta 8080, stream MJPEG consumido pelo widget do cncjs
- **Rede**: servidor IP fixo `10.10.10.190/24`, gateway `10.10.10.1`, hostname `printbox` / `printbox.local`
- **Usuário**: `galvani` (servidor)

## Source of truth

1. `docs/ACMER-K1-User-Manual-EN.pdf` — tabelas de potência/velocidade
2. Firmware da máquina (`$$`) — parâmetros de hardware
3. `docs/ESPECIFICACOES-K1.md` — consolida 1–2

## Regras absolutas

- `$x=` do firmware **nunca se altera**. Fábrica está correto.
- Potência/velocidade **sempre** da tabela 7W (manual oficial ACMER).
- **Nunca** propor troca de hardware (WiFi module, board swap) como solução primária.

## Convenções

- Tabela de materiais: fonte única no manual oficial ACMER
- Markdown: pipe tables, sem HTML
- PDF: `ACMER-K1-User-Manual-EN.pdf` (ASCII)

## Comandos

```bash
# Extrair texto do manual
pdftotext -layout docs/ACMER-K1-User-Manual-EN.pdf /tmp/k1_manual.txt

# Versão do cncjs no servidor
cncjs --version
```

## Limites

- Sem build/test/run — repo de documentação
- Não recomendar software de controle laser sem verificar manutenção recente
- **Nada de Wine, Bottles, LaserGRBL, ACMER Studio, Rayforge** — isso foi removido e não volta
