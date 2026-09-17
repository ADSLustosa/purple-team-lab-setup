# 🐉 Instalação do Kali Linux

Instalação via ISO oficial (não imagem pronta), para dominar o processo completo de particionamento e configuração.

## 1. Baixar a ISO oficial

```
https://www.kali.org/get-kali/#kali-installer-images
```

Escolha a versão **"Installer" (64-bit)**. Valide a integridade:

```powershell
Get-FileHash "C:\Users\<usuario>\Downloads\kali-linux-*-installer-amd64.iso" -Algorithm SHA256
```

Compare com o hash publicado na página oficial.

## 2. Criar a VM via VBoxManage

```powershell
$vbox = "C:\Program Files\Oracle\VirtualBox\VBoxManage.exe"

New-Item -Path "D:\VMs\Kali-Attacker" -ItemType Directory -Force

& $vbox createvm --name "Kali-Attacker" --ostype "Debian_64" --register
& $vbox modifyvm "Kali-Attacker" --memory 4096 --cpus 2 --vram 32

& $vbox createmedium disk --filename "D:\VMs\Kali-Attacker\Kali-Attacker.vdi" --size 40960

& $vbox storagectl "Kali-Attacker" --name "SATA Controller" --add sata --controller IntelAHCI
& $vbox storageattach "Kali-Attacker" --storagectl "SATA Controller" --port 0 --device 0 --type hdd --medium "D:\VMs\Kali-Attacker\Kali-Attacker.vdi"

& $vbox storagectl "Kali-Attacker" --name "IDE Controller" --add ide
& $vbox storageattach "Kali-Attacker" --storagectl "IDE Controller" --port 0 --device 0 --type dvddrive --medium "C:\Users\<usuario>\Downloads\kali-linux-<versao>-installer-amd64.iso"

& $vbox modifyvm "Kali-Attacker" --nic1 nat
& $vbox startvm "Kali-Attacker"
```

## 3. Instalação guiada

No menu de boot, selecione **"Graphical install"**. Siga as telas:

1. Idioma (recomendado: English, facilita busca de soluções técnicas depois)
2. Localização: Brazil
3. Layout de teclado
4. Hostname / domínio
5. Criação de usuário e senha
6. **Particionamento de disco** — use "Guided - use entire disk" para o lab (não é ambiente de produção)
7. Instalação dos pacotes base (leva alguns minutos)
8. Instalação do GRUB no disco principal

## 4. Guest Additions (integração com o host)

```bash
sudo apt update
sudo apt install -y build-essential dkms linux-headers-$(uname -r)
```

No menu do VirtualBox: `Dispositivos > Inserir imagem de CD das Adições de Convidado`

```bash
sudo mkdir -p /media/cdrom
sudo mount /dev/sr0 /media/cdrom
cd /media/cdrom
sudo bash ./VBoxLinuxAdditions.run
sudo reboot
```

Habilitar clipboard: `Dispositivos > Área de Transferência Compartilhada > Bidirecional`

---

## ⚠️ Problemas encontrados

### Senha rejeitada na tela de login
**Sintoma:** senha criada na instalação não era aceita no LightDM.
**Causa provável:** incompatibilidade de layout de teclado.
**Solução:** boot no GRUB → `Advanced options` → `recovery mode` → `root shell` →
```bash
mount -o remount,rw /
mount -t proc proc /proc
passwd <usuario>
reboot -f
```

### Guest Additions: `mount: Can't open blockdev`
**Sintoma:** erro ao montar `/dev/cdrom`.
**Causa:** imagem de CD não estava de fato inserida na VM.
**Solução:** reinserir via menu `Dispositivos`, validar com `lsblk` (procurar `sr0` com ~50MB), montar via `/dev/sr0` em vez do link simbólico.

### `linux-headers-$(uname -r)` não encontrado
**Sintoma:** `Unable to locate package linux-headers-X.X.X+kali-amd64`.
**Causa:** kernel rodando mais novo que os headers disponíveis no espelho do repositório.
**Solução:**
```bash
sudo apt install -y linux-headers-amd64
# se persistir:
sudo apt full-upgrade -y && sudo reboot
```
