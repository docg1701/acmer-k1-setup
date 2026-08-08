# Manual: Gravadora a Laser ACMER K1 + Software Rayforge

> Pesquisa compilada em 2026-08-08 a partir de fontes oficiais (acmerlaser.com,
> rayforge.org, GitHub) e reviews independentes. Ver [Fontes](#fontes) no final.

---

## Parte 1 — ACMER K1 (Gravadora a Laser Portátil Fechada)

### 1.1 O que é

A **ACMER K1** é uma gravadora/cortadora a laser de diodo azul, **totalmente
fechada (Classe 1)**, portátil (2,4 kg) e de baixo custo (US$ 199–359 conforme
a potência). É voltada para iniciantes, DIY, artesanato e pequenos negócios.

### 1.2 Especificações técnicas

| Especificação | Valor |
|---|---|
| Tipo de laser | Diodo azul 455 nm (±5 nm), com beam-combining |
| Opções de potência | 2.5W / 3.5W / 7W / 12W |
| Área de gravação | 150 × 150 mm (5,9 × 5,9 pol) |
| Tamanho do spot | 0,08 × 0,08 mm (anunciado); 0,06 mm medido em teste |
| Precisão de repetição | 0,01 mm |
| Velocidade máx. de gravação | 10.000 mm/min |
| Dimensões | 286 × 273 × 190 mm |
| Peso | 2,4 kg (5,3 lb) |
| Classe de segurança | Classe 1 (fechada — sem necessidade de óculos com a tampa fechada) |
| Conectividade | USB / compatível com GRBL |
| Ruído | ~59–61 dB |
| Materiais | Madeira, bambu, papel, couro, acrílico escuro, ardósia, alumínio anodizado |
| Software compatível | AcmerTool (oficial), LightBurn, LaserGRBL, **Rayforge** (GRBL) |
| Sistemas operacionais | Windows, macOS, Linux |
| Arquivos suportados | NC, DXF, BMP, JPG, PNG, SVG etc. |

### 1.3 Segurança (diferencial do produto)

- **Classe 1 fechada**: a luz nociva fica contida; **não precisa de óculos de
  proteção** durante a operação com a tampa fechada (os óculos incluídos são
  úteis para manutenção/limpeza da lente).
- **Sensor de tampa**: abrir a tampa pausa o trabalho em ~1 segundo; fechar
  retoma de onde parou.
- **Sensor de inclinação**: derrubou a máquina, ela desliga antes de causar
  problema.
- **Ventilador de exaustão embutido**: reduz fumaça e odor; para uso indoor
  intenso, recomenda-se o purificador de ar da ACMER (US$ 309) ou exaustão
  para janela.

### 1.4 Foco

Processo simples, com folha de medição de distância focal incluída:

1. Coloque a folha sobre o material.
2. Solte o botão lateral.
3. Deslize o módulo laser para baixo até o bico tocar a folha.
4. Aperte o botão. Pronto.

O bico protetor é magnético (fácil de remover para limpeza). **Não há air
assist de fábrica** — há um furo lateral na carcaça por onde dá para passar
uma mangueira e improvisar (compressor da ACMER custa US$ 59).

### 1.5 O que vem na caixa

Máquina, módulo laser, fonte, cabo USB, óculos de segurança, um pedaço de
basswood (madeira de teste), dois cartões de alumínio revestido, papel kraft
e chaves hex. Montagem em minutos: encaixar o módulo, conectar o cabo e
gravar (funciona em menos de 5 minutos após abrir a caixa).

### 1.6 Resultados por material (teste com 7W, review Hoffman)

| Material | Resultado |
|---|---|
| Madeira (compensado de bétula 3 mm) | Corte consistente a ~200 mm/min (lento por falta de air assist); gravação forte, até 10.000 mm/min com bom detalhe em fotos |
| Couro | Destaque — limpo, profundidade uniforme, quase sem escurecimento |
| Acrílico preto | Corta bem (diodo azul não corta acrílico transparente) |
| Ardósia | Gravação nítida e consistente |
| Alumínio anodizado | Difícil de obter contraste confiável |
| Aço inox | Apenas marcações cinza-claras (sem cor; precisa de mais potência) |
| Papel, bambu, MDF fino | Bem |

O 12W corta até **8 mm de madeira Paulownia em uma passada**.

### 1.7 Software da ACMER (AcmerTool / ACMER Studio)

- Software oficial e gratuito da ACMER, otimizado para o K1.
- Faz conexão do dispositivo, seleção de modo de gravação, edição gráfica e
  parâmetros em um só lugar.
- **Fraquezas conhecidas (reviews)**: preview em baixa qualidade (JPEG),
  otimização de percurso ruim (grava meio caractere, pula para outro e
  volta), tempo restante exibido em decimal (ex.: 3.85 min), sem menu para
  alterar parâmetros da máquina.
- Recomendação dos reviewers: para uso sério, usar LightBurn ou Rayforge.

### 1.8 Drivers e firmware

Página oficial de downloads: `https://acmerlaser.com/pages/download`

- **Drivers K1** (Google Drive): `https://drive.google.com/drive/folders/1l_SdpABXqe01gIdmSW0AkYbXRfl3Nd_K`
- **Firmware K1** (Google Drive): `https://drive.google.com/drive/folders/1M5pc4m_38LjX32b6TZgqO_e4kc5XL02h`

### 1.9 Pontos fortes e fracos (resumo de reviews)

### Prós

- Encaixotada Classe 1 — segura para casa, sem óculos obrigatórios.
- Portátil (2,4 kg), montagem instantânea.
- Barata para o que oferece (7W: US$ 299; 12W: US$ 359; 3.5W: US$ 259; 2.5W: US$ 199).
- Silenciosa (59–61 dB), LED interno ilumina a área de trabalho.
- Compatível com LightBurn/LaserGRBL/Rayforge (padrão GRBL).

### Contras

- Área de trabalho pequena (150 × 150 mm).
- Sem air assist de fábrica → corte mais lento e bordas piores que lasers
  abertos de mesma potência.
- Sem fim de curso máximo (dá para "ranger" as correias).
- Botão de emergência na traseira.
- Sem câmera, detecção de chama ou air assist inclusos.
- Review relatou **ondulações no eixo Y** em uma unidade (defeito de QC —
  rodar teste de linhas nos dois eixos antes de projetos reais).

### 1.10 Preços de referência (2026)

- K1 2.5W — US$ 199
- K1 3.5W — US$ 259
- K1 7W — US$ 299
- K1 12W — US$ 359
- Compressor air assist — US$ 59
- Purificador de ar — US$ 309

---

## Parte 2 — Rayforge (Software livre de controle a laser)

### 2.1 O que é

Rayforge é um **software livre e de código aberto** (MIT) de CAD 2D, gerador
de G-code e controle de máquina para gravadoras/cortadoras a laser baseadas
em **GRBL, Marlin, Ruida e Smoothieware** (e OctoPrint). Feito com GTK4 e
Libadwaita, com interface nativa para **Linux, macOS e Windows**. Gratuito —
diferente do LightBurn (pago) e mais completo que o AcmerTool.

- Site: <https://rayforge.org>
- Código: <https://github.com/barebaric/rayforge>
- Discord: <https://discord.gg/sTHNdTtpQJ>
- Autor: Samuel Abels (knipknap)
- Idiomas: inglês, português, espanhol, alemão, francês, ucraniano e chinês

### 2.2 Versões (agosto/2026)

- **Estável atual: 1.8.5** (Flatpak/Flathub; é a versão em uso — Flatpak 1.8).
- Em beta: **1.9.0-beta4** (provavelmente a última beta antes do release final;
  reescrita do canvas 3D, unidade métrica/imperial, importação de SVG por
  cores, assistente de configuração de máquina com busca AI de specs).
- Snap Store: 1.8.x (canal stable).

### 2.3 Funcionalidades principais

### Design & edição

- Editor de sketch paramétrico (restrições geométricas e dimensionais).
- Canvas 2D completo: alinhamento, transformação, medição, zoom, pan.
- Operações por camada (ex.: gravar e depois cortar no mesmo job).
- Sistema de matéria-prima (stock) com espessura e material.
- Undo/redo completo.
- Importa **SVG, DXF, PDF, JPEG, PNG, BMP, Ruida (.rd) e LightBurn (.lbdev)**;
  exporta SVG e DXF. Projeto próprio em `.ryp` (formato compactado).

### Operações & toolpaths

- Contorno (corte), gravação raster (com preenchimento cross-hatch),
  shrink wrap, gravação em profundidade (2.5D), e frame.
- Corte multi-passe com step-down configurável.
- **Suporte a 4º eixo (rotativo)** — modo eixo verdadeiro ou substituição
  de eixo para máquinas hobbistas.
- **Simulação 3D animada** (playback com scrubber e velocidade 1x–16x).
- Holding tabs (guias de fixação) manuais ou automáticas.
- Overscan e compensação de kerf (largura do corte).
- Dithering Floyd-Steinberg e Bayer para raster de alta qualidade.
- Pós-processadores: lead-in/lead-out, merge de linhas sobrepostas, crop
  ao limite do stock.
- Otimização de percurso (travel time), suavização de caminhos, interpolação
  por tamanho do spot.

### Controle de máquina

- Perfis multi-máquina com troca instantânea.
- 6 sistemas de coordenadas de trabalho (G54–G59) por camada.
- **No-go zones** (áreas proibidas) com detecção de colisão.
- Contador de horas de máquina com lembretes de manutenção.
- Leitura/escrita de parâmetros do firmware GRBL (`$$`) direto da UI.
- Curvas de arco (G2/G3) e bezier (G5) com linearização automática.
- Múltiplos lasers por job.
- Dialetos G-code: GRBL, Smoothieware, Marlin, LinuxCNC, Mach4 + editor
  de dialetos customizados.
- Macros e hooks G-code (antes/depois do job, com substituição de variáveis).
- Pre-flight checks: limites, área de trabalho, no-go zones.
- Console G-code interativo com syntax highlighting.

### Materiais & presets

- Biblioteca com 60+ materiais embutidos + bibliotecas do usuário.
- Sistema de **receitas** (presets) que casam automaticamente material,
  espessura, máquina e cabeçote.
- **Material Test Grid**: grade de teste potência × velocidade para achar
  os parâmetros ideais (com opção air assist e calibração de offset
  bidirecional).

### Workflow & automação

- Integração de **câmera USB** (alinhamento, posicionamento, calibração
  fisheye).
- **Gerador de workpiece por IA** (prompt de texto → SVG) via provedores
  compatíveis com OpenAI.
- Alinhamento print & cut (marcas de registro) com wizard guiado.
- **Modo CLI/headless** para automação em lote.
- Modo projetor (projeta toolpaths na mesa da máquina).

### Plataforma

- Interface GTK4/Libadwaita com temas system/light/dark.
- Sistema de addons (gerenciador embutido, comunidade).
- Suporte a dispositivos: GRBL (serial, telnet, rede WiFi/Ethernet),
  Smoothieware (telnet), Marlin (serial), Ruida (UDP), OctoPrint (HTTP API).
- Update checker automático via GitHub Releases.

### 2.4 Instalação — Flatpak (a versão do usuário, 1.8)

Fonte: <https://flathub.org/apps/org.rayforge.rayforge> (ID: `org.rayforge.rayforge`)

```bash
# Instalar
flatpak install flathub org.rayforge.rayforge

# Rodar
flatpak run org.rayforge.rayforge
```

- Arquiteturas: x86_64 e aarch64.
- Para acesso à porta serial (K1 via USB), o Flatpak precisa de permissão de
  dispositivo. Se a máquina não aparecer, garanta que o usuário está no grupo
  `dialout` e que o Flatpak tem acesso ao dispositivo serial (via portal
  `org.freedesktop.portal.Device`/USB ou `--device=all` se necessário).

> Nota: com Flatpak, permissões de serial/hotplug funcionam de forma
> diferente do Snap — no Snap há o passo explícito
> `sudo snap connect rayforge:serial-port`. No Flatpak, verifique o acesso
> ao device `/dev/ttyUSB*`/`/dev/ttyACM*` do seu sistema.

### 2.5 Instalação — outras formas

**Snap (Linux):**

```bash
sudo snap install rayforge
sudo usermod -a -G dialout $USER   # Debian-based; relogar depois
sudo snap set system experimental.hotplug=true
sudo snap connect rayforge:serial-port
sudo snap connect rayforge:camera   # opcional
snap connections rayforge           # verificar
```

**Debian/Ubuntu (PPA):**

```bash
# Launchpad PPA: https://launchpad.net/~knipknap/+archive/ubuntu/rayforge
# pacote .deb também disponível nas releases do GitHub
```

**Windows:** instalador `.exe` nas releases do GitHub
(<https://github.com/barebaric/rayforge/releases>).

**macOS:** `.dmg`/`.zip` (arm, intel, universal) nas releases.

**PyPI:** `pip install rayforge` (pacote python, requer Python ≥ 3.10).

### 2.6 Requisitos

| Recurso | Instalado | Recomendado |
|---|---|---|
| Disco | < 1 GB | varia por projeto |
| RAM | 1 GB | projetos complexos podem consumir 10 GB+ |

### 2.7 Primeira configuração (wizard)

1. Abra **Settings → Machines** (ou `ctrl+,`) → **Add Machine**.
2. Escolha perfil embutido (pré-preenche controlador, área e cabeçote),
   importe um perfil (inclui `.lbdev` do LightBurn), ou **Device Not Listed**
   para configurar tudo manualmente.
3. **Controlador**: escolha o firmware (para o K1: **GRBL**).
4. **Conexão**: para serial — `/dev/ttyUSB0` ou `/dev/ttyACM0` no Linux,
   `COM3` no Windows; baud rate (GRBL padrão: 115200).
5. **Discover**: clique **Probe Now** para ler configuração da máquina
   (área, velocidades, aceleração) automaticamente.
6. **AI Spec Lookup** (opcional): informar vendor/modelo e buscar specs por
   IA; ou preencher manualmente.
7. **Hardware**: área X/Y (K1: **150 × 150 mm**), origem (0,0), direção dos
   eixos, limites, velocidades, aceleração.
8. **Head**: potência máxima (S-value — K1 7W = 7000/1000? usar o valor da
   máquina; tipicamente 1000 em GRBL), tamanho do spot (0,08 mm),
   frequência PWM, distância focal.
9. **Rotary/Câmeras**: opcional.
10. **Review & Name**: revisar, nomear, **Create Machine**.

A conexão é automática ao iniciar (status no canto inferior esquerdo).

### 2.8 Quick Start — primeiro job

1. **Importar design**: SVG, DXF, PDF, `.rd` ou imagem (JPEG/PNG/BMP).
   Sem design? Use o sketcher ou baixe SVGs gratuitos (Flaticon, SVG Repo).
2. **Posicionar**: pan (botão do meio ou espaço+arrastar), zoom (scroll),
   mover (arrastar), rotacionar/escalar (alças de seleção).
3. **Atribuir operação**: selecionar o objeto → `Operations → Add Operation`
   (`ctrl+shift+a`) → escolher tipo (Contour = cortar; Raster Engrave =
   gravar; Depth Engrave = profundidade) → configurar **Power** (comece
   baixo!), **Speed** (mm/min), **Passes**.
4. **Preview 3D**: `View → 3D Preview` ou `F12` — verifique percursos,
   ordem, objetos errados, excedentes da área de trabalho.
5. **Enviar para a máquina**:
   - Material no lugar + laser focado.
   - Posicione o laser com os jog controls (`View → Control Panel` ou
     `ctrl+l`).
   - **Frame** (`Machine → Frame`): traça o contorno do desenho em baixa
     potência para confirmar posição.
   - `Machine → Start Job`. Acompanhe no status bar (progresso + tempo
     estimado). Pause/Stop no control panel. **Esc só sai do modo
     simulação, não para o job.**
6. **Finalizar**: espere a exaustão limpar a fumaça, remova a peça, limpe a
   mesa.

**Dicas:** salve com `ctrl+s`; sempre teste em material de sobra; anote
parâmetros bons por material (biblioteca de materiais); mantenha a lente
limpa e correias tensionadas; use air assist se tiver.

### 2.9 Release notes relevantes da linha 1.8.x

- **1.8.5** (estável atual): upgrade raygeo 1.21.3 (corrige wavefronts
  duplicados e modo raster mask_scan/dither ignorando step_power); seleções
  em grupo não resetam mais posições relativas (#311).
- **1.8.4**: modo Speed vs Offset no material test grid (calibração de
  offset bidirecional empírica, #312); refatoração do pipeline (assembler
  registry).
- **1.8.3**: seletor de idioma em runtime (#303); gizmo de arrastar;
  suporte a tool numbers > 255; toggle air assist no material test grid;
  correção de G-code com Y duplicado em modo axis replacement (erro GRBL 25).
- **1.8.2**: variante de protocolo GRBL configurável para Longer Ray5;
  texto via Pango (fallback de fonte robusto).
- (1.9.0-beta): unidade métrica/imperial, import SVG por cores, array/pattern
  tool, correção de largura de ponto (dot width), suporte asyncio no pipeline,
  motor 3D reescrito.

---

## Parte 3 — ACMER K1 + Rayforge (integração)

### 3.1 Compatibilidade

O K1 é uma máquina **GRBL** via USB. O Rayforge suporta GRBL por serial
desde a versão 0.13 — ou seja, **funciona com o K1**, dando acesso a todos
os recursos que o AcmerTool não tem (otimização de percurso, preview 3D,
test grid, biblioteca de materiais, receitas etc.).

> Dica de setup equivalente à do LightBurn: tipo de dispositivo **GRBL**,
> área de trabalho **150 × 150 mm**, origem do job no **canto inferior
> esquerdo**.

### 3.2 Checklist de conexão (Linux)

1. Instale o Rayforge (Flatpak 1.8 — ver seção 2.4).
2. Conecte o K1 por USB e verifique o device:

   ```bash
   ls -l /dev/ttyUSB* /dev/ttyACM*
   ```

3. Usuário no grupo `dialout` (relogar depois):

   ```bash
   sudo usermod -a -G dialout $USER
   ```

4. No wizard da máquina: GRBL → porta serial (`/dev/ttyUSB0` etc.) →
   baud rate do K1 (padrão GRBL: 115200; se falhar, tente 9600/57600) →
   Probe Now.
5. Hardware: 150 × 150 mm; Head: spot 0,08 mm, potência máx. conforme a
   versão do módulo.
6. Status "Connected" no canto inferior esquerdo → pronto.

### 3.3 Parâmetros iniciais sugeridos (ponto de partida)

| Material | Operação | Power | Speed | Passes |
|---|---|---|---|---|
| Madeira 3 mm (bétula) | Corte | comece 60–80% | ~200 mm/min | 1–2 |
| Madeira | Gravação | 20–50% | até 10.000 mm/min | 1 |
| Couro | Gravação | 20–40% | 3.000–6.000 mm/min | 1 |
| Acrílico preto | Corte | 70–100% | 150–250 mm/min | 1–2 |

Sempre rode o **Material Test Grid** do Rayforge para calibrar no seu
módulo específico. Como o K1 não tem air assist, espere cortes mais lentos
que lasers abertos equivalentes.

### 3.4 Dicas de manutenção

- Limpe a lente regularmente (bico magnético facilita).
- Verifique tensão das correias (o K1 não tem fim de curso máximo — cuidado
  para não forçar os eixos até o limite).
- Rode um teste de linhas nos eixos X e Y ao receber a máquina (defeito de
  QC com ondulação no Y já foi reportado).
- Use purificador de ar ou ventile bem o ambiente.

---

## Parte 4 — Configuração verificada do Rayforge para o K1 7W (2026-08-08)

> Configuração feita campo a campo, validando cada valor contra a saída `$$`
> do firmware da própria máquina (fonte da verdade) e a spec oficial ACMER
> (capturada em 2026-08-08).

### Firmware confirmado via console (`$$`)

| Parâmetro | Valor | Uso |
|---|---|---|
| $32 | 1 | Modo laser ativado (S-value controla o laser) |
| $30 / $31 | 1000 / 0 | S-value máx/mín |
| $100 / $101 | 80 / 80 | steps/mm X/Y (1 passo = 0,0125 mm) |
| $110 / $111 | 10000 / 10000 | Velocidade máx. X/Y (mm/min) |
| $120 / $121 | 300 / 300 | Aceleração X/Y (mm/s²) |
| $130 / $131 | 150 / 150 | Área de trabalho X/Y (mm) |
| $22 / $23 | 1 / 3 | Homing ativado, origem canto inferior esquerdo |
| $21 / $20 | 1 / 0 | Hard limits ON, soft limits OFF |
| Versão | Grbl 1.1h (fork ACMER) | Suporta G5 bezier (testado) e parâmetros extras $41–$48 |

### Configuração final do Rayforge

| Seção | Campo | Valor | Fonte |
|---|---|---|---|
| Machine | Name | Acmer K1 | — |
| Driver | GRBL (Serial) | Porta /dev/ttyACM0, baud 115200 | console funcionou |
| Driver | Poll device status | ON | USB nativo estável |
| Driver | Deadlock detection | OFF | sem ALARM:3 falso |
| Driver | RX Buffer Override | 0 (auto) | padrão |
| Speeds | Max Travel Speed | 3000 (fixo — driver não suporta) | sem impacto real |
| Speeds | Max Cut Speed | **10000** | $110 |
| Speeds | Acceleration | **300** | $120 |
| Axes | X/Y Extent | 150 / 150 | $130/$131 |
| Axes | Origin | Bottom Left | $22+$23 |
| Axes | Reverse X/Y/Z | OFF | $3 já trata inversão; sem Z motorizado |
| Work Area | Margins | 0 / 0 / 0 / 0 | área toda utilizável |
| Work Area | Origin Is Coordinate Zero | OFF | sem efeito com margens 0 |
| Soft Limits | Custom | OFF (usa bounds 150×150) | proteção de jog |
| Path | Support Arcs | ON | firmware tem G2/G3 |
| Path | Support Bézier | **ON** | testado: G5 aceito em modo normal e check ($C) |
| Path | Tolerance | 0,030 | 2,6× menor que o spot — imperceptível |
| Homing | Home On Start | **OFF + homing manual** | evita crash por objeto esquecido dentro |
| Homing | Single Axis Homing | ON | GRBL 1.1 suporta |
| Homing | Clear Alarm On Connect | OFF | desbloqueio consciente |
| Precision | G-code Precision | 3 | 12× mais fino que o passo da máquina |
| Dialect | Grbl (Compat) | selecionado | perfil embutido padrão |
| Device Settings | — | **NÃO TOCAR** | escrita direta no firmware EEPROM |
| Laser Head | Name / Tool / Type | ACMER Blue Laser 7w / 0 / Diode | — |
| Laser Head | Max Power | 1000 | $30 |
| Laser Head | Focus Power | 1,00% | só usado em focus mode (sem Z motorizado = inerte) |
| Laser Head | Spot Size X/Y | **0,080 / 0,080** | spec oficial ACMER |
| Framing | Power / Speed / Repeat | 1,00% / 0 / 20 | perfil K1 (1% × 20 ≈ visível sem queimar) |
| Framing | Pause at Corners | 0 | desligado |
| 3D Model | head-diode / scale 307,69 | manter | cosmético — só preview 3D |
| Rotary | — | NÃO configurar | K1 não tem eixo A no firmware |
| Camera | — | NÃO configurar | K1 não tem câmera |
| No-go zones | — | pular | opcional, sem risco |
| Maintenance | — | nada | máquina nova, 0h |

### Procedimento de primeiro uso (seguro)

1. **Homing manual**: abrir a tampa → conferir área vazia → `$H` no console
   (ou botão Home no Rayforge) → fechar a tampa.
2. **Teste de jog**: mover +X e +Y alguns mm; coordenadas positivas = direção
   correta; se invertido, ativar Reverse do eixo afetado.
3. **Teste de linhas**: gravar linhas nos eixos X e Y (QC — ondulação no Y já
   foi reportada em unidades defeituosas).
4. **Material Test Grid** do Rayforge: potência × velocidade em material de
   sobra antes do primeiro job real.
5. Começar com potência baixa e conferir com a tampa fechada.

---

## Fontes

1. ACMER K1 oficial (loja): <https://acmerlaser.com/products/acmer-k1-enclosed-portable-laser-engraver>
2. ACMER EU: <https://eu.acmerlaser.com/products/acmer-k1-enclosed-portable-laser-engraver>
3. Downloads ACMER (drivers/firmware K1): <https://acmerlaser.com/pages/download>
4. Review Hoffman Engineering (teste 1 mês, 7W): <https://www.hoffman.engineering/blog/acmer-k1-review-portable-laser-worth-buying/>
5. 4evatech (specs detalhadas): <https://4evatech.com/acmer-k1/>
6. Rayforge oficial: <https://rayforge.org/>
7. Rayforge GitHub: <https://github.com/barebaric/rayforge>
8. Rayforge Releases: <https://github.com/barebaric/rayforge/releases>
9. Rayforge Flathub (Flatpak 1.8): <https://flathub.org/apps/org.rayforge.rayforge>
10. Rayforge Snap: <https://snapcraft.io/rayforge>
11. Rayforge Installation: <https://rayforge.org/docs/getting-started/installation/>
12. Rayforge First Time Setup: <https://rayforge.org/docs/getting-started/first-time-setup/>
13. Rayforge Quick Start: <https://rayforge.org/docs/getting-started/quick-start/>
14. Rayforge PyPI: <https://pypi.org/project/rayforge/>
15. Reviews em vídeo: <https://www.youtube.com/watch?v=UfR1fuH7z_c> e <https://www.youtube.com/watch?v=WyPaQc8XoEE>

---

## Part 5 — Tabela oficial de parâmetros ACMER (do manual em PDF)

Fonte: `ACMER K1-User Manual（English）.pdf` (seção 7, "Recommended parameters for common materials") — tabela **7W Compressed Spot**, a única aplicável ao módulo 7W do usuário.

### Tabela única — Cut + Engrave (7W Compressed Spot, manual oficial pág. 20)

| Material | Cut Power | Cut Speed (mm/min) | Cut Passes | Engrave Power | Engrave Speed (mm/min) | Engrave Passes |
|---|---|---|---|---|---|---|
| Kraft paper 0,5mm | 100% | 1500 | 1 | 50% | 8000 | 1 |
| Kraft paper 1,0mm | 100% | 1000 | 1 | 50% | 8000 | 1 |
| Kraft paper 2,0mm | 100% | 300 | 1 | 50% | 8000 | 1 |
| Plywood 2,0mm | 100% | 200 | 1 | 50% | 6000 | 1 |
| Solid wood 2,0mm | 95% | 200 | 2 | 80% | 3500 | 1 |
| Bamboo 2,0mm | 95% | 150 | 1 | 80% | 4500 | 1 |
| Red Acrylic 1,0mm | 100% | 150 | 1 | — | — | — |
| Red Acrylic 2,0mm | 100% | 100 | 1 | — | — | — |
| Black Acrylic 1mm | 100% | 150 | 1 | — | — | — |
| Black Acrylic 2mm | 100% | 100 | 1 | — | — | — |
| Light-colored Felt 1mm | 80% | 1500 | 1 | 50% | 8000 | 1 |
| Cork | — | — | — | 60% | 5000 | 1 |
| Transparent Acrylic (need blacking) | — | — | — | 70% | 6000 | 1 |
| Glass (need blacking) | — | — | — | 100% | 2000 | 1 |
| Dark Felt | — | — | — | 50% | 8000 | 1 |
| Leather | — | — | — | 50% | 8000 | 1 |
| Silica gel | — | — | — | 50% | 2000 | 1 |
| Cobblestone | — | — | — | 90% | 80 | 1 |
| Ceramics | — | — | — | 100% | 2000 | 1 |
| Black alumina | — | — | — | 80% | 7000 | 1 |
| Tin plate | — | — | — | 90% | 3000 | 1 |
| Non-reflective Stainless steel (matte) | — | — | — | 90% | 200 | 1 |
| Non-reflective Stainless steel (smooth) | — | — | — | 90% | 150 | 2 |

Corte = M3 (laser constante), gravação = M4 (potência dinâmica) — automático no Rayforge. "—" = manual não traz valor. Tabela é para kraft paper; papel comum mais fino pode exigir menos potência (test grid decide).

### Configuração final do Rayforge (resumo da Parte 4)

### Configuração dos steps da plaquinha (papel 0,5mm, 7W)

- **Raster Engrave** (texto): Power 50%, Speed 8000, Line Spacing 0,1, Scan Mode Segmented
- **Contour do texto (DESENHO)**: Power 50%, Speed 8000, Passes 1 — marca o contorno SEM cortar
- **Contour do retângulo (CORTE)**: Power 100%, Speed 1500, Passes 1, Kerf 0,080, Cut Side Centerline
- Ordem no workflow strip: engrave → contour texto (desenho) → contour retângulo (corte, último)
- **Regra**: o power/speed decide desenho vs corte. 50%/8000 = gravação (tabela engrave); 100%/1500 = corte (tabela cut). Nunca usar valores de corte no contorno do texto — recorta as letras.
- Laser options M3/M4: automático no Rayforge (M3 corte, M4 gravação)

### Lição registrada

Papel fino corta com **power alto + velocidade alta** (menor energia por mm), não power alto + velocidade baixa. 80%/500 mm/min ≈ 0,67 J/mm (queima); 100%/1500 ≈ 0,28 J/mm (corta limpo).

### BUG conhecido Rayforge 1.8.5 — "Job dependencies are not ready" sem erro visível

- **Sintoma**: envio falha com "sending failed: job dependencies are not ready"; Recalculate parece não fazer nada; nenhuma mensagem/ícone de erro na UI.
- **Causa raiz** (verificado no código `rayforge/pipeline/stage/step_stage.py` v1.8.5): `collect_assembly_info` itera todos os workpieces da camada; se um workpiece não tem fill para o tipo de step (ex.: retângulo só de contorno numa camada com step Raster Engrave), `get_workpiece_handle` retorna `None` → `ValueError` → `except ValueError` retorna `(None, [])` → `launch_task` recebe `assembly_info` vazio, faz `return` sem marcar step como válido → step permanece DIRTY → pipeline nunca conclui.
- **Workaround**: manter workpieces sem fill em camada separada dos steps de raster. Ex.: Layer 1 = texto (Raster+Contour), Layer 2 = retângulo (Contour de corte).
- **Extra**: se `rx:511` no status GRBL (buffer cheio), power-cycle na K1.
- **Homing stall pós-jog no cncjs**: após mover eixos manualmente, homing pode travar em `<Home>`. Reset: `$X` no console MDI ou power-cycle.
