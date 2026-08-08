# ACMER K1 Setup

Configuração validada da gravadora ACMER K1 (laser diodo azul 7W, 455nm, área 150×150) com Rayforge 1.8.5, e servidor de impressão dedicado (Debian + cncjs) para operar sem cabo USB no PC principal.

**Tudo aqui foi verificado contra o firmware da máquina (`$$`), o manual oficial da ACMER e o código-fonte do Rayforge 1.8.5.** Nada é chute.

## Documentos

| Arquivo | Conteúdo |
|---|---|
| `docs/MANUAL-RAYFORGE-K1.md` | Configuração completa do Rayforge para a K1: device settings, materiais, tabela oficial 7W (corte/gravação), workflow de plaquinha, bugs conhecidos da 1.8.5 |
| `docs/MANUAL-SERVIDOR-K1.md` | Servidor de impressão: Debian + WiFi + cncjs + USB, fluxo "desenha no PC → servidor executa → PC desliga" |
| `docs/ACMER-K1-User-Manual-EN.pdf` | Manual oficial da ACMER (fonte primária das tabelas de potência/velocidade) |

## Stack

- **Máquina**: ACMER K1 7W — GRBL, USB serial 115200 (`/dev/ttyACM0`), sem WiFi de fábrica
- **Design/CAM**: Rayforge 1.8.5 (Flatpak `org.rayforge.rayforge`) — steps, potência, sanity check, export G-code
- **Servidor de impressão**: Debian (netinst, console) + cncjs 1.11.x — web UI, streama o job pelo USB local, PC pode desligar
- **Rede**: WiFi entre PC e servidor usada apenas para o upload do arquivo

## Fluxo rápido

1. Desenhe e configure no Rayforge (potência/velocidade = tabela oficial 7W)
2. `File → Export G-code...`
3. Upload no cncjs (`http://<servidor>:8000`) → Run
4. Desligue o PC — o servidor executa e você acompanha pelo browser

## Regras que não devem ser quebradas

- **Nunca alterar valores de firmware `$x=`** — os de fábrica estão corretos
- **Nunca inventar potência/velocidade** — usar a tabela oficial 7W (Parte 5 do manual)
- **Geometria sempre em coordenadas positivas** no projeto (sanity check da 1.8.5 falha com valores negativos)
- Ver `AGENTS.md` para o contexto completo de trabalho neste repositório.
