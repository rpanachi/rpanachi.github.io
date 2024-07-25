---
layout: post
title: "[pt-BR] Transforme seu roteador wi-fi em um NAS e Media Server UPnP/DLNA com OpenWRT"
date: 2013-09-24 00:00:00
categories: openwrt hacking mybrain
excerpt: Laugh in the face of danger
disqus: true
archive: true
---


> The Linux philosophy is 'Laugh in the face of danger'. Oops. Wrong One. 'Do it yourself'. Yes, that's it.<br/>
> ―  Linus

<b>TL;DR</b> [OpenWRT](https://openwrt.org/) é uma distro Linux embarcável em roteadores que permite customizar e instalar serviços sem a necessidade de compilar um novo firmware. Em outras palavras, é um "firmware on steroids" para roteadores.

## Mas por quê?
* Economia. Apenas substituindo o firmware do roteador é possível adicionar funcionalidade e recursos, economizando uma grana preciosa com dispositivos semelhantes, como AppleTV, NAS dedicado, etc.
* Funcionalidade. Provavelmente o roteador fica ligado 100% do tempo na sua casa, sendo utilizado apenas para compartilhar a internet. Praticamente um servidor disponível 24 horas por dia sem utilizaçao! Não mais…
* Diversão. Substituir o firmware do seu roteador por uma distro baseada em Linux e configurar todos os serviços manualmente… é diversão pura!
* Por que eu quero (e posso)!

## Instalação

<b>Atenção: existe a possibilidade (embora pequena) de algo sair muito errado e você ganhar um peso de papel. Prossiga por sua conta e risco... ou se tiver coragem!</b>

O primeiro passo é descobrir o modelo do seu roteador e verificar a compatibilidade nesta página: <http://wiki.openwrt.org/toh/start>. Se não encontrar o modelo ou houver um aviso de não-compatibilidade, sinto muito mas não vai rolar para você.

Se seu roteador for compatível, haverá o target da imagem e um link para uma wiki com um how-to específico do modelo, instruções de instalação, resolução de problemas, etc. Procure pelo nome da release que você deverá baixar daqui <http://downloads.openwrt.org>. A imagem tem o formato <i>openwrt-TARGET-generic-MODELO-VERSAO-squashfs-factory.bin</i>

Por exemplo, meu roteador é um <i>TP-Link WR1043ND</i>. Procurando na página <http://wiki.openwrt.org/toh/start>, o target compatível com meu roteador é <i>ar71xx</i>:

![OpenWRT downloads page](/assets/images/0_mhdepUyx8jJt2Qvu.webp)

Acessei a página de downloads, naveguei pelos diretórios da release, depois target e encontrei o arquivo <i>openwrt-ar71xx-generic-tl-wr1043nd-v1-squashfs-factory.bin</i> para download.

![OpenWRT downloads page](/assets/images/0_75wTLjZloj7K3muC.webp)

Quando baixar a imagem, é só fazer a atualização de firmware normalmente no seu roteador pelo admin:

![TP-Link admin page](/assets/images/0_RU6NFeEoy7Ah_UYy.webp)

Confirme e reze muito! Aguarde o roteador atualizar e reiniciar.

Deste ponto em diante, será necessário conectar no roteador com cabo, para setar uma senha de root e configurar o wifi. Este processo é relativamente trivial, basta utilizar a interface web do admin, sem segredo.

![OpenWRT settings](/assets/images/0_Yn4God7687sKVtU6.webp)

Uma vez definida a senha e o wifi configurado, é possível acessar seu roteador via ssh:


```
$ ssh root@192.168.1.1
root@192.168.1.1's password:
 
BusyBox v1.19.4 (2013-03-14 11:28:31 UTC) built-in shell (ash)
Enter 'help' for a list of built-in commands.
  _______                     ________        __
 |       |.-----.-----.-----.|  |  |  |.----.|  |_
 |   -   ||  _  |  -__|     ||  |  |  ||   _||   _|
 |_______||   __|_____|__|__||________||__|  |____|
          |__| W I R E L E S S   F R E E D O M
 -----------------------------------------------------
 ATTITUDE ADJUSTMENT (12.09, r36088) 
 ----------------------------------------------------- 
  * 1/4 oz Vodka      Pour all ingredients into mixing 
  * 1/4 oz Gin        tin with ice, strain into glass. 
  * 1/4 oz Amaretto 
  * 1/4 oz Triple sec 
  * 1/4 oz Peach schnapps 
  * 1/4 oz Sour mix 
  * 1 splash Cranberry juice 
 ----------------------------------------------------
root@OpenWrt:~#
```

Se você chegou até aqui, parabéns! Você teve coragem. E felizmente a parte difícil já passou.

Nota: dependendo do seu roteador, a versão do OpenWRT pode variar. Leia a [wiki](http://wiki.openwrt.org/) do modelo do seu roteador para instruções específicas e resolução de problemas.

<b>Importante</b>: caso o pior aconteça (como acabar a energia no meio do processo de flash da firmware) e você não queira utilizar seu roteador como peso de papel, tente seguir os procedimento de "debricking" em <http://wiki.openwrt.org/doc/howto/generic.debrick>

## Configurando o NAS

Para o NAS, vamos montar as partições do HD externo e configurar uma partição de swap pois o roteador provavelmente não tem memória interna suficiente para dar conta do recado.

Neste exemplo, vou usar meu HD de 1TB com uma partição formatada em ext4 e outra partição de 1GB formatada como linux+swap. Não vou usar FAT32 nem NTFS e nem recomendo! A idéia aqui é deixar o HD plugado eternamente no roteador e acessá-lo pela rede.

### Instalando os pacotes necessários

Para nossa sorte, o OpenWRT conta com um gerenciador de pacotes que facilita (e muito) a instalação das libs e módulos necessários para montar o HD externo e compartilhá-lo na rede via Samba ou NFS.

Logue no roteador via ssh e instale os seguintes pacotes:

```
root@OpenWrt:~# opkg update 
root@OpenWrt:~# opkg install kmod-usb-storage block-mount kmod-fs-ext4
```

O ultimo pacote vai depender do sistema de arquivos do seu HD. Por exemplo, kmod-fs-ext2, etc. Veja os módulos disponíveis em <http://wiki.openwrt.org/doc/howto/storage>

Verifique se as partições já foram detectadas rodando `blkid`:

```
root@OpenWrt:~# blkid
/dev/mtdblock2: TYPE="squashfs"
/dev/sda1: LABEL="MEDIA" UUID="1ecad5f1–0000–0000–000f-e8b7f6ac651d" TYPE="ext4"
```

### Montando as partições do HD

Agora vamos configurar o fstab para montar as partições automaticamente quando o roteador for ligado. Edite o arquivo `/etc/config/fstab` com o seguinte (use o vi):

```
config global automount
        option from_fstab 1
        option anon_mount 0
config global autoswap
        option from_fstab 1
        option anon_swap 0
config mount
        option target   /mnt/media
        option device   /dev/sda1
        option fstype   ext4
        option options  rw,sync
        option enabled  1
        option enabled_fsck 0
config swap
        option device   /dev/sda2         
        option enabled  1
```

No meu caso, vou montar a partição `/dev/sda1` em `/mnt/media`, então é necessário criar o diretório de destino, ativar e iniciar o serviço `fstab`:

```
root@OpenWrt:~# mkdir /mnt/media
root@OpenWrt:~# /etc/init.d/fstab enable
root@OpenWrt:~# /etc/init.d/fstab start
```

Pronto, seu HD externo pode ser acessado em /mnt/media e o swap foi montado na segunda partição.

### Compartilhando na rede

Vou utilizar NFS para compartilhar a partição na rede, mas você também pode utilizar Samba seguindo <http://wiki.openwrt.org/doc/howto/cifs.server>

```
root@OpenWrt:~# opkg update
root@OpenWrt:~# opkg install nfs-kernel-server
```

Edite (ou crie) o arquivo `/etc/exports` com o conteúdo:

```
/mnt/media 192.168.1.0/255.255.255.0(rw,sync,no_subtree_check)
```

Ative e inicie os serviços necessários:

```
root@OpenWrt:~# /etc/init.d/portmap start
root@OpenWrt:~# /etc/init.d/portmap enable
root@OpenWrt:~# /etc/init.d/nfsd start
root@OpenWrt:~# /etc/init.d/nfsd enable
```

Pronto! O server (roteador) está configurado. Para montar esta partição no client, por exemplo Ubuntu, siga:

```
$ sudo apt-get install nfs-common
$ sudo mkdir /media/nas
$ sudo mount -t nfs 192.168.1.1:/mnt/media /media/nas
```

Groovy! Seu NAS foi montado no seu desktop em `/media/nas`. Pode compartilhar seus arquivos a vontade!

## Configurando o Media Server (minidlna)

Logue no roteador e instale os pacotes necessários:

```
root@OpenWrt:~# opkg update
root@OpenWrt:~# opkg install minidlna
```

Para configurar, basta editar o arquivo `/etc/config/minidlna` como segue:

```
config minidlna config
  option 'enabled' '1'
  option port '8200'
  option interface 'br-lan'
  option friendly_name 'OpenWrt DLNA Server'
  option db_dir '/mnt/media/minidlna/db'
  option log_dir '/mnt/media/minidlna/log'
  option inotify '1'
  option enable_tivo '0'
  option strict_dlna '0'
  option presentation_url ''
  option notify_interval '900'
  option serial '12345678'
  option model_number '1'
  option root_container '.'
  list media_dir '/mnt/media'
  option album_art_names 'Cover.jpg/cover.jpg/AlbumArtSmall.jpg/albumartsmall.jpg/AlbumArt.jpg/albumart.jpg/Album.jpg/album.jpg/Folder.jpg/folder.jpg/Thumb.jpg/thumb.jpg'
```

Ative e inicie o serviço:

```
root@OpenWrt:~# /etc/init.d/minidlna enable
root@OpenWrt:~# /etc/init.d/minidlna start
```

Pronto! O Media Server está configurado e compartilhando seus arquivos via DLNA na rede. Aqui na minha SmartTV aparece assim:

![Media Server on Smart TV](/assets/images/0_57VW_86gWm9QEafE.webp)

## Conclusão

Embora relativamente complicado, todo procedimento para instalação e configuração não é nada diferente do que estamos acostumados a fazer diariamente no Linux, como devops, seja configurando uma VPS ou montando um NAS na rede.

A liberdade que o OpenWRT oferece é imensa. O repositório é recheado com pacotes bem úteis e muito fáceis de configurar. Sem falar na economia comparando com um aparelho como AppleTV (que não ofecererá tantos recursos) ou outros media servers do mercado.

## Referências
* <http://wiki.openwrt.org/doc/howto/usb.storage>
* <http://wiki.openwrt.org/doc/howto/nfs.server>
* <http://wiki.openwrt.org/doc/uci/minidlna>
* <http://wiki.openwrt.org/doc/uci/transmission>
* <http://wiki.openwrt.org/toh/tp-link/tl-wr1043nd>
* <http://www.rodrigorodrigues.info/index.php/2012/01/openwrt-fazendo-magica-com-linux-no-roteador-parte-1/>
* <http://knowhow.bart.prokop.name/install/openwrt/wr1043nd>

E aí, tem coragem??? Poste sua experiência nos comentários! :)
