---
title: "Como instalar o Bareos no Linux para criar backups otimizados!"
date: 2025-02-14T15:00:03+00:00
author: "Vinicius R. Rocha"
categories: ["Linux", "Terminal", "Backup", "Bareos"]
tags: ["terminal", "nuvem", "servidor", "backup"]
---

Neste tutorial, irei demonstrar como instalar o Bareos Server no Alma Linux. A instalação foi realizada em um ambiente de testes com máquina virtual. Caso utilize um VPS, ajuste as informações conforme necessário.
A versão utilizada foi o **Alma Linux 9.5** totalmente atualizado.

## O que é o Bareos?

**Ba**ckup & **Re**covery **O**pen **S**ource Software

Desde 2012, o Bareos fornece é uma solução robusta de backup e recuperação de dados de código aberto, garantindo que seus dados permaneçam seguros e completamente sob seu controle em todos os principais sistemas operacionais. 
Como Linux e Windows.

## O que você precisa?

- Um servidor **Alma Linux** totalmente atualizado;
- Acesso **SSH** e permissão de **root**;
- Determinação para assumir o controle e gerenciar os backups da sua organização.

## Atualizando as dependências do servidor

O primeiro passo é atualizar as dependências do seu servidor para garantir uma instalação eficaz:

````bash
~# yum update -y
````

## Instalando pacotes importantes

````bash
~# yum install epel-release
~# yum install wget
````

## Adicionando os repositórios do Bareos

Vamos executar o download de um script em sh do repositório do Bareos:

````bash
~# wget https://download.bareos.org/current/EL_9/add_bareos_repositories.sh
````

Execute o script para adicionar o repositório:

````bash
~# sh add_bareos_repositories.sh
````

A saída será essa:

````bash
Repository https://download.bareos.org/current/EL_9 successfully added.
````

## Instalando o Bareos

Após adicionar o repositório, instale o Bareos:

````bash
~# yum install bareos -y
````

## Instalando o PostgreSQL

O Bareos utiliza, na versão atual, um banco de dados PostgreSQL, é dentro dele que ele armazena informações sobre os backups. Vamos baixar e instalar o repositório do PostgreSQL na máquina:

````bash
~# yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-9-x86_64/pgdg-redhat-repo-latest.noarch.rpm
````

Agora vamos desabilitar os módulos pré-empacotados do PostgreSQL nesta distribuição:

````bash
~# yum -qy module disable postgresql
````

Instale o PostgreSQL 17:

````bash
~# yum install postgresql17-server -y
````

Inicie o banco de dados e habilite a iniciação dele com o SO:

````bash
~# sudo /usr/pgsql-17/bin/postgresql-17-setup initdb
````
````bash
~# systemctl enable postgresql-17
````
````bash
~# systemctl start postgresql-17
````

## Criando as databases que o Bareos utiliza dentro do PostgreSQL

Como o Bareos utiliza um banco de dados pra armazenar as informações, vamos criar o banco principal e suas respectivas tabelas:

````bash
~# su postgres -c /usr/lib/bareos/scripts/create_bareos_database
````
````bash
~# su postgres -c /usr/lib/bareos/scripts/make_bareos_tables
````
````bash
~# su postgres -c /usr/lib/bareos/scripts/grant_bareos_privileges
````

Por último vamos ativar os serviços do Bareos, Bareos Director, Storage Daemon e File Daemon:

````bash
~# systemctl enable --now bareos-dir
````
````bash
~# systemctl enable --now bareos-sd
````
````bash
~# systemctl enable --now bareos-fd
````

## Finalizando

Instalação do Bareos finalizada, para acessar o console dele e começar a executar comandos digite:

````bash
~# bconsole
````
