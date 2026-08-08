# ACMER K1 Setup

Setup validado da gravadora a laser **ACMER K1 7W** com **Rayforge 1.8.5** e
servidor de impressão dedicado **printbox** (**Debian 13 + cncjs 1.11.2**).

## O que é

Este repositório documenta a configuração completa de uma ACMER K1 (laser diodo
azul 7W, 455 nm, 150×150 mm, GRBL) operada remotamente via servidor de
impressão. Tudo verificado contra o firmware da máquina (`$$`), o manual oficial
da ACMER e o código-fonte do Rayforge 1.8.5.

## Servidor printbox

Mini PC com Debian 13 console, conectado à K1 por USB e acessível via WiFi na
rede local.

| Serviço | Porta | Função |
|---|---|---|
| cncjs 1.11.2 | 8000 | Web UI, upload e stream de G-code |
| ustreamer 5.4 | 8080 | Stream MJPEG da webcam USB, widget do cncjs |
| Avahi | — | `printbox.local` |

| Configuração | Valor |
|---|---|
| IP fixo | `10.10.10.190/24` |
| Gateway | `10.10.10.1` |
| DNS | `8.8.8.8`, `8.8.4.4` |
| Interface WiFi | `wlp1s0` |
| Power save | off (serviço systemd) |
| Serial K1 | `/dev/ttyACM0`, 115200 baud |
| Usuário | `galvani` (grupos: dialout, tty, sudo, netdev) |
| SO | Debian 13.6, kernel 6.12 |

## Stack

```text
[PC] Rayforge (Flatpak) — desenho, steps, export G-code
 │
 │  WiFi — upload do arquivo
 │
[printbox] Debian 13 + cncjs + ustreamer
 │  ├── cncjs :8000 — recebe .nc, streama via USB
 │  └── ustreamer :8080 — webcam
 │
 │  USB serial 115200
 │
[K1] GRBL — executa
```

## Documentos

| Arquivo | Conteúdo |
|---|---|
| `docs/MANUAL-RAYFORGE-K1.md` | Specs da máquina, Rayforge 1.8.5, tabela oficial 7W, bugs conhecidos, workflow |
| `docs/MANUAL-SERVIDOR-K1.md` | Debian 13 + cncjs + systemd + webcam, passo a passo do `dd` ao primeiro job |
| `docs/ACMER-K1-User-Manual-EN.pdf` | Manual oficial ACMER — fonte primária das tabelas de potência/velocidade |

## Fluxo rápido

1. Configure o Rayforge (`MANUAL-RAYFORGE-K1.md`, Parte 4)
2. Desenhe, configure steps e **Export G-code**
3. Acesse `http://10.10.10.190:8000` → Upload → Run
4. Pode desligar o PC — o servidor segue executando

## Regras imutáveis

- Firmware da K1 (`$x=`) **não se altera** — valores de fábrica são corretos
- Potência/velocidade **sempre da tabela oficial 7W** (manual Parte 5)
- Coordenadas de projeto **sempre positivas** (sanity check do Rayforge)
- Para contribuir neste repo, leia `AGENTS.md` antes

## Licença

Documentação — uso livre. PDF oficial ACMER: copyright ACMER.
