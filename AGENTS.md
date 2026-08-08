# AGENTS.md

## Purpose

Documentação validada do setup da gravadora **ACMER K1** (laser diodo azul 7W,
455nm, área 150×150, firmware GRBL) com **Rayforge 1.8.5** e servidor de
impressão **Debian + cncjs**. Este repositório **não contém código** — apenas
manuais e configurações. Idioma dos documentos: **pt-BR**.

## Project facts (não re-verificar sem motivo)

- Máquina: ACMER K1 7W — GRBL, USB serial 115200 (`/dev/ttyACM0`), **sem WiFi de fábrica**
- Software de design: Rayforge 1.8.5 via Flatpak (`org.rayforge.rayforge`) — código-fonte local: `/tmp/rayforge-1.8.5/`
- Servidor: Debian netinst (console puro) + **cncjs 1.11.x** (Node), web UI porta 8000
- Tabela oficial de potência/velocidade: manual ACMER, seção "7W Compressed Spot" (PDF em `docs/`)

## Source of truth (nesta ordem)

1. **Manual oficial ACMER** (`docs/ACMER-K1-User-Manual-EN.pdf`) — potência/velocidade/passes
2. **Firmware da máquina (`$$`)** — parâmetros de hardware (nunca alterar `$x=`)
3. **Código-fonte do Rayforge 1.8.5** — comportamento do software/UI
4. Documentos deste repo (que consolidam 1–3)

## Regras absolutas

- **NUNCA alterar valores de firmware `$x=`** — fábrica está correto; Rayforge apenas lê
- **NUNCA inventar potência/velocidade** — usar a tabela oficial 7W (MANUAL-RAYFORGE-K1.md, Parte 5)
- **NUNCA afirmar comportamento do Rayforge sem conferir o código-fonte** — versões mudam; citar a versão em toda afirmação
- **NUNCA sugerir "passar por cima" do sanity check** — coordenadas negativas = risco de crash da máquina ($21=1)

## Bugs conhecidos do Rayforge 1.8.5 (documentados — não re-diagnosticar)

- **"Job dependencies are not ready"**: `step_stage.py` — workpiece "vazio" para um step trava o pipeline silenciosamente. Workaround: workpieces distintas em camadas separadas
- **Sanity check com coordenadas negativas**: geometria local do sketch pode ser negativa mesmo com canvas "ok". Workaround: centralizar/posicionar tudo em coords positivas
- **Campo thickness (mm)**: inert no K1 (sem eixo Z) — só afeta preview 3D

## Convenções de edição

- DRY: tabela oficial de materiais vive em **um** lugar (MANUAL-RAYFORGE-K1.md, Parte 5) — atualizar lá
- Manter tabelas Markdown simples (pipe tables), sem HTML
- PDF oficial: nome ASCII (`ACMER-K1-User-Manual-EN.pdf`)
- Nomes de materiais no Rayforge: Papel = kraft; Papelão = corrugado

## Comandos

```bash
# Extrair texto do manual oficial (para busca)
pdftotext -layout docs/ACMER-K1-User-Manual-EN.pdf /tmp/k1_manual.txt

# Verificar versão do cncjs no servidor
cnc -V
```

Sem build/test/run — repositório de documentação.

## Limites

- Não propor mudanças de hardware (WiFi module, board swap) como solução primária — o K1 é GRBL/USB e o setup cncjs já resolve
- Não recomendar ferramentas de controle laser sem verificar manutenção/releases atuais (ex.: LaserWeb4 está estagnado desde 2018 — não usar)
