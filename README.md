# Manual de instalação pihole (docker)

## Introdução

Este manual conta com os passos para instalar o sistema operativo e o software piHole para funcionar em conjunto com outros container docker, como, por exemplo, o Nextcloud e servidores SAMBA.

## Setup

Para esse guia, será utilizado um Raspberry Pi 5 4 GB como máquina principal. Caso queira, é possível utilizar qualquer máquina que seja x86 ou ARM64 (aarch64), tendo em mente que essa máquina deve contar com um suporte no mínimo OK.

### Instalação do OS (Sistema Operacional)

Como utilizarei um Raspberry Pi, irei utilizar um método mais simples e convencional para realizar a instalação do sistema operacional. Para isso, utilizarei o [RaspberryPi Imager](https://www.raspberrypi.com/software/). Como estou utilizando Linux para fazer esse processo, recomendo baixar a [AppImage](https://downloads.raspberrypi.com/imager/imager_latest_amd64.AppImage) para te salvar de dor de cabeça e perda de tempo com versões desatualizadas disponibilizadas por cada Distro.

Após baixar o arquivo AppImage, entre na pasta onde o arquivo foi salvo e execute-o com o comando: `sudo ./imager_2.0.4_amd64.AppImage`

**Passo a passo**:

1. Selecione o dispositivo que irá utilizar:
   ![exemplo](https://github.com/ChancellorMarko/manual_piHole/blob/main/img/1_screen.png)
2. Selecione seu sistema operacional de preferência, no meu caso utilizarei outros sistemas operacionais da própria Raspberry, como o Rasp OS Lite (64 bit):
   ![exemplo](https://github.com/ChancellorMarko/manual_piHole/blob/main/img/2_screen.png)
3. Insira as informações pertinentes de sua preferência conforme o que for solicitado.
4. Após configurar todas as informações pertinentes nas telas anteriores, você terá uma tela de acesso remoto que, dependendo da sua intenção, pode ser ou não interessante. Caso você queira acessar essa máquina remotamente para configuração via sua rede local (LAN), habilite a função SSH. Você tem duas opções para login via SSH:
   - Senha: com essa opção, você poderá escolher uma senha para realizar o login via sua rede, porém essa opção é insegura caso queira deixar essa máquina exposta na rede pública.
   - Chave SSH: uma das opções mais seguras caso queira deixar a máquina exposta na rede pública ou local.
   - Passo a passo: ![Configuração SSH](https://github.com/ChancellorMarko/manual_piHole/blob/main/configuracao_ssh.md).
5. Caso queira, pode ativar a opção de Raspberry Pi Connect, no meu caso deixarei desativado.
6. Continue para a escrita das informações no cartão SD (**Atenção: todos os dados presentes nele serão apagados**).
   ![exemplo](https://github.com/ChancellorMarko/manual_piHole/blob/main/img/5_screen.png)

### Login via SSH

Com sua máquina rodando e conectada à internet, você deverá descobrir o IP dessa máquina na sua rede local para realizar a conexão via SSH. Após descobrir o IP da sua máquina, geralmente algo parecido com 192.168.0.XXX, você poderá fazer login:

Para isso, utilize o comando abaixo para adicionar sua chave customizada:

```bash
ssh-add server-key-file-name
```

Após isso, tente realizar o login via comando no terminal:

```bash
ssh seu-usuario@ip-da-sua-maquina
```

Caso tenha problemas, tente usar a chave diretamente no comando:

```bash
ssh -i server-key-file-name seu-usuario@ip-da-sua-maquina
```

Com tudo funcionando corretamente, mova as chaves para a pasta onde deveriam estar:

```bash
mv server-key-file-name server-key-file-name.pub ~/.ssh/
```

---

### Instalação do Docker

Esta seção será destinada à instalação do Docker para realizar a instalação e configuração do Nextcloud All-in-One.

Para instalar o docker em um computador linux, utilize os seguintes comandos ou, caso queira, verifique diretamente o guia oficial em [Docker download.](https://docs.docker.com/engine/install/).

- **Adicione o repositório oficial do Docker e instale-o:**

<details>
<summary>Debian</summary>

Primeiramente adicione a chave:

```bash
# Add Docker's official GPG key:
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/debian
Suites: $(. /etc/os-release && echo "$VERSION_CODENAME")
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update
```

Instale todos os requerimentos:

```bash
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

</details>

<details>
<summary>Fedora</summary>

Primeiramente adicione a chave:

```bash
sudo dnf config-manager addrepo --from-repofile https://download.docker.com/linux/fedora/docker-ce.repo
```

Instale todos os requerimentos:

```bash
sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

</details>

<details>
<summary>RHEL</summary>

Primeiramente adicione a chave:

```bash
sudo dnf -y install dnf-plugins-core
sudo dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
```

Instale todos os requerimentos:

```bash
sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

</details>

<details>
<summary>Ubuntu</summary>

Primeiramente adicione a chave:

```bash
# Add Docker's official GPG key:
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update
```

Instale todos os requerimentos:

```bash
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

</details>

---

## Instalando Pi-Hole

### Configurar o seu IP para estático

Dependendo da distribuição utilizada você pode contar com uma interface gráfica para realizar essa configuração, porém no meu caso, utilizei a distribuição mínima o que impossibilita essa regalia.

Para configurar o IP estático numa distribuição baseada em Debian/Ubuntu como o Armbian, atualize a configuração do Netplan:

1. Primeiramente, descubra qual interface o seu dispositivo está utilizando:

```bash
sudo ip addr
```

Geralmente algo parecido com `eth0`, `enp4s3`, `wlan0` or `lan`.

2. Segundo, navegue até a pasta do Netplan:

```bash
cd /etc/netplan/
```

2. Verifique a configuração padrão e faça um backup:

No meu caso o arquivo padrão é o `armbian.yaml`.

```bash
sudo cp armbian.yaml armbian.yaml.bak
```

3. Entre no arquivo e modifique a configuração:

```bash
sudo nano armbian.yaml
```
<details>
<summary>Para rede Wifi</summary>

A configuração deve ficar similar a essa caso esteja utilizando wifi:

Arquivo original:
```yaml
network:
  version: 2
  renderer: networkd
  wifis:
    wlan0: # <- interface utilizada para conexão wireless
      dhcp4: true
      dhcp6: true
      macaddress: "xx:xx:xx:xx:xx:xx"
      access-points:
        "Nome-de-sua-rede-wifi":
          auth:
            key-management: "psk"
            password: "senha-de-sua-rede-wifi"
```

Arquivo modificado:
```yaml
network:
  version: 2
  renderer: networkd
  wifis:
    wlan0:
      dhcp4: false
      dhcp6: false
      addresses:
      - 192.168.0.XXX/24 # IP que você gostaria do seu computador manter na rede local
      routes:
      - to: default
        via: 192.168.0.1 # o gateway do seu modem é geralmente o valor especificado
      macaddress: "xx:xx:xx:xx:xx:xx" # mantenha o seu endereço mac como no arquivo original
      access-points:
        "Nome-de-sua-rede-wifi":
          auth:
            key-management: "psk"
            password: "senha-de-sua-rede-wifi"
```

**Para salvar o arquivo modificado, Ctrl + S, Ctrl + X**

</details>

<details>
<summary>Para rede cabeada</summary>

A configuração deve ficar similar a essa, caso esteja usando ethernet:

```yaml
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: false
      dhcp6: false
      link-local: []
      addresses:
        - 192.168.0.XXX/24 # IP que você gostaria do seu computador manter na rede local
      routes:
        - to: default
          via: 192.168.0.1 # o gateway do seu modem é geralmente o valor especificado
      macaddress: "xx:xx:xx:xx:xx:xx" # mantenha o seu endereço mac como no arquivo original
```

Coso tenha mais dúvidas, verifique a [documentação do Netplan](https://netplan.readthedocs.io/en/latest/using-static-ip-addresses/).

</details>

4. Após as mudanças no arquivo, teste as configurações com:

```bash
sudo netplan try
```

As vezes é necessário alterar as permissões dos arquivos para root:

```bash
sudo chmod 600 /etc/netplan/*.yaml
```

**Caso tenha problemas na configuração visite este [material adicional](https://docs.armbian.com/User-Guide_Networking/) para auxílio.**

5. Caso a configuração esteja funcionando corretamente, aplique-a para tornar ela padrão:

```bash
sudo netplan apply
```

Com isso, agora você possui um computador conectado a rede Wi-Fi e com um IP estático na sua rede local.

### Liberar as portas `53:53/tcp e udp`:

Caso você esteja utilizando uma distribuição baseada em Debian/Ubuntu ou fedora, é possível que as portas `53:53/tcp` e `53:53/udp` estejam sendo utilizadas pelo Systemd-Resolved, isso impedirá que o Docker utilize as portas impedindo o correto funcionamento da ferramenta. [(Documentação com explicação)](https://docs.pi-hole.net/docker/tips-and-tricks/)

Para resolver isso desative o stub-listener e reinicie o serviço `resolved`:

- Desabilite o stub-listener com esse comando que cria um diretório e adiciona a configuração necessária:

```bash
sudo sh -c 'mkdir -p /etc/systemd/resolved.conf.d && printf "[Resolve]\nDNSStubListener=no\n" | tee /etc/systemd/resolved.conf.d/no-stub.conf'
```

- Altere o symlink de `/etc/resolve.conf` para `/run/systemd/resolve/resolv.conf` que será atualizado juntamente com o sistema:

```bash
sudo sh -c 'rm -f /etc/resolv.conf && ln -s /run/systemd/resolve/resolv.conf /etc/resolv.conf'
```

- Reinicie o serviço systemd-resolved:

```bash
sudo systemctl restart systemd-resolved.service
```

- **Atenção em alguns casos o comando pode não funcionar corretamente, então você pode ter que modificar manualmente o arquivo ´resolved.conf´**:

Navegue até a pasta de configuração do `resolved`:

```bash
cd /etc/systemd/
```

Modifique a configuração `DNSStubListener=no`, retirando o caractere `#` do inicio:

```bash                         
#  This file is part of systemd.
#
#  systemd is free software; you can redistribute it and/or modify it under the
#  terms of the GNU Lesser General Public License as published by the Free
#  Software Foundation; either version 2.1 of the License, or (at your option)
#  any later version.
#
# Entries in this file show the compile time defaults. Local configuration
# should be created by either modifying this file (or a copy of it placed in
# /etc/ if the original file is shipped in /usr/), or by creating "drop-ins" in
# the /etc/systemd/resolved.conf.d/ directory. The latter is generally
# recommended. Defaults can be restored by simply deleting the main
# configuration file and all drop-ins located in /etc/.
#
# Use 'systemd-analyze cat-config systemd/resolved.conf' to display the full config.
#
# See resolved.conf(5) for details.

[Resolve]
# Some examples of DNS servers which may be used for DNS= and FallbackDNS=:
# Cloudflare: 1.1.1.1#cloudflare-dns.com 1.0.0.1#cloudflare-dns.com 2606:4700:4700::1111#cl>
# Google:     8.8.8.8#dns.google 8.8.4.4#dns.google 2001:4860:4860::8888#dns.google 2001:48>
# Quad9:      9.9.9.9#dns.quad9.net 149.112.112.112#dns.quad9.net 2620:fe::fe#dns.quad9.net>
#DNS=
#FallbackDNS=
#Domains=
#DNSSEC=no
#DNSOverTLS=no
#MulticastDNS=yes
#LLMNR=yes
#Cache=yes
#CacheFromLocalhost=no
DNSStubListener=no #<-- Dessa forma, remova o #
#DNSStubListenerExtra=
#ReadEtcHosts=yes
#ResolveUnicastSingleLabel=no
#StaleRetentionSec=0
```

Com essas alterações você tera desativado o serviço que por padrão utiliza a porta `53:53/tcp e udp` do sistema. Agora podemos continuar para a instalação do Pihole em um contêiner Docker.

### Configurar o `docker-compose.yml`

Para configurar o arquivo necessário para rodar o aplicativo dentro da máquina virtual Docker, será utilizado o arquivo de [configuração padrão](https://github.com/pi-hole/docker-pi-hole), porém, com algumas modificações.

Passos para a configuração e criação do contêiner:

1. Primeiramente crie uma pasta que será responsável por armazenar as configurações do docker-compose.yml e uma pasta de configuração do pihole:

Navegue até sua pasta `home/usuário`:

```bash
cd ~
```

Crie a pasta que irá armazenar os arquivos e configurações:

```bash
mkdir pihole
```

Crie o arquivo de configuração `docker-compose.yml`:

```bash
touch docker-compose.yml
```

2. Insira a configuração do pihole no arquivo criado:

```bash
nano docker-compose.yml
```

Insira essa configuração:

```yml
# More info at https://github.com/pi-hole/docker-pi-hole/ and https://docs.pi-hole.net/
services:
  pihole:
    container_name: pihole
    image: pihole/pihole:latest
    ports:
      # DNS Ports
      - "53:53/tcp"
      - "53:53/udp"
      # Default HTTP Port
      - "80:80/tcp"
      # Default HTTPs Port. FTL will generate a self-signed certificate
      - "443:443/tcp"
      # Uncomment the line below if you are using Pi-hole as your DHCP server
      #- "67:67/udp"
      # Uncomment the line below if you are using Pi-hole as your NTP server
      #- "123:123/udp"
    environment:
      # Set the appropriate timezone for your location (https://en.wikipedia.org/wiki/List_of_tz_database_time_zones), e.g:
      TZ: 'Europe/London'
      # Set a password to access the web interface. Not setting one will result in a random password being assigned
      FTLCONF_webserver_api_password: 'correct horse battery staple'
      # If using Docker's default `bridge` network setting the dns listening mode should be set to 'ALL'
      FTLCONF_dns_listeningMode: 'ALL'
    # Volumes store your data between container upgrades
    volumes:
      # For persisting Pi-hole's databases and common configuration file
      - './etc-pihole:/etc/pihole'
      # Uncomment the below if you have custom dnsmasq config files that you want to persist. Not needed for most starting fresh with Pi-hole v6. If you're upgrading from v5 you and have used this directory before, you should keep it enabled for the first v6 container start to allow for a complete migration. It can be removed afterwards. Needs environment variable FTLCONF_misc_etc_dnsmasq_d: 'true'
      #- './etc-dnsmasq.d:/etc/dnsmasq.d'
    cap_add:
      # See https://docs.pi-hole.net/docker/#note-on-capabilities
      # Required if you are using Pi-hole as your DHCP server, else not needed
      - NET_ADMIN
      # Required if you are using Pi-hole as your NTP client to be able to set the host's system time
      - SYS_TIME
      # Optional, if Pi-hole should get some more processing time
      - SYS_NICE
    restart: unless-stopped
```

3. Modificação das portas utilizadas:

Caso você não tenha mais nenhum serviço que esteja rodando e ocupando as portas `80` e `443`, não será necessário a alteração dessas duas portas no arquivo, porém caso você tenha, assim como eu, outros containers utilizando essas duas portas será necessário remapear para algo próximo, como:

```yml
services:
  pihole:
    container_name: pihole
    image: pihole/pihole:latest
    ports:
      # DNS Ports
      - "53:53/tcp"
      - "53:53/udp"
      # Default HTTP Port
      - "8081:80/tcp" #<-- altere essas portas para algo do tipo ou alguma de sua preferência
      # Default HTTPs Port. FTL will generate a self-signed certificate
      - "8444:443/tcp" #<-- altere essas portas para algo do tipo ou alguma de sua preferência
```

4. Configure a senha utilizada na interface da aplicação pihole (**crie uma senha segura**):

```yml
FTLCONF_webserver_api_password: 'uma senha bem segura'
```

Pronto, agora você tem o `docker-compose.yml` configurado e pronto para realizar a um `docker compose up -d`.

### Subir o contêiner do Pihole

Para subir o contêiner use:

```bash
sudo docker compose up -d
```

Caso tenha algum problema com as portas `53` volte ao de liberar as porta e verifique a documentação de apoio. Lembre de reiniciar o serviço `resolved` com `sudo systemctl restart systemd-resolved.service` para a configuração ser aplicada!

Caso os problemas seja com portas `80` e `443` muito provavelmente as portas que você escolheu já estão sendo utilizadas por algum outro serviço, recomendo que leia a documentação de apoio e verifique quais portas são usadas por outros aplicativos instalados no seu sistema.

Para entrar na interface do Pihole, use o IP de sua maquina:

IP de sua maquina: `https://192.168.0.XXX:8444/admin/login`.

IP de exemplo: [https://192.168.0.10:8444/admin/login](https://192.168.0.10:8444/admin/login).

Lembrando que caso tenha escolhido uma porta diferente para HTTPS seu endereço devera refletir essa mudança.

## Conteúdo adicional

Esta parte não é necessária para a instalação do Pihole, porém é recomendada e bem interessante para ter seu próprio fornecedor de lista de DNSs.

### Instalação do Unbound

Unbound é um resolvedor de DNS que funciona dentro de seu próprio dispositivo, isso permite que você filtre ainda mais as requisições de DNS que são permitidas ou não em sua rede local.

Caso queira saber mais sobre o Unbound, a documentação no site do [pihole](https://docs.pi-hole.net/guides/dns/unbound/) é bem fácil de entender e utilizar.

Para instalar, utilize:

```bash
sudo apt install unbound
```

Faça o download das listas de servidores mais recente:

```bash
wget https://www.internic.net/domain/named.root -qO- | sudo tee /var/lib/unbound/root.hints
```

[Documentação para instalação e configuração.](https://unbound.docs.nlnetlabs.nl/en/latest/)

### Configure o Unbound para funcionar com o Pihole

Para configurar o Unbound para funcionar com o Pihole vamos criar ou modificar o arquivo de configurações em `unbound/conf.d/`.

Para isso, navegue até a pasta do Umbound:

```bash
cd /etc/unbound/unbound.conf.d/
```

Crie o arquivo de configurações para o Pihole:

```bash
sudo touch pihole.conf
```

Adicione a seguinte configuração dentro do arquivo:

```bash
sudo nano pihole.conf
```

```conf
server:
    # If no logfile is specified, syslog is used
    # logfile: "/var/log/unbound/unbound.log"
    verbosity: 0

    interface: 127.0.0.1
    port: 5335
    do-ip4: yes
    do-udp: yes
    do-tcp: yes

    # May be set to no if you don't have IPv6 connectivity
    do-ip6: yes

    # You want to leave this to no unless you have *native* IPv6. With 6to4 and
    # Terredo tunnels your web browser should favor IPv4 for the same reasons
    prefer-ip6: no

    # Use this only when you downloaded the list of primary root servers!
    # If you use the default dns-root-data package, unbound will find it automatically
    #root-hints: "/var/lib/unbound/root.hints"

    # Trust glue only if it is within the server's authority
    harden-glue: yes

    # Require DNSSEC data for trust-anchored zones, if such data is absent, the zone becomes BOGUS
    harden-dnssec-stripped: yes

    # Don't use Capitalization randomization as it known to cause DNSSEC issues sometimes
    # see https://discourse.pi-hole.net/t/unbound-stubby-or-dnscrypt-proxy/9378 for further details
    use-caps-for-id: no

    # Reduce EDNS reassembly buffer size.
    # IP fragmentation is unreliable on the Internet today, and can cause
    # transmission failures when large DNS messages are sent via UDP. Even
    # when fragmentation does work, it may not be secure; it is theoretically
    # possible to spoof parts of a fragmented DNS message, without easy
    # detection at the receiving end. Recently, there was an excellent study
    # >>> Defragmenting DNS - Determining the optimal maximum UDP response size for DNS <<<
    # by Axel Koolhaas, and Tjeerd Slokker (https://indico.dns-oarc.net/event/36/contributions/776/)
    # in collaboration with NLnet Labs explored DNS using real world data from the
    # the RIPE Atlas probes and the researchers suggested different values for
    # IPv4 and IPv6 and in different scenarios. They advise that servers should
    # be configured to limit DNS messages sent over UDP to a size that will not
    # trigger fragmentation on typical network links. DNS servers can switch
    # from UDP to TCP when a DNS response is too big to fit in this limited
    # buffer size. This value has also been suggested in DNS Flag Day 2020.
    edns-buffer-size: 1232

    # Perform prefetching of close to expired message cache entries
    # This only applies to domains that have been frequently queried
    prefetch: yes

    # One thread should be sufficient, can be increased on beefy machines. In reality for most users running on small networks or on a single machine, it should be unnecessary to seek performance enhancement by increasing num-threads above 1.
    num-threads: 1

    # Ensure kernel buffer is large enough to not lose messages in traffic spikes
    so-rcvbuf: 1m

    # Ensure privacy of local IP ranges
    private-address: 192.168.0.0/16
    private-address: 169.254.0.0/16
    private-address: 172.16.0.0/12
    private-address: 10.0.0.0/8
    private-address: fd00::/8
    private-address: fe80::/10

    # Ensure no reverse queries to non-public IP ranges (RFC6303 4.2)
    private-address: 192.0.2.0/24
    private-address: 198.51.100.0/24
    private-address: 203.0.113.0/24
    private-address: 255.255.255.255/32
    private-address: 2001:db8::/32
```

Ela pode ser encontrada [aqui](https://docs.pi-hole.net/guides/dns/unbound/).

Reinicie o serviço do Unbound:

```bash
sudo service unbound restart
```

E teste se ele está funcionando corretamente:

```bash
dig pi-hole.net @127.0.0.1 -p 5335
```

### Configure seu servidor DNS no Pihole:

Para isso é necessário apenas a inserção do endereço local host nas configurações customizadas.

Deixe da forma que aparece na imagem abaixo:

![exemplo](https://github.com/ChancellorMarko/manual_piHole/blob/main/img/6_screen.png)

Caso queira, é passível deixar os outros serviços de DNS ligados para caso o Unbound falhe ou o serviço não iniciar novamente, seu Pihole pode ter uma conexão de backup.

Por fim, não se esqueça de colocar o IP de seu servidor como servidor DNS padrão em seu roteador, geralmente localizado nas configurações de Servidor DHCP do seu roteador ou modem, para ter cobertura total de todos os dispositivos em sua rede local.

**Fim do Guia**