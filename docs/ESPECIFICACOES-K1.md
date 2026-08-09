# ACMER K1 7W — Especificações de Hardware, Firmware e Materiais

> **Hardware**: `MANUAL-RAYFORGE-K1.md` (git, 2026-08-08) + manual oficial ACMER.
> **Parâmetros `$$`**: dump do firmware da máquina (2026-08-08), validado campo a campo.
> **Materiais**: `ACMER K1-User Manual（English）.pdf`, seção 7 — **7W Compressed Spot**.

---

## 1. Hardware

| Especificação | Valor |
|---|---|
| Tipo de laser | Diodo azul 455 nm (±5 nm), beam-combining |
| Potência | 7W |
| Área de trabalho | 150 × 150 mm |
| Tamanho do spot | 0,08 × 0,08 mm (anunciado); 0,06 mm medido |
| Precisão de repetição | 0,01 mm |
| Velocidade máx. de gravação | 10.000 mm/min |
| Dimensões | 286 × 273 × 190 mm |
| Peso | 2,4 kg |
| Classe de segurança | Classe 1 (fechada — sem óculos com tampa fechada) |
| Conectividade | USB / compatível com GRBL |
| Baud rate | 115200 |
| Device serial | `/dev/ttyACM0` |
| Ruído | ~59–61 dB |
| Firmware | Grbl 1.1h (fork ACMER) |

### Segurança

- **Sensor de tampa**: abrir pausa o trabalho em ~1s; fechar retoma
- **Sensor de inclinação**: derruba → desliga antes de causar problema
- **Ventilador de exaustão embutido**: reduz fumaça/odor
- **Air assist**: não tem de fábrica; furo lateral na carcaça para mangueira

### Foco

1. Colocar folha de medição sobre o material
2. Soltar botão lateral
3. Deslizar módulo laser até o bico tocar a folha
4. Apertar botão

Bico protetor magnético (fácil remover para limpeza).

---

## 2. Parâmetros GRBL (dump `$$`)

> Valores de fábrica — **não se alteram**. Referência para configurar qualquer
> software que converse com o firmware da K1.

| Parâmetro | Valor | Descrição |
|---|---|---|
| `$20` | 0 | Soft limits OFF |
| `$21` | 1 | Hard limits ON |
| `$22` | 1 | Homing cycle enable |
| `$23` | 3 | Homing direction invert (origem canto inf. esq.) |
| `$30` | 1000 | S-value máx (PWM) |
| `$31` | 0 | S-value mín (PWM) |
| `$32` | 1 | Laser mode ON |
| `$100` | 80 | Steps/mm — eixo X (1 passo = 0,0125 mm) |
| `$101` | 80 | Steps/mm — eixo Y (1 passo = 0,0125 mm) |
| `$110` | 10000 | Max rate — eixo X (mm/min) |
| `$111` | 10000 | Max rate — eixo Y (mm/min) |
| `$120` | 300 | Aceleração — eixo X (mm/s²) |
| `$121` | 300 | Aceleração — eixo Y (mm/s²) |
| `$130` | 150 | Max travel — eixo X (mm) |
| `$131` | 150 | Max travel — eixo Y (mm) |

### Notas sobre o firmware

- **Grbl 1.1h (fork ACMER)**: suporta G5 bezier e parâmetros extras `$41–$48`
- **G-code precision recomendada**: 3 casas decimais (12× mais fino que o passo da máquina)
- **Arcos**: G2/G3 suportados nativamente
- **Limites**: hard limits ON protege contra crash em fim de curso
- **Homing**: ao ligar, a máquina faz homing automático; se houver objeto no caminho, pode crashar — fazer homing manual com tampa aberta e área vazia (`$H` no console)
- **$3 já trata inversão de direção dos eixos** — não usar Reverse no software para compensar direção

---

## 3. Tabela de parâmetros — materiais (7W Compressed Spot)

### Corte (M3)

| Material | Potência | Velocidade (mm/min) | Passes |
|---|---|---|---|
| Kraft paper 0,5mm | 100% | 1500 | 1 |
| Kraft paper 1,0mm | 100% | 1000 | 1 |
| Kraft paper 2,0mm | 100% | 300 | 1 |
| Plywood 2,0mm | 100% | 200 | 1 |
| Solid wood 2,0mm | 95% | 200 | 2 |
| Bamboo 2,0mm | 95% | 150 | 1 |
| Red Acrylic 1,0mm | 100% | 150 | 1 |
| Red Acrylic 2,0mm | 100% | 100 | 1 |
| Black Acrylic 1mm | 100% | 150 | 1 |
| Black Acrylic 2mm | 100% | 100 | 1 |
| Light-colored Felt 1mm | 80% | 1500 | 1 |

### Gravação (M4, 10 lines/mm)

| Material | Potência | Velocidade (mm/min) | Passes |
|---|---|---|---|
| Kraft paper | 50% | 8000 | 1 |
| Plywood | 50% | 6000 | 1 |
| Solid wood | 80% | 3500 | 1 |
| Bamboo | 80% | 4500 | 1 |
| Cork | 60% | 5000 | 1 |
| Leather | 50% | 8000 | 1 |
| Light-colored Felt | 50% | 8000 | 1 |
| Dark Felt | 50% | 8000 | 1 |
| Transparent Acrylic* | 70% | 6000 | 1 |
| Glass* | 100% | 2000 | 1 |
| Silica gel | 50% | 2000 | 1 |
| Cobblestone | 90% | 80 | 1 |
| Ceramics | 100% | 2000 | 1 |
| Black alumina | 80% | 7000 | 1 |
| Tin plate | 90% | 3000 | 1 |
| Stainless steel (matte) | 90% | 200 | 1 |
| Stainless steel (smooth) | 90% | 150 | 2 |

\* Requer pintura preta prévia na superfície.

---

## 4. Papelão ondulado (caixa) — corrugado 4mm

> **Não consta na tabela oficial ACMER.** Valores abaixo = extrapolação da
> tabela 7W (kraft paper 2mm sólido: 100% @ 300 mm/min) + dados comunitários
> de máquinas com laser azul ~5.5–7W (Ortur). Corrugado tem 2 camadas finas
> de liner + miolo vazado → **menos fibra que papel sólido da mesma
> espessura** → corta mais rápido que kraft 2mm.
>
> ⚠️ Validar com grade de teste antes de job real. Se não atravessar:
> **reduzir velocidade ou 2 passes** — nunca subir potência acima de 100% e
> cuidado com queima (papel/celulose pega fogo fácil em velocidade baixa).

### Configuração ACMER Studio — material custom "Papelão corrugado 4mm"

| Operação | Power% | Speed (mm/min) | Passes | Laser |
|---|---|---|---|---|
| **Line** | 35 | 5000 | 1 | M4 |
| **Fill** | 50 | 8000 | 1 | M4, 10 lines/mm |
| **Cut** | 100 | 450 | 1 | M3 |

Faixa de teste do Cut: **350–600 mm/min** (450 = meio-termo entre kraft 2mm
sólido @ 300 e corrugado 3mm Ortur @ 500).

### Referências comunitárias (diodo azul, corrugado)

| Fonte | Laser | Material | Operação | Speed (mm/min) | Power | Passes |
|---|---|---|---|---|---|---|
| Ortur oficial | 5.5W | Corrugado 3mm | Cut | 500 | 100% | 1 |
| ZapCraft | 5W | 1–3mm | Cut | 330 | 70% | 2 |
| ZapCraft | 10W | 1–3mm | Cut | 600 | 70% | 1 |
| Atomstack A20 | 20W | Cardboard | Cut | 800 | 50% | 1 |

Fontes: <https://ortur.net/pages/materials-reference> ·
<https://zapcraft.net/material-settings/cardboard-laser-settings-starting-points/> ·
<https://www.bonnycreations.com/settings/materials/corrugated-cardboard>

---

## 5. Kraft paper — contorno vs preenchimento

| Operação | Comando | Potência | Velocidade (mm/min) |
|---|---|---|---|
| Corte (contorno) | M3 | 100% | 1500 |
| Gravação (preenchimento) | M4 | 50% | 8000 |
| Marcação de contorno (sem cortar) | M4 | 50% | 8000 |

---

## 6. Termoplásticos impressos 3D (PLA, ABS, PETG)

> **Não constam na tabela oficial ACMER.** Valores = extrapolação da tabela
> 7W (acrílico preto/vermelho — termoplástico mais próximo: 1mm @ 150,
> 2mm @ 100 mm/min) + dados comunitários (xTool M1/F1, Batch Studio,
> ComMarker). ⚠️ **Validar com grade de teste** antes de job real.
>
> ⚠️ **Fumaça tóxica**: ABS libera estireno (e cianeto em degradação
> térmica); PLA e PETG liberam VOCs. **Exaustão obrigatória** — o gabinete
> da K1 contém a fumaça mas **não filtra**. ABS: só com exaustão boa.

### 6.1. Regras gerais (por que cor escura)

- Diodo azul 455nm: **só cores escuras absorvem**. Teste xTool F1 (diodo
  azul 10W): PLA branco = nenhum efeito; PLA cinza = derretimento irregular;
  **preto/vermelho = grava limpo**.
- **PLA**: ponto de fusão baixo (150–180°C) — potência alta derrete e
  borbulha em vez de gravar. Sempre velocidade alta + potência baixa.
- **Peça impressa**: imprimir com a face a gravar no bottom e **≥6 camadas
  sólidas** nela (senão o laser atravessa o infill). Layer lines deixam a
  gravação com profundidade irregular — gravação rasa é mais limpa.

### 6.2. PLA escuro — ACMER Studio

| Operação | Power% | Speed (mm/min) | Passes | Laser |
|---|---|---|---|---|
| **Line** | 35 | 2000 | 1 | M4 |
| **Fill** | 30 | 4000 | 1 | M4, 10 lines/mm |
| **Cut 1mm** | 100 | 200 | 1 | M3 |
| **Cut 2mm** | 100 | 120 | 1–2 | M3 |
| **Cut 3mm** | 100 | 80 | 2 | M3 |

Teste Fill: 2000–6000 mm/min (derreteu → sobe velocidade). Cut: se não
atravessar, reduz velocidade — nunca acima de 100%. **4–5mm: corte não é
realista com 7W** (muitas passes, borda derretida) — gravar ok, cortar
não.

### 6.3. ABS escuro — ACMER Studio

| Operação | Power% | Speed (mm/min) | Passes | Laser |
|---|---|---|---|---|
| **Line** | 30 | 2000 | 1 | M4 |
| **Fill** | 25 | 3000 | 1 | M4, 10 lines/mm |
| **Cut 1mm** | 100 | 150 | 1 | M3 |
| **Cut 2mm** | 100 | 80 | 1–2 | M3 |

> **ABS derrete antes de queimar** (consenso comunitário: Bambu Lab forum,
> Batch Studio) — corte acima de 2mm vira gosma. Prefira gravação; corte só
> fino. Fumaça: estireno — exaustão forte.

### 6.4. PETG — ACMER Studio

| Operação | Power% | Speed (mm/min) | Passes | Laser |
|---|---|---|---|---|
| **Line** | 30 | 2000 | 1 | M4 |
| **Fill** | 25 | 4000 | 1 | M4, 10 lines/mm |
| **Cut 1mm** | 100 | 150 | 1 | M3 |

> PETG absorve pior o 455nm e derrete formando rebarba — corte só fino
> (1mm). Fumaça: VOCs.

### 6.5. Fontes

- <https://www.batchmade.studio/laser-engrave-3d-prints> (xTool F1: cores
  PLA/ABS)
- <https://blog.commarker.com/archives/57631> (PLA: fusão 150–180°C,
  velocidade alta + potência baixa)
- <https://xtool.zendesk.com/hc/en-us/articles/15017421813911> (acrílicos,
  referência de termoplásticos)
- <https://forum.bambulab.com/t/laser-engraving-3d-printed-parts/74296>
  (ABS derrete antes de queimar)
- <https://lahobbyguy.com/bb/viewtopic.php?t=24> (Ortur 7W: evitar 100%
  em engrave)
