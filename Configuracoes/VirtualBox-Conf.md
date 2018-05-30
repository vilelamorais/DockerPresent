Construção do ambiente virtual (utilizando VirtualBox)

Desenho de rede


|-----------|     |---------------------------------------------------------|
|           |     |                                                         |
|  Acesso   |     |    Equipamento Host (Virtualizador)                     |
|     a     |-----|            |                                            |
| internet  |     |            |                                            |
|           |     |  |---------|---------- VirtualBox -------------------|  |
|-----------|     |  |         |                                         |  |
      |           |  |         | Host-only adapter (entrada 2)           |  |
      |           |  |         |                                         |  |
      |           |  |   |-----------|                                   |  |
      |           |  |   |           |                                   |  |
      |           |  |   |  Debian   |                                   |  |
      |-----------|--|---|           |---------|                         |  |
   Bridged        |  |   |   NAT     |         |                         |  |
   adapter        |  |   |           |         |  Internal network       |  |
  (entrada 1)     |  |   |-----------|         |   (entrada 3)           |  |
                  |  |                         |                         |  |
                  |  |                         |                         |  |
                  |  |         |---------------|----------------|        |  |
                  |  |         |               |                |        |  |
                  |  |   |-----------|   |-----------|   |-----------|   |  |
                  |  |   |           |   |           |   |           |   |  |
                  |  |   |           |   |           |   |           |   |  |
                  |  |   |   BOX 01  |   |   BOX 02  |   |  BOX N+1  |   |  |
                  |  |   |           |   |           |   |           |   |  |
                  |  |   |           |   |           |   |           |   |  |
                  |  |   |-----------|   |-----------|   |-----------|   |  |
                  |  |                                                   |  |
                  |  |---------------------------------------------------|  |
                  |                                                         |
                  |---------------------------------------------------------|


Elementos de configuração

- Acesso a internet

Não existem restrições ou características especiais para a conexão com a internet.

- Equipamento Host

Não existem restrições ou características especiais para o equipamento utilizado.

- VirtualBox

Utilizada a versão 5.2.10 com Oracle VM VirtualBox Extension Pack 5.2.10

Para a composição da rede virtualizada utilizada uma rede interna ao VirtualBox utilizando o range ﻿192.168.57.0/24.
Para a criação da rede interna utiliza os seguintes comandos:

VBoxManage hostonlyif create
   >> como resultado do comando foi criada a rede vboxnet1

VBoxManage hostonlyif ipconfig vboxnet1 --ip ﻿192.168.57.3 --netmask 255.255.255.0
   >> como resultado do comando a range de ip foi alterado para o range especificado

O comando pode variar entre maiúscula e minúscula dependendo do sistema operacional utilizado. Para a apresentação criada o sistema operacional principal foi o MacOS.

- Debian NAT

Para criar a máquina virtual que será a centralizadora de requisições para a internet (default gateway) foi utilizada a distribuição Debian versão 9.4.0-amd64.
A maquina virtual precisa ter 3 interfaces de rede sendo:
   * Adapter 1
     Marcado como ativo
     Attached to: Bridge adapter
     Name: en1:Wi-Fi (AirPort) >> neste caso para o uso da interface de rede wireless da máquina host
     Não foram adicionadas configurações avançadas
   * Adapter 2
     Marcado como ativo
     Attached to: Host-only adapter
     Name: vboxnet1 >> criada anteriormente
     Não foram adicionadas configurações avançadas
   * Adapter 3
     Marcado como ativo
     Attached to: Internal Network
     Name: DEB_NET >> nome usado para criar a rede interna, criado no momento da definição da interface de rede
     Não foram adicionadas configurações avançadas

Não existem requisitos definidos para a configuração da máquina de forma geral, somente para configuração de NETWORK e NAT.

Para a definição das placas de rede foi editado o arquivo /etc/network/interfaces (no uso de SO Debian) e incluida as seguintes definições:

#######=========================================================================
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# Configuração para obter IP do roteador/acesso de internet
auto enp0s3
allow-hotplug enp0s3
iface enp0s3 inet dhcp

# Interface de acesso entre a maquina virtual e o host
auto enp0s8
allow-hotplug enp0s8
iface enp0s8 inet static
      address 192.168.57.10
      netmask 255.255.255.0

# Interface para configuração da rede interna do virtual box
auto enp0s9
allow-hotplug enp0s9
iface enp0s9 inet static
      address 192.168.2.7
      netmask 255.255.255.0

#######=========================================================================

Para o SO CentOS, considere as configurações a seguir:

Editar os arquivos de cada interface contído no diretório /etc/sysconfig/network-scripts/

## ifcfg-enp0s3 ================================================================

TYPE="Ethernet"
PROXY_METHOD="none"
BROWSER_ONLY="no"
BOOTPROTO="dhcp"
DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
IPV6INIT="no"
NAME="enp0s3"
UUID="aecc1275-c529-42bf-b35b-78f8da8ec578"
DEVICE="enp0s3"
ONBOOT="yes"

## ifcfg-enp0s8 ================================================================

TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=none
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=no
NAME=enp0s8
UUID=27a67e49-5740-4368-b509-9de96d45945f
DEVICE=enp0s8
ONBOOT=yes
IPADDR=192.168.57.11
PREFIX=24
GATEWAY=192.168.57.3

## ifcfg-enp0s9 ================================================================

TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=none
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=no
NAME=enp0s9
UUID=8d25a481-382b-4fae-b7bf-989609b3189f
DEVICE=enp0s9
ONBOOT=yes
IPADDR=192.168.2.8
PREFIX=24

#######=========================================================================

Para realizar o compartilhamento e definição desta maquina virtual, foi utilizado o IPTABLES para as definições de NAT e mascaramento da REDE

Para realizar o encaminhamento de requisições o Kernel deve ser configurado para isso. Foi feito utilizando o comando:

sysctl -w net.ipv4.ip_forward=1

Para definir as regras de encaminhamento, inicialmente foi realizada a limpeza de todas as regras das tabelas do IPTABLES

iptables --flush
iptables --table nat --flush
iptables --delete-chain
iptables --table nat --delete-chain

E definida a regra de NAT e Mascaramento para a interface de acesso a internet

iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE
iptables -I FORWARD -o enp0s3 -s 192.168.0.0/16 -j ACCEPT
iptables -I INPUT -s 192.168.0.0/16 -j ACCEPT

Adiciona as entradas de DNS no arquivo resolv.conf conforme abaixo
nameserver 8.8.8.8
nameserver 8.8.4.4

Algumas fontes e referências utilizadas:
- Configuração de redes virtuais
https://precisionsec.com/virtualbox-host-only-network-cuckoo-sandbox-0-4-2/
https://superuser.com/questions/429432/how-can-i-configure-a-dhcp-server-assigned-to-a-host-only-net-in-virtualbox
https://thornelabs.net/2015/08/24/virtualbox-commands-cheat-sheet.html
https://www.virtualbox.org/manual/ch08.html#vboxmanage-hostonlyif

- Configuração de NAT e mascaramento de redes internas com iptables
https://medium.com/@TarunChinmai/sharing-internet-connection-from-a-linux-machine-over-ethernet-a5cbbd775a4f
http://www.linuxandubuntu.com/home/how-to-configure-iptables-firewall-in-linux
https://www.systutorials.com/1372/setting-up-gateway-using-iptables-and-route-on-linux/
https://www.karlrupp.net/en/computer/nat_tutorial
https://www.howtoforge.com/internet-connection-sharing-masquerading-on-linux
https://www.howtoforge.com/nat_iptables

- Configuração de interface de rede para CetnOS
https://www.centos.org/docs/5/html/Deployment_Guide-en-US/s1-networkscripts-interfaces.html

- Docker
