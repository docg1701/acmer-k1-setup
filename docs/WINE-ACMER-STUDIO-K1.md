# MANUAL — ACMER Studio V1.4.0 no Linux Mint via Wine

> **Data**: 2026-08-08
> **Sistema**: Linux Mint 22.3 Cinnamon (x86_64)
> **Wine**: wine-9.0 (Ubuntu 9.0~repack-4build3)
> **Winetricks**: 20240105
> **Resultado**: ACMER Studio abre, interface funcional, geração de G-code operacional

## 0. Pré-requisitos

Wine já instalado:

```bash
sudo dpkg --add-architecture i386
sudo mkdir -pm755 /etc/apt/keyrings
sudo wget -O /etc/apt/keyrings/winehq-archive.key https://dl.winehq.org/wine-builds/winehq.key
sudo wget -nc -P /etc/apt/sources.list.d/ https://dl.winehq.org/wine-builds/ubuntu/dists/jammy/winehq-jammy.sources
sudo apt update
sudo apt install --install-recommends winehq-staging winetricks
```

Verifique:

```bash
wine --version   # wine-9.0
```

## 1. Download do ACMER Studio

1. Acesse https://acmerlaser.com/pages/acmer-software-download
2. Clique no botão de download da seção **ACMER Studio V1.4.0**
3. Salve o `.exe` (ex.: `ACMER_Studio_Setup_V1.4.0.exe`)

## 2. Instalação

Antes de instalar, configure a versão do Windows no Wine:

```bash
winecfg
```

Na aba **Applications**, selecione **Windows 10** (não XP 64 bits).

Depois instale:

```bash
wine ~/Downloads/ACMER_Studio_Setup_*.exe
```

Siga o instalador: Next → Next → Install → Finish. Marque "Create desktop shortcut" se oferecido.

## 3. Instalar Visual C++ Runtime (vcrun2019)

O ACMER Studio depende de `concrt140.dll` e outros runtimes do Visual C++ 2019.
Sem isso, o programa crasha no lançamento.

```bash
winetricks vcrun2019
```

**SHA256 mismatch esperado**: o pacote no servidor da Microsoft é atualizado com
frequência e o checksum do winetricks fica desatualizado. Responda **Y** para continuar
em ambas as perguntas (x86 e x64).

**Erro no vc_redist.x64.exe**: a instalação do x64 pode retornar status 102. Se
acontecer, instale manualmente:

```bash
cd ~/.cache/winetricks/vcrun2019
wine vc_redist.x64.exe /quiet /norestart
```

## 4. Corrigir PATH de DLLs nativas

Os módulos nativos do ACMER Studio (OpenCV, Ceres, OR-Tools, ONNX Runtime etc.)
ficam em subdiretórios de `resources/tools/win/`. O loader do Wine não busca
dentro dessas pastas — os DLLs existem mas não são encontrados.

Adicione os diretórios ao PATH do Wine:

```bash
wine reg add "HKLM\System\CurrentControlSet\Control\Session Manager\Environment" \
  /v PATH /t REG_EXPAND_SZ \
  /d "C:\Program Files\ACMER Studio\resources\tools\win\Calibrate;C:\Program Files\ACMER Studio\resources\tools\win\PathOpt;C:\Program Files\ACMER Studio\resources\tools\win\BatchDup;%PATH%" \
  /f
```

Saída esperada: `reg: A operação foi completada com sucesso`

## 5. Abrir o ACMER Studio

```bash
wine "C:\Program Files\ACMER Studio\ACMER Studio.exe"
```

Warnings cosméticos normais que **não afetam o funcionamento**:

- `ntlm_check_version ntlm_auth was not found` — resolva com `sudo apt install winbind` (opcional)
- `RoGetActivationFactory Failed to find library for L"Windows.Devices.Input.PenDevice"` — API UWP, sem suporte Wine (irrelevante)
- `wglSetPixelFormatWINE failed` — renderização 3D com glitches visuais no preview (não afeta G-code)
- `EGL Driver message (Error) eglCreateContext` — mesmo do acima
- `remote_font_face_source.cc(372)] NOTREACHED hit` — fallback de fontes do Electron, inofensivo

## 6. Conectar na K1 (opcional, só se a máquina estiver conectada via USB neste PC)

```bash
ln -sf /dev/ttyACM0 ~/.wine/dosdevices/com10
```

No ACMER Studio, selecione **COM10** com **115200 baud**.

Se a K1 estiver no servidor remoto (printbox), ignore este passo.
Gere o G-code, salve como `.nc` ou `.gcode`, e faça upload via cncjs em
`http://10.10.10.190:8000`.

## 7. Fluxo completo

```text
[Seu PC - Linux Mint]
  │
  ├── Wine + ACMER Studio V1.4.0
  │     ├── Design (vetores, texto, imagens)
  │     ├── Configurar potência, velocidade, passes
  │     └── Exportar G-code (.nc)
  │
  │  WiFi — upload via cncjs web UI
  ▼
[printbox - Debian 13]
  │  cncjs :8000 — recebe .nc, streama via USB
  │  ustreamer :8080 — webcam
  │
  │  USB serial 115200
  ▼
[ACMER K1] GRBL — executa
```

## 8. Tabela de potência/velocidade (7W)

Consulte `docs/ACMER-K1-User-Manual-EN.pdf` — tabela oficial do fabricante.

| Material | Operação | Potência (%) | Velocidade (mm/min) | Passes |
|---|---|---|---|---|
| Compensado 3 mm | Corte | 100 | 200 | 1 |
| Papel kraft | Gravação | 30 | 6000 | 1 |
| Couro | Gravação | 40 | 4000 | 1 |
| Acrílico escuro | Corte | 100 | 150 | 2 |
| Madeira | Gravação | 60 | 8000 | 1 |

## 8. Ajuste de janela (Mint/Cinnamon)

O ACMER Studio usa Electron com title bar customizada. No Cinnamon, maximizar
empilha a barra do sistema sobre a do app.

**Correção**: abra o `winecfg`, aba **Graphics**, desmarque **todas** as opções
em "Window settings" ("Allow the window manager to decorate the windows" e
"Allow the window manager to control the windows").

O ACMER Studio agora maximiza limpo, só com a barra do Electron.

## 9. Troubleshooting

### "concrt140.dll aborting"

```bash
winetricks vcrun2019
```

Se o x64 falhar, instale manualmente (seção 3).

### DLLs nativas não encontradas (opencv, ortools, ceres, onnxruntime...)

Reexecute o comando `wine reg add` da seção 4.

### ACMER Studio não abre (tela preta ou crash silencioso)

```bash
# Limpar prefixo e reinstalar
rm -rf ~/.wine
wineboot -u
# Reinstale o ACMER Studio + vcrun2019 + PATH
```

### Erro "WSALookupServiceBegin failed with: 8"

Inofensivo. O Electron tenta detectar mudanças de rede e falha no Wine. Não
afeta nada.

### Preview 3D não renderiza (tela branca ou glitch)

Limitação do Wine/OpenGL. O G-code gerado não é afetado. Use o preview 2D
(linhas de contorno) como alternativa.

### NTLM / ntlm_auth warnings

```bash
sudo apt install winbind
```

---

## Notas finais

- **Calibração de câmera**: não funciona (irrelevante para K1, não tem câmera)
- **Smart autofill / IA**: não testado (depende de onnxruntime carregar modelo)
- **Path optimization / nesting**: não testado
- **Geração de G-code básica (corte vetorial, gravação)**: funcional
- **Conexão direta via USB/COM**: funcional com o link simbólico

O ACMER Studio é um app Electron + módulos nativos C++. Wine 9.0 suporta
Electron razoavelmente bem — o programa abre e a interface responde. Os módulos
nativos mais pesados (calibração, otimização) têm dependências não triviais e
podem falhar silenciosamente. Para uso básico (design vetorial + export G-code),
funciona.
