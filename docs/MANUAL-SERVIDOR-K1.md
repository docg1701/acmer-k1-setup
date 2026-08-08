# MANUAL — Servidor de Impressão da K1 (Rayforge + cncjs)

> **Arquitetura**: você desenha e gera o job no Rayforge (seu PC), envia o arquivo
> G-code para o cncjs rodando no mini PC (servidor), e o servidor executa a K1
> direto pelo USB. Seu PC pode desligar — o servidor cuida de tudo, e você
> acompanha pelo browser (PC ou celular).
>
> **Data**: 2026-08-08 · **cncjs v1.11.x** (verificado: releases ativos em 2026)

```text
[Seu PC]  Rayforge — desenha, configura steps, exporta G-code
    │
    │  WiFi (rede local) — só upload do arquivo
    ▼
[mini PC]  Debian + cncjs — recebe o arquivo, streama via USB
    │  USB serial 115200
    ▼
[K1]  GRBL — executa o job
```

**Regra de ouro**: toda configuração (potência, velocidade, passes, ordem dos
steps, sanity check) acontece no Rayforge, na hora do **Export G-code**. O cncjs
não tem configuração de material/potência — ele só manda o arquivo para a
máquina, byte por byte. Nada se repete no servidor.

---

## 1. Instalação do Debian no mini PC

1. Baixe o **Debian netinst amd64**: <https://www.debian.org/distrib/>
2. Grave num pendrive (ex.: `dd if=debian.iso of=/dev/sdX bs=4M status=progress`)
3. Boot pelo pendrive e instale:
   - **Hostname**: `printbox`
   - **Usuário**: `galvani` + senha
   - **Rede**: durante a instalação, conecte no **WiFi** (o instalador configura)
   - **Particionamento**: guided, disco inteiro (o disco não tem nada importante)
   - **Software selection**: marque **SSH server** + **standard system utilities**
     — **sem desktop, sem GNOME/KDE** (console puro; ninguém vai usar monitor)
4. Reinicie. Teste com `ip a` — deve aparecer o IP do WiFi.

---

## 2. Pós-instalação (uma vez só)

```bash
# Atualizar o sistema
sudo apt update && sudo apt upgrade -y

# Ferramentas básicas
sudo apt install -y curl iw git

# Usuário no grupo dialout (acesso à porta serial da K1)
sudo usermod -aG dialout $USER
# (faça logout/login depois — ou reinicie)
```

### 2.1. Desativar WiFi power save (obrigatório)

O power save causa **picos de latência** no WiFi — e é exatamente o que pode
travar um job no meio (a máquina para com o laser aceso). Desligar:

```bash
# Ver a interface (geralmente wlan0)
ip a

# Desligar na hora:
sudo iw dev wlan0 set power_save off

# Confirmar:
iw dev wlan0 get power_save
```

Para persistir após reboot, crie o serviço:

```bash
sudo tee /etc/systemd/system/wifi-powersave-off.service > /dev/null <<'EOF'
[Unit]
Description=Disable WiFi power save
After=network-online.target
Wants=network-online.target

[Service]
Type=oneshot
ExecStart=/usr/sbin/iw dev wlan0 set power_save off

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl enable --now wifi-powersave-off.service
```

### 2.2. IP fixo

Configurar direto no Debian (sem depender de DHCP reservation do roteador).
Edite `/etc/network/interfaces`:

```bash
sudo nano /etc/network/interfaces
```

Substitua o conteúdo por:

```text
auto lo
iface lo inet loopback

auto wlan0
iface wlan0 inet static
    address 10.10.10.190/24
    gateway 10.10.10.1
    wpa-ssid NOME-DA-REDE
    wpa-psk SENHA-DA-REDE
```

> Substitua `NOME-DA-REDE` e `SENHA-DA-REDE` pelos dados do seu WiFi.
> Se o WiFi tiver caracteres especiais na senha, use aspas: `wpa-psk "senha"`.

Aplicar:

```bash
sudo systemctl restart networking
# ou reboot
```

Confirme:

```bash
ip a show wlan0   # deve mostrar 10.10.10.190
ping 10.10.10.1   # deve responder
```

Anote `10.10.10.190` — esse IP vai no browser para sempre.

### 2.3. Teste do SSH (opcional, mas recomendado)

O mini PC não tem monitor — gerencie por SSH do seu PC:

```bash
ssh galvani@10.10.10.190
```

---

## 3. Instalação do cncjs

```bash
# Node.js + npm (Debian 13 traz Node 20.x — cncjs pede ≥12)
sudo apt install -y nodejs npm

# cncjs (instalação global; --unsafe-perm é exigido com sudo)
sudo npm install --unsafe-perm -g cncjs@latest

# Confirmar versão (deve mostrar 1.11.x)
cncjs --version
```

### 3.1. Configuração mínima (`~/.cncrc`)

```bash
mkdir -p ~/gcode
cat > ~/.cncrc <<'EOF'
{
  "controller": "Grbl",
  "baudrates": [115200, 250000],
  "watchDirectory": "/home/galvani/gcode"
}
EOF
```

- `controller: Grbl` — a K1 é GRBL
- `baudrates: 115200` — baud da K1 (confirmado na config atual do Rayforge)
- `watchDirectory` — pasta monitorada: arquivos jogados ali aparecem na web UI
- O cncjs adiciona automaticamente `state` e `secret` ao salvar preferências pela web UI

### 3.2. Serviço systemd (sobe sozinho no boot)

```bash
sudo tee /etc/systemd/system/cncjs.service > /dev/null <<'EOF'
[Unit]
Description=cncjs web controller
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=galvani
ExecStart=/usr/local/bin/cncjs -H 0.0.0.0 -p 8000 --controller Grbl
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl enable --now cncjs.service

# Conferir que está no ar:
ss -ltn | grep 8000
```

> Se `which cncjs` mostrar outro caminho, ajuste o `ExecStart`.

---

## 4. Conexão USB com a K1

1. **Pluga o cabo USB da K1 no mini PC** e liga a máquina.
2. Descubra o device:

   ```bash
   ls /dev/ttyACM* /dev/ttyUSB*
   ```

   → na prática, será `/dev/ttyACM0` (mesmo device que o seu PC usa hoje).
3. Confirme que é a K1:

   ```bash
   lsusb
   ```

   (procure pela placa — CH340 aparece como `1a86:7523`, WCH/ACM como CDC)

**Se o device mudar** (K1 religada pode virar `ttyACM1`): reinicie o cncjs:

```bash
sudo systemctl restart cncjs
```

O device é `/dev/ttyACM0` — use esse nome no cncjs.

---

## 5. Configuração do cncjs (web UI)

1. No browser do seu PC: **`http://10.10.10.190:8000`**
2. **Connect** (canto superior):
   - **Controller**: `Grbl`
   - **Serial port**: `/dev/ttyACM0`
   - **Baud rate**: `115200`
   - **Connect**
3. Deve aparecer o status GRBL (Idle, posição, versão do firmware).
4. **Teste de sanidade**: use o painel de jog (setas) — a K1 deve mover.
5. **NÃO mexa** nos sliders de override (feedrate/laser) — o job usa os valores
   do arquivo; override só se você quiser alterar ao vivo, conscientemente.

---

## 6. Fluxo do dia a dia

### 6.1. No Rayforge (seu PC) — gerar o job

1. Desenha e configura tudo como você já faz (steps, potência, sanity check)
2. **`File → Export G-code...`** → salve ex.: `plaquinha.nc`
   - O sanity check roda na exportação — coordenadas negativas aparecem aqui
   - O arquivo contém TUDO: potência, velocidade, passes, ordem dos steps

### 6.2. Enviar para o servidor

**Forma 1 — browser**: cncjs → aba G-code → **Upload** → seleciona `plaquinha.nc`

**Forma 2 — pasta monitorada**: copie o arquivo para `~/gcode` no mini PC
(`scp plaquinha.nc galvani@10.10.10.190:/home/galvani/gcode/`) — ele aparece na web UI
sozinho.

### 6.3. Executar

1. Selecione o arquivo na lista → **Load**
2. Confira o visualizador (toolpath) e a posição da máquina
3. **Run** → o servidor streama o job pelo USB
4. **Desliga o PC** — o job continua; acompanhe do celular no
   `http://10.10.10.190:8000`
5. Botões disponíveis durante o job: **Pause**, **Resume**, **Cancel**

---

## 7. Testes recomendados na primeira semana

1. **Teste de conexão**: conectar, ler status, jog em todas as direções
2. **Job pequeno**: uma plaquinha de papel (já validada) via cncjs
3. **Teste de "PC desligado"**: inicie um job, desligue o notebook, confirme
   que o job termina e o servidor mostra 100%
4. **Teste de pause/cancel**: pause no meio, resume, depois cancel — a K1
   responde como no Rayforge

---

## 8. Manutenção

```bash
# Atualizar o cncjs
sudo npm install --unsafe-perm -g cncjs@latest
sudo systemctl restart cncjs

# Ver logs
journalctl -u cncjs -f

# Reiniciar tudo (mini PC)
sudo reboot
```

Depois do reboot, o WiFi power save off e o cncjs sobem sozinhos (systemd).
Confira com `iw dev wlan0 get power_save` e `ss -ltn | grep 8000`.

---

## 9. Troubleshooting

| Sintoma | Causa | Solução |
|---|---|---|
| Web UI não abre | cncjs não subiu / IP errado | `ss -ltn \| grep 8000`; `ip a`; reinicie o serviço |
| "No serial port found" | usuário sem dialout / USB solto | `sudo usermod -aG dialout $USER` + relogin; cheque `ls /dev/ttyACM*` |
| Porta mudou após religar a K1 | re-enumeração USB | `sudo systemctl restart cncjs` (ou regra udev, seção 4) |
| Job trava no meio / máquina para com laser aceso | WiFi power save ativo | `iw dev wlan0 get power_save` → deve dar `off` |
| "Port in use" ao conectar | outro programa abriu a serial | nada mais pode usar a porta; reinicie o cncjs |
| Celular não acessa | AP isolation no roteador | desative "isolamento de clientes" no WiFi |
| Job inicia mas fica lento/com gaps | WiFi 2.4GHz congestionado | prefira 5GHz no mini PC (se suportar) |

---

## 10. Segurança (uma regra só)

**Não abra a porta 8000 para a internet** (nada de port forwarding no
roteador). O cncjs é para uso na sua rede local — e na LAN doméstica, sem
senha, é o padrão de uso normal dessa ferramenta. Se um dia precisar de acesso
externo, monte uma VPN — não exponha a porta.

---

## Resumo da arquitetura (o que roda onde)

| Componente | Onde | Papel |
|---|---|---|
| Rayforge | Seu PC | Design, steps, potência, sanity, **export do G-code** |
| cncjs | mini PC (Debian) | Web UI, recebe o arquivo, **streama via USB** |
| K1 | — | Executa (GRBL) |
| WiFi | entre PC e mini PC | Só o upload do arquivo (o job roda local no servidor) |
