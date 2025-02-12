---
title: "Como instalar o Nextcloud no Linux para criar uma Nuvem Privada!"
date: 2025-02-12T11:00:03+00:00
author: "Vinicius R. Rocha"
categories: ["Linux", "Terminal", "Nextcloud"]
tags: ["nextcloud", "terminal", "nuvem", "servidor"]
---

Neste tutorial, irei demonstrar como instalar o Nextcloud no Ubuntu. A instalação foi realizada em um ambiente de testes com máquina virtual. Caso utilize um VPS, ajuste as informações conforme necessário.
A versão utilizada foi o **Ubuntu Server 24.04**, mas os passos devem funcionar em outras versões anteriores como **22.04** ou **20.08**, ou até mesmo **18.04**.

## O que é o Nextcloud?

O Nextcloud é um software open-source que permite hospedar e compartilhar arquivos de forma segura e privativa. Ele é semelhante ao Dropbox, mas com código-fonte aberto, proporcionando mais controle e privacidade sobre seus arquivos, além da possibilidade de personalizar a instalação.
Se deseja uma solução de armazenamento onde tenha total controle sobre o servidor e a plataforma, o Nextcloud é a melhor escolha.

## O que você precisa?

- Um servidor **Ubuntu** atualizado (com IP estático ou domínio);
- Acesso **SSH** e permissão de **root**;
- Determinação para assumir o controle da sua privacidade.

## Atualizando as dependências do servidor

O primeiro passo é atualizar as dependências do seu servidor para garantir uma instalação eficaz:

```bash
~# apt update && apt upgrade -y
```

## Instalando o Apache2 no Ubuntu

Usaremos o Apache2, um dos servidores web mais populares. Para instalá-lo, execute:

```bash
~# apt install apache2
```

Habilite o serviço para iniciar com o sistema:

```bash
~# systemctl enable apache2.service
```

Neste ponto você precisa redirecionar as portas 80 e 443 para o IP interno da máquina, e depois, liberar essas portas no seu firewall de borda, para ficarem acessíveis ao mundo, ou apenas ao seu IP público.
Para testar se o Apache2 está funcionando, acesse o IP público do servidor ou o domínio pelo navegador. Exemplo da página que deve ser retornada:<br><br>

![Apache](/images/apache2.png)

<br><br>
## Instalando o MariaDB no Ubuntu

O MariaDB é uma ramificação do MySQL e será usado como banco de dados. Instale-o com:

```bash
~# apt install mariadb-server mariadb-client
```

Habilite o serviço:

```bash
~# systemctl enable mariadb.service
```

Agora, execute a configuração inicial:

```bash
~# mysql_secure_installation
```

Responda às perguntas conforme abaixo:

```plaintext
Enter current password for root (enter for none): [Deixe vazio]
Switch to unix_socket authentication [Y/n] Y
Change the root password? [Y/n]: Y
New password: [Digite uma nova senha]
Re-enter new password: [Confirme a senha]
Remove anonymous users? [Y/n]: Y
Disallow root login remotely? [Y/n]: Y
Remove test database and access to it? [Y/n]: Y
Reload privilege tables now? [Y/n]: Y
```

Reinicie o serviço:

```bash
~# systemctl restart mariadb.service
```

## Configurando o banco de dados do Nextcloud

Acesse o MariaDB:

```bash
~# mysql -u root -p
```

Crie o banco de dados:

```sql
CREATE DATABASE nextcloud;
```

Crie um usuário para o banco:

```sql
CREATE USER 'nextcloud'@'localhost' IDENTIFIED BY 'nextcloud';
```

Dê as permissões necessárias:

```sql
GRANT ALL ON nextcloud.* TO 'nextcloud'@'localhost' IDENTIFIED BY 'nextcloud' WITH GRANT OPTION;
```

Atualize os privilégios e saia:

```sql
FLUSH PRIVILEGES;
EXIT;
```

## Instalando o PHP 8.1 no Ubuntu Server

Adicione o repositório necessário:

```bash
~# apt install software-properties-common
~# add-apt-repository ppa:ondrej/php
~# apt update
```

Instale os pacotes do PHP:

```bash
~# apt install php8.1 libapache2-mod-php8.1 php8.1-common php8.1-gmp php8.1-curl php8.1-intl php8.1-mbstring php8.1-xmlrpc php8.1-mysql php8.1-gd php8.1-xml php8.1-cli php8.1-zip
```

Neste momento, caso não possua o vim ou o nano na máquina, instale.

Edite o arquivo de configuração:

```bash
~# vim /etc/php/8.1/apache2/php.ini
```

Modifique os seguintes valores:

```ini
file_uploads = On
allow_url_fopen = On
memory_limit = 512M
upload_max_filesize = 300M
max_execution_time = 360
```

Reinicie o Apache para aplicar as configurações:

```bash
~# systemctl restart apache2.service
```

## Baixando e instalando o Nextcloud

Baixe a versão mais recente dentro da sua home mesmo:

```bash
~# wget https://download.nextcloud.com/server/releases/latest.zip
```

Neste ponto instale o **zip** e o **unzip** (caso não tenha), vamos precisar:

```bash
~# apt install zip unzip
```

Extraia o arquivo:

```bash
~# unzip latest.zip
```

Mova a pasta recém extraída para o diretório padrão do Apache:

```bash
~# mv nextcloud /var/www/html/
```

Defina as permissões corretas:

```bash
~# chown -R www-data:www-data /var/www/html/nextcloud
```

## Configurando o servidor Apache

Crie um arquivo de configuração:

```bash
 ~# vim /etc/apache2/sites-available/nextcloud.conf
```

Adicione o seguinte conteúdo e edite conforme preferir:

```apache
<VirtualHost *:80>
   ServerAdmin admin@seudominio.com
   DocumentRoot "/var/www/html/nextcloud"
   ServerName seudominio.com
   <Directory "/var/www/html/nextcloud/">
      Options MultiViews FollowSymlinks
      AllowOverride All
      Order allow,deny
      Allow from all
   </Directory>
   TransferLog /var/log/apache2/nextcloud_access.log
   ErrorLog /var/log/apache2/nextcloud_error.log
</VirtualHost>
```

Salve o arquivo e desative a configuração padrão do Apache:

```bash
~# a2dissite 000-default
~# a2ensite nextcloud
```

Reinicie o Apache:

```bash
~# systemctl restart apache2
```

## Finalizando a instalação do Nextcloud

Acesse o endereço de IP público do seu servidor no navegador. Preencha os dados para criar um usuário administrador e conectar ao banco de dados com as informações configuradas anteriormente. Exemplo:<br><br>

![Nextcloud Install](/images/nextcloud_install.png)

Após preencher os dados clique em **Instalar** e aguarde, o Nextcloud estará pronto para uso.

## Conclusão

O Nextcloud é uma excelente opção para armazenar arquivos de forma segura, seja para uso pessoal ou empresarial. Se ainda não testou, vale a pena experimentar!

## Dica adicional!

É possível agora, com o Nextcloud funcionando, instalar um certificado SSL na sua URL própria, vamos lá!

Primeiro instale as bibliotecas do Python para o Certbot do Apache:

```bash
 ~# apt install python3-certbot-apache
```

Depois execute este comando para criar o certificado:

```bash
 ~# certbot --apache --agree-tos --preferred-challenges http -d seudominio.com.br
```

Aqui responda com um endereço de e-mail para receber notificações sobre o certificado:

````bash
Enter email address (used for urgent renewal and security notices)
 (Enter 'c' to cancel): suaconta@seudominio.com.br
````

Em seguida:

````bash
(Y)es/(N)o: Y
````

A resposta será essa, caso dê tudo certo com o certificado:
````bash
Deploying certificate
Successfully deployed certificate for seudominio.com.br to /etc/apache2/sites-available/nextcloud-le-ssl.conf
Congratulations! You have successfully enabled HTTPS on https://seudominio.com.br
````
