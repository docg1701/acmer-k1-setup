# MANUAL — ACMER Studio V1.4.0 no Linux Mint via Wine do sistema

> **Data**: 2026-08-08
> **Sistema**: Linux Mint 22.3 Cinnamon (x86_64)
> **Wine**: wine-9.0
> **ACMER Studio**: V1.4.0 (15/07/2026)
> **DXVK**: 2.5.3 (Vulkan 1.2 — compatível com Mesa 25.2.8)

---

## 1. Configurar Wine

```bash
winecfg
```

Aba **Applications**: **Windows 10**.

---

## 2. Instalar ACMER Studio

```bash
wine ~/Downloads/ACMER_Studio_Setup_V1.4.0.exe
```

Next → Next → Install → Finish.

---

## 3. Dependências

```bash
winetricks --force vcrun2019
cd ~/.cache/winetricks/vcrun2019
wine vc_redist.x64.exe /quiet /norestart

winetricks --force corefonts
```

`vc_redist.x64.exe` falha com status 102 via winetricks — instalação manual.
`corefonts`: ~5 MB (Arial, Times, Courier). Se texto serrilhado, `allfonts` depois.

---

## 4. PATH das DLLs nativas

```bash
wine reg add "HKLM\System\CurrentControlSet\Control\Session Manager\Environment" \
  /v PATH /t REG_EXPAND_SZ \
  /d "C:\Program Files\ACMER Studio\resources\tools\win\Calibrate;C:\Program Files\ACMER Studio\resources\tools\win\PathOpt;C:\Program Files\ACMER Studio\resources\tools\win\BatchDup;%PATH%" \
  /f
```

---

## 5. DXVK 2.5.3

O Mesa 25.2.8 do Mint 22.3 não suporta `khrLoadStoreOpNone` (Vulkan 1.3).
DXVK ≥3.0 exige essa extensão. Use a versão 2.5.3:

```bash
cd /tmp
wget https://github.com/doitsujin/dxvk/releases/download/v2.5.3/dxvk-2.5.3.tar.gz
tar xf dxvk-2.5.3.tar.gz
cp dxvk-2.5.3/x64/*.dll ~/.wine/drive_c/windows/system32/

wine reg add "HKCU\Software\Wine\DllOverrides" /v "d3d11" /t REG_SZ /d "native,builtin" /f
wine reg add "HKCU\Software\Wine\DllOverrides" /v "dxgi" /t REG_SZ /d "native,builtin" /f
wine reg add "HKCU\Software\Wine\DllOverrides" /v "d3d10core" /t REG_SZ /d "native,builtin" /f
```

Sem DXVK: grid da área de trabalho e camadas de imagem não renderizam.

---

## 6. Configuração de janela (Mint/Cinnamon)

```bash
winecfg
```

Aba **Graphics** → desmarque tudo em "Window settings":

- ❌ Allow the window manager to decorate the windows
- ❌ Allow the window manager to control the windows

---

## 7. Mapear diretório home

```bash
winecfg
```

Aba **Drives** → **Add**:
- Letra: `D:`
- Pasta: `/home/galvani`

Agora o ACMER Studio acessa teus arquivos em `D:\`.

---

## 8. Abrir

```bash
wine "C:\Program Files\ACMER Studio\ACMER Studio.exe"
```

---

## 9. Conexão com a K1 (opcional)

Se a K1 estiver conectada via USB neste PC:

```bash
ln -sf /dev/ttyACM0 ~/.wine/dosdevices/com10
```

No ACMER Studio: **COM10** a **115200**.

Se a K1 estiver no servidor printbox, ignore — exporte o G-code e faça upload
via `http://10.10.10.190:8000`.

---

## 10. Warnings normais (ignorar)

| Warning | Impacto |
|---|---|
| `WSALookupServiceBegin failed with: 8` | Nenhum — detecção de rede do Electron |
| `RoGetActivationFactory Failed ... PenDevice` | Nenhum — API UWP |
| `DXVK: No adapters found` | DXVK instalado corretamente evita isso |
| `ntlm_check_version` | Nenhum — resolva com `sudo apt install winbind` |

---

## 11. Limitações conhecidas

- **Calibração de câmera**: não funciona (K1 não tem câmera)
- **Smart autofill / IA**: não testado
- **Path optimization / nesting**: não testado
- **Geração de G-code (corte vetorial, gravação)**: funcional
- **ACMER Studio é Windows-only**: Electron + módulos C++ nativos. Wine 9.0 +
  DXVK 2.5.3 funciona. Atualizações futuras do ACMER Studio podem quebrar.
