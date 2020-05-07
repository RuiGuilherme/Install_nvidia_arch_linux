# Nvidia GTX820M Arch Linux

Primeiramente gostaria agradecer o [felipefacundes](https://github.com/felipefacundes) pelas suas 24 horas de apoio e suporte nesta instalação longa e cheias de supresas. 

**Telegram: @FeFacundes**


## Neofetch

```bash
                   -`                    ArchLinux@ArchLinux 
                  .o+`                   ---------------------- 
                 `ooo/                   OS: Arch Linux x86_64 
                `+oooo:                  Host: Aspire E5-571G V1.26 
               `+oooooo:                 Kernel: 5.3.7-arch1-1-ARCH 
               -+oooooo+:                Uptime: 19 mins 
             `/:-:++oooo+:               Packages: 1324 (pacman) 
            `/++++/+++++++:              Shell: bash 5.0.11 
           `/++++++++++++++:             Resolution: 1366x768 
          `/+++ooooooooooooo/`           WM: sway 
         ./ooosssso++osssssso+`          Theme: Breeze [GTK2/3] 
        .oossssso-````/ossssss+`         Icons: candy-icons [GTK2/3] 
       -osssssso.      :ssssssso.        Terminal: termite 
      :osssssss/        osssso+++.       Terminal Font: Hack Nerd Font 12 
     /ossssssss/        +ssssooo/-       CPU: Intel i7-5500U (4) @ 3.000GHz 
   `/ossssso+/:-        -:/+osssso+-     GPU: Intel HD Graphics 5500 
  `+sso+:-`                 `.-/+oso:    GPU: NVIDIA GeForce 610M/710M/810M/820M / GT 620M/625M/630M/720M 
 `++:.                           `-/+/   Memory: 1774MiB / 7883MiB 
 .`                                 `/
```

## Instalação
Primeiro passo é instalar as dependências básicas do sistema, tanto da Intel quanto da Nvidia 

`Habilite o Multilib em /etc/pacman.conf`

`Retire a hashtag antes das duas linhas: [multilib] e Include = /etc/pacman.d/mirrorlist`

### Nvidia

> pacman -Syy nvidia-dkms linux-headers dkms nvidia-settings lib32-libvdpau lib32-libglvnd libglvnd libvdpau nvidia-utils opencl-nvidia xsettingsd xsettings-client ffnvcodec-headers libxnvctrl xf86-video-nouveau lib32-nvidia-utils lib32-opencl-nvidia nccl nvidia-cg-toolkit


### Intel

>pacman -Syy lib32-vulkan-intel lib32-mesa lib32-libva1-intel-driver lib32-libva-intel-driver libva1-intel-driver libva-utils intel-opencl-clang intel-media-driver intel-graphics-compiler lib32-libglvnd libglvnd linux-headers dkms intel-gpu-tools intel-gmmlib intel-compute-runtime i810-dri xf86-video-intel vulkan-intel mesa libva-intel-driver iucode-tool intel-ucode intel-tbb

Créditos: [Instalação Arch, por Felipe Facundes](https://github.com/felipefacundes/desktop/tree/master/Arch_linux_Install#preparar-para-jogos-todas-%C3%A0s-depend%C3%AAncias-necess%C3%A1rias-inclusive-para-aumentar-consideravelmente-a-performance-em-jogos)

##### Observações: 
- Irei usar a [SDDM](https://wiki.archlinux.org/index.php/SDDM) como DM
- Irei usar o [KDE Plasma Xorg](https://wiki.archlinux.org/index.php/KDE) como DE
- Irei usar o driver dkms
- GeFoce GT 820M é uma placa Fermi, então só tem suporte no driver legacy 390xx, pode ser consultado aqui: [Link](https://www.nvidia.com/en-us/drivers/unix/legacy-gpu/)
- Irei usar o [optimus-manager](https://aur.archlinux.org/packages/optimus-manager/) para controlar e alternar entre os gráficos. 
#### Configurando o sistema para receber o driver (Não se esqueça de salvar)

`sudo nano /usr/local/bin/optimus.sh`

```bash
#!/bin/sh

xrandr --setprovideroutputsource modesetting NVIDIA-0
xrandr --auto
```
`sudo chmod a+rx /usr/local/bin/optimus.sh`

================================

`sudo nano /usr/share/sddm/scripts/Xsetup`

```bash
#!/bin/sh

xrandr --setprovideroutputsource modesetting NVIDIA-0
xrandr --auto
```

`sudo chmod +x /usr/share/sddm/scripts/Xsetup`

================================

`sudo nano /etc/X11/xorg.conf.d/optimus.conf`


```bash
Section "Module"
    Load "modesetting"
EndSection

Section "Device"
    Identifier "nvidia"
    Driver "nvidia"
    #BusID "PCI:1:0:0"
    Option "AllowEmptyInitialConfiguration"
EndSection
```

##### Observação

A linha de BusID é opcional, porém você tem que alterar o "1" para o ID da sua GPU, no meu caso eu apenas deletei essa linha(Já que o BusID é opciona), caso queria saber qual é o Bus de seu equipamento entre com o seguinte comando `lspci -k | grep -A 2 -E "(VGA|3D)"` e altere de acordo com a saída o arquivo `/etc/X11/xorg.conf.d/optimus.conf`

================================

### Vamos checar se está tudo ok.

Verifique a saída dos arquivos e veja se tudo foi salvo corretamente.

`cat /usr/local/bin/optimus.sh`

`cat /usr/share/sddm/scripts/Xsetup`

`cat /etc/X11/xorg.conf.d/optimus.conf`

## Iniciando os serviços

#### Observações
O bbswitch-dkms, optimus-manager-qt ou o optimus-manager-qt-git provavelmente não iram funcionar, mas você pode tentar. ;) 

Como já foi dito, vamos usar o **optimus-manager**

`yay -S nvidia-390xx-dkms nvidia-390xx-settings opencl-nvidia-390xx lib32-nvidia-390xx-utils lib32-opencl-nvidia-390xx nvidia-390xx-utils optimus-manager`

`reboot`

Quando a maquina voltar 

`optimus-manager --switch nvidia`
ou
`optimus-manager --switch hybrid`
[Mais detalhes](https://github.com/Askannz/optimus-manager)

Ele vai fazer logout, para checar qual driver está rodando basta executar `glxinfo -B`

##### A saída deve ser algo assim:

```bash

[ArchLinux@ArchLinux ~]$ glxinfo -B
name of display: :0.0
display: :0  screen: 0
direct rendering: Yes
Memory info (GL_NVX_gpu_memory_info):
    Dedicated video memory: 2048 MB
    Total available memory: 2048 MB
    Currently available dedicated video memory: 1964 MB
OpenGL vendor string: NVIDIA Corporation
OpenGL renderer string: GeForce 820M/PCIe/SSE2
OpenGL core profile version string: 4.6.0 NVIDIA 390.129
OpenGL core profile shading language version string: 4.60 NVIDIA
OpenGL core profile context flags: (none)
OpenGL core profile profile mask: core profile

OpenGL version string: 4.6.0 NVIDIA 390.129
OpenGL shading language version string: 4.60 NVIDIA
OpenGL context flags: (none)
OpenGL profile mask: (none)

OpenGL ES profile version string: OpenGL ES 3.2 NVIDIA 390.129
OpenGL ES profile shading language version string: OpenGL ES GLSL ES 3.20
```
