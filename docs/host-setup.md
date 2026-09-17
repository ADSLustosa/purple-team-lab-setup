# 🖥️ Configuração do Host

Este guia documenta a preparação do host Windows para rodar o laboratório com performance adequada, sem conflitos de virtualização.

## Pré-requisitos

- Windows 10/11
- Virtualização de hardware habilitada na BIOS (Intel VT-x / AMD-V)
- Oracle VirtualBox 7.x + Extension Pack

## 1. Desabilitar Hyper-V e Virtual Machine Platform

O Hyper-V do Windows compete pelo mesmo hypervisor que o VirtualBox precisa usar diretamente. Rode como Administrador:

```powershell
Disable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V-All -NoRestart
Disable-WindowsOptionalFeature -Online -FeatureName VirtualMachinePlatform -NoRestart
bcdedit /set hypervisorlaunchtype off
Restart-Computer -Force
```

Valide após o reboot:

```powershell
Get-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V-All
Get-WindowsOptionalFeature -Online -FeatureName VirtualMachinePlatform
```

Ambos devem retornar `State: Disabled`.

## 2. Habilitar AMD-V (SVM) ou Intel VT-x na BIOS

Em placas-mãe Gigabyte (testado em B550M Aorus Elite), o SVM Mode vem desabilitado por padrão:

1. Reinicie e pressione `Delete` para entrar na BIOS
2. Pressione `F2` para o modo avançado
3. Navegue até `M.I.T. > Advanced Frequency Settings > Advanced CPU Core Settings`
4. Altere `SVM Mode` para `Enabled`
5. Pressione `F10` para salvar e sair

Valide no Windows:

```powershell
Get-ComputerInfo -Property "HyperV*" | Select-Object HyperVRequirementVirtualizationFirmwareEnabled
```

Deve retornar `True`.

## 3. Instalar o VirtualBox

1. Baixe em [virtualbox.org/wiki/Downloads](https://www.virtualbox.org/wiki/Downloads) (Windows hosts)
2. Baixe também o **Extension Pack** na mesma página
3. Instale o VirtualBox, aceitando os drivers de rede/USB
4. Instale o Extension Pack via `Arquivo > Preferências > Extensões`

Valide:

```powershell
& "C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" --version
& "C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" list extpacks
```

## 4. Definir pasta padrão de VMs no drive de melhor performance

Se você tem SSD NVMe separado do drive do sistema, direcione as VMs para lá:

```powershell
& "C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" setproperty machinefolder "D:\VMs"
```

---

## ⚠️ Problemas encontrados

### Conflito Hyper-V x VirtualBox
**Sintoma:** VM não ligava / performance ruim.
**Causa:** Hyper-V/VirtualMachinePlatform ativos disputando o hypervisor.
**Solução:** ver seção 1 acima.

### AMD-V (SVM) desabilitado na BIOS
**Sintoma:** `VERR_SVM_DISABLED` ao ligar a VM, mesmo com Hyper-V já desativado.
**Causa:** BIOS de placas Gigabyte vem com SVM desabilitado por padrão.
**Solução:** ver seção 2 acima.
