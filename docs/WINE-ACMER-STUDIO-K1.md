# MANUAL — ACMER Studio V1.4.0 no Linux Mint via Wine do sistema

> **Data**: 2026-08-08
> **Sistema**: Linux Mint 22.3 Cinnamon (x86_64)
> **Wine**: wine-9.0
> **ACMER Studio**: V1.4.0 (15/07/2026)
> **Resultado**: funcional — design, G-code, interface completa

---

## 1. Configurar Wine

```bash
winecfg
```

Na aba **Applications**, Versão do Windows: **Windows 10**.

---

## 2. Baixar e instalar o ACMER Studio

1. Acesse https://acmerlaser.com/pages/acmer-software-download
2. Baixe o instalador (V1.4.0)
3. Execute:

```bash
wine ~/Downloads/ACMER_Studio_Setup_V1.4.0.exe
```

Siga o instalador: Next → Next → Install → Finish.

---

## 3. Instalar dependências via winetricks

### 3.1 Visual C++ Runtime 2019

```bash
winetricks --force vcrun2019
```

Se o `vc_redist.x64.exe` falhar com status 102 após o --force:

```bash
cd ~/.cache/winetricks/vcrun2019
wine vc_redist.x64.exe /quiet /norestart
```

### 3.2 Fontes do Windows

```bash
winetricks --force allfonts
```

~250 MB. Corrige texto serrilhado no Electron do ACMER Studio.

---

## 4. PATH das DLLs nativas

Os módulos C++ do ACMER Studio (OpenCV, Ceres, OR-Tools, ONNX Runtime) ficam em
subdiretórios de `resources/tools/win/`. O Wine não busca nessas pastas.

```bash
wine reg add "HKLM\System\CurrentControlSet\Control\Session Manager\Environment" \
  /v PATH /t REG_EXPAND_SZ \
  /d "C:\Program Files\ACMER Studio\resources\tools\win\Calibrate;C:\Program Files\ACMER Studio\resources\tools\win\PathOpt;C:\Program Files\ACMER Studio\resources\tools\win\BatchDup;%PATH%" \
  /f
```

Saída esperada: `reg: A operação foi completada com sucesso`

---

## 5. Configuração de janela (Mint/Cinnamon)

O ACMER Studio usa Electron com title bar customizada. Maximizar empilha a barra
do Cinnamon sobre a do app.

```bash
winecfg
```

Aba **Graphics** → desmarque **todas** as opções em "Window settings":

- ❌ Allow the window manager to decorate the windows
- ❌ Allow the window manager to control the windows

---

## 6. Mapear diretório home

Abra o `winecfg`, aba **Drives** → **Add**.

- Letra: `D:`
- Pasta: `/home/galvani`

Agora o ACMER Studio acessa teus arquivos em `D:\`.

---

## 7. Abrir o programa

```bash
wine "C:\Program Files\ACMER Studio\ACMER Studio.exe"
```

### Warnings normais (ignorar)

| Warning | Impacto |
|---|---|
| `WSALookupServiceBegin failed with: 8` | Nenhum — detecção de rede do Electron |
| `RoGetActivationFactory Failed ... PenDevice` | Nenhum — API UWP, sem suporte no Wine |
| `wglSetPixelFormatWINE failed` | Visual — preview 3D pode ter glitch. G-code não afetado |
| `ntlm_check_version ntlm_auth was not found` | Nenhum — resolva com `sudo apt install winbind` |
| `remote_font_face_source.cc NOTREACHED` | Nenhum — fallback de fonte |

---

## 8. Conexão com a K1 (opcional)

Se a K1 estiver conectada via USB neste PC:

```bash
ln -sf /dev/ttyACM0 ~/.wine/dosdevices/com10
```

No ACMER Studio, **COM10** a **115200**.

Se a K1 estiver no servidor printbox, ignore. Apenas exporte o G-code e faça
upload via `http://10.10.10.190:8000`.

---

## 9. Limitações conhecidas

- **Calibração de câmera**: não funciona (irrelevante, K1 não tem câmera)
- **Smart autofill / IA**: não testado (depende de onnxruntime)
- **Path optimization / nesting**: não testado
- **Preview 3D**: glitches visuais, não afeta G-code
- **ACMER Studio é Windows-only**: Electron + C++ nativo. Wine 9.0 funciona, mas
  atualizações futuras do ACMER Studio podem quebrar compatibilidade
