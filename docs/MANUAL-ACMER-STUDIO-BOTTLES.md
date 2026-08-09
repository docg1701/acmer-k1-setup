# MANUAL — ACMER Studio no Linux (Bottles + GE-Proton)

> **Contexto**: o ACMER Studio mais novo é distribuído como `.exe` para Windows
> e roda no Linux via **Bottles** (Flatpak) com o runner **GE-Proton11-3**.
> Aqui documentamos o passo a passo validado: instalação, permissão de
> filesystem, runner, ajustes de `winecfg` e o que já funciona.
>
> **Data**: 2026-08-09 · **Bottles** (Flatpak) · **GE-Proton11-3** · **ACMER Studio**
> (executável: `download/ACMERStudio.exe`)

**Status**: programa roda, renderiza corretamente, salva projetos e exporta
G-code. **Pendente**: teclado não funciona para editar nome de arquivo (e em
outros campos de texto) — análise via log no final deste documento.

---

## 1. Instalar o Bottles (Flatpak)

```bash
flatpak install flathub com.usebottles.bottles
```

---

## 2. Criar o bottle apontando para um diretório em `~/`

1. Abra o Bottles → **+ Novo bottle**.
2. Dê um nome (ex.: `acmer-studio`).
3. Em **Diretório**, aponte para uma pasta sua em `~/` (ex.:
   `~/acmer-studio-files`) — **fora** do sandbox do Flatpak.
4. O Bottles detecta que o diretório não é acessível dentro do sandbox e
   oferece um **comando a executar no terminal** que concede essa permissão.

> O comando exato é gerado pelo próprio Bottles — execute o que ele mostrar.
> O padrão é um override de filesystem, algo como:
>
> ```bash
> flatpak override --user --filesystem=/home/galvani/acmer-studio-files com.usebottles.bottles
> ```

5. Execute o comando num terminal, **feche o Bottles** e recomece o processo,
   apontando para o **mesmo diretório**.
6. Agora o Bottles aceita o diretório — crie o bottle normalmente.

---

## 3. Runner: GE-Proton11-3

O bottle deve usar o runner **GE-Proton11-3**:

- Bottles → **Preferências** → **Runners** → instalar o **GE-Proton11-3**
  (ou em **+ Novo bottle** → runner GE-Proton11-3).

---

## 4. Ajuste obrigatório no winecfg (janelas)

Sem este passo o programa não inicia direito. Com o bottle selecionado:

1. **Console** (do bottle) → rode:
   ```bash
   winecfg
   ```
2. Aba **Graphics** → **desmarque** os dois itens:
   - `Permitir que o gerenciador de janelas controle as janelas`
   - `Permitir que o gerenciador de janelas decore as janelas`

---

## 5. Rodar o ACMER Studio

```bash
# Console do bottle
wine /caminho/para/ACMERStudio.exe
```

**Validado — funciona:**
- Interface renderiza corretamente
- Projetos são salvos
- Exportação de G-code funciona

---

## 6. Problema pendente: teclado no diálogo de exportar G-code

**Sintoma**: ao abrir o diálogo de exportar G-code, não dá para digitar o nome
do arquivo; o teclado não funciona e a janela principal "perde toda a
funcionalidade" (clica, mas nada responde).

### 6.1. Fatos do log (run 2026-08-09)

| Achado | Leitura |
|---|---|
| App é **Electron + Node** (erros `network_change_notifier_win.cc`, logs JS do renderer, mensagens IPC `msg { type: 'init', ... }`) | Entrada de teclado passa pelo Chromium; diálogos nativos vão pro comdlg32 do Wine |
| Texto chinês corrompido (mojibake GBK) nos logs | App ACMER em chinês; IME/teclado do Chromium é sensível a isso |
| `DXVK: v3.0.2` + `AMD Radeon Graphics (RADV RENOIR)` | iGPU AMD (Ryzen 4000 APU); render via Vulkan/DXVK |
| Segunda init do DXVK no fim do log (`03b4:... Game: ACMER Studio.exe`) | Provável restart do processo GPU do Chromium na hora do diálogo |
| `opencv_*3415.dll`, `ceres.dll`, `ortools.dll`, `libprotobuf.dll`, `tinyxml2.dll`, `onnxruntime.dll`, `libsodium.dll` não encontrados | Ferramentas **Calibrate**, **PathOpt** e **BatchDup (smart autofill)** não carregam — não é o problema do teclado |
| `libvkd3d-utils-1.dll` / `wined3d.dll` não encontrados | Ruído do fallback DXCore; render já é pelo DXVK. Cosmético |
| `WSALookupServiceBegin failed: 8` | App tentando rede; inofensivo |

### 6.2. Diagnóstico

O sintoma (janela principal desabilitada + diálogo que não recebe teclado) é o
comportamento clássico de **diálogo modal nativo** (comdlg32 → `GetSaveFileNameW`)
rodando com **"Allow the window manager to control the windows" DESLIGADO**.

Quando o WM control está off, o Wine gerencia o foco X11 sozinho — e em diálogos
modais o foco de teclado não chega ao campo de nome. A janela principal fica
desabilitada pela modalidade (por isso "clica mas não funciona"), e o diálogo
espera digitação que nunca chega = trava. Isso é bug conhecido do Wine (ver
WineHQ), e ainda é agravado pelo runner **GE-Proton**, que carrega um patch de
foco da Valve que "pode quebrar diálogos modais" (commit `d30ce49` do wine/Proton).

### 6.3. Correções a testar (nesta ordem)

1. **Virtual desktop** (fix clássico): `winecfg` → aba **Graphics** → marcar
   **"Emulate a virtual desktop"** (ex.: 1920×1080). Com isso o Wine controla
   todo o foco internamente, e o diálogo modal volta a receber teclado. Testar se
   o app ainda inicia com o virtual desktop ativo.
2. **Clique dentro do campo de nome** antes de digitar (o diálogo modal do
   Electron/Chromium muitas vezes só recebe foco de teclado após um clique).
3. **Testar Esc/Enter** no diálogo: se fechar, confirma que é o modal travado por
   foco (não é hang do renderer).
4. Se o virtual desktop quebrar a renderização: trocar o runner para **sys-wine**
   (Wine puro, sem os patches de foco do Proton).
5. Se ainda travar: variável de ambiente do bottle
   `ELECTRON_DISABLE_GPU=1` (render por software do Chromium — mais estável no
   Wine, custa performance).

### 6.4. Verificação das DLLs (2026-08-09, no disco)

As DLLs acusadas no log **não estão faltando** — estão nas pastas das
ferramentas em `resources\tools\win\`:

| DLL | Local | Status |
|---|---|---|
| `opencv_*3415.dll`, `ceres.dll`, `gflags.dll`, `glog.dll`, `P3Calibration.dll` | `Calibrate\` | Presentes — erro é de caminho de busca |
| `ortools.dll`, `libprotobuf.dll`, `tinyxml2.dll`, `abseil_dll.dll`, `libscip.dll`, `re2.dll` | `PathOpt\` | Presentes — idem |
| `onnxruntime.dll`, `libsodium.dll`, `opencv_*` | `BatchDup\` | Presentes — idem |
| `libvkd3d-utils-1.dll` | system32 do bottle | **Ausente de verdade** — existe no runner `ge-proton11-3` (`share/default_pfx/.../system32/`) |

O Wine não acha as DLLs das ferramentas porque a ordem de busca é: diretório
do exe → system32 → **PATH** — e as pastas `Calibrate/PathOpt/BatchDup` não
estão no PATH do bottle (no Windows o app resolve isso de outra forma).

**Fix (validar no console, depois tornar permanente):**

```bash
# teste: no console do bottle
export PATH="Z:\\acmer-bottle\\drive_c\\Program Files\\ACMER Studio\\resources\\tools\\win\\Calibrate;Z:\\acmer-bottle\\drive_c\\Program Files\\ACMER Studio\\resources\\tools\\win\\PathOpt;Z:\\acmer-bottle\\drive_c\\Program Files\\ACMER Studio\\resources\\tools\\win\\BatchDup;$PATH"
cd "Z:\\acmer-bottle\\drive_c\\Program Files\\ACMER Studio"
wine ACMERStudio.exe

# permanente (via registro do bottle)
reg add "HKCU\\Environment" /v Path /t REG_EXPAND_SZ /d "Z:\\acmer-bottle\\drive_c\\Program Files\\ACMER Studio\\resources\\tools\\win\\Calibrate;Z:\\acmer-bottle\\drive_c\\Program Files\\ACMER Studio\\resources\\tools\\win\\PathOpt;Z:\\acmer-bottle\\drive_c\\Program Files\\ACMER Studio\\resources\\tools\\win\\BatchDup;%PATH%"
```

`libvkd3d-utils-1.dll`: dependência do `wined3d.dll` (fallback DXCore).
Inofensiva — renderização vai pelo DXVK. Silencia copiando do runner para o
system32 do bottle:

```bash
cp ~/.var/app/com.usebottles.bottles/data/bottles/runners/ge-proton11-3/files/share/default_pfx/drive_c/windows/system32/libvkd3d-utils-1.dll "drive_c/windows/system32/"
```
