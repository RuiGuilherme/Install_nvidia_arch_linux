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

#### Configurando o sistema para receber o driver (Não se esqueça de salvar)

`sudo nano /etc/modprobe.d/nvidia.conf`

```bash
blacklist nouveau
blacklist nvidiafb
blacklist rivafb
```

================================

`sudo nano /etc/modprobe.d/nvidia-drm.conf`

```bash
options nvidia_drm modeset=1
```

================================

`sudo nano /usr/local/bin/optimus.sh`

```bash
#!/bin/sh

xrandr --setprovideroutputsource modesetting NVIDIA-0
xrandr --auto
```
`sudo chmod a+rx /usr/local/bin/optimus.sh`

================================

`sudo nano /usr/local/share/optimus.desktop`


```bash
[Desktop Entry]
Type=Application
Name=Optimus
Exec=sh -c "xrandr --setprovideroutputsource modesetting NVIDIA-0; xrandr --auto"
NoDisplay=true
X-GNOME-Autostart-Phase=DisplayServer
```

================================

`sudo ln -s /usr/local/share/optimus.desktop /usr/share/gdm/greeter/autostart/optimus.desktop`

Talvez esse primeiro ln dê algum problema, mas não vai atrapalhar nos proximos passos.

`sudo ln -s /usr/local/share/optimus.desktop /etc/xdg/autostart/optimus.desktop`

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
    BusID "PCI:1:0:0"
    Option "AllowEmptyInitialConfiguration"
EndSection
```

##### Observação

A linha de BusID é opcional, porém você tem que alterar o "1" para o ID da sua GPU, no meu caso eu apenas deletei essa linha(Já que o BusID é opciona), caso queria saber qual é o Bus de seu equipamento entre com o seguinte comando `lspci -k | grep -A 2 -E "(VGA|3D)"` e altere de acordo com a saída o arquivo `/optimus.conf`

================================

### Vamos checar se está tudo ok.

Verifique a saída dos arquivos e veja se tudo foi salvo corretamente.

`cat /etc/modprobe.d/nvidia.conf`

`cat /etc/modprobe.d/nvidia-drm.conf`

`cat /usr/local/bin/optimus.sh`

`cat /usr/local/share/optimus.desktop`

`cat /usr/share/sddm/scripts/Xsetup`

`cat /etc/X11/xorg.conf.d/optimus.conf`

## Iniciando os serviços

#### Observações
O bbswitch-dkms, optimus-manager-qt ou o optimus-manager-qt-git provavelmente não iram funcionar, mas você pode tentar. ;) 

No caso vamos usar o **optimus-manager**

`yay -S optimus-manager`

Logo após instalar, tente mudar para a nvidia executando o seguinte comando

`optimus-manager --switch nvidia`

E por final(caso ele não faça logout): `reboot`

Caso esteja usando o SwayWM o sistema não vai aplicar e nem fazer logout(Sway não suporta driver fechado). Nesta situação será preciso usar o i3wm ou alguma DE(KDE, Gnome...).

Outra observação é que você precisa de uma DM(gdm, sddm, lightdm...) para poder iniciar o driver(Infelizmente usando o sx o driver da nvidia não inicia de forma alguma, sendo necesario uma DM - No caso eu usei o ssdm).

Após o sistema voltar e entrar em sua DE você pode checar qual driver está rodando executando o comando `glxinfo -B`.

Caso ainda não esteja funcionando é muito provavel que você esteja rodando uma versão do driver não suportada pela sua placa de vídeo (Como foi o meu caso), a GTX 820M só é suportada pela versão 430(No aguardo da 440 hehe) e n tem suporte para versão 435.

Entre com o seguinte comando `pacman -Ss nvidia` e caso a saída de informações desse comando seja os drivers 430, 435 ou o beta(440) é provavel que o seu o problema seja a versão do driver, neste caso eu recomendo usar a versão 390xx executando o seguinte comando: `sudo pacman -S nvidia-390xx-dkms  nvidia-390xx-settings opencl-nvidia-390xx lib32-nvidia-390xx-utils lib32-opencl-nvidia-390xx nvidia-390xx-utils`

Caso você tenha algum problema com dependência e conflitos, remova os pacotes conflitantes usando o `-Rdd`

E logo em seguida execute o comando `optimus-manager --switch nvidia`(é normal ele fazer logout), faça login na sua DE e em seguida execute o comando `glxinfo -B` para checar qual driver está rodando.

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



## Créditos
[felipefacundes](https://github.com/felipefacundes) - Salvador no Telegram. 
