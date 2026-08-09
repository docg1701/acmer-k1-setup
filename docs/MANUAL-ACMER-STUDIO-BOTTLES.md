# MANUAL — ACMER Studio no Linux (Bottles + GE-Proton)

> **Contexto**: o ACMER Studio mais novo é distribuído como `.exe` para Windows
> e roda no Linux via **Bottles** (Flatpak) com o runner **GE-Proton11-3**.
> Aqui documentamos o passo a passo validado: instalação, permissão de
> filesystem, runner, ajustes de `winecfg` e o que já funciona.
>
> **Data**: 2026-08-09 · **Bottles** (Flatpak) · **GE-Proton11-3** · **ACMER Studio**
> (executável: `download/ACMERStudio.exe`)

**Status**: programa roda, renderiza corretamente, salva projetos, exporta
G-code e edita nome de arquivo. **Resolvido**: problema de teclado no diálogo
— causa: Wine em modo não-gerenciado + diálogo modal nativo; correção:
**virtual desktop** (seção 4).

---

## Resumo rápido (passo a passo)

1. Instalar Bottles: `flatpak install flathub com.usebottles.bottles`
2. Criar o bottle apontando para um diretório em `~/`; executar o comando de
   permissão de filesystem que o Bottles oferece (terminal), fechar e
   recomeçar apontando para o mesmo diretório
3. Runner do bottle: escolher **GE-Proton11-3** no ato de criação do bottle
   (instalar antes, se preciso: Bottles → Preferências → Runners)
4. Instalar o ACMER Studio no bottle
5. `winecfg` (console do bottle) → aba **Graphics**:
   - desmarcar *"Permitir que o gerenciador de janelas controle as janelas"*
   - desmarcar *"Permitir que o gerenciador de janelas decore as janelas"*
   - marcar **"Emulate a virtual desktop"** (1920×1080) ← resolve o teclado
     nos diálogos de salvar/exportar
6. Copiar as DLLs das ferramentas para o diretório do exe (bloco de comando
   na seção 6.4) — sem isso, Calibrate/PathOpt/BatchDup não carregam
7. Rodar o ACMER Studio pelo Bottles

Detalhes: seções abaixo.

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

O bottle deve usar o runner **GE-Proton11-3**, escolhido **no ato da criação**
do bottle (+ Novo bottle → campo Runner).

Se o runner ainda não estiver instalado: Bottles → **Preferências** →
**Runners** → instalar o **GE-Proton11-3** antes de criar o bottle.

---

## 4. Ajustes no winecfg (janelas) — obrigatórios

Sem estes passos o programa não inicia direito. Com o bottle selecionado:

1. **Console** (do bottle) → rode:
   ```bash
   winecfg
   ```
2. Aba **Graphics** → **desmarque** os dois itens:
   - `Permitir que o gerenciador de janelas controle as janelas`
   - `Permitir que o gerenciador de janelas decore as janelas`
3. Ainda em **Graphics** → marque **"Emulate a virtual desktop"** com
   resolução cheia (ex.: 1920×1080).

> **Por que o virtual desktop**: com o WM control desligado, o Wine gerencia
> o foco X11 sozinho e diálogos modais nativos (salvar/exportar G-code) não
> recebem foco de teclado — não dá para digitar o nome do arquivo e a janela
> parece travar (bug conhecido do Wine). Com o virtual desktop ativo, o Wine
> controla o foco internamente e o diálogo volta a funcionar. Validado em
> 2026-08-09.

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

## 6. Histórico: teclado no diálogo de exportar G-code (resolvido)

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

**Evidência no app (app.asar, 2026-08-09)**: o export de G-code e o
salvar/salvar-como usam `dialog.showSaveDialog` do Electron → no Windows isso
abre o **diálogo nativo Win32** (comdlg32 → `GetSaveFileNameW`), uma janela
modal separada, dona da janela principal.

O sintoma (janela principal desabilitada + campo de nome sem teclado) é o
comportamento clássico do Wine em **modo não-gerenciado** ("Allow the window
manager to control the windows" DESLIGADO — necessário pro app iniciar): o
Wine cuida do foco X11 sozinho e **diálogos modais não recebem foco de
tecleado** — as teclas não chegam ao campo; a janela dona fica desabilitada
pela modalidade (por isso "clica mas não funciona"). Bug documentado no Wine
há anos, com a MESMA combinação de config (decorate OFF + control OFF + sem
virtual desktop). O GE-Proton ainda agrava: carrega patch de foco da Valve que
"pode quebrar diálogos modais" (commit `d30ce49`).

Não é crash do Chromium — é o modal esperando digitação que nunca chega
(confirmar com Esc: se fechar, é foco, não hang).

### 6.2.1. Correções (priorizadas)

**✅ Resolvido em 2026-08-09: virtual desktop.** `winecfg` → Graphics →
**"Emulate a virtual desktop"** (1920×1080) — testado e funcionando: o app
inicia, o diálogo de exportar G-code recebe teclado e o nome do arquivo é
editável. O Wine vira dono de todas as janelas e controla o foco
internamente; o WM nunca toca nas janelas — resolve os dois lados (app inicia
sem interferência do WM; modal recebe teclado).

Demais opções testadas/manuais (mantidas como referência, não usadas):

1. **Clique no campo de nome antes de digitar** (custo zero): diálogos modais
   do Electron/Chromium às vezes só recebem teclado após um clique
   (electron/electron#42948).
2. **Runner sys-wine** (sem os patches de foco do Proton): trocar
   GE-Proton11-3 por sys-wine/kron4ek (já instalados no Bottles) e revalidar
   app + diálogo.
3. **`ELECTRON_DISABLE_GPU=1`** no bottle: só se o travamento parecer hang do
   renderer depois dos testes acima.

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
do exe → system32 → PATH — e as pastas `Calibrate/PathOpt/BatchDup` não
estão no PATH do bottle (no Windows o app resolve isso de outra forma).

**Fix (aplicado em 2026-08-09): copiar as DLLs das ferramentas para o
diretório do exe** — primeiro lugar da busca, funciona sem registro e sem
PATH (o console do Bottles é cmd do Wine; `reg add` em `HKCU\Environment`
não funciona porque a chave não existe no Wine).

Bloco único, pronto para copiar e colar no terminal Linux:

```bash
APP="/home/galvani/acmer-studio/acmer-bottle/drive_c/Program Files/ACMER Studio"
BOTTLE="/home/galvani/acmer-studio/acmer-bottle/drive_c/windows"
RUNNER="$HOME/.var/app/com.usebottles.bottles/data/bottles/runners/ge-proton11-3"

# DLLs das ferramentas (Calibrate/PathOpt/BatchDup) -> diretório do exe
cp "$APP/resources/tools/win/Calibrate/"*.dll "$APP/"
cp "$APP/resources/tools/win/PathOpt/"*.dll "$APP/"
cp "$APP/resources/tools/win/BatchDup/"*.dll "$APP/"

# libvkd3d-utils-1.dll (ausente do bottle, existe no runner) -> system32/syswow64
cp "$RUNNER/files/share/default_pfx/drive_c/windows/system32/libvkd3d-utils-1.dll" "$BOTTLE/system32/"
cp "$RUNNER/files/share/default_pfx/drive_c/windows/syswow64/libvkd3d-utils-1.dll" "$BOTTLE/syswow64/"
```

(34 DLLs + 2 do vkd3d. Reverter: `rm` das DLLs copiadas — só as que não são
do Electron; ou reinstalar o app.)

**Validado em run de 2026-08-09 14:23**: erros `err:module:import_dll` de
opencv/ceres/ortools/tinyxml2/onnxruntime/libsodium **não aparecem mais**.
Ferramentas Calibrate/PathOpt/BatchDup carregam.

`libvkd3d-utils-1.dll`: dependência do `wined3d.dll` (fallback DXCore).
Inofensiva — renderização vai pelo DXVK. Silenciada copiando do runner para o
system32/syswow64 do bottle (feito em 2026-08-09):

```bash
cp ~/.var/app/com.usebottles.bottles/data/bottles/runners/ge-proton11-3/files/share/default_pfx/drive_c/windows/system32/libvkd3d-utils-1.dll "/home/galvani/acmer-studio/acmer-bottle/drive_c/windows/system32/"
cp ~/.var/app/com.usebottles.bottles/data/bottles/runners/ge-proton11-3/files/share/default_pfx/drive_c/windows/syswow64/libvkd3d-utils-1.dll "/home/galvani/acmer-studio/acmer-bottle/drive_c/windows/syswow64/"
```

Erros que permanecem no log (cosméticos): `network_change_notifier` (app
buscando rede), `RoGetActivationFactory PenDevice` e `com_get_class_object`
(serviços Windows inexistentes no Wine), `QueryInterface` (ruído do DXVK).
Nenhum afeta o funcionamento.
