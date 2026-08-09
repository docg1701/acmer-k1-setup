# ACMER K1 Setup

Setup da gravadora a laser **ACMER K1 7W** com **Wine + LaserGRBL / ACMER Studio**
e servidor de impressão dedicado **printbox** (**Debian 13 + cncjs 1.11.2**).

## O que é

Este repositório documenta a configuração completa de uma ACMER K1 (laser diodo
azul 7W, 455 nm, 150×150 mm, GRBL) operada remotamente via servidor de
impressão. Tudo verificado contra o firmware da máquina (`$$`) e o manual oficial
da ACMER.

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
[Seu PC - Linux Mint]  Wine + LaserGRBL ou ACMER Studio — design e export G-code
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
| `docs/WINE-ACMER-STUDIO-K1.md` | Instalação do ACMER Studio V1.4.0 no Wine (testado e funcional) |
| `docs/BOTTLES-ACMER-STUDIO-K1.md` | Instalação do ACMER Studio V1.4.0 via Bottles (snapshot e backup) |
| `docs/WINE-SETUP-K1.md` | Instalação do LaserGRBL no Wine (alternativa) |
| `docs/MANUAL-SERVIDOR-K1.md` | Debian 13 + cncjs + systemd + webcam, passo a passo |
| `docs/ACMER-K1-User-Manual-EN.pdf` | Manual oficial ACMER — fonte primária das tabelas de potência/velocidade |

## Fluxo rápido

1. Gere o G-code no ACMER Studio (Wine): `docs/WINE-ACMER-STUDIO-K1.md`
2. Acesse `http://10.10.10.190:8000` → Upload → Run
3. Pode desligar o PC — o servidor segue executando

## Regras imutáveis

- Firmware da K1 (`$x=`) **não se altera** — valores de fábrica são corretos
- Potência/velocidade **sempre da tabela oficial 7W** (manual ACMER)
- Para contribuir neste repo, leia `AGENTS.md` antes

## Licença

Documentação — uso livre. PDF oficial ACMER: copyright ACMER.
