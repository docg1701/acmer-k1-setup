# AGENTS.md

## Contexto do projeto

Repositório de documentação do setup da gravadora **ACMER K1 7W** com
**Rayforge 1.8.5** (Flatpak) e servidor de impressão **Debian + cncjs**.
Idioma: **pt-BR**. Sem código — apenas manuais, tabelas e configurações.

## Stack (fatos — não questionar)

- **Máquina**: ACMER K1 7W, GRBL 1.1h fork ACMER, USB serial `/dev/ttyACM0` 115200, sem WiFi
- **Design**: Rayforge 1.8.5 Flatpak (`org.rayforge.rayforge`), código-fonte em `/var/lib/flatpak/app/org.rayforge.rayforge/`
- **Servidor**: Debian 13 (console) + cncjs 1.11.2 (Node 20), porta 8000, hostname `printbox`
- **Rede**: servidor IP fixo `10.10.10.190/24`, gateway `10.10.10.1`, hostname `printbox` / `printbox.local`
- **Usuário**: `galvani` (servidor)

## Source of truth

1. `docs/ACMER-K1-User-Manual-EN.pdf` — tabelas de potência/velocidade
2. Firmware da máquina (`$$`) — parâmetros de hardware
3. Código-fonte do Rayforge 1.8.5 (Flatpak) — comportamento do software
4. Documentos deste repo (consolidam 1–3)

## Regras absolutas

- `$x=` do firmware **nunca se altera**. Fábrica está correto.
- Potência/velocidade **sempre** da tabela 7W (`MANUAL-RAYFORGE-K1.md`, Parte 5).
- Afirmações sobre Rayforge **só com código-fonte**. Citar versão em toda afirmação.
- **Nunca** sugerir bypass do sanity check — coordenadas negativas crasham a máquina ($21=1).
- **Nunca** propor troca de hardware (WiFi module, board swap) como solução primária.

## Bugs conhecidos (não re-diagnosticar)

- **"Job dependencies are not ready"** (v1.8.5): `step_stage.py` — workpiece sem fill para o step → ValueError → step DIRTY. Workaround: workpieces incompatíveis em camadas separadas.
- **Sanity check + coordenadas negativas**: geometria local pode ser negativa com canvas "ok". Workaround: centralizar tudo em coords positivas.
- **Thickness (mm)**: inerte no K1 (sem eixo Z). Só afeta preview 3D.
- **Homing stall pós-jog no cncjs**: máquina fica presa em `<Home>` após mover eixos manualmente e tentar homing. Reset: `$X` no console MDI ou power-cycle.

## Convenções

- Tabela de materiais: fonte única em `MANUAL-RAYFORGE-K1.md`, Parte 5
- Markdown: pipe tables, sem HTML
- PDF: `ACMER-K1-User-Manual-EN.pdf` (ASCII)
- Nomes Rayforge: Papel = kraft, Papelão = corrugado

## Comandos

```bash
# Extrair texto do manual
pdftotext -layout docs/ACMER-K1-User-Manual-EN.pdf /tmp/k1_manual.txt

# Versão do cncjs no servidor
cncjs --version

# Código-fonte do Rayforge
find /var/lib/flatpak/app/org.rayforge.rayforge -name "*.py" -path "*/rayforge/*"
```

## Limites

- Sem build/test/run — repo de documentação
- Não recomendar software de controle laser sem verificar manutenção recente (LaserWeb4: estagnado desde 2018)
