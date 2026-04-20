# Configuração de SSH

Aqui você vai encontrar uma breve configuração para realizar login via SSH em um Raspberry Pi 5, utilizando o aplicativo de confiuração próprio da Raspberrypi.

## Configuração com senha

Para essa configuração:

- Habilite a função SSH;
- Clique em usar autenticação com senha 'Use Password Authentication';
- Insira uma senha segura para sua maquina.

## Configuração com Chave SSH (SSH Key)

Para essa configuração você poderá usar um gerador de chaves SSH como o ssh-keygen ou [wpoven.com](https://www.wpoven.com/tools/create-ssh-key). Recomendo utilizar o seu próprio computador para realizar esse processo.

1. Para gerar uma chave e salva-la em um arquivo:

    Para sistemas mais antigos (caso tenha incompatibilidade com padrões de encriptação mais novos):

    ```bash
    ssh-keygen -f ~/server-key-file-name -t ecdsa -b 521
    ```

    Para sistemas mais novos:

    ```bash
    ssh-keygen -f ~/server-key-file-name -t ed25519
    ```

    - `ssh-keygen`: é o programa nativo linux que permite gerar chaves SSH;
    - `-f ~/server-key-file-name`: é o nome do arquivo que será guardada sua nova chave, ele pode ser modificado a sua escolha como o nome da maquina que você esta acessando;
    - `-t ed25519` ou `-t ecdsa -b 521`: são os métodos utilizados para realizar uma encriptação e gerar uma chave única.

2. Insira a chave no local indicado pela interface clicando em procurar 'Browse'.
   ![exemplo](https://github.com/ChancellorMarko/manual_piHole/blob/main/img/4_screen.png)