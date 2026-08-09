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

## 4. Papelão ondulado (caixa)

> Estimativa baseada em kraft paper 2,0mm. Papelão é mais denso e espesso (3–5mm).
> Testar grade de potência/velocidade para calibrar.

### Corte (M3)

| Espessura | Potência | Velocidade (mm/min) | Passes |
|---|---|---|---|
| 3mm | 100% | 150 | 1–2 |
| 4–5mm | 100% | 100 | 2–3 |

### Gravação (M4, 10 lines/mm)

| Profundidade | Potência | Velocidade (mm/min) | Passes |
|---|---|---|---|
| Superficial (marcação) | 30–40% | 6000–8000 | 1 |
| Média | 50–60% | 4000–5000 | 1 |
| Profunda | 70–80% | 2000–3000 | 1 |

---

## 5. Kraft paper — contorno vs preenchimento

| Operação | Comando | Potência | Velocidade (mm/min) |
|---|---|---|---|
| Corte (contorno) | M3 | 100% | 1500 |
| Gravação (preenchimento) | M4 | 50% | 8000 |
| Marcação de contorno (sem cortar) | M4 | 50% | 8000 |
