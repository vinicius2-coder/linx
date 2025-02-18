---
title: "Como configurar um backup job no Bareos!"
date: 2025-02-18T10:00:03+00:00
author: "Vinicius R. Rocha"
categories: ["Linux", "Terminal", "Backup", "Bareos"]
tags: ["terminal", "nuvem", "servidor", "backup"]
---

Neste tutorial, irei demonstrar como configurar a conexão entre a máquina Director e a máquina Client no Bareos, e depois como configurar um backup job. A versão utilizada foi o **Alma Linux 9.5** totalmente atualizado, tanto
Director como no Client.

## O que é o Bareos?

**Ba**ckup & **Re**covery **O**pen **S**ource Software

Desde 2012, o Bareos fornece uma solução robusta de backup e recuperação de dados de código aberto, garantindo que seus dados permaneçam seguros e completamente sob seu controle em todos os principais sistemas operacionais como 
Linux e Windows.

## O que você precisa?

- Um servidor **Alma Linux** rodando o Bareos Server;
- Acesso **SSH** e permissão de **root**;

## Configurando a conexão entre as máquinas

Seguindo os passos da configuração do Bareos, anteriormente instalamos o Bareos Server em uma vm, depois instalamos o Bareos UI, para acessos web, e depois instalamos o Bareos FD (Client) em uma vm convidada, agora vamos configurar
a conexão entre as vms.

Dentro da vm Director (Server) acesse o **bconsole** e execute este comando:

````bash
* configure add client name=seu-servidor-cliente address=ip-do-servidor-cliente password=uma-senha-forte
````

Ele vai retornar essa saída:

````bash
Exported resource file "/etc/bareos/bareos-dir-export/client/seu-servidor-cliente/bareos-fd.d/director/bareos-dir.conf":
Director {
  Name = "bareos-dir"
  Password = "[md5]********************************"  
}
Created resource config file "/etc/bareos/bareos-dir.d/client/seu-servidor-cliente.conf":
Client {
  Name = "seu-servidor-cliente"
  Address = "ip-do-servidor-cliente"
  Password = "uma-senha-forte"
}
*
````

Depois, precisamos copiar o arquivo de conf. gerado no Director para a máquina Client, isso pode ser executado com um simples SCP:

````bash
~# scp /etc/bareos/bareos-dir-export/client/seu-server/bareos-fd.d/director/bareos-dir.conf root@seu-servidor-cliente:/etc/bareos/bareos-fd.d/director/
````

## Sempre lembre de substituir as informações necessárias.

Após copiar o arquivo de conf. execute um restart no bareos-fd na vm cliente:

````bash
~# systemctl restart bareos-fd
````

Por último pode ser interessante testar ping e telnet nas portas 9101, 9102 e 9103 entre as vms, tudo tem que fechar tanto com IP como com hostname.

## Criando e executando um backup job

Agora que a conexão entre as vms está ok, vamos criar um backup job da vm cliente completo e depois executá-lo. No Bareos, tudo é configurado baseado em arquivos de configuração ou conf.

Para isso, primeiro, vamos criar um arquivo conf. com o schedule, para definir o horário que este backup job vai rodar. Execute este comando:

````bash
~# vim /etc/bareos/bareos-dir.d/schedule/Tarde.conf
````

Neste caso criei o arquivo Tarde.conf.

Dentro dele coloque este código:

````bash
Schedule {
  # name (required)
  Name = "Tarde"

  # run time
  # allot of options are available,
  # you can create multiple "Run =" times
  # for now lets keep it simple
  # this will run once a day, every day at 21:10
  Run = daily at 16:00
}
````

O arquivo define que o backup job vai rodar todos os dias as 16:00h.

Depois precisamos colocar o bareos como dono do arquivo:

````bash
~# chown bareos.bareos Tarde.conf
````

Em seguida precisamos informar ao Bareos sobre o que queremos fazer backup na vm cliente, a fonte do backup, faremos isso criando um novo arquivo de conf. no diretório **fileset**, assim:

````bash
~# vim /etc/bareos/bareos-dir.d/fileset/BackupCompleto.conf
````

Dentro do arquivo coloque este código:

````bash
FileSet {
  # name (required)
  Name = "BackupCompleto"

  # include directory
  Include {
    Options {
      # calculate a signature for all files
      # both MD5/SHA1 are available, this is definitly for long term storage
      # a good idea to activate, note that the hash ads a bit of storage overhead
      Signature = MD5
      
      # compress every file with compression software
      # best known are : LZ4/GZIP (see manual for the others)
      # LZ4 is super fast in both compression and decompression
      # I would activate this always.
      Compression = LZ4
      
      # if supported by the OS, the read time won't be adapted
      # this would generate a bunch of writes for no reason on the client machine.
      noatime = yes
    }
    # the directory we are including
    # note: no trailing slashes!
    File = /
  }
}
````

No meu caso coloquei o diretório **/** na linha que começa com **File**, ou seja, a vm linux completa.

Estamos quase no final!

A próxima etapa é configurar as definições do backup job, executamos isso criando um novo arquivo:

````bash
~# vim /etc/bareos/bareos-dir.d/jobdefs/JobPadrao.conf
````

Dentro do arquivo coloque este código:

````bash
JobDefs {
  # name (required)
  Name = "JobPadrao"
  
  # type can be backup/restore/verify
  Type = Backup
  
  # the default level bareos will try
  # can also be Full/Differential(since last full)/Incremental(since last incremental)
  Level = Incremental
  
  # the default client, to be overwritten by the job.conf
  Client = bareos-fd
  
  # what files to include/exclude
  FileSet = "BackupCompleto"
  
  # the schedule we just created
  Schedule = "Tarde"
  
  # where to store it
  Storage = File
  
  # the message reporting
  Messages = Standard
  
  # the pool where to store it
  Pool = Incremental
  
  # the higher the value priority the lower it will be dropped in the queue
  # so for important jobs priority=1 will run first
  Priority = 10
  
  # the bootstrap file keeps a "log" of all the backups, and gets rewritten every time a 
  # full backup is made, it can be used during recovery
  Write Bootstrap = "/var/lib/bareos/%c.bsr"
  
  # in case these value's get overwritten
  # define where would be a good pool to write 
  # note that full backup will be used atleast once because no full
  # backup will exist
  Full Backup Pool = Full
  Differential Backup Pool = Differential
  Incremental Backup Pool = Incremental
}
````

Repare acima que o **Fileset** e o **Schedule** foram alterados, para os nomes dos dois arquivos que foram criados anteriormente.

E agora, por último, vamos criar o arquivo principal de conf. que "chama" os outros arquivos para executar o backup job:

````bash
~# vim /etc/bareos/bareos-dir.d/job/Backup.conf
````

Dentro do arquivo coloque este código:

````bash
Job {
  # required
  Name = "Backup"

  # the default settings
  JobDefs = "JobPadrao"

  # overwrite the client here
  Client = "seu-servidor-cliente"
}
````

Aqui agora apenas referencie o nome do arquivo conf. de definição, no meu caso é o JobPadrao.conf e o nome do seu servidor cliente. E é claro um nome para este arquivo.

Basta agora acessar o **bconsole** e executar um reload, para que o backup job fique ativo e pronto para rodar no horário programado.
