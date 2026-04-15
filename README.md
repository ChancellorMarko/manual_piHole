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
   ![exemplo](https://github.com/ChancellorMarko/manual_hospedagem_nextcloud/blob/main/img/1_screen.png)
2. Selecione seu sistema operacional de preferência, no meu caso utilizarei outros sistemas operacionais da própria Raspberry, como o Rasp OS Lite (64 bit):
   ![exemplo](https://github.com/ChancellorMarko/manual_hospedagem_nextcloud/blob/main/img/2_screen.png)
3. Insira as informações pertinentes de sua preferência conforme o que for solicitado.
4. Após configurar todas as informações pertinentes nas telas anteriores, você terá uma tela de acesso remoto que, dependendo da sua intenção, pode ser ou não interessante. Caso você queira acessar essa máquina remotamente para configuração via sua rede local (LAN), habilite a função SSH. Você tem duas opções para login via SSH:
   - Senha: com essa opção, você poderá escolher uma senha para realizar o login via sua rede, porém essa opção é insegura caso queira deixar essa máquina exposta na rede pública.
   - Chave SSH: uma das opções mais seguras caso queira deixar a máquina exposta na rede pública ou local.
   - Passo a passo: ![Configuração SSH](https://github.com/ChancellorMarko/manual_hospedagem_nextcloud/blob/main/Configura%C3%A7%C3%A3o%20SSH.md).
5. Caso queira, pode ativar a opção de Raspberry Pi Connect, no meu caso deixarei desativado.
6. Continue para a escrita das informações no cartão SD (**Atenção: todos os dados presentes nele serão apagados**).
   ![exemplo](https://github.com/ChancellorMarko/manual_hospedagem_nextcloud/blob/main/img/5_screen.png)

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

## Instalação do Docker

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

### Configurar o `docker-compose.yml`

Para configurar o arquivo necessário para rodar o aplicativo dentro da máquina virtual Docker, será utilizado o arquivo de configuração padrão, porém, com algumas modificações, no [Github do Pi-Hole](https://github.com/pi-hole/docker-pi-hole).

- Exemplo do arquivo:

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

Passos para a configuração e criação do contêiner:

1. Liberar as portas `53:53/tcp e udp`:

Caso você esteja utilizando uma distribuição baseada em Debian/Ubuntu ou fedora, é possível que as portas `53:53/tcp` e `53:53/udp` estejam sendo utilizadas pelo Systemd-Resolved, isso impedirá que o Docker utilize as portas impedindo o correto funcionamento da ferramenta. [(Documentação com explicação)](https://docs.pi-hole.net/docker/tips-and-tricks/)

Para resolver isso desative o stub-listener e reinicie o serviço resolve:

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

2. Configuração das portas utilizadas:

Caso você não tenha mais nenhum serviço que esteja rodando e ocupando as portas `80` e `443`, não será necessário a alteração dessas duas portas no arquivo, porém caso você tenha, assim como eu, outros containers utilizando essas duas portas será necessário remapear para algo próximo:

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
      - "8081:80/tcp" # altere essas portas para algo do tipo ou alguma de sua preferencia
      # Default HTTPs Port. FTL will generate a self-signed certificate
      - "8444:443/tcp" # altere essas portas para algo do tipo ou alguma de sua preferencia
```
