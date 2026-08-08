# ACMER K1 Setup

Setup validado da gravadora a laser **ACMER K1 7W** com **Rayforge 1.8.5** e
servidor de impressão dedicado (**Debian + cncjs**) para operar sem USB no PC
principal.

## A quem serve

Para quem tem uma ACMER K1 (laser diodo azul 7W, 455nm, 150×150mm, GRBL) e quer:

- Configurar o Rayforge corretamente (device, materiais, sanity check)
- Ter uma **tabela de parâmetros confiável** (potência/velocidade/passes por material)
- Montar um **servidor de impressão** (mini PC com Debian) para não depender do
  notebook conectado por USB durante os jobs

## Pré-requisitos

- ACMER K1 (qualquer potência; a tabela cobre 7W)
- PC com Linux e Flatpak instalado (Rayforge)
- Mini PC ou Raspberry Pi para o servidor (Debian netinst)
- Rede WiFi local

## Documentos

| Arquivo | Para quem | Conteúdo |
|---|---|---|
| `docs/MANUAL-RAYFORGE-K1.md` | Quem opera a K1 | Specs da máquina, Rayforge 1.8.5, tabela oficial 7W, bugs, workflow |
| `docs/MANUAL-SERVIDOR-K1.md` | Quem monta o servidor | Debian + cncjs + systemd, passo a passo do `dd` ao primeiro job |
| `docs/ACMER-K1-User-Manual-EN.pdf` | Referência | Manual oficial ACMER — fonte primária das tabelas |

## Stack

```
[PC] Rayforge (Flatpak) — desenho, steps, export G-code
 │
 │  WiFi — upload do arquivo
 │
[Servidor] Debian + cncjs — recebe .nc, streama via USB
 │
 │  USB serial 115200
 │
[K1] GRBL — executa
```

## Fluxo rápido

1. Configure o Rayforge (siga `MANUAL-RAYFORGE-K1.md`, Parte 4)
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
