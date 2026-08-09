# MANUAL — LaserGRBL no Linux Mint via Wine

> **Data**: 2026-08-08
> **Sistema**: Linux Mint 22.3 Cinnamon (x86_64)
> **Wine**: wine-9.0 (Ubuntu 9.0~repack-4build3)
> **Alternativa**: LaserGRBL — gratuito, leve, comprovado no Wine

## 1. Instalar Wine

```bash
sudo dpkg --add-architecture i386
sudo mkdir -pm755 /etc/apt/keyrings
sudo wget -O /etc/apt/keyrings/winehq-archive.key https://dl.winehq.org/wine-builds/winehq.key
sudo wget -nc -P /etc/apt/sources.list.d/ https://dl.winehq.org/wine-builds/ubuntu/dists/jammy/winehq-jammy.sources
sudo apt update
sudo apt install --install-recommends winehq-staging winetricks
```

Verifique: `wine --version`

## 2. Download e instalação

1. Baixe o instalador: https://lasergrbl.com (botão Download)
2. Execute:

```bash
wine ~/Downloads/LaserGRBL_*.exe
```

3. Durante a instalação, marque **"Create a desktop shortcut"**.

## 3. Pós-instalação

Se o LaserGRBL não abrir, instale o .NET Framework:

```bash
winetricks dotnet48
```

## 4. Link da porta serial (COM → USB)

```bash
mkdir -p ~/.wine/dosdevices
ln -sf /dev/ttyACM0 ~/.wine/dosdevices/com10
```

Se `com10` já existir como arquivo, remova antes: `rm ~/.wine/dosdevices/com10`.

No LaserGRBL, selecione **COM10** a **115200**.

## 5. Limitação conhecida

O **Raster Image Import** (imagem → G-code de varredura) não funciona no Wine.
Use o ACMER Studio para raster: veja `docs/WINE-ACMER-STUDIO-K1.md`.

Corte vetorial e gravação de contorno funcionam normalmente.

## 6. Alternativa: PlayOnLinux

Script oficial do repositório LaserGRBL:

1. `sudo apt install playonlinux`
2. Baixe: https://github.com/arkypita/LaserGRBL/raw/master/POL_LaserGRBL_setup.sh
3. PlayOnLinux → **Tools → Execute local script** → selecione o `.sh`
4. Testado no Debian Bookworm 64-bit com Wine 9.0-rc5

## 7. Configuração da K1

| Parâmetro | Valor |
|---|---|
| Firmware | GRBL 1.1h (fork ACMER) |
| Baud rate | 115200 |
| Área de trabalho | 150 × 150 mm |
| Laser | Diodo 7W, 455 nm |
| $32 (Laser Mode) | 1 |

### Tabela de potência/velocidade (7W)

Consulte `docs/ACMER-K1-User-Manual-EN.pdf` — tabela oficial do fabricante.

## 8. Troubleshooting

### COM port não aparece

```bash
lsusb | grep -i "ch340\|ch341\|acmer"
dmesg | grep tty
cd ~/.wine/dosdevices
rm -f com10
ln -sf /dev/ttyACM0 com10
```

### Erro .NET

```bash
winetricks dotnet48
```

### Permissão da porta serial

```bash
sudo usermod -a -G dialout $USER
# Re-login necessário
```

## Fontes

- Guia AranaCorp: https://www.aranacorp.com/en/installing-lasergrbl-under-linux/
- Script PlayOnLinux oficial: https://github.com/arkypita/LaserGRBL/blob/master/POL_LaserGRBL_setup.sh
- FAQ LaserGRBL (Linux): https://lasergrbl.com/faq/
- WineHQ: https://wiki.winehq.org/Ubuntu
