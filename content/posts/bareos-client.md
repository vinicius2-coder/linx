---
title: "Como instalar o Bareos Client em uma máquina!"
date: 2025-02-17T14:00:03+00:00
author: "Vinicius R. Rocha"
categories: ["Linux", "Terminal", "Backup", "Bareos"]
tags: ["terminal", "nuvem", "servidor", "backup"]
---

Neste tutorial, irei demonstrar como instalar o Bareos Client em uma máquina convidada. A instalação foi realizada em um ambiente de testes com máquina virtual. Caso utilize um VPS, ajuste as informações conforme necessário.
A versão utilizada foi o **Alma Linux 9.5** totalmente atualizado.

## O que é o Bareos?

**Ba**ckup & **Re**covery **O**pen **S**ource Software

Desde 2012, o Bareos fornece é uma solução robusta de backup e recuperação de dados de código aberto, garantindo que seus dados permaneçam seguros e completamente sob seu controle em todos os principais sistemas operacionais. 
Como Linux e Windows.

## O que você precisa?

- Um servidor **Alma Linux** rodando o Bareos Server;
- Acesso **SSH** e permissão de **root**;

## Instalando o software

Primeiro vamos adicionar o repositório do Bareos na máquina, executando este comando:

````bash
~# wget https://download.bareos.org/current/EL_9/add_bareos_repositories.sh 
````

Depois precisamos atualizar os repositórios da máquina:

````bash
~# yum update
````

Agora vamos instalar os pacotes do bareos-fd, o único software utilizado por uma máquina convidada:

````bash
~# yum install bareos-fd
````

Por último vamos ativar e habilitar o serviço para iniciar com a máquina:
````bash
~# systemctl enable bareos-fd
````
````bash
~# systemctl start bareos-fd
````

## Pronto, o Bareos já está instalado em uma máquina como cliente e está pronto para ser utilizado.

## Dica adicional

Neste ponto é importante se atentar ao firewall da máquina, dependendo de qual software está rodando, seu estado, regras de acesso http e https e etc, o range de portas 9100 a 9103 precisa estar liberado para 
comunicação entre as máquinas cliente e director.
