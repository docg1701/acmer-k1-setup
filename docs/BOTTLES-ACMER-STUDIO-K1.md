# MANUAL — ACMER Studio V1.4.0 no Linux Mint via Bottles

> **Data**: 2026-08-08
> **Sistema**: Linux Mint 22.3 Cinnamon (x86_64)
> **Bottles**: 65.3 (Flatpak)
> **Objetivo**: ACMER Studio V1.4.0 em ambiente Wine isolado, reprodutível via snapshot
> **Runner**: `caffe` (recomendado — base Wine 9.x, melhor compatibilidade com Electron)

## 0. Por que Bottles

Bottles é um gerenciador gráfico de prefixos Wine com:

- **Snapshots**: salva o estado completo da bottle. Se algo quebrar, restaura em 1 clique
- **Backup**: exporta a bottle inteira como `.tar.gz`, portável entre máquinas
- **Dependencies**: instala vcrun2019, dotnet, mono etc. com um clique, sem terminal
- **Flatpak**: sandbox, não suja o sistema, atualizações automáticas
- **Runners**: troque entre Wine, Wine-GE, Proton-GE por bottle

PlayOnLinux é legado, sem snapshot, interface pior. Bottles é o substituto moderno.

## 1. Instalar Bottles

```bash
flatpak install flathub com.usebottles.bottles
```

Abra pelo menu de aplicativos ou:

```bash
flatpak run com.usebottles.bottles
```

## 2. Criar a bottle

1. **Create Bottle** (botão `+` no canto superior esquerdo)
2. Nome: `acmer-studio`
3. Environment: **Application**
4. Runner: **caffe** (recomendado — `wine-11` funciona mas tem glitch de janela, `soda` não funciona)
5. Clique **Create**

Aguarde a criação (~30 segundos).

## 3. Instalar dependências

1. Dentro da bottle `acmer-studio`, aba **Dependencies**
2. Busque e instale **vcredist2019** — resolve `concrt140.dll` e runtimes Visual C++
3. Busque e instale **allfonts** — fontes do Windows (Arial, Times, etc.).
   ~250 MB, demora alguns minutos. Corrige texto serrilhado no Electron.

## 4. Instalar o ACMER Studio

1. Na bottle, clique **Run Executable**
2. Selecione o `ACMER_Studio_Setup_V1.4.0.exe` que você baixou
3. Siga o instalador: Next → Next → Install → Finish

## 5. Corrigir PATH de DLLs nativas

Os módulos C++ do ACMER Studio (OpenCV, Ceres, OR-Tools, ONNX Runtime) ficam
em subdiretórios e o loader do Wine não os encontra. Precisa adicionar ao PATH.

Na bottle, vá em **Tools → Command Line** e cole:

```bash
reg add "HKLM\System\CurrentControlSet\Control\Session Manager\Environment" /v PATH /t REG_EXPAND_SZ /d "C:\Program Files\ACMER Studio\resources\tools\win\Calibrate;C:\Program Files\ACMER Studio\resources\tools\win\PathOpt;C:\Program Files\ACMER Studio\resources\tools\win\BatchDup;%PATH%" /f
```

Ou via GUI: **Opções → Registry Rules** →
- Key: `HKLM\System\CurrentControlSet\Control\Session Manager\Environment`
- Value: `PATH`
- Type: `REG_SZ`
- Data: `C:\Program Files\ACMER Studio\resources\tools\win\Calibrate;C:\Program Files\ACMER Studio\resources\tools\win\PathOpt;C:\Program Files\ACMER Studio\resources\tools\win\BatchDup;%PATH%`

Saída esperada: `The operation completed successfully`.

## 6. Desabilitar decoração de janela (Mint/Cinnamon)

O ACMER Studio usa Electron com title bar customizada. No Cinnamon, maximizar
empilha a barra do sistema sobre a do app.

Na bottle, **Settings → Display**, desmarque **Window manager decorations**.

Isso equivale a desmarcar todas as opções de janela no `winecfg` clássico.

## 7. Criar atalho de lançamento

1. Na bottle, aba **Programs**
2. Clique **Add Program** → selecione `C:\Program Files\ACMER Studio\ACMER Studio.exe`
3. Renomeie como quiser (ex.: "ACMER Studio")
4. Botão `...` ao lado → **Add to Desktop** (cria atalho `.desktop`)

Agora o ACMER Studio aparece no menu do Mint e pode ser fixado na barra de tarefas.

## 8. Snapshot (golden state)

Tudo funcionando: botão **⋮** na bottle → **Snapshot** → dê um nome (ex.: `golden`).

15 segundos para salvar. Esse snapshot captura:

- Wine prefix completo
- ACMER Studio instalado e configurado
- DLLs nativos com PATH correto
- Todas as dependências

Se algo quebrar no futuro: botão **Snapshots** → `golden` → **Restore**.
Menos de 1 minuto e está de volta.

## 9. Backup completo (portável)

Na lista de snapshots, botão **Export** no snapshot `golden`.
Gera um `.tar.gz` que pode ser restaurado em qualquer máquina com Bottles
via **Import Snapshot**.

## 10. Conectar na K1 (opcional)

Se a K1 estiver conectada via USB neste PC:

1. Na bottle, **Settings → Devices**
2. Em **Serial Ports**, adicione `/dev/ttyACM0` mapeado para `COM10`

No ACMER Studio, selecione **COM10** com **115200 baud**.

Se a K1 estiver no servidor remoto (printbox), ignore este passo — apenas exporte
o G-code e faça upload via cncjs em `http://10.10.10.190:8000`.

## 11. Atualizar o ACMER Studio

Quando sair nova versão:

1. **Snapshot** antes de atualizar
2. Baixe o novo `.exe`
3. **Run Executable** com o novo instalador
4. Reaplique o PATH de DLLs (seção 5) — o instalador pode sobrescrever
5. Teste → se funcionar, **Snapshot** com o nome da nova versão
6. Se quebrar, **Restore** do snapshot anterior

## 12. Warnings normais (ignorar)

- `WSALookupServiceBegin failed with: 8` — detecção de rede do Electron
- `RoGetActivationFactory Failed to find library for L"Windows.Devices.Input.PenDevice"` — API UWP
- `wglSetPixelFormatWINE failed` — renderização OpenGL/EGL
- `remote_font_face_source.cc(372)] NOTREACHED hit` — fallback de fontes

Nenhum afeta geração de G-code.

## 13. Troubleshooting

### Bottle não cria (erro de runner)

Em Settings, troque o runner para **soda-9.0** ou **wine-ge-8** e tente novamente.
O sys-wine-9.0 é o testado, mas qualquer Wine 9.x serve.

### vcrun2019 não instala pela GUI

Abra Tools → Command Line e execute:

```bash
winetricks -q vcrun2019
```

### Erro "DLL not found" (opencv, ceres, ortools...)

O PATH da seção 5 foi perdido. Reaplique o comando `wine reg add`.

### Restauração de snapshot falha

Feche o Bottles. Via terminal:

```bash
flatpak kill com.usebottles.bottles
```

Reabra e tente novamente.
