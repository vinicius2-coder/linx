---
title: "Como instalar o Bareos-UI para gerenciar seus backups na Web!"
date: 2025-02-14T17:00:03+00:00
author: "Vinicius R. Rocha"
categories: ["Linux", "Terminal", "Backup", "Bareos"]
tags: ["terminal", "nuvem", "servidor", "backup"]
---

Neste tutorial, irei demonstrar como instalar os pacotes que habilitam a UI - Interface Gráfica - no Bareos Server. A instalação foi realizada em um ambiente de testes com máquina virtual. Caso utilize um VPS, ajuste as informações conforme necessário.
A versão utilizada foi o **Alma Linux 9.5** totalmente atualizado já rodando um Bareos Server.

## O que é o Bareos?

**Ba**ckup & **Re**covery **O**pen **S**ource Software

Desde 2012, o Bareos fornece é uma solução robusta de backup e recuperação de dados de código aberto, garantindo que seus dados permaneçam seguros e completamente sob seu controle em todos os principais sistemas operacionais. 
Como Linux e Windows.

## O que você precisa?

- Um servidor **Alma Linux** rodando o Bareos Server;
- Acesso **SSH** e permissão de **root**;

## Instalando os pacotes da UI

Primeiro vamos instalar os pacotes do bareos-ui, a interface gráfica:

````bash
~# yum install bareos-webui -y
````

Agora ajuste os parâmetros padrão do SELinux:

````bash
~# setsebool -P httpd_can_network_connect on
````

## Configurando o Bareos dentro do bconsole

Neste ponto, acesse o **bconsole** e faça um reload:

````bash
~# bconsole
Connecting to Director localhost:9101
 Encryption: TLS_CHACHA20_POLY1305_SHA256 TLSv1.3
1000 OK: bareos-dir Version: 24.0.1~pre67.4ee24a825 (12 February 2025)
Bareos community build (UNSUPPORTED).
Get professional support from https://www.bareos.com
You are connected using the default console

Enter a period (.) to cancel a command.
*reload
reloaded
*
````

Depois, ainda dentro do **bconsole** execute este comando para criar um usuário administrador, substituindo os campos necessários:

````bash
* configure add console name=admin password=yoursecret profile=webui-admin tlsenable=false
````

## Reiniciando os serviços e acessando o Bareos-UI

Depois de criar o usuário, saia do **bconsole** e reinicie o webserver e o bareos-dir:

````bash
~# systemctl restart httpd
````
````bash
~# systemctl restart bareos-dir
````

## Acessando a interface gráfica:

Basta acessar a interface acessando assim:
http://seuIPlocal/bareos-webui/

![bareos-webui](/images/bareos-webui/)


## Dica adicional

Neste ponto é importante se atentar ao firewall da máquina, dependendo de qual software está rodando, seu estado, regras de acesso http e https e etc, o acesso ao Bareos UI pode ser barrado.





